# CloudCanva — Complete Technical Interview Preparation Guide

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Architecture Deep Dive](#2-architecture-deep-dive)
3. [5-Layer Pipeline Explained](#3-5-layer-pipeline-explained)
4. [Backend Deep Dive](#4-backend-deep-dive)
5. [Frontend Deep Dive](#5-frontend-deep-dive)
6. [AWS Services — Why Each One](#6-aws-services--why-each-one)
7. [Security Implementation](#7-security-implementation)
8. [IAM & Quota Systems](#8-iam--quota-systems)
9. [Terraform Generation](#9-terraform-generation)
10. [Technical Decisions & Tradeoffs](#10-technical-decisions--tradeoffs)
11. [Interview Questions & Answers](#11-interview-questions--answers)
12. [Potential Follow-up Questions](#12-potential-follow-up-questions)

---

## 1. Project Overview

### What is CloudCanva?
CloudCanva is a full-stack web application that converts natural language prompts into validated AWS architecture diagrams and production-ready Terraform code.

### The Problem It Solves
- Writing Terraform manually is slow, error-prone, requires HCL expertise
- Security misconfigurations (open SSH, public databases) are #1 cause of AWS breaches
- No existing tool combines: visual design + AI + security + IAM + quotas + Terraform
- AWS Infra Composer outputs CloudFormation only, has no AI, no security auto-fix

### How It Works (One Paragraph)
User types "Build a 3-tier web app with RDS" → Amazon Bedrock (Claude Sonnet 4) generates
structured architecture JSON following 18 strict rules → a deterministic post-processing
function fixes any AI errors (missing refs, wrong AMIs, insecure ports) → user previews on
React Flow canvas → clicks Export → 19 Jinja2 templates render main.tf → ZIP uploaded to S3
→ metadata saved to DynamoDB → user downloads and runs terraform apply.

---

## 2. Architecture Deep Dive

### High-Level Architecture
```
User → Amplify (Next.js frontend) → Cognito (auth) → API Gateway (HTTPS proxy)
    → EC2 (FastAPI backend) → Bedrock / S3 / DynamoDB / STS / IAM
    → Output: Terraform ZIP → terraform apply → AWS Infrastructure Live
```

### Why This Architecture?
- **Amplify**: Zero-ops frontend hosting with native Next.js SSR + Cognito integration
- **API Gateway**: Free HTTPS endpoint (browsers block HTTP from HTTPS pages)
- **EC2**: Persistent process for FastAPI, file system for temp zips, Bedrock calls take 5-8s
- **Cognito**: Serverless JWT auth, no custom auth code needed
- **S3 + DynamoDB**: File storage + metadata separated (right tool for each job)

### Data Flow
1. Frontend sends JWT + request → API Gateway
2. API Gateway forwards to EC2:8000
3. FastAPI verifies JWT via Cognito JWKS
4. Processes request (AI generation / Terraform export / validation)
5. Uses S3, DynamoDB, Bedrock, STS as needed
6. Returns response to frontend

---

## 3. 5-Layer Pipeline Explained

### Layer 1: User Input
- **Manual Mode**: Drag AWS services from sidebar onto React Flow canvas, connect with edges, configure properties
- **AI Mode**: Type natural language prompt in dialog
- Frontend sends POST /ai/generate with {prompt, region}

### Layer 2: AI Generation (Amazon Bedrock)
- Calls Claude Sonnet 4 via Bedrock API in eu-north-1
- System prompt contains 18 strict rules:
  - Rule 1: Region-locked AZs (only use eu-north-1a/b/c for eu-north-1)
  - Rule 2: Region-correct AMI (ami-09a9858973b288bdd for eu-north-1)
  - Rule 3-7: Mandatory refs (Subnet→VPC, EC2→Subnet, SG→VPC, IGW→VPC, RT→VPC)
  - Rule 8-9: Node positioning (VPC at y=50, subnets at y=200, services at y=400)
  - Rule 10: Add edges for connections
  - Rule 11: Include proper properties (cidr_block, ami, engine, etc.)
  - Rule 12: Always include refs object even if empty
  - Rule 13: Return ONLY JSON, no markdown
  - Rule 14-18: Security rules (no SSH to 0.0.0.0/0, restrict DB ports, HTTP/HTTPS only public, DB in private subnet, always include egress rule)
- Output: Structured JSON with {summary, nodes[], edges[]}

### Layer 3: Post-Processing (Deterministic Safety Net)
- Function: `post_process_ai_response()` in main.py
- NOT AI — pure Python code. Same input = same output every time.
- What it fixes:
  - Missing refs.vpc on Subnet/SG/IGW/RouteTable → links to first VPC node
  - Missing refs.subnet on EC2 → links to first Subnet
  - Wrong AMI (different region) → replaces with correct region AMI
  - Wrong AZ (us-east-1a in eu-north-1) → replaces with correct AZ
  - SSH (22) open to 0.0.0.0/0 → restricts to 10.0.0.0/16 (VPC CIDR)
  - DB ports (3306/5432/6379) open to 0.0.0.0/0 → restricts to 10.0.1.0/24
  - LoadBalancer missing subnets → assigns first 2 subnet IDs
  - RDS missing subnets → assigns 2 subnets for multi-AZ
  - refs key missing entirely → creates empty refs: {}

### Layer 4: Preview & Apply
- Frontend receives corrected JSON
- Shows preview dialog: summary, service list, connections
- User can: Regenerate (try again) or Apply to Canvas
- Apply pushes nodes + edges into React Flow state

### Layer 5: Canvas Rendering & Terraform Export
- React Flow renders nodes as draggable cards with edges
- User can further edit properties, add/remove services
- Click Export → POST /generate sends canvas JSON to backend
- Backend: TerraformOrchestrator → 19 Jinja2 templates → main.tf → ZIP → S3
- Returns ZIP download to user

---

## 4. Backend Deep Dive

### File Structure
```
backend/
├── main.py                 # FastAPI app, 15+ endpoints, all business logic
├── auth.py                 # Cognito JWT verification middleware
├── template_registry.py    # Maps service types → Jinja2 template files
├── requirements.txt        # Python dependencies
├── generators/
│   ├── orcherastor.py      # Iterates services, renders templates
│   └── generic_template_generator.py  # Renders single template with config
├── templates/aws/          # 19 Jinja2 Terraform templates
│   ├── provider.tf.j2
│   ├── vpc.tf.j2
│   ├── subnet.tf.j2
│   ├── ec2.tf.j2
│   ├── rds.tf.j2
│   ├── internet_gateway.tf.j2
│   ├── nat_gateway.tf.j2
│   ├── eks.tf.j2
│   ├── s3/bucket.tf.j2
│   ├── ecs/ecs.tf.j2
│   ├── eip/elastic_ip.tf.j2, eip_association.tf.j2
│   ├── load_balancer/load_balancer.tf.j2, target_group.tf.j2
│   ├── routing/route_table.tf.j2, route_table_association.tf.j2
│   └── security_group/security_group.tf.j2, ingress.tf.j2, egress.tf.j2
```

### Key Endpoints
| Endpoint | Method | What it does |
|----------|--------|-------------|
| / | GET | Health check |
| /regions | GET | Returns 6 regions with AMIs, AZs |
| /ai/generate | POST | Bedrock AI → post-process → return JSON |
| /generate | POST | Terraform generation → ZIP → S3 → download |
| /validate | POST | Check architecture for errors/warnings |
| /projects | GET | List user's saved projects from DynamoDB |
| /projects/{name}/download | GET | Presigned S3 URL for ZIP download |
| /projects/{name} | DELETE | Remove from S3 + DynamoDB |
| /check-quotas | POST | Static quota warnings |
| /check-quotas-live | POST | Live quota check via AssumeRole |
| /generate-iam-policy | POST | Minimum IAM policy JSON |
| /validate-iam-permissions | POST | SimulatePrincipalPolicy check |

### How Authentication Works (auth.py)
1. Frontend sends `Authorization: Bearer <jwt-token>` header
2. Backend extracts token from header
3. Fetches Cognito JWKS (public keys) from:
   `https://cognito-idp.{region}.amazonaws.com/{pool_id}/.well-known/jwks.json`
4. Caches keys for 1 hour (performance optimization)
5. Finds the key matching token's `kid` header
6. Verifies RS256 signature
7. Checks `token_use == "access"` and `exp` (not expired)
8. Returns decoded claims (username, sub, etc.)
9. All endpoints use `Depends(get_current_user)` for protection

### How Terraform Generation Works
1. `generate_tf_code(config_data, region)` called
2. Creates `TemplateRegistry` — loads Jinja2 Environment with template dir
3. Creates `TerraformOrchestrator` — receives config + registry + region
4. Orchestrator iterates `connected_services`:
   - Detects AWS provider → renders `provider "aws" {}` block
   - For each service: constructs template name `{provider}_{type}`
   - Looks up template in registry map
   - `GenericTemplateGenerator.render()` → passes service config to template
   - Appends rendered block to tf_blocks list
5. Writes all blocks to `/tmp/xxx/tf_output/infrastructure/main.tf`
6. `shutil.make_archive()` → creates .zip
7. Uploads to S3: `users/{email}/{project_name}.zip`
8. Saves to DynamoDB: user_email, project_name, s3_key, created_at, node_count
9. Returns FileResponse (user downloads ZIP)

### Template Registry Mapping (19 templates)
```python
"aws_aws_instance" → "aws/ec2.tf.j2"
"aws_provider" → "aws/provider.tf.j2"
"aws_vpc" → "aws/vpc.tf.j2"
"aws_subnet" → "aws/subnet.tf.j2"
"aws_internet_gateway" → "aws/internet_gateway.tf.j2"
"aws_nat_gateway" → "aws/nat_gateway.tf.j2"
"aws_security_group" → "aws/security_group/security_group.tf.j2"
"aws_vpc_security_group_ingress_rule" → "aws/security_group/vpc_security_group_ingress_rule.tf.j2"
"aws_vpc_security_group_egress_rule" → "aws/security_group/vpc_security_group_egress_rule.tf.j2"
"aws_route_table" → "aws/routing/route_table.tf.j2"
"aws_route_table_association" → "aws/routing/route_table_association.tf.j2"
"aws_s3_bucket" → "aws/s3/bucket.tf.j2"
"aws_load_balancer" → "aws/load_balancer/load_balancer.tf.j2"
"aws_target_group" → "aws/load_balancer/target_group.tf.j2"
"aws_rds_instance" → "aws/rds.tf.j2"
"aws_ecs_cluster" → "aws/ecs/ecs.tf.j2"
"aws_eks_cluster" → "aws/eks.tf.j2"
"aws_aws_eip" → "aws/eip/elastic_ip.tf.j2"
"aws_eip_association" → "aws/eip/eip_association.tf.j2"
```

---

## 5. Frontend Deep Dive

### File Structure
```
frontend/
├── app/
│   ├── page.tsx              # Home page (Canvas + controls)
│   ├── layout.tsx            # Root layout
│   ├── globals.css           # Tailwind styles
│   ├── signin/page.tsx       # Login page
│   ├── signup/page.tsx       # Registration page
│   ├── projects/page.tsx     # Saved projects list
│   └── forgot-password/page.tsx
├── components/
│   ├── Canvas.tsx            # React Flow canvas (core component)
│   ├── AiArchitectureGenerator.tsx  # AI dialog
│   ├── IAMPolicyGenerator.tsx       # IAM policy UI
│   ├── ValidationPanel.tsx          # Validation results
│   ├── TerraformPreview.tsx         # Live TF preview
│   ├── AuthGuard.tsx                # Protected route wrapper
│   ├── base-node.tsx                # Custom node component
│   └── ui/                          # shadcn/ui components
├── context/
│   └── AuthContext.tsx       # Cognito auth state
├── lib/
│   ├── api.ts                # Axios instance (baseURL from env)
│   └── cognito.ts            # Amplify Cognito config
└── .env.local                # Environment variables
```

### Key Frontend Technologies
| Tech | Why used |
|------|----------|
| Next.js 15 | SSR, app router, API routes, optimized builds |
| React 19 | Latest features, concurrent rendering |
| TypeScript | Type safety, better IDE support |
| Tailwind CSS | Utility-first styling, fast development |
| React Flow | Interactive node-edge canvas (drag, connect, zoom) |
| shadcn/ui | Pre-built accessible UI components (Dialog, Button, Input) |
| AWS Amplify (SDK) | Cognito integration (signIn, signUp, getCurrentUser) |
| Axios | HTTP client for API calls |
| Zustand/Context | State management (AuthContext) |
| dagre | Auto-layout algorithm for node positioning |

### How Canvas.tsx Works
- Uses React Flow library for the interactive canvas
- Nodes represent AWS services (VPC, EC2, Subnet, etc.)
- Edges represent connections/dependencies
- Key state: `nodes[]`, `edges[]`, `selectedNode`
- Key functions:
  - `onConnect()` — creates edge when user draws connection
  - `onNodesChange()` — handles drag, select, delete
  - `handleExport()` — generates config JSON, sends to /generate
  - `validateInfra()` — sends to /validate endpoint
  - `generateConfig()` — transforms React Flow nodes to backend format

### How AI Generator Component Works
1. User opens dialog, types prompt
2. On "Generate": calls `api.post("/ai/generate", {prompt, region})`
3. Backend returns `{summary, nodes[], edges[]}`
4. Shows preview in dialog (service list + connections)
5. On "Apply to Canvas": calls `onArchitectureGenerated(nodes, edges)`
6. Parent component sets these into React Flow state
7. Canvas re-renders with new nodes

### Authentication Flow (Frontend)
1. `cognito.ts` configures Amplify with pool_id, client_id, region
2. `AuthContext.tsx` wraps app, checks `getCurrentUser()` on load
3. If not authenticated → redirect to /signin
4. On sign-in: Amplify calls Cognito → receives tokens
5. `api.ts` interceptor adds `Authorization: Bearer {token}` to every request
6. `AuthGuard.tsx` component wraps protected pages

---

## 6. AWS Services — Why Each One

### Amazon Bedrock (Claude Sonnet 4)
**What**: Managed AI service — call Claude model via API
**Why Bedrock (not OpenAI/Gemini)**: Data stays in AWS, IAM auth (no API keys), same billing
**Why Bedrock (not SageMaker)**: Just need inference, not training. Bedrock = one API call. SageMaker = manage endpoints, instances, scaling.
**Why Claude Sonnet 4 (not Haiku/Opus)**: Haiku too small for 18-rule complex prompts. Opus too slow/expensive. Sonnet = best cost-to-quality ratio.

### Amazon Cognito
**What**: Managed user authentication (signup, signin, JWT tokens)
**Why Cognito (not custom auth)**: Zero auth code to write, JWT out of box, password hashing handled, free 50K users
**Why Cognito (not IAM Identity Center)**: Identity Center is for workforce SSO, not customer-facing apps
**How it works**: User signs in → Cognito issues JWT → frontend sends with every request → backend verifies via JWKS

### Amazon EC2
**What**: Virtual server running FastAPI backend
**Why EC2 (not Lambda)**: Persistent process (no cold starts), Bedrock takes 5-8s (Lambda 30s timeout risky), file system for temp zips, simple deployment
**Why EC2 (not ECS/Fargate)**: Single app, no need for containers/orchestration. Overkill complexity for one FastAPI server.
**Why EC2 (not App Runner)**: More control, better debugging, established patterns

### AWS Amplify
**What**: Frontend hosting platform for Next.js
**Why Amplify (not S3+CloudFront static)**: Next.js needs SSR support. S3 only serves static files.
**Why Amplify (not EC2 for frontend)**: Zero ops — no nginx, no SSL certs, auto CI/CD from Git
**Why Amplify (not Vercel/Netlify)**: Native Cognito integration, same AWS ecosystem, same billing

### Amazon S3
**What**: Object storage for Terraform ZIP files
**Why S3 (not EFS)**: EFS expensive, needs EC2 mount, no presigned URLs
**Why S3 (not EBS)**: EBS attached to single EC2, can't serve downloads
**Why S3 (not DynamoDB)**: DynamoDB has 400KB item limit, wrong tool for binary files
**Key feature used**: Presigned URLs (10-min expiry) for secure downloads without exposing bucket

### Amazon DynamoDB
**What**: NoSQL database for project metadata
**Why DynamoDB (not RDS)**: Serverless (no provisioning), fast key-value lookups, no connection pools, pay-per-request
**Why DynamoDB (not S3 alone)**: S3 ListObjects is slow, can't query by user_email, no sorting
**Schema**: Partition key = user_email, Sort key = project_name. Fields: s3_key, created_at, node_count

### API Gateway
**What**: HTTPS proxy in front of EC2
**Why needed**: Amplify is HTTPS, EC2 is HTTP. Browsers block mixed content. Gateway gives free HTTPS.
**Config**: HTTP API → Route ANY /{proxy+} → Integration http://EC2-IP:8000/{proxy}

### AWS STS
**What**: Security Token Service — for AssumeRole
**Why used**: Live quota check uses user's credentials. STS assumes their role to query their account's actual resource usage.

---

## 7. Security Implementation

### 18 AI Rules (Prompt Engineering)
Rules 1-13: Schema and correctness
Rules 14-18: Security enforcement:
- Rule 14: NEVER SSH (22) open to 0.0.0.0/0 → use VPC CIDR
- Rule 15: NEVER DB ports (3306/5432/6379) open to 0.0.0.0/0 → use app subnet CIDR
- Rule 16: ONLY 80/443 allowed from 0.0.0.0/0
- Rule 17: Databases MUST be in private subnets (map_public_ip_on_launch = false)
- Rule 18: Always include egress rule (protocol -1, 0.0.0.0/0)

### Post-Processing Security Fixes
Even if AI ignores rules, code enforces:
```python
sensitive_ports = {22: "SSH", 3306: "MySQL", 5432: "PostgreSQL", 6379: "Redis", 27017: "MongoDB"}

if cidr == "0.0.0.0/0" and port in sensitive_ports:
    if port == 22:
        rule["cidr_blocks"] = ["10.0.0.0/16"]   # VPC-only SSH
    else:
        rule["cidr_blocks"] = ["10.0.1.0/24"]   # App subnet only
```

### Why This Matters
- SSH open to internet → bots brute-force within minutes
- DB ports public → data theft, ransomware, deletion
- DB in public subnet → gets public IP, directly reachable
- CloudCanva makes insecure exports architecturally impossible

### Cognito JWT Security
- RS256 signature verification (asymmetric — can't be forged)
- Token expiration checked
- JWKS keys cached 1 hour, rotated by Cognito automatically
- token_use = "access" verified (prevents using ID tokens)

---

## 8. IAM & Quota Systems

### IAM Policy Generator (/generate-iam-policy)
**What**: Looks at user's design → computes minimum IAM permissions needed to deploy

**How**: Maps 14 resource types to specific IAM actions:
```
vpc → ec2:CreateVpc, DeleteVpc, DescribeVpcs, ModifyVpcAttribute, CreateTags
subnet → ec2:CreateSubnet, DeleteSubnet, DescribeSubnets, ModifySubnetAttribute, CreateTags
aws_instance → ec2:RunInstances, TerminateInstances, DescribeInstances, CreateTags
security_group → ec2:CreateSecurityGroup, DeleteSecurityGroup, DescribeSecurityGroups, CreateTags
internet_gateway → ec2:CreateInternetGateway, DeleteInternetGateway, AttachInternetGateway, DetachInternetGateway
route_table → ec2:CreateRouteTable, DeleteRouteTable, CreateRoute, DeleteRoute, AssociateRouteTable
s3_bucket → s3:CreateBucket, DeleteBucket, ListBucket, PutBucketTagging
load_balancer → elasticloadbalancing:CreateLoadBalancer, DeleteLoadBalancer, DescribeLoadBalancers
```

**Output**: Ready-to-use JSON policy document with exact actions needed

### IAM Permission Simulation (/validate-iam-permissions)
**What**: Tests if user's IAM role can actually deploy the design
**How**: Uses `iam:SimulatePrincipalPolicy` API
**Process**:
1. User provides role_arn or access keys
2. Backend determines required actions from design
3. Calls SimulatePrincipalPolicy in batches of 50
4. Returns: allowed/denied per action + "can_deploy: true/false"

### Quota Validation — Static (/check-quotas)
**What**: Compares design resource counts vs known AWS defaults
**Limits checked**: 5 VPCs, 5 EIPs, 20 EC2s, 200 Subnets, 500 SGs, 5 IGWs
**No credentials needed** — just counts what's in the design

### Quota Validation — Live (/check-quotas-live)
**What**: Checks user's actual AWS usage + planned additions
**How**:
1. User provides credentials (AssumeRole or access keys)
2. Backend calls: describe_vpcs, describe_addresses, describe_instances
3. Computes: current_count + design_count vs limit
4. Returns: "VPCs: 3 existing + 2 new = 5/5 → WARNING"

---

## 9. Terraform Generation

### What is Terraform?
Infrastructure as Code tool. Write code (HCL) → terraform apply → AWS resources created.
Industry standard, multi-cloud (AWS + Azure + GCP), 3000+ providers.

### What is HCL?
HashiCorp Configuration Language. Declarative config (describe WHAT, not HOW).
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

### Why Terraform over CloudFormation?
- Multi-cloud (not AWS-locked)
- Larger community
- Faster provider updates
- Better state management
- Industry standard (most companies use it)

### How Jinja2 Templates Work
Jinja2 is a Python templating engine. We use .tf.j2 files that have placeholders:
```jinja2
resource "aws_vpc" "{{ id.replace('-', '_') }}" {
  cidr_block = "{{ cidr_block }}"
  enable_dns_support = {{ enable_dns_support | default(true) | lower }}
  tags = {
    Name = "{{ name }}"
  }
}
```
When rendered with config `{id: "vpc-1", cidr_block: "10.0.0.0/16", name: "main-vpc"}`:
```hcl
resource "aws_vpc" "vpc_1" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  tags = {
    Name = "main-vpc"
  }
}
```

### Cross-Resource References
The `refs` system connects resources in generated Terraform:
- Subnet has `refs.vpc` → renders as `vpc_id = aws_vpc.{vpc_ref}.id`
- EC2 has `refs.subnet` → renders as `subnet_id = aws_subnet.{subnet_ref}.id`
- EC2 has `refs.security_group` → renders as `vpc_security_group_ids = [aws_security_group.{sg_ref}.id]`

---

## 10. Technical Decisions & Tradeoffs

### Why FastAPI (not Flask/Django)?
- Async support (needed for Bedrock calls)
- Built-in request validation (Pydantic)
- Auto-generated API docs (/docs)
- Dependency injection (Depends for auth)
- Faster than Flask for I/O-bound work

### Why React Flow (not D3.js/Cytoscape)?
- Built specifically for node-edge diagrams
- Drag-and-drop, zoom, pan out of box
- Custom node components
- Edge connection handling built-in
- Active community, React-native

### Why Next.js (not plain React/Vite)?
- Server-side rendering (better SEO, faster initial load)
- App router with file-based routing
- API routes (could use for BFF pattern)
- Built-in optimization (images, fonts, code splitting)
- Amplify native support

### Why Jinja2 (not string concatenation)?
- Proper templating with conditionals/loops
- Separation of template from logic
- Easy to add new resource types (just add .tf.j2 file)
- Readable, maintainable templates
- Industry-standard Python templating

### Why nohup/systemd (not PM2/Docker)?
- EC2 with Python — PM2 is for Node.js
- Docker adds unnecessary layer for single app
- systemd is Linux-native, auto-restart, auto-start on boot
- Simplest reliable solution

### Tradeoffs Accepted
| Decision | Tradeoff | Why acceptable |
|----------|----------|---------------|
| EC2 over Lambda | Pay for idle time | Bedrock needs 8s, Lambda timeout too risky |
| Single main.tf | No modules | Simpler output for beginners, can enhance later |
| 18 AI rules | Long prompt | Ensures reliable output, reduces post-processing |
| Post-processing | Extra latency (~100ms) | Guarantees correctness, negligible time |
| REGION_CONFIG dict | Manual AMI updates | Rarely changes, simple to maintain |

---

## 11. Interview Questions & Answers

### General Project Questions

**Q: What does CloudCanva do in one sentence?**
A: It converts natural language or visual designs into validated, secure, deployment-ready Terraform code for AWS infrastructure.

**Q: What's your role in the project?**
A: I built the entire platform end-to-end — FastAPI backend (15+ endpoints), Bedrock AI integration with 18-rule prompt engineering, 19 Jinja2 Terraform templates, IAM policy generator, quota validation, Cognito auth, S3+DynamoDB persistence, and the Next.js frontend with React Flow canvas.

**Q: What didn't you do?**
A: Production deployment (EC2 provisioning, DNS, CI/CD pipeline setup) — that was handled separately.

**Q: How long did it take?**
A: Built within the internship duration. Each mentor feedback (add AI, add security, add IAM) was shipped within the same sprint.

### Architecture Questions

**Q: Why not use a single service for everything (like Lambda)?**
A: Lambda has cold starts (5-15s), 30s API Gateway timeout (Bedrock takes 8s), no persistent file system for temp zips, 250MB package limit. EC2 gives us a persistent process, full control, and predictable performance.

**Q: Why separate S3 and DynamoDB? Why not store everything in one?**
A: Different tools for different jobs. S3 is for binary file storage (zips) — cheap, scalable, presigned URLs. DynamoDB is for fast metadata lookups by user — serverless, millisecond queries. Storing binary files in DynamoDB hits the 400KB item limit. Querying S3 by user_email is slow and expensive.

**Q: Why API Gateway between Amplify and EC2?**
A: Mixed content issue. Amplify serves over HTTPS. Browsers block HTTPS pages from calling HTTP endpoints. API Gateway gives us a free HTTPS URL that proxies to the HTTP EC2 backend.

**Q: How does authentication work?**
A: Cognito issues JWT tokens on sign-in. Frontend sends token in Authorization header. Backend fetches Cognito's JWKS (public keys), verifies RS256 signature, checks expiration and token_use. If valid, request proceeds. If not, 401 returned.

### AI & Prompt Engineering Questions

**Q: Why 18 rules in the system prompt?**
A: LLMs are probabilistic — they follow instructions most of the time, not all. 18 rules cover: output format (JSON only), correctness (refs, AMIs, AZs), positioning (visual layout), and security (port restrictions). More rules = more reliable output.

**Q: What if the AI doesn't follow the rules?**
A: That's what Layer 3 (post-processing) handles. It's a deterministic Python function that fixes every known failure pattern. Even if AI ignores all 18 rules, the output is still correct after post-processing.

**Q: Why Claude Sonnet and not GPT-4?**
A: Stays in AWS (no external API keys, data stays private), IAM-based auth (same as rest of infra), best structured JSON adherence among models we tested, and Bedrock pricing is pay-per-token with no subscription.

**Q: How do you handle Bedrock failures?**
A: If Bedrock throws an exception (timeout, throttling, etc.), the backend returns a 500 error with details. The frontend has a mock fallback generator for development/demo — keyword matching produces canned architectures.

**Q: What's the temperature setting?**
A: Currently default (no explicit temperature). Could add 0.7 for more varied outputs on repeated prompts.

### Security Questions

**Q: What happens if SSH is open to 0.0.0.0/0?**
A: Bots scan all AWS IPs 24/7 for port 22. Within minutes, brute-force attacks start. If password is weak, attacker gets shell access → data theft, crypto mining, lateral movement. CloudCanva makes this impossible — post-processing auto-restricts to VPC CIDR.

**Q: What if a database is in a public subnet?**
A: It gets a public IP. Combined with a misconfigured security group, anyone on the internet can connect directly. Ransomware bots encrypt data within hours. CloudCanva forces `map_public_ip_on_launch = false` for DB instances.

**Q: How is the JWT secured?**
A: RS256 (asymmetric) — private key stays with Cognito, we only use the public key to verify. Can't be forged. Tokens expire (short-lived). JWKS rotated automatically by Cognito.

### Terraform Questions

**Q: What's the difference between Terraform and CloudFormation?**
A: Terraform: multi-cloud, HCL language, larger community, state file you manage. CloudFormation: AWS-only, YAML/JSON, AWS manages state, tightly integrated with AWS but vendor-locked.

**Q: How do cross-resource references work in your generated Terraform?**
A: Through the `refs` system. If a Subnet has `refs.vpc = "vpc-1"`, the template renders `vpc_id = aws_vpc.vpc_1.id`. Terraform resolves this during plan/apply — it knows to create VPC first, then Subnet.

**Q: What happens when user runs terraform apply?**
A: Terraform reads main.tf → builds dependency graph → creates resources in correct order → stores state locally. If something fails, it reports which resource and why.

**Q: Why generate a flat main.tf instead of modules?**
A: Simplicity for our target audience (beginners, students). Modules add complexity (variables, outputs, module blocks). A single main.tf is easier to read, understand, and debug. Future enhancement could add module support.

### Frontend Questions

**Q: Why React Flow?**
A: Built specifically for node-edge interactive diagrams. Drag-and-drop, zoom, pan, custom nodes, edge connection — all built-in. D3.js would require building all of this from scratch.

**Q: How do you manage state between AI generation and canvas?**
A: AI dialog returns nodes/edges → calls `onArchitectureGenerated(nodes, edges)` → parent component sets them into React Flow's state via `setNodes()` and `setEdges()`. React Flow re-renders automatically.

**Q: How does the export work from frontend?**
A: Canvas component has `generateConfig()` that transforms React Flow nodes into the backend's expected JSON format (connected_services with services array). POSTs to /generate. Backend returns ZIP. Frontend creates a download link and triggers click.

### AWS-Specific Questions

**Q: What's a presigned URL?**
A: A temporary URL (we set 10-min expiry) that gives access to a specific S3 object without making the bucket public. Backend generates it using `generate_presigned_url()`. User's browser downloads directly from S3.

**Q: What's AssumeRole?**
A: STS operation that gives temporary credentials for a different IAM role. We use it for live quota checks — the user provides their role ARN, we assume it to query their account's resources.

**Q: What's SimulatePrincipalPolicy?**
A: IAM API that tests whether a principal (role/user) has permission to perform specific actions — without actually performing them. We use it to check "will terraform apply work?" before the user tries.

**Q: Why DynamoDB partition key = user_email?**
A: All queries are "give me all projects for this user." Partition key on user_email means these queries are single-partition lookups — millisecond response time. Sort key on project_name allows ordering and individual item access.

---

## 12. Potential Follow-up Questions

### Scalability
**Q: What if 1000 users use it simultaneously?**
A: EC2 would need scaling — either vertical (bigger instance) or horizontal (multiple EC2s behind ALB). DynamoDB scales automatically. S3 scales infinitely. Bedrock has per-account throttling limits. Would add API rate limiting on /ai/generate.

**Q: What's the bottleneck?**
A: Bedrock API calls (5-8s per request). Everything else is sub-second. Could add response caching for identical prompts.

### Error Handling
**Q: What if Bedrock returns invalid JSON?**
A: The backend strips markdown fences (```), trims whitespace, then tries json.loads(). If it still fails, the endpoint returns 500 with error details. Frontend shows error toast.

**Q: What if S3 upload fails?**
A: Caught in try/except, logged, but doesn't block the response — user still gets the ZIP directly via FileResponse. S3 is for persistence (re-download later), not the primary delivery.

**Q: What if DynamoDB save fails?**
A: Same — caught, logged, non-blocking. User gets their ZIP. They just won't see it in "My Projects" list.

### Improvements
**Q: What would you do differently if starting over?**
A: 
- Use modules in Terraform output (not flat main.tf)
- Add Terraform remote state generation (S3 backend + DynamoDB locking)
- Cache Bedrock responses for identical prompts
- Add WebSocket for real-time generation progress
- Use ECS Fargate for easier scaling (now that I've learned containers)

**Q: What's missing?**
A: Cost estimation, Terraform plan preview, more templates (Lambda, API Gateway, CloudFront), multi-cloud support, collaboration features.

### Concepts They Might Probe

**Q: Explain CORS.**
A: Cross-Origin Resource Sharing. Browser security that blocks frontend (origin A) from calling backend (origin B) unless backend explicitly allows it. FastAPI CORS middleware adds `Access-Control-Allow-Origin: *` headers. API Gateway also needs CORS configured.

**Q: Explain JWT.**
A: JSON Web Token. Three parts: header (algorithm), payload (claims like username, exp), signature (proves it's not tampered). Cognito signs with RS256 (private key). We verify with public key (JWKS). Stateless — no session storage needed.

**Q: Explain REST API.**
A: Representational State Transfer. HTTP methods map to operations: GET (read), POST (create), PUT (update), DELETE (remove). Stateless — each request contains all info needed. Our backend has 15+ REST endpoints.

**Q: What's Infrastructure as Code?**
A: Defining infrastructure (VPCs, servers, databases) in code files instead of clicking through a console. Benefits: version control, repeatable, reviewable, automated. Terraform is the most popular IaC tool.

**Q: Explain the difference between public and private subnets.**
A: Public subnet has a route to Internet Gateway + instances get public IPs. Private subnet has no internet route + no public IPs. Databases go in private (unreachable from internet). Web servers go in public (users need to access them).

**Q: What's a VPC?**
A: Virtual Private Cloud — an isolated network in AWS. You define CIDR block (IP range), create subnets inside it, attach gateways, configure route tables. All resources live inside a VPC.

**Q: What's an AMI?**
A: Amazon Machine Image — a snapshot/template for creating EC2 instances. Contains the OS + pre-installed software. AMIs are region-specific (an AMI in us-east-1 doesn't exist in eu-north-1). That's why CloudCanva auto-corrects AMIs per region.

**Q: What's an Availability Zone?**
A: A physically separate data center within a region. eu-north-1 has 3 AZs: eu-north-1a, 1b, 1c. Spreading resources across AZs gives high availability — if one AZ goes down, others keep running.

---

## Quick Revision Checklist

Before the interview, make sure you can explain:
- [ ] The 5-layer pipeline (draw it on paper)
- [ ] Why each AWS service was chosen (not alternatives)
- [ ] How authentication works end-to-end
- [ ] How Terraform generation works (config → template → render → zip)
- [ ] What post-processing fixes and why it exists
- [ ] The security guardrails (which ports restricted, why)
- [ ] IAM policy generator vs IAM permission simulation (different things!)
- [ ] Static vs live quota check (what's different)
- [ ] How frontend and backend communicate (JWT + HTTPS + API Gateway)
- [ ] What a presigned URL is and why you use it
- [ ] Why Terraform over CloudFormation
- [ ] Your contribution (everything except deployment)


---
---

# PART 2: DETAILED TECH STACK EXPLANATIONS

---

## Next.js 15 — Complete Explanation

### What is Next.js?
Next.js is a React framework built by Vercel. While React is a UI library (renders components), Next.js adds everything needed for production: routing, server-side rendering, static generation, API routes, and optimized builds.

### Why Next.js over plain React (Vite/CRA)?
| Feature | Plain React (Vite) | Next.js |
|---------|-------------------|---------|
| Routing | Need react-router (manual) | File-based (automatic) |
| SSR | Not built-in | Built-in |
| SEO | Poor (client-side only) | Great (SSR/SSG) |
| Code splitting | Manual | Automatic per page |
| Image optimization | Manual | Built-in next/image |
| API routes | Need separate server | Built-in |
| Build output | Static SPA | Static + Server + Edge |

### App Router (Next.js 13+)
Next.js 15 uses the App Router (not Pages Router). Key differences:
- Files in `app/` directory define routes
- `app/page.tsx` = home page (/)
- `app/projects/page.tsx` = /projects
- `app/signin/page.tsx` = /signin
- `layout.tsx` wraps all pages (persistent UI like nav)
- `"use client"` directive marks client components
- Server Components are default (better performance)

### How We Use Next.js in CloudCanva
```
app/
├── page.tsx          → / (canvas page, main app)
├── layout.tsx        → wraps all pages (ThemeProvider, metadata)
├── globals.css       → Tailwind + custom styles
├── signin/page.tsx   → /signin (login form)
├── signup/page.tsx   → /signup (registration form)
├── projects/page.tsx → /projects (saved projects list)
└── forgot-password/  → /forgot-password
```

### Key Next.js Concepts Used
1. **Client Components** (`"use client"`): Canvas, AI dialog, forms — need browser APIs (useState, useEffect, event handlers)
2. **File-based routing**: Each folder in `app/` = a route
3. **Layout nesting**: Root layout applies to all pages
4. **Environment variables**: `NEXT_PUBLIC_` prefix exposes to browser
5. **Dynamic imports**: Heavy components loaded on demand

---

## React 19 — Complete Explanation

### What is React?
A JavaScript UI library for building user interfaces with components. Components are reusable pieces of UI that manage their own state and render based on data.

### Key React Concepts Used in CloudCanva

#### 1. Components
Everything is a component:
```tsx
// Functional component
export function AiArchitectureGenerator({ onArchitectureGenerated }) {
  return <Dialog>...</Dialog>;
}
```

#### 2. State (useState)
```tsx
const [nodes, setNodes] = useState([]);        // canvas nodes
const [prompt, setPrompt] = useState("");       // AI input
const [isLoading, setIsLoading] = useState(false);  // loading spinner
```
When state changes → component re-renders → UI updates.

#### 3. Effects (useEffect)
```tsx
useEffect(() => {
  loadProjects();  // runs once on component mount
}, []);
```
Side effects: API calls, subscriptions, DOM manipulation.

#### 4. Props
Data passed from parent to child:
```tsx
<Canvas aiNodes={nodes} aiEdges={edges} selectedRegion="eu-north-1" />
```

#### 5. Context (useContext)
Shared state without prop drilling:
```tsx
// AuthContext provides { user, isAuthenticated, isLoading } to entire app
const { isAuthenticated } = useAuth();
```

#### 6. Refs (useRef)
```tsx
const reactFlowWrapper = useRef(null);  // reference to DOM element
```

### React 19 Specific Features
- Concurrent rendering (smoother UI updates)
- Automatic batching of state updates
- Improved Suspense boundaries
- Server Components (used by Next.js)

---

## TypeScript — Complete Explanation

### What is TypeScript?
TypeScript = JavaScript + static types. You define types for variables, function parameters, return values. Compiler catches errors at build time instead of runtime.

### How We Use TypeScript in CloudCanva

#### Interface Definitions
```tsx
interface Project {
  project_name: string;
  created_at: string;
  node_count: number;
  s3_key: string;
}

interface AiArchitectureGeneratorProps {
  onArchitectureGenerated: (nodes: any[], edges: any[]) => void;
}
```

#### Type Safety Benefits
```tsx
// Without TS: runtime error if project.node_count is undefined
// With TS: compiler warns at build time

const [projects, setProjects] = useState<Project[]>([]);
// Now TypeScript knows projects is an array of Project objects
// Autocomplete works: projects[0].project_name ✓
// Typo caught: projects[0].porject_name ✗ (compiler error)
```

#### Why TypeScript in CloudCanva?
- Canvas has complex node/edge data structures
- API responses need clear type definitions
- Refactoring is safer (rename shows all usages)
- IDE autocomplete speeds up development
- Catches null/undefined errors at compile time

---

## Tailwind CSS — Complete Explanation

### What is Tailwind?
Utility-first CSS framework. Instead of writing custom CSS classes, you compose styles from pre-built utility classes directly in HTML/JSX.

### Traditional CSS vs Tailwind
```css
/* Traditional */
.button {
  background-color: #0ea5e9;
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
  font-weight: 500;
}
```
```tsx
/* Tailwind */
<button className="bg-sky-500 text-white px-4 py-2 rounded-lg font-medium">
```

### Tailwind Classes Used in CloudCanva
```
bg-[#0f172a]     → custom dark background
text-white       → white text
border-slate-700 → gray border
rounded-xl       → large border radius
p-4              → padding 16px
gap-2            → flex gap 8px
hover:bg-sky-600 → hover state
transition-colors → smooth color transitions
```

### Why Tailwind (not plain CSS/styled-components)?
- No context switching (styles in same file as JSX)
- No naming (no .btn-primary-large-active class names)
- Consistent design system (spacing, colors predefined)
- Smaller bundle (only used classes included)
- Fast iteration (change className, see result)
- Dark theme support built-in

---

## React Flow — Complete Explanation

### What is React Flow?
A React library for building interactive node-based editors and diagrams. Used for: flowcharts, mind maps, workflow builders, data pipelines, and in our case — AWS architecture diagrams.

### Core Concepts
1. **Nodes**: Draggable boxes on the canvas (each AWS service is a node)
2. **Edges**: Lines connecting nodes (show relationships/dependencies)
3. **Handles**: Connection points on nodes (where edges attach)
4. **Viewport**: The visible area (zoom, pan)

### How CloudCanva Uses React Flow
```tsx
<ReactFlow
  nodes={nodes}           // AWS service nodes
  edges={edges}           // connections between services
  onConnect={onConnect}   // handle new connection
  onNodesChange={onNodesChange}  // drag, select, delete
  onEdgesChange={onEdgesChange}
  nodeTypes={nodeTypes}   // custom node components
  fitView                 // auto-zoom to fit all nodes
>
  <Background />
  <Controls />
  <Panel position="top-center">...</Panel>
</ReactFlow>
```

### Node Data Structure
```tsx
{
  id: "vpc-1",
  type: "service",
  position: { x: 200, y: 50 },
  data: {
    label: "VPC",
    type: "network",
    provider: "aws",
    rawLabel: "vpc",
    serviceName: "Main VPC",
    properties: {
      name: "main-vpc",
      cidr_block: "10.0.0.0/16",
      refs: {},
      tags: {}
    }
  }
}
```

### Edge Data Structure
```tsx
{
  id: "edge-1",
  source: "ec2-1",    // from this node
  target: "subnet-1", // to this node
}
```

### Why React Flow (not D3.js/Vis.js/Cytoscape)?
| Feature | React Flow | D3.js | Cytoscape |
|---------|-----------|-------|-----------|
| React integration | Native | Wrapper needed | Wrapper needed |
| Drag-and-drop | Built-in | Manual | Built-in |
| Custom nodes | Easy (React components) | SVG manipulation | Limited |
| Edge routing | Built-in | Manual | Built-in |
| Minimap | Built-in plugin | Manual | Plugin |
| Learning curve | Low (React devs) | Very high | Medium |
| Performance | Optimized | Manual optimization | Good |

---

## FastAPI — Complete Explanation

### What is FastAPI?
A modern Python web framework for building APIs. Built on Starlette (async) and Pydantic (validation). Known for being fast, easy, and auto-generating documentation.

### Why FastAPI (not Flask/Django/Express)?
| Feature | FastAPI | Flask | Django | Express |
|---------|---------|-------|--------|---------|
| Async | Native | Extension | Extension | Native |
| Validation | Pydantic (automatic) | Manual | Forms/Serializers | Manual |
| API docs | Auto (/docs) | Manual | DRF has it | Manual |
| Performance | Very fast (Starlette) | Slower | Slower | Fast |
| Type hints | Full support | Optional | Optional | N/A (JS) |
| Dependency injection | Built-in (Depends) | Manual | Middleware | Manual |
| Python | Yes | Yes | Yes | No (JS) |

### Key FastAPI Features Used in CloudCanva

#### 1. Route Decorators
```python
@app.get("/")           # GET request to /
@app.post("/generate")  # POST request to /generate
@app.delete("/projects/{project_name}")  # DELETE with path parameter
```

#### 2. Dependency Injection
```python
@app.post("/generate")
async def generate(payload: Dict[str, Any], user: dict = Depends(get_current_user)):
    # get_current_user runs first, verifies JWT
    # If invalid → 401 returned automatically
    # If valid → user dict passed to function
```

#### 3. Request Body Validation
```python
async def generate(payload: Dict[str, Any]):
    # FastAPI auto-parses JSON body into Python dict
    # If body isn't valid JSON → 422 error automatically
```

#### 4. CORS Middleware
```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],       # allow all origins
    allow_credentials=True,
    allow_methods=["*"],       # allow all HTTP methods
    allow_headers=["*"],       # allow all headers
)
```

#### 5. Response Types
```python
return {"message": "ok"}           # JSON response (default)
return FileResponse(path=zip_path)  # File download
raise HTTPException(status_code=500, detail="error message")  # Error
```

### How FastAPI Handles a Request (Step by Step)
1. Request arrives at EC2:8000
2. CORS middleware adds headers (allows cross-origin)
3. Route matched (e.g., POST /ai/generate)
4. Dependencies resolved (get_current_user verifies JWT)
5. Request body parsed into Python dict
6. Handler function executes (calls Bedrock, etc.)
7. Response serialized to JSON
8. Response sent back with headers

---

## Python — Key Libraries Used

### Boto3 (AWS SDK for Python)
Boto3 is the official AWS SDK (Software Development Kit) for Python. It allows Python applications to interact programmatically with AWS services such as Amazon S3, EC2, DynamoDB, Bedrock, Cognito, IAM, STS, and many others.

Instead of manually calling AWS REST APIs, developers use Boto3's Python methods to perform AWS operations.
```python
import boto3

# Create clients for different AWS services
s3_client = boto3.client("s3", region_name="eu-north-1")
dynamodb = boto3.resource("dynamodb", region_name="eu-north-1")
bedrock_client = boto3.client("bedrock-runtime", region_name="eu-north-1")
sts_client = boto3.client("sts", region_name="eu-north-1")

# S3 operations
s3_client.upload_file(local_path, bucket, s3_key)
s3_client.generate_presigned_url("get_object", Params={...}, ExpiresIn=600)
s3_client.delete_object(Bucket=bucket, Key=key)

# DynamoDB operations
table = dynamodb.Table("cloudcanva-projects")
table.put_item(Item={...})
table.query(KeyConditionExpression=Key("user_email").eq(email))
table.get_item(Key={...})
table.delete_item(Key={...})

# Bedrock operations
response = bedrock_client.invoke_model(modelId="eu.anthropic.claude-sonnet-4-6", body=json_body)

# STS operations
sts_client.assume_role(RoleArn=arn, RoleSessionName="session")
```

### Jinja2 (Template Engine)
```python
from jinja2 import FileSystemLoader, Environment

env = Environment(loader=FileSystemLoader("templates"))
template = env.get_template("aws/vpc.tf.j2")
rendered = template.render({"cidr_block": "10.0.0.0/16", "name": "main-vpc"})
```

### python-jose (JWT Verification)
```python
from jose import jwt, JWTError

claims = jwt.decode(
    token,
    public_key,        # from Cognito JWKS
    algorithms=["RS256"],
    issuer=ISSUER,
    options={"verify_exp": True}
)
```

---

## Jinja2 Templates — Deep Explanation

### What is Jinja2?
A Python templating engine. You write template files with placeholders (`{{ }}`), then render them with data. The placeholders get replaced with actual values.

### Syntax
```jinja2
{{ variable }}                    → output a value
{% if condition %}...{% endif %}  → conditional
{% for item in list %}...{% endfor %} → loop
{{ value | default("fallback") }} → filter with default
{{ value | lower }}               → convert to lowercase
```

### Real Example: VPC Template (vpc.tf.j2)
```jinja2
resource "aws_vpc" "{{ id.replace('-', '_') }}" {
  cidr_block           = "{{ cidr_block }}"
  enable_dns_support   = {{ enable_dns_support | default(true) | lower }}
  enable_dns_hostnames = {{ enable_dns_hostname | default(true) | lower }}
  instance_tenancy     = "{{ instance_tenancy | default('default') }}"

  tags = {
    Name = "{{ name }}"
    {% if tags %}
    {% for key, value in tags.items() %}
    {{ key }} = "{{ value }}"
    {% endfor %}
    {% endif %}
  }
}
```

### Rendered Output (with data)
Input: `{id: "vpc-1", cidr_block: "10.0.0.0/16", name: "main-vpc", tags: {}}`
```hcl
resource "aws_vpc" "vpc_1" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  instance_tenancy     = "default"

  tags = {
    Name = "main-vpc"
  }
}
```

### Real Example: EC2 Template (ec2.tf.j2)
```jinja2
resource "aws_instance" "{{ id.replace('-', '_') }}" {
  ami           = "{{ ami }}"
  instance_type = "{{ instance_type }}"

  {% if refs.subnet %}
  subnet_id = aws_subnet.{{ refs.subnet.replace('-', '_') }}.id
  {% endif %}

  {% if refs.security_group %}
  vpc_security_group_ids = [aws_security_group.{{ refs.security_group.replace('-', '_') }}.id]
  {% endif %}

  tags = {
    Name = "{{ name }}"
  }
}
```

### Why Jinja2 (not f-strings/string concat)?
- Clean separation of template from logic
- Conditionals and loops in templates
- Filters (default values, case conversion)
- Adding new resource = just add a .tf.j2 file (no code changes)
- Battle-tested, used in Ansible, Flask, Django

---

## Terraform — Complete Explanation

### What is Terraform?
An Infrastructure as Code (IaC) tool by HashiCorp. You describe infrastructure in HCL (HashiCorp Configuration Language), then:
- `terraform init` → downloads required providers
- `terraform plan` → shows what will be created/changed
- `terraform apply` → creates the actual AWS resources
- `terraform destroy` → removes everything

### HCL Syntax
```hcl
# Provider (which cloud + region)
provider "aws" {
  region = "eu-north-1"
}

# Resource (what to create)
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "main-vpc"
  }
}

# Reference another resource
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id    # ← references VPC above
  cidr_block = "10.0.1.0/24"
}
```

### How Terraform Knows the Order
Terraform builds a dependency graph from references:
- Subnet references VPC (`aws_vpc.main.id`) → VPC created first
- EC2 references Subnet → Subnet created first
- No explicit ordering needed — Terraform figures it out

### State File
Terraform stores what it created in `terraform.tfstate`. This lets it:
- Know what exists (for plan/destroy)
- Track changes (what was modified)
- Map resource names to real IDs

### Why Terraform (not CloudFormation)?
| Aspect | Terraform | CloudFormation |
|--------|-----------|---------------|
| Cloud support | AWS, Azure, GCP, 3000+ | AWS only |
| Language | HCL (concise) | YAML/JSON (verbose) |
| Community | Massive, open-source | AWS-only ecosystem |
| State | You manage (S3 backend) | AWS manages |
| Speed | Fast (parallel) | Slower (sequential) |
| Modules | First-class, reusable | Nested stacks (clunky) |
| Debugging | Clear error messages | Cryptic rollback errors |
| Learning | Widely transferable skill | Only useful for AWS |

---

## Amazon Cognito — Complete Explanation

### What is Cognito?
A managed authentication service. Handles: user sign-up, sign-in, password reset, email verification, JWT token issuance, MFA — all without writing auth code.

### Key Components
1. **User Pool**: Database of users (emails, passwords, attributes)
2. **App Client**: Frontend connects to this (client ID, no secret for public apps)
3. **JWKS Endpoint**: Public keys for verifying JWT tokens
4. **Hosted UI** (optional): Pre-built login pages (we use custom UI instead)

### How It Works in CloudCanva
```
1. User enters email + password on /signin page
2. Frontend calls Cognito via Amplify SDK: signIn({username, password})
3. Cognito verifies credentials against User Pool
4. If valid → returns tokens:
   - Access Token (used for API auth)
   - ID Token (contains user info)
   - Refresh Token (get new tokens when expired)
5. Frontend stores Access Token
6. Every API request: Authorization: Bearer {access_token}
7. Backend verifies token using JWKS public keys
```

### JWT Token Structure
```
Header: { "alg": "RS256", "kid": "abc123" }
Payload: {
  "sub": "uuid-of-user",
  "token_use": "access",
  "scope": "aws.cognito.signin.user.admin",
  "iss": "https://cognito-idp.ap-south-1.amazonaws.com/ap-south-1_xxx",
  "exp": 1719700000,
  "username": "user@example.com"
}
Signature: RS256(header + payload, cognito_private_key)
```

### RS256 vs HS256
- **HS256** (symmetric): Same secret to sign and verify. If backend is compromised, attacker can forge tokens.
- **RS256** (asymmetric): Private key signs (only Cognito has it). Public key verifies (anyone can verify, no one can forge). More secure for distributed systems.

### Why Cognito (not build custom auth)?
Building auth from scratch requires:
- Password hashing (bcrypt/argon2)
- Database for users
- Token signing/rotation
- Email verification flow
- Password reset flow
- Rate limiting on login
- Brute-force protection
- MFA implementation
- JWKS endpoint management

Cognito does ALL of this with zero code. Configuration only.

---

## Amazon Bedrock — Complete Explanation

### What is Bedrock?
A managed service to use foundation models (LLMs) via API. No infrastructure to manage — just call the API with a prompt, get a response.

### Available Models on Bedrock
- Claude (Anthropic) — Claude Sonnet 4, Claude Haiku, Claude Opus
- Llama (Meta)
- Titan (Amazon's own model)
- Mistral
- Stable Diffusion (images)

### Why Claude Sonnet 4 Specifically?
| Model | Strength | Weakness for our use case |
|-------|----------|--------------------------|
| Claude Haiku | Fast, cheap | Too small — struggles with 18 rules and large JSON |
| Claude Sonnet 4 | Best balance of speed + quality | None significant |
| Claude Opus | Most capable | Slower, more expensive, overkill |
| GPT-4 (OpenAI) | Very capable | External to AWS, data leaves boundary |
| Llama (Meta) | Open source | Worse instruction following for structured output |

### How We Call Bedrock
```python
client = boto3.client("bedrock-runtime", region_name="eu-north-1")

body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 4096,
    "messages": [{"role": "user", "content": system_prompt + user_prompt}]
})

response = client.invoke_model(
    modelId="eu.anthropic.claude-sonnet-4-6",
    body=body,
    contentType="application/json"
)

result = json.loads(response["body"].read())
ai_text = result["content"][0]["text"]
architecture = json.loads(ai_text)  # parse JSON from AI response
```

### Prompt Engineering Details
The system prompt is ~2000 characters with 18 rules. Key design decisions:
- Rules are numbered (AI follows numbered lists better)
- Critical rules are CAPITALIZED ("NEVER", "MUST", "ALWAYS")
- Examples provided in the format string (shows exact JSON structure)
- Region-specific values injected dynamically (correct AMI, correct AZs)
- Security rules are last (most important = most recent in context)

### Error Handling
```python
try:
    result = generate_with_bedrock(prompt, region, region_conf)
    result = post_process_ai_response(result, region, region_conf)
    return result
except Exception as e:
    raise HTTPException(status_code=500, detail=f"AI generation failed: {str(e)}")
```

### Cost
- Pay per token (input + output)
- Claude Sonnet 4: ~$3/million input tokens, ~$15/million output tokens
- Average request: ~2000 input tokens + ~2000 output tokens ≈ $0.04 per generation
- Very cheap at low scale

---

## Amazon S3 — Complete Explanation

### What is S3?
Simple Storage Service — object storage. Store any file (binary, text, images) with a unique key. Infinitely scalable, 99.999999999% durability (11 nines).

### Key Concepts
- **Bucket**: Container for objects (like a folder). Globally unique name.
- **Object**: File stored in bucket. Has key (path), value (content), metadata.
- **Key**: Path to object. Example: `users/user@email.com/my-project.zip`
- **Presigned URL**: Temporary URL granting access to specific object.

### How CloudCanva Uses S3
```python
# Upload ZIP after Terraform generation
s3_client.upload_file(
    Filename=zip_file_path,           # local file
    Bucket="cloudcanva-terraform-exports",  # bucket name
    Key=f"users/{email}/{project_name}.zip"  # S3 path
)

# Generate presigned URL for download (expires in 10 min)
presigned_url = s3_client.generate_presigned_url(
    "get_object",
    Params={"Bucket": bucket, "Key": s3_key},
    ExpiresIn=600  # seconds
)

# Delete object when user deletes project
s3_client.delete_object(Bucket=bucket, Key=s3_key)
```

### What is a Presigned URL?
A URL with temporary credentials baked in. Allows downloading a private S3 object without making the bucket public.

Format: `https://bucket.s3.amazonaws.com/key?X-Amz-Algorithm=...&X-Amz-Credential=...&X-Amz-Expires=600&X-Amz-Signature=...`

After 600 seconds → URL expires → access denied.

### Why Presigned URLs (not public bucket)?
- Public bucket: ANYONE can access ALL files forever
- Presigned: SPECIFIC user can access SPECIFIC file for LIMITED time
- Security principle: least privilege access

---

## Amazon DynamoDB — Complete Explanation

### What is DynamoDB?
A managed NoSQL key-value database. Serverless — no provisioning, scales automatically, millisecond reads.

### Key Concepts
- **Table**: Collection of items (like a SQL table)
- **Item**: A single record (like a SQL row)
- **Partition Key**: Primary lookup field (distributes data)
- **Sort Key**: Secondary ordering field (within a partition)
- **Query**: Fast lookup by partition key (+ optional sort key filter)
- **Scan**: Read entire table (slow, expensive — avoid)

### CloudCanva DynamoDB Schema
```
Table: cloudcanva-projects
├── Partition Key: user_email (String)
├── Sort Key: project_name (String)
└── Attributes:
    ├── s3_key (String) — path to ZIP in S3
    ├── created_at (String) — ISO timestamp
    └── node_count (Number) — resources in design
```

### Why This Schema?
- Query by user_email → returns ALL projects for that user (fast, single partition)
- Sort by project_name → allows getting specific project
- Composite key (user_email + project_name) → unique per item
- No need for secondary indexes at this scale

### Operations Used
```python
# Save project
table.put_item(Item={
    "user_email": "user@email.com",
    "project_name": "my-vpc-setup",
    "s3_key": "users/user@email.com/my-vpc-setup.zip",
    "created_at": "2026-06-29T10:30:00Z",
    "node_count": 7
})

# List user's projects
response = table.query(
    KeyConditionExpression=Key("user_email").eq("user@email.com")
)
projects = response["Items"]

# Get specific project
response = table.get_item(Key={
    "user_email": "user@email.com",
    "project_name": "my-vpc-setup"
})

# Delete project
table.delete_item(Key={
    "user_email": "user@email.com",
    "project_name": "my-vpc-setup"
})
```

### DynamoDB vs RDS (SQL) for This Use Case
| Aspect | DynamoDB | RDS (PostgreSQL) |
|--------|----------|-----------------|
| Setup | Zero (serverless) | Provision instance, configure, maintain |
| Scaling | Automatic | Manual (resize instance) |
| Cost at low scale | ~$0 (pay-per-request) | ~$15-50/month minimum |
| Query pattern | Key-value lookup | Complex queries, joins |
| Schema changes | Just add attributes | ALTER TABLE migrations |
| Connection management | HTTP API (no pools) | Connection pooling required |
| Our need | Simple: get projects by user | Overkill complexity |

---

## Amazon EC2 — Complete Explanation

### What is EC2?
Elastic Compute Cloud — virtual servers in AWS. You choose instance type (CPU, RAM), AMI (operating system), and run anything you want.

### Our EC2 Setup
- **Instance type**: t3.micro (2 vCPU, 1GB RAM) — enough for FastAPI
- **AMI**: Ubuntu 24.04
- **Storage**: Default EBS (8GB gp3)
- **Security Group**: Port 8000 open (FastAPI), Port 22 (SSH)
- **IAM Role**: EpoxyChronicleInstanceRole (Bedrock + S3 + DynamoDB access)
- **Region**: eu-north-1

### Why EC2 (Detailed Comparison with Lambda)
| Scenario | EC2 | Lambda |
|----------|-----|--------|
| Bedrock call (8 seconds) | No problem — persistent process | Risky — API Gateway has 30s timeout |
| Write temp files (Terraform generation) | Full file system, unlimited | /tmp only, 512MB limit, ephemeral |
| Cold start | None — always running | 5-15s first invocation |
| Cost at low traffic | ~$8/month (t3.micro) | Possibly cheaper (pay-per-invocation) |
| Cost at high traffic | Same $8/month | Could get expensive ($$$) |
| Deployment | SSH + git pull + restart | Package code + upload + configure |
| Debugging | SSH in, check logs, test locally | CloudWatch logs (delayed, fragmented) |
| Dependencies | pip install anything | Layer management, package size limits |

### IAM Instance Role
EC2 doesn't need hardcoded credentials. Instead, it has an IAM role attached. Boto3 automatically picks up credentials from the instance metadata service.

```python
# No credentials in code!
s3_client = boto3.client("s3", region_name="eu-north-1")
# Boto3 auto-uses instance role credentials
```

Role permissions needed:
- `bedrock:InvokeModel` — AI generation
- `dynamodb:Query/PutItem/GetItem/DeleteItem` — project CRUD
- `s3:PutObject/GetObject/DeleteObject` — ZIP storage
- `sts:AssumeRole` — for live quota checks

---

## AWS Amplify — Complete Explanation

### What is Amplify?
A platform for hosting full-stack web apps. Handles: build, deploy, SSL, CDN, custom domains, CI/CD — all from pushing to Git.

### How We Use Amplify
1. Connected GitHub repo (frontend folder)
2. Amplify detects Next.js → uses appropriate build settings
3. On every git push → auto-builds → auto-deploys
4. Provides HTTPS URL: `https://main.dayrf9zus4ibl.amplifyapp.com`
5. Environment variables set in Amplify Console

### Amplify Build Process
```
1. Clones repo from GitHub
2. Runs: npm install
3. Runs: npm run build (next build)
4. Deploys built output to CDN edge locations
5. SSL certificate auto-provisioned
6. URL goes live
```

### Amplify SDK (frontend auth)
```typescript
import { Amplify } from "aws-amplify";
import { signIn, signUp, getCurrentUser } from "aws-amplify/auth";

Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: process.env.NEXT_PUBLIC_COGNITO_USER_POOL_ID,
      userPoolClientId: process.env.NEXT_PUBLIC_COGNITO_CLIENT_ID,
    }
  }
});

// Sign in
const result = await signIn({ username: email, password });

// Get current user
const user = await getCurrentUser();
```

---
---

# PART 3: NETWORKING & AWS CONCEPTS FOR INTERVIEW

---

## VPC (Virtual Private Cloud)
An isolated virtual network in AWS. You define:
- CIDR block (IP address range): e.g., 10.0.0.0/16 = 65,536 IP addresses
- Subnets within the VPC
- Gateways (Internet Gateway, NAT Gateway)
- Route tables (traffic rules)
- Security Groups (firewall rules)

Everything runs inside a VPC. It's your private network in the cloud.

## Subnet
A subdivision of a VPC's IP range.
- **Public subnet**: Has route to Internet Gateway. Instances get public IP.
- **Private subnet**: No route to internet. Instances have no public IP. Use NAT Gateway for outbound internet.

Example:
- VPC: 10.0.0.0/16 (65,536 IPs)
- Public Subnet: 10.0.1.0/24 (256 IPs)
- Private Subnet: 10.0.2.0/24 (256 IPs)

## CIDR Block
Notation for IP address ranges. `10.0.0.0/16` means:
- First 16 bits are fixed (10.0)
- Last 16 bits are flexible (0.0 to 255.255)
- = 65,536 possible addresses

`/24` = 256 addresses, `/32` = 1 address (single IP)

## Internet Gateway (IGW)
Allows resources in public subnets to reach the internet.
- Attached to a VPC
- Route table has: `0.0.0.0/0 → IGW` (all internet traffic goes through IGW)
- Without IGW, nothing in the VPC can reach the internet

## NAT Gateway
Allows private subnet resources to reach the internet (for updates, API calls) WITHOUT giving them a public IP.
- Placed in a public subnet
- Private route table has: `0.0.0.0/0 → NAT Gateway`
- Internet → NAT → private resource: BLOCKED (inbound)
- Private resource → NAT → Internet: ALLOWED (outbound only)

## Route Table
Defines where network traffic goes.
```
Destination     | Target
10.0.0.0/16     | local (within VPC)
0.0.0.0/0       | igw-xxx (internet)
```
- Public route table: has IGW route → instances can reach internet
- Private route table: no IGW route (or NAT only) → no direct internet access

## Security Group
A virtual firewall for resources. Has inbound (ingress) and outbound (egress) rules.
- **Stateful**: If you allow inbound traffic, response is automatically allowed outbound.
- Rules specify: protocol, port range, source/destination CIDR

Example:
```
Inbound:
  TCP 80 from 0.0.0.0/0 (HTTP from anywhere)
  TCP 22 from 10.0.0.0/16 (SSH from VPC only)
Outbound:
  All traffic to 0.0.0.0/0 (allow everything out)
```

## Elastic IP (EIP)
A static public IP address. Without EIP, EC2's public IP changes on stop/start. With EIP, it stays the same.

## AMI (Amazon Machine Image)
A template for EC2 instances. Contains: operating system, pre-installed software, configuration. AMIs are region-specific — must use correct AMI for the region you're deploying in.

## Availability Zone (AZ)
A physically separate data center within a region. eu-north-1 has:
- eu-north-1a (data center A)
- eu-north-1b (data center B)
- eu-north-1c (data center C)

Spreading across AZs = high availability. If one AZ goes down, others keep running.

## Load Balancer (ALB)
Application Load Balancer distributes incoming traffic across multiple targets (EC2 instances).
- Needs subnets in 2+ AZs
- Health checks — stops sending traffic to unhealthy instances
- HTTPS termination — handles SSL certificates

## Target Group
Where the Load Balancer sends traffic. Contains a list of targets (EC2 instances) + health check configuration.

## RDS (Relational Database Service)
Managed database service. You choose engine (PostgreSQL, MySQL), instance class, storage. AWS handles: backups, patching, failover.
- Needs a DB Subnet Group (subnets in 2+ AZs)
- Should be in private subnets (no public access)

---
---

# PART 4: ADDITIONAL INTERVIEW SCENARIOS

---

## Scenario-Based Questions

**Q: A user creates a design with 6 VPCs. What happens?**
A: The static quota check (`/check-quotas`) counts 6 VPCs and returns a warning: "Your design creates 6 VPCs. AWS default limit is 5 per region. Deployment WILL fail." The user sees this warning before exporting.

**Q: A user types "give me infrastructure" (vague prompt). What happens?**
A: Bedrock interprets it as best it can — likely generates a basic EC2 setup (VPC + Subnet + EC2 + SG). Post-processing ensures all refs are correct. User previews and can regenerate with a better prompt.

**Q: The AI generates SSH open to 0.0.0.0/0 despite the rules. What happens?**
A: Post-processing catches it. The function checks every ingress rule: if port 22 has cidr 0.0.0.0/0, it replaces with 10.0.0.0/16. The user NEVER sees or exports an insecure configuration.

**Q: A user selects ap-south-1 but AI outputs us-east-1 AMIs. What happens?**
A: Post-processing checks every EC2 node's AMI against a list of known wrong-region AMIs. If it doesn't match the region's correct AMI, it's replaced. AZ values are also checked — if they don't start with the selected region prefix, they're replaced.

**Q: DynamoDB is down. Can users still export?**
A: Yes. DynamoDB save is in a try/except with non-blocking error handling. The ZIP is still generated, uploaded to S3, and returned to the user. They just won't see it in "My Projects." The export itself doesn't depend on DynamoDB.

**Q: S3 is down. Can users still export?**
A: Partially. The ZIP is generated locally on EC2 and returned directly via FileResponse. S3 upload fails (logged), but user still downloads the file. They just can't re-download later from "My Projects."

**Q: How do you handle concurrent users?**
A: FastAPI is async — handles multiple requests concurrently. Each request generates its own temp directory (unique path via `tempfile.mkdtemp()`). S3 keys include user email (no collisions). DynamoDB has per-user partition (no conflicts).

**Q: What if Bedrock is throttled (too many requests)?**
A: The endpoint catches the exception and returns 500 with details. No fallback to mock in production — user sees "AI generation failed" and can retry. Could add retry with exponential backoff in future.

**Q: How would you add a new AWS resource type (e.g., Lambda)?**
A:
1. Create template: `templates/aws/lambda/function.tf.j2`
2. Add to registry: `"aws_lambda_function": "aws/lambda/function.tf.j2"`
3. Add to AI prompt: include Lambda in the list of supported labels
4. Add to frontend sidebar: new service card for Lambda
5. Add IAM mapping: `"lambda_function": ["lambda:CreateFunction", "lambda:DeleteFunction", ...]`
That's it — clean separation of concerns makes extension easy.

---

## System Design Questions They Might Ask

**Q: How would you make this highly available?**
A:
- Backend: Multiple EC2 instances behind an ALB, across 2+ AZs
- Or migrate to ECS Fargate (auto-scaling containers)
- S3 and DynamoDB are already highly available (AWS-managed)
- Cognito is already HA (AWS-managed)
- Add health checks and auto-recovery

**Q: How would you handle 10,000 concurrent users?**
A:
- Horizontal scaling: ECS Fargate with auto-scaling (CPU/memory based)
- Rate limiting on /ai/generate (Bedrock has per-account limits)
- Cache identical prompts (Redis/ElastiCache)
- CDN for static assets (already on Amplify)
- DynamoDB auto-scales (on-demand billing)
- S3 handles unlimited concurrent uploads

**Q: How would you add multi-cloud (Azure)?**
A:
- Add Azure templates folder: `templates/azure/`
- Add Azure provider template
- Extend template registry with `azure_` prefixed keys
- Update AI prompt to support Azure resource types
- Add Azure AMI/region config
- Frontend: add Azure services to sidebar
- Same architecture, just more templates

**Q: How would you add cost estimation?**
A:
- Create a pricing dictionary: `{"t3.micro": 0.0104, "t3.small": 0.0208, ...}` (hourly rates)
- For each resource in design, look up hourly cost
- Multiply by 730 hours (monthly)
- Sum all resources → total monthly estimate
- Show on canvas as a running total

---

## Final Tips for the Interview

1. **Start with the big picture** — describe the 5-layer pipeline before diving into details
2. **Use numbers** — "18 rules", "19 templates", "15+ endpoints", "6 regions", "14 resource types"
3. **Explain WHY, not just WHAT** — "We use DynamoDB because..." not just "We use DynamoDB"
4. **Mention tradeoffs** — shows maturity ("EC2 over Lambda because Bedrock takes 8s")
5. **Security is a differentiator** — emphasize the auto-enforcement, not just checking
6. **Know alternatives** — for every choice, know what you didn't pick and why
7. **Admit what you didn't do** — "I didn't handle deployment" shows honesty
8. **Draw diagrams** — if asked to explain architecture, draw the 3-layer diagram (Frontend → Backend → AWS Services)
