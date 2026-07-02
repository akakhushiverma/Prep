# CloudCanva — Detailed Code Walkthrough Script

**Total Duration: ~20-25 minutes**
**Style: Show each file, explain what it does section by section. No code reading — just point and explain.**

---

## INTRO (30 sec)

> "This is the complete code walkthrough of CloudCanva. I'll go through every file — what it does, what it handles, and how files connect to each other. Starting with the backend."

---

## BACKEND

---

### main.py — The Core Server

**[Open: backend/main.py]**

> "This is the main file — the FastAPI server. Everything starts here. Let me go section by section."

**[Point to: imports section, lines 1-15]**

> "Top section — imports. We're importing FastAPI for the web framework, Boto3 for all AWS service calls, Jinja2 related classes from our custom modules, and standard libraries like json, os, shutil, tempfile for file handling."

**[Point to: CORS middleware, lines 18-25]**

> "CORS middleware — this allows our frontend (on a different domain) to make API calls to this backend. Without this, browsers would block cross-origin requests."

**[Point to: Configuration section, lines 28-32]**

> "Configuration — the AWS region we're operating in (eu-north-1), the S3 bucket name where we store exported Terraform zips, and the DynamoDB table name where we save project metadata."

**[Point to: REGION_CONFIG dictionary, lines 34-70]**

> "REGION_CONFIG — this is our multi-region support. A dictionary mapping 6 AWS regions to their correct Amazon Linux AMI, Ubuntu AMI, and availability zones. When a user selects a region, this config is used throughout the pipeline — in the AI prompt, in post-processing, and in template generation. This ensures you never get a us-east-1 AMI when deploying in eu-north-1."

**[Point to: AWS Clients, lines 72-75]**

> "AWS clients initialization — S3 client for file uploads/downloads, DynamoDB resource for project storage. These use the EC2 instance role for credentials — no keys hardcoded anywhere."

---

**[Point to: GET / endpoint]**

> "Health check — just returns a message confirming the server is running. Used by API Gateway health checks and quick testing."

**[Point to: GET /regions endpoint]**

> "Regions endpoint — returns all 6 supported regions with their AMIs and availability zones. The frontend uses this to populate the region selector dropdown."

---

**[Point to: POST /generate endpoint]**

> "This is the Terraform EXPORT endpoint. When user clicks Export on the canvas, this runs. What it does step by step:"
> "1. Extracts project name and selected region from the payload"
> "2. Gets the user email from the verified JWT token"
> "3. Calls generate_tf_code() — which renders all Jinja2 templates into main.tf and creates a ZIP"
> "4. Uploads the ZIP to S3 under the user's folder"
> "5. Saves metadata to DynamoDB — user email, project name, S3 key, timestamp, node count"
> "6. Returns the ZIP file directly to the user's browser for download"

---

**[Point to: GET /projects endpoint]**

> "Lists all saved projects for the logged-in user. Queries DynamoDB using the user's email as partition key. Returns projects sorted by newest first."

**[Point to: GET /projects/{name}/download endpoint]**

> "Download endpoint — looks up the project in DynamoDB, gets the S3 key, generates a presigned URL valid for 10 minutes, and returns it. The frontend opens this URL to trigger the download."

**[Point to: DELETE /projects/{name} endpoint]**

> "Delete endpoint — removes the project from both S3 (the actual ZIP file) and DynamoDB (the metadata). Both deletions are attempted — if one fails, the other still proceeds."

---

**[Point to: POST /validate endpoint]**

> "Architecture validation — receives the design, checks for common errors: missing AMIs on EC2, missing CIDR blocks on VPCs/Subnets, subnets not linked to VPCs, security groups not linked to VPCs. Also checks security warnings — SSH or DB ports open to the internet."

---

**[Point to: POST /ai/generate endpoint]**

> "The AI generation endpoint — the core of CloudCanva. Receives a natural language prompt and region. Gets the region config (correct AMI, AZs), calls generate_with_bedrock() to get architecture from Claude, then passes it through post_process_ai_response() to fix any issues. Returns the corrected architecture JSON to the frontend."

---

**[Point to: AWS_DEFAULT_LIMITS dictionary]**

> "Static quota limits — maps resource types to their AWS default limits. 5 VPCs, 5 EIPs, 20 EC2 instances, 200 subnets, 500 security groups, 5 internet gateways per region."

**[Point to: POST /check-quotas endpoint]**

> "Static quota check — counts how many of each resource the design creates, compares against these limits, and returns warnings. No AWS credentials needed — it just counts what's in the design."

**[Point to: POST /check-quotas-live endpoint]**

> "Live quota check — this one actually calls AWS. User provides their credentials (either a role ARN for AssumeRole or direct access keys). We use those to call describe_vpcs, describe_addresses, describe_instances — count what they already have running, add what the design will create, and check if they'll exceed limits. Returns results like 'VPCs: 3 existing + 2 new = 5/5 — WARNING'."

---

**[Point to: RESOURCE_IAM_ACTIONS dictionary]**

> "IAM action mappings — this maps 14 resource types to their exact IAM permissions. For example, VPC needs CreateVpc, DeleteVpc, DescribeVpcs, ModifyVpcAttribute, CreateTags. EC2 needs RunInstances, TerminateInstances, etc. This is how we generate minimum-privilege policies."

**[Point to: POST /generate-iam-policy endpoint]**

> "IAM policy generator — scans the user's design, looks up which resource types they're using, collects all required actions from the mapping above, and outputs a complete JSON IAM policy document. Users can copy-paste this into AWS Console to create the exact policy they need."

**[Point to: POST /validate-iam-permissions endpoint]**

> "IAM permission simulation — goes one step further. Takes the user's role ARN, determines what actions are needed for their design, then calls SimulatePrincipalPolicy to test each action against their role. Returns allowed/denied for each action, plus a 'can_deploy: true/false' summary."

---

**[Point to: post_process_ai_response function]**

> "The post-processing safety net — this is Layer 3 of our pipeline. It's NOT AI — it's pure deterministic code. Same input always gives same output."

> "What it does:"
> "First — finds the VPC node ID and all subnet IDs in the architecture (discovery pass)"
> "Then — loops through every node and applies fixes:"
> "- Subnets missing refs.vpc → linked to the VPC"
> "- EC2 missing refs.subnet → linked to first subnet"
> "- Security Groups missing refs.vpc → linked to VPC"
> "- Internet Gateways and Route Tables → same, linked to VPC"
> "- EC2 with wrong AMI (from different region) → replaced with correct one"
> "- Subnets with wrong AZ → replaced with correct region AZ"
> "- SSH port 22 open to 0.0.0.0/0 → restricted to VPC CIDR 10.0.0.0/16"
> "- DB ports (3306, 5432, 6379) open to internet → restricted to app subnet 10.0.1.0/24"
> "- Load Balancer missing subnets → gets first 2 subnet IDs"
> "- RDS missing subnets → gets 2 subnets for multi-AZ requirement"

---

**[Point to: generate_with_bedrock function]**

> "The Bedrock AI call. Constructs a system prompt with 18 rules telling Claude exactly how to format the response — JSON schema, region-specific values, security rules, positioning rules. Sends it to Bedrock, parses the JSON response, handles any markdown fencing the AI might add."

**[Point to: generate_tf_code function]**

> "Terraform code generation. Creates the TemplateRegistry, creates the TerraformOrchestrator, calls generate(). Takes the returned blocks, writes them to a temp directory as main.tf, zips it, returns the zip path."

**[Point to: validate_architecture function]**

> "Validation logic — checks each service for required fields. EC2 needs AMI and instance_type. VPC needs cidr_block. Subnet needs cidr_block and refs.vpc. Security group needs refs.vpc. Also checks security rules — warns about open sensitive ports."

**[Point to: mock_ai_generate function]**

> "Fallback mock generator — if Bedrock is unreachable, returns hardcoded architectures based on keywords in the prompt. For development and demo purposes only."

---

### auth.py — Authentication

**[Open: backend/auth.py]**

> "Cognito JWT verification middleware."

> "Top section — configuration. Cognito region, user pool ID, app client ID. These identify which Cognito User Pool we're verifying against. The JWKS URL is built from these — that's where Cognito publishes its public keys."

> "JWKS cache — stores the public keys for 1 hour. Avoids calling Cognito on every single request."

> "_get_jwks() — fetches public keys from Cognito's JWKS endpoint. If cache is fresh (under 1 hour), returns cached. Otherwise makes an HTTP call to get fresh keys."

> "_get_public_key() — takes a JWT token, reads its 'kid' (key ID) from the header, and finds the matching public key from JWKS. Each token tells you which key signed it."

> "verify_token() — the actual verification. Decodes the JWT using RS256 algorithm with the public key. Checks: is the signature valid? Is the issuer correct? Is it expired? Is token_use 'access'? If any check fails → 401 error."

> "get_current_user() — FastAPI dependency function. Extracts 'Bearer' token from Authorization header, calls verify_token, returns the decoded claims (username, email, etc). Every protected endpoint has Depends(get_current_user) — runs this before the endpoint logic."

---

### template_registry.py — Template Mapping

**[Open: backend/template_registry.py]**

> "This file maps service type names to their Jinja2 template file paths. 19 mappings total."

> "The _template_map dictionary — keys are like 'aws_vpc', 'aws_aws_instance', 'aws_subnet'. Values are file paths like 'aws/vpc.tf.j2', 'aws/ec2.tf.j2'."

> "Constructor — initializes the Jinja2 Environment with the templates folder. Uses FileSystemLoader to find templates, enables trim_blocks and lstrip_blocks for clean output."

> "get_template() — takes a template name, looks it up in the map, returns the loaded Jinja2 template object. If not found, raises an error."

> "This is what makes adding new resources easy — just add a new .tf.j2 file and one line to this map."

---

### generators/orcherastor.py — Terraform Orchestrator

**[Open: backend/generators/orcherastor.py]**

> "The orchestrator — drives the Terraform generation loop."

> "Constructor takes: config_data (the architecture), template_registry, and region."

> "generate() method:"
> "1. Loops through connected_services in the config"
> "2. For each infrastructure group — checks if any service uses AWS provider"
> "3. If AWS detected — renders the provider block first (with correct region)"
> "4. Then loops through each service: constructs template name as provider_type (e.g., 'aws_vpc'), creates a GenericTemplateGenerator, calls render()"
> "5. Appends each rendered Terraform block to the output list"
> "6. Returns a dict of infrastructure groups with their Terraform blocks"

> "Also supports Kubernetes — if provider is 'kubernetes', stores output separately as YAML instead of Terraform."

---

### generators/generic_template_generator.py — Template Renderer

**[Open: backend/generators/generic_template_generator.py]**

> "The simplest file in the project. Takes a config dict, a template registry, and a template name. The render() method gets the template from registry and calls template.render() with the config. Returns the rendered Terraform string."

> "One class, one method, one responsibility — render a single template with a single service's configuration."

---

### templates/aws/ — Jinja2 Terraform Templates

**[Open: backend/templates/aws/ folder, show the structure]**

> "19 template files organized by resource type."

> "Root level templates: vpc.tf.j2, subnet.tf.j2, ec2.tf.j2, rds.tf.j2, internet_gateway.tf.j2, nat_gateway.tf.j2, eks.tf.j2, provider.tf.j2"

> "Subfolders for grouped resources:"
> "- security_group/ — security_group.tf.j2, vpc_security_group_ingress_rule.tf.j2, vpc_security_group_egress_rule.tf.j2"
> "- routing/ — route_table.tf.j2, route_table_association.tf.j2"
> "- load_balancer/ — load_balancer.tf.j2, target_group.tf.j2"
> "- s3/ — bucket.tf.j2"
> "- ecs/ — ecs.tf.j2"
> "- eip/ — elastic_ip.tf.j2, eip_association.tf.j2"

> "Each template has Jinja2 placeholders like double curly braces for variables, if blocks for conditional attributes, and for loops for things like tags. The refs system renders cross-resource references — like subnet referencing VPC ID."

---

## FRONTEND

---

### app/ folder — Pages and Routing

**[Open: frontend/app/ folder]**

> "Next.js app router — each folder is a route."

**[Point to: app/layout.tsx]**

> "Root layout — wraps every page. Sets up the HTML structure, imports global CSS, applies metadata (title: CloudCanva, description). Uses ThemeProvider for dark mode support."

**[Point to: app/globals.css]**

> "Global styles — imports Tailwind CSS base, components, utilities. Has some custom CSS variables for the dark theme colors."

**[Point to: app/page.tsx]**

> "Home page — the main canvas view. This is what users see after login. It renders:"
> "- The top toolbar with AI Generate button, Templates, My Projects, region selector, user info"
> "- The Canvas component (React Flow)"
> "- Passes AI-generated nodes/edges to Canvas when AI generates"
> "- Has handleAiGenerated callback that receives nodes from AI dialog and passes to canvas"

**[Point to: app/signin/page.tsx]**

> "Login page — email and password form. Calls Amplify signIn(). If successful, redirects to home. Handles errors: wrong password, unverified email, additional MFA steps."

**[Point to: app/signup/page.tsx]**

> "Registration — email, password, confirm password. Calls Amplify signUp(). After signup, shows a verification code input — user enters the code from their email to confirm."

**[Point to: app/projects/page.tsx]**

> "My Projects page. On load, calls GET /projects to fetch user's saved architectures. Shows:"
> "- Stats bar: total projects, total resources, last export date"
> "- Search bar to filter projects by name"
> "- Project cards with: name, date, resource count, Download button, Delete button"
> "- Download triggers presigned URL from S3"
> "- Delete removes from both S3 and DynamoDB with confirmation dialog"

**[Point to: app/forgot-password/page.tsx]**

> "Password reset — enter email, receive code, enter code + new password. Uses Amplify's resetPassword and confirmResetPassword."

---

### components/ folder — UI Components

**[Open: frontend/components/ folder]**

**[Point to: Canvas.tsx]**

> "The core component — 1000+ lines. Manages the entire React Flow canvas. Key things it handles:"
> "- Node state (all AWS services on canvas)"
> "- Edge state (connections between services)"
> "- Drag and drop from sidebar (adding new services)"
> "- onConnect — creating edges when user draws a connection"
> "- Selected node editing — property panel on the right"
> "- Export dialog — name the project, send to backend, trigger download"
> "- Validate button — sends architecture to /validate endpoint"
> "- Clear canvas dialog"
> "- Region awareness — passes selected region to all operations"

**[Point to: AiArchitectureGenerator.tsx]**

> "AI dialog component. Has a pool of 16 example prompts — randomly picks 4 each time dialog opens. User types prompt, clicks Generate, it calls POST /ai/generate. Response is shown as a preview: service list and connections. User can Regenerate or Apply to Canvas. Apply pushes nodes/edges up to the parent."

**[Point to: IAMPolicyGenerator.tsx]**

> "IAM policy component. Two functions: Generate Policy (calls /generate-iam-policy, shows JSON) and Validate Role (calls /validate-iam-permissions with user's credentials, shows allowed/denied per action)."

**[Point to: ValidationPanel.tsx]**

> "Shows validation results — errors in red, warnings in yellow. Lists issues like 'Subnet not linked to VPC' or 'SSH port open to internet'."

**[Point to: TerraformPreview.tsx]**

> "Live Terraform preview — generates a rough preview of what the Terraform will look like based on current canvas nodes. Not the final output (that's from the backend templates), but gives users a quick idea."

**[Point to: AuthGuard.tsx]**

> "Route protection wrapper. Checks if user is authenticated via AuthContext. If not (and not loading), redirects to /signin. Wraps all protected pages."

**[Point to: base-node.tsx]**

> "Custom React Flow node component — defines how each AWS service card looks on the canvas. The visual box with rounded corners, selected state, etc."

**[Point to: getPropertiesByType.tsx]**

> "Returns the correct property fields for each service type. EC2 needs ami, instance_type. VPC needs cidr_block. Subnet needs cidr_block, availability_zone. Used when rendering the property editor panel."

**[Point to: dependencyCleaner.tsx]**

> "Cleanup utility — when a node or edge is deleted, this handles removing dangling references. If you delete a VPC, subnets that referenced it get their refs cleaned up."

---

### lib/ folder — Utilities

**[Open: frontend/lib/ folder]**

**[Point to: api.ts]**

> "Axios HTTP client. Creates an instance with baseURL from environment variable (points to API Gateway URL in production, localhost in dev). Has a request interceptor that fetches the current JWT token from Amplify and adds it as Authorization Bearer header to every outgoing request."

**[Point to: cognito.ts]**

> "Amplify configuration. Calls Amplify.configure() with Cognito User Pool ID, Client ID, and region from environment variables. This one file connects the frontend to Cognito — enables signIn, signUp, getCurrentUser throughout the app."

---

### context/ folder — State Management

**[Open: frontend/context/ folder]**

**[Point to: AuthContext.tsx]**

> "Authentication context provider. Wraps the entire app. On component mount, calls getCurrentUser() to check if user is already signed in (token still valid). Provides three values to all child components: user object, isAuthenticated boolean, and isLoading state. Components use useAuth() hook to access these."

---

### Other Frontend Files

**[Point to: .env.local]**

> "Environment variables — Cognito pool ID, client ID, region, and the API URL. In production this points to the API Gateway HTTPS URL. In development it points to localhost:8000."

**[Point to: package.json]**

> "Dependencies — Next.js 15, React 19, React Flow for the canvas, AWS Amplify for auth, Axios for HTTP, Tailwind for styling, shadcn/ui for components, dagre for auto-layout, zod for validation, lucide-react for icons."

**[Point to: tailwind.config]**

> "Tailwind configuration — custom colors, animations, plugins. Defines the dark theme palette used throughout the app."

---

## THE COMPLETE FLOW (2 minutes)

**[Show: either draw diagram or just speak over the file tree]**

> "Let me trace the complete flow one more time:"

> "User opens the app → Next.js serves the page from Amplify"

> "AuthGuard checks → AuthContext calls getCurrentUser() → Cognito validates session"

> "User types prompt in AiArchitectureGenerator → sends POST /ai/generate to API Gateway"

> "API Gateway forwards to EC2:8000 → auth.py verifies JWT → main.py handles request"

> "generate_with_bedrock() sends prompt + 18 rules to Bedrock Claude → gets JSON back"

> "post_process_ai_response() fixes refs, AMIs, AZs, ports → guaranteed valid"

> "JSON returned to frontend → preview shown → user clicks Apply → nodes on canvas"

> "User edits on canvas → clicks Export → Canvas.tsx calls generateConfig() → sends to POST /generate"

> "TerraformOrchestrator loops services → template_registry maps each to .tf.j2 file → generic_template_generator renders each → all blocks collected"

> "main.tf written → zipped → uploaded to S3 → metadata saved to DynamoDB → ZIP returned"

> "User downloads ZIP → runs terraform init + apply → AWS infrastructure is live"

> "That's the entire codebase of CloudCanva. Each file has one clear responsibility. The backend handles AI, rendering, auth, and storage. The frontend handles the visual experience. They connect through REST APIs secured by JWT tokens."

---

## RECORDING ORDER SUMMARY

1. Open backend/ folder → explain main.py (biggest file, most time)
2. Show auth.py → explain auth flow
3. Show template_registry.py → explain mapping
4. Show generators/ → explain orchestrator and renderer
5. Show templates/ folder → explain Jinja2 templates
6. Switch to frontend/ → explain app/ pages
7. Show components/ → explain each major component
8. Show lib/ and context/ → explain api client and auth state
9. End with the flow summary connecting everything
