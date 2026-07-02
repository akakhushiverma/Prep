# 🎯 Complete Interview Preparation Guide — Khushi Verma

> **B.Tech CSE | MNNIT Allahabad | CPI: 8.54 | AWS Support Engineering Intern @ Amazon**
> LeetCode Rating: 1759 | Codeforces Rating: 1149

---

## Table of Contents

1. [HR & Behavioral Questions](#section-1-hr--behavioral-questions)
2. [Amazon Leadership Principles](#section-2-amazon-leadership-principles)
3. [Deep Dive — CloudCanva](#section-3-deep-dive--cloudcanva)
4. [Deep Dive — TripSync](#section-4-deep-dive--tripsync)
5. [Deep Dive — IntelliQuery](#section-5-deep-dive--intelliquery)
6. [Data Structures & Algorithms](#section-6-data-structures--algorithms)
7. [Object Oriented Programming](#section-7-object-oriented-programming)
8. [Operating Systems](#section-8-operating-systems)
9. [Computer Networks](#section-9-computer-networks)
10. [Databases](#section-10-databases)
11. [System Design](#section-11-system-design)
12. [React & Frontend](#section-12-react--frontend)
13. [Node.js & Backend](#section-13-nodejs--backend)
14. [Python & FastAPI](#section-14-python--fastapi)
15. [Situational & Scenario Questions](#section-15-situational--scenario-questions)
16. [Questions to Ask the Interviewer](#section-16-questions-to-ask-the-interviewer)
17. [Coding Interview Strategy](#section-17-coding-interview-strategy)
18. [Self Introduction Scripts](#section-18-self-introduction-scripts)
19. [Weaknesses & Gap Areas](#section-19-weaknesses--gap-areas)
20. [Mock Interview Simulations](#section-20-mock-interview-simulations)

---

# SECTION 1: HR & BEHAVIORAL QUESTIONS

## 🔥 1. Tell Me About Yourself (2-Minute Structured Answer)

**Ideal Answer:**

"Hi, I'm Khushi Verma, a pre-final year B.Tech student in Computer Science at MNNIT Allahabad with a CPI of 8.54. I'm currently working as an AWS Support Engineering Intern at Amazon in Bangalore.

At Amazon, I built CloudCanva — an AI-powered platform for visual AWS infrastructure design. It lets users design cloud architectures either through drag-and-drop or by describing what they want in natural language, and the system generates secure, validated Terraform code with one click. I worked across the full stack — Next.js and React Flow on the frontend, FastAPI on the backend, and Amazon Bedrock for AI capabilities.

Before my internship, I built two significant projects. TripSync is a multi-agent travel planning platform where five AI agents run in parallel to plan complete trips — handling weather, flights, hotels, events, and routing simultaneously. IntelliQuery is an NL-to-database platform where users can query databases in plain English, powered by a LangGraph pipeline with security guardrails.

I'm passionate about building intelligent systems that simplify complex workflows. My LeetCode rating of 1759 reflects my problem-solving abilities, and being selected as an AFE-FFE Scholar among 40,000+ applicants motivates me to keep pushing my boundaries."

---

## 🔥 2. Why Amazon / Why This Role?

**Ideal Answer:**

"Three things drew me to Amazon. First, the Leadership Principles — especially Customer Obsession and Bias for Action — align deeply with how I approach building software. When I built CloudCanva, every design decision started with 'how does this make the developer's life easier?'

Second, the scale. Amazon operates at a level where engineering decisions impact millions of users. Working with services like Bedrock, S3, DynamoDB, and Cognito during my internship showed me how these building blocks enable innovation at scale.

Third, the learning culture. In just my internship, I went from understanding individual AWS services to building a platform that orchestrates them together with AI. The growth trajectory here is unmatched."

---

## 🔥 3. Why Should We Hire You?

**Ideal Answer:**

"Three reasons. First, I've demonstrated I can deliver end-to-end products independently. CloudCanva has 15+ API endpoints, 19 Terraform templates, AI integration, and security guardrails — all built during my internship.

Second, I learn fast and go deep. I picked up Amazon Bedrock, Terraform templating with Jinja2, and IAM permission modeling within weeks and built production-quality features with them.

Third, I think about the whole picture. I don't just write code — I think about security (IAM least privilege, quota validation), user experience (natural language workflows), and maintainability (modular architecture). My projects consistently go beyond the minimum requirement."

---

## ⭐ 4. What Are Your Strengths and Weaknesses?

**Strengths:**

"My biggest strength is the ability to connect different technologies into cohesive systems. In CloudCanva, I connected React Flow, Amazon Bedrock, FastAPI, Terraform, IAM, and quota APIs into a seamless workflow. I enjoy understanding how things fit together.

My second strength is persistence. When I was implementing deterministic post-processing for Bedrock's output, the AI sometimes generated invalid configurations. Instead of accepting this, I built an 18-rule validation system that ensures every output is structurally correct."

**Weaknesses (Honest + Growth-oriented):**

"I sometimes over-engineer initial solutions. With IntelliQuery, I built a 7-node LangGraph pipeline when the MVP could have started with 3-4 nodes. I've learned to ship iteratively — get the core working first, then add complexity when justified by user needs.

I'm also working on my system design skills at scale. I can design applications well, but thinking about million-user scenarios with distributed systems is an area I'm actively studying."

---

## ⭐ 5. Where Do You See Yourself in 5 Years?

**Ideal Answer:**

"In 5 years, I see myself as a senior engineer who owns critical systems end-to-end. I want to be the person who can take an ambiguous problem — like 'make infrastructure design accessible to non-experts' — and turn it into a working product that scales.

Specifically, I'm drawn to the intersection of AI and developer tools. My work on CloudCanva showed me how AI can fundamentally change how people interact with complex systems. I want to go deeper in this space — building AI-powered platforms that make software engineering more accessible and efficient."

---

## ⭐ 6. Tell Me About a Challenge You Faced and How You Solved It

**Ideal Answer (STAR Format):**

**Situation:** While building CloudCanva, Amazon Bedrock (Claude Sonnet 4) would sometimes generate AWS infrastructure configurations with incorrect resource references — for example, a security group referencing a VPC that didn't exist in the design.

**Task:** I needed to ensure 100% structural correctness in generated Terraform code, because even one invalid reference would cause deployment failures for users.

**Action:** I designed an 18-rule deterministic post-processing system. Instead of relying solely on the LLM to be correct, I built a validation layer that:
- Verified cross-resource references (every security group has a valid VPC)
- Checked naming conventions against AWS constraints
- Validated IAM policy structures
- Ensured quota limits weren't exceeded

I also implemented prompt engineering techniques to reduce errors at the generation stage itself.

**Result:** The system now produces structurally valid Terraform configurations consistently. Users can export and apply configurations without manual fixups, which was the core value proposition of the product.

---

## ⭐ 7. Tell Me About a Time You Worked in a Team

**Ideal Answer (STAR Format):**

**Situation:** During Smart India Hackathon 2025, our team of 6 had 36 hours to build a complete solution. We secured 4th position at the institute level.

**Task:** We needed to divide work efficiently across frontend, backend, ML, and presentation tracks while keeping the system integrated.

**Action:** I took ownership of the system architecture upfront. I defined the API contracts first so team members could work in parallel without blocking each other. I paired with teammates who were stuck — helping the ML team integrate their model with our backend, and helping frontend team consume the APIs correctly. I also set up a shared testing environment so everyone could validate their components together.

**Result:** We delivered a fully working prototype within 36 hours and secured 4th position. The key was clear API contracts early — nobody was blocked waiting for someone else's code.

---

## ⭐ 8. Tell Me About a Time You Failed

**Ideal Answer (STAR Format):**

**Situation:** In the early stages of IntelliQuery, I tried to build the NL-to-SQL pipeline as a single monolithic chain in LangChain. It handled simple queries but completely broke on complex multi-table joins and ambiguous queries.

**Task:** I needed a system that could handle diverse query complexities while being maintainable and debuggable.

**Action:** I stepped back and redesigned the entire pipeline using LangGraph instead. I split it into 7 specialized nodes — each handling a specific responsibility (intent classification, schema understanding, query generation, validation, etc.). This two-phase multi-agent approach meant each node could be tested and improved independently.

**Result:** The redesigned system handles complex queries reliably, and the modular architecture made it easy to add features like destructive query blocking and RBAC enforcement. The failure taught me that premature optimization of architecture is less important than getting the decomposition right.

---

## 9. Tell Me About Your Greatest Achievement

**Ideal Answer:**

"Being selected as an AFE-FFE Scholar among 40,000+ applicants is my proudest achievement — not just because of the numbers, but because it validated my journey. Coming from a UP Board school to MNNIT to Amazon was not a straightforward path.

But in terms of technical achievement, building CloudCanva during my Amazon internship stands out. Taking a complex problem — making AWS infrastructure design accessible through AI — and delivering a working platform with security guardrails, IAM generation, and Terraform export showed me I can handle ambiguity and deliver."

---

## 10. How Do You Handle Pressure and Deadlines?

**Ideal Answer:**

"I break problems down and prioritize ruthlessly. During my internship, I had to deliver CloudCanva with multiple complex features — AI integration, 19 Terraform templates, IAM simulation, quota checks — within a limited timeframe.

My approach: I identified the critical path first (AI → Terraform generation → export was the core value). I built that end-to-end first, then layered on security guardrails, quota validation, and additional templates. This way, I always had a working demo even while adding features.

For the TripSync hackathon project, we had 36 hours. I wrote the multi-agent orchestrator skeleton first so the team could develop individual agents in parallel without blocking."

---

## 11. Tell Me About a Time You Had a Conflict with a Teammate

**Ideal Answer (STAR Format):**

**Situation:** During IntelliQuery development, my teammate wanted to use a simple regex-based approach for detecting destructive queries (DROP, DELETE), while I advocated for AST-based parsing of the generated SQL.

**Task:** We needed a reliable destructive query blocker that wouldn't have false positives or false negatives.

**Action:** Instead of insisting on my approach, I proposed we test both. I wrote 50 test cases covering edge cases — queries with 'DROP' in string literals, subqueries with DELETE, and various obfuscation patterns. The regex approach failed on 12 of them. This data made the decision clear without it becoming a personal argument.

**Result:** We went with the AST-based approach, and it caught every destructive query in production testing. I learned that data resolves technical disagreements better than opinions.

---

## 12. What Motivates You?

**Ideal Answer:**

"Two things: solving problems that make complex things simpler, and continuous learning. CloudCanva was born from the insight that AWS infrastructure design shouldn't require memorizing service configurations — AI can handle that. Making that vision real motivated me through every debugging session.

I'm also motivated by measurable growth. Going from zero knowledge of LangGraph to building a 7-node multi-agent pipeline, or from basic AWS understanding to building a platform that generates infrastructure — that growth is addictive."

---

## 13. Why Did You Choose Computer Science?

**Ideal Answer:**

"I chose CS because I love building things that people use. In school, I started with competitive programming — the problem-solving aspect hooked me. But what truly drew me in was the realization that code can automate entire workflows. When I built my first web application and someone actually used it, that was the moment I knew this was my field.

Now, working at the intersection of AI and developer tools, I'm even more convinced. The problems in CS are infinite and evolving — that's exciting to me."

---

## 14. Tell Me About a Time You Learned Something Quickly

**Ideal Answer (STAR Format):**

**Situation:** At the start of my Amazon internship, I needed to integrate Amazon Bedrock into CloudCanva. I had never worked with Bedrock, Terraform templating with Jinja2, or IAM policy modeling before.

**Task:** I needed to become productive with these technologies within 2-3 weeks to meet project timelines.

**Action:** I took a structured approach: First, I read Bedrock documentation and made small test calls. Then I studied existing Terraform configurations to understand patterns. For IAM, I used AWS documentation and experimented in a sandbox account. I built small proof-of-concept components before integrating them into the main project. Within 2 weeks, I had a working prototype that generated Terraform from natural language.

**Result:** I went from zero Bedrock/Terraform knowledge to building 19 production-quality templates and an 18-rule prompt engineering system. The key was learning by building small things quickly rather than trying to understand everything theoretically first.

---

## 15. How Do You Prioritize Tasks?

**Ideal Answer:**

"I use a combination of impact and dependency analysis. In CloudCanva:

1. **Critical path first** — The AI → Terraform → Export flow was the core value, so I built that end-to-end first
2. **Unblock others** — I created API contracts early so frontend work could proceed in parallel
3. **High-impact, low-effort** — Security guardrails added massive value with relatively simple rule-based logic
4. **Nice-to-have last** — Architecture preview and advanced visualization came after core features were stable

I also maintain a daily priority list and revisit it if new information changes the landscape."

---

**📌 KEY POINTS TO REMEMBER — Section 1:**
- Always use STAR format for behavioral answers
- Connect every answer back to YOUR specific projects
- Show self-awareness in weakness answers
- Demonstrate growth mindset
- Keep answers to 1.5-2 minutes max
- Practice these out loud, not just reading

---

# SECTION 2: AMAZON LEADERSHIP PRINCIPLES

## LP 1: Customer Obsession 🔥

**Meaning:** Leaders start with the customer and work backwards. They work vigorously to earn and keep customer trust. Although leaders pay attention to competitors, they obsess over customers.

**Questions:**

**Q1: Tell me about a time you went above and beyond for a customer/user.**

**Answer (STAR):**

*Situation:* While building CloudCanva, I noticed that users who weren't AWS experts struggled with understanding which resources needed to be connected (e.g., a Lambda needs an IAM role, an EC2 needs a security group attached to a VPC).

*Task:* Make the platform usable by developers who aren't AWS infrastructure experts.

*Action:* I implemented cross-resource reference mapping — when a user adds an EC2 instance, the system automatically suggests required dependencies (VPC, subnet, security group) and shows visual connections on the React Flow canvas. I also added the natural language workflow so users could describe what they wanted in plain English instead of manually configuring each resource.

*Result:* The platform became accessible to developers who previously needed a dedicated DevOps engineer to set up infrastructure. The natural language path reduced design time from hours to minutes.

**Q2: Describe a time you made a decision based on customer needs rather than technical elegance.**

**Answer (STAR):**

*Situation:* In IntelliQuery, I initially designed the NL-to-database pipeline to auto-execute queries immediately for a smoother UX.

*Task:* Balance user experience with safety.

*Action:* After considering that users might accidentally express destructive intents in natural language ("remove all old records"), I added an interactive intent confirmation step. Technically, this added latency and complexity to the pipeline. But it prevented catastrophic mistakes.

*Result:* The confirmation step caught several potentially destructive queries during testing that would have wiped data. Users appreciated the safety net over raw speed.

**Q3: How do you determine what customers really need vs. what they ask for?**

**Answer:** "In CloudCanva, users asked for 'better Terraform generation.' But diving deeper, the real need was confidence — they wanted to know the generated code was correct and secure before applying it. That's why I built quota validation, IAM least-privilege checks, and architecture preview. The feature they asked for was better AI; the need they had was trust in the output."

---

## LP 2: Ownership 🔥

**Meaning:** Leaders are owners. They think long term and don't sacrifice long-term value for short-term results. They act on behalf of the entire company, beyond just their own team.

**Questions:**

**Q1: Tell me about a time you took ownership of something beyond your job description.**

**Answer (STAR):**

*Situation:* During my internship, CloudCanva's scope was initially just the AI-to-Terraform pipeline. But I noticed there was no way for users to validate whether their design would actually work within AWS account limits.

*Task:* The quota validation wasn't in my assigned scope, but without it, users would generate Terraform that would fail on `terraform apply`.

*Action:* I proactively built both static quota checks (comparing against known defaults) and live quota checks (querying actual account limits via AWS APIs). I also added IAM permission simulation so users could see if their current permissions allowed the resources they designed.

*Result:* This prevented users from generating and attempting to deploy invalid configurations. My manager was impressed that I identified and solved a problem nobody had explicitly assigned.

**Q2: Describe a time you saw a problem and fixed it without being asked.**

**Answer (STAR):**

*Situation:* In TripSync, the hotel agent was returning results without price normalization — some APIs returned per-night prices, others total-stay prices, and currencies varied.

*Task:* Users were comparing incomparable numbers, leading to bad booking decisions.

*Action:* Without being asked, I built a price normalization layer that converted all prices to per-night rates in the user's preferred currency. I also added the budget optimizer agent that factored in the normalized prices to give a clear budget breakdown.

*Result:* The budget breakdown became one of TripSync's most praised features in our demo. It started as me noticing a data inconsistency and fixing it proactively.

---

## LP 3: Invent and Simplify ⭐

**Meaning:** Leaders expect and require innovation and invention from their teams. They are externally aware, look for new ideas everywhere, and are not limited by "not invented here."

**Questions:**

**Q1: Tell me about a time you simplified a complex process.**

**Answer (STAR):**

*Situation:* AWS infrastructure design traditionally requires: learning service documentation → writing Terraform/CloudFormation manually → debugging errors → iterating. This takes hours for experienced engineers and is nearly impossible for beginners.

*Task:* Simplify this to minutes instead of hours.

*Action:* I invented a dual-workflow approach in CloudCanva: (1) Natural language — describe what you want, AI generates it, (2) Visual drag-and-drop for those who want manual control. Both paths lead to validated, exportable Terraform. The 18-rule prompt engineering system and deterministic post-processing ensure correctness without user effort.

*Result:* A process that took hours (learn → write → debug) became: describe → validate → export in minutes. The simplification wasn't dumbing down — it was automating the expertise.

**Q2: Tell me about an innovative solution you created.**

**Answer:** "IntelliQuery's two-phase multi-agent pipeline was innovative because it solved a problem others treat as a single-step translation (NL → SQL). By decomposing it into intent classification → schema understanding → query generation → validation → visualization, each phase could be optimized independently. The destructive query blocking emerged naturally from this architecture — it's just a validation node in the pipeline. Monolithic approaches can't add such safety features easily."

---

## LP 4: Are Right, A Lot ⭐

**Meaning:** Leaders have strong judgment and good instincts. They seek diverse perspectives and work to disconfirm their beliefs.

**Questions:**

**Q1: Tell me about a time you made a decision with incomplete information.**

**Answer (STAR):**

*Situation:* When designing IntelliQuery's architecture, I had to choose between a monolithic LangChain chain vs. LangGraph's graph-based approach. LangGraph was newer with less documentation and community support.

*Task:* Choose the right architecture without being able to fully evaluate LangGraph's production readiness.

*Action:* I built a small prototype with both approaches. The monolithic chain worked for simple queries but became unmaintainable when I added intent confirmation and destructive query blocking. LangGraph's node-based architecture made these additions natural. I made the decision based on extensibility needs, not just current requirements.

*Result:* The LangGraph choice paid off — adding RBAC enforcement, auto-visualization, and multi-database support later was straightforward because each was just a new node. If I'd chosen the monolithic approach, these additions would have required a rewrite.

**Q2: Tell me about a time you changed your mind based on new data.**

**Answer:** "Initially in TripSync, I used pure API calls for travel data. But after testing, I found that flight APIs had poor coverage for certain routes. I changed my approach to use SerpAPI for broader coverage, even though it meant less structured data. The data showed users needed coverage over structure."

---

## LP 5: Learn and Be Curious 🔥

**Meaning:** Leaders are never done learning and always seek to improve themselves. They are curious about new possibilities and act to explore them.

**Questions:**

**Q1: Tell me about a time you learned a new technology to solve a problem.**

**Answer (STAR):**

*Situation:* For my Amazon internship project, I needed to work with Amazon Bedrock, Terraform templating, React Flow, and IAM policy modeling — none of which I had prior experience with.

*Task:* Become productive with 4+ new technologies within weeks.

*Action:* I structured my learning: Week 1 — Bedrock API calls and prompt engineering basics. Week 2 — Terraform syntax and Jinja2 templating. Week 3 — React Flow for canvas rendering. I learned by building incrementally — each day produced a working mini-feature. I read official docs, studied existing implementations in the codebase, and asked senior engineers specific questions (not vague "how does this work" but "why does this component use this pattern").

*Result:* Within 3 weeks I had a working end-to-end prototype. By the end of the internship, I'd built 19 Terraform templates, an 18-rule prompt engineering system, and a full React Flow canvas. The key was learning by building, not learning then building.

**Q2: What's the most recent thing you taught yourself?**

**Answer:** "LangGraph — I taught myself graph-based AI orchestration to build IntelliQuery. What fascinated me was how it models AI workflows as directed graphs with state management. I went from reading the docs to building a 7-node production pipeline with conditional edges, state persistence, and error recovery. It fundamentally changed how I think about AI system design."

---

## LP 6: Hire and Develop the Best

**Meaning:** Leaders raise the performance bar with every hire and promotion. They recognize exceptional talent and move them throughout the organization.

**Questions:**

**Q1: Tell me about a time you helped someone else succeed.**

**Answer (STAR):**

*Situation:* During the SIH hackathon, two teammates were new to React and struggled with state management for our frontend.

*Task:* Help them become productive quickly without taking over their work.

*Action:* I spent 2 hours pair-programming with them, explaining useState/useEffect patterns through our actual codebase rather than abstract tutorials. I created a component template they could follow and reviewed their code with explanations rather than just fixes.

*Result:* They delivered their assigned components on time and later told me they used the same patterns in their own projects. Teaching solidified my own understanding too.

---

## LP 7: Insist on the Highest Standards 🔥

**Meaning:** Leaders have relentlessly high standards. They continually raise the bar and drive their teams to deliver high-quality products.

**Questions:**

**Q1: Tell me about a time you refused to compromise on quality.**

**Answer (STAR):**

*Situation:* In CloudCanva, the initial Terraform generation from Bedrock was about 85% accurate — most configurations were correct but some had invalid resource references or missing required fields.

*Task:* 85% sounds high, but in infrastructure-as-code, even one error means deployment failure. I needed 100% structural correctness.

*Action:* Instead of shipping with a disclaimer ("please review generated code"), I built the 18-rule deterministic post-processing layer. Every generated configuration passes through validation that checks cross-resource references, required fields, naming conventions, and IAM policy structure. If validation fails, the system regenerates rather than showing broken output.

*Result:* Users can trust the exported Terraform and apply it directly. The extra week spent on validation saved users from debugging cryptic Terraform errors — which would have undermined trust in the entire platform.

**Q2: Describe a time your high standards caused friction.**

**Answer:** "In IntelliQuery, I insisted on implementing RBAC even for the MVP demo. My teammate felt it was overkill for a prototype. But I argued that an NL-to-database tool without access control is a security liability from day one. We spent the extra time, and during our demo, the first question was 'how do you handle permissions?' — we had a complete answer."

---

## LP 8: Think Big

**Meaning:** Thinking small is a self-fulfilling prophecy. Leaders create and communicate a bold direction that inspires results.

**Questions:**

**Q1: Tell me about a time you proposed something ambitious.**

**Answer (STAR):**

*Situation:* When I started my Amazon internship, the original scope was a simpler Terraform generation tool — take a form input, generate a template.

*Task:* I saw an opportunity for something much more impactful.

*Action:* I proposed CloudCanva — a full visual design platform with AI-powered natural language workflows, not just a form-to-template converter. I presented a vision where any developer, regardless of AWS expertise, could design secure infrastructure visually. This required adding React Flow canvas, Bedrock integration, security guardrails, and quota validation — significantly more than the original scope.

*Result:* The expanded vision was approved. The final product is a platform that reimagines infrastructure design, not just a template generator. Thinking big meant building something genuinely novel rather than incremental.

---

## LP 9: Bias for Action 🔥

**Meaning:** Speed matters in business. Many decisions are reversible and do not need extensive study. We value calculated risk-taking.

**Questions:**

**Q1: Tell me about a time you took action without waiting for perfect information.**

**Answer (STAR):**

*Situation:* During TripSync development, I needed to decide between building a custom flight data scraper vs. using SerpAPI for travel data. Building custom scrapers would give more structured data but would take 2+ weeks.

*Task:* Deliver a working demo within the project timeline.

*Action:* I chose SerpAPI immediately and started building. The data was less structured, but I could have results within days instead of weeks. I built a normalization layer on top to handle the less structured data. If SerpAPI proved insufficient later, I could replace it without changing the rest of the architecture.

*Result:* We had a working flight search within 3 days. The normalization approach worked well enough that we never needed the custom scraper. The bias for action saved two weeks.

**Q2: Describe a calculated risk you took.**

**Answer:** "Choosing LangGraph over LangChain for IntelliQuery was a calculated risk — LangGraph was newer and less documented. But I mitigated it by building a small prototype first (2 days). When the prototype showed it could handle our use case cleanly, I committed. If it had failed, I'd have lost 2 days, not 2 months."

---

## LP 10: Frugality

**Meaning:** Accomplish more with less. Constraints breed resourcefulness, self-sufficiency, and invention.

**Questions:**

**Q1: Tell me about a time you accomplished something with limited resources.**

**Answer (STAR):**

*Situation:* For TripSync, I needed real-time travel data from multiple sources (flights, hotels, weather, routes) but had no budget for premium APIs.

*Task:* Build a comprehensive travel planning platform using only free-tier APIs.

*Action:* I used free tiers strategically: OpenWeatherMap free tier for weather, OpenRouteService free tier for routing, SerpAPI limited free calls for flights/hotels, and Google Calendar API (free with OAuth2). I designed the multi-agent orchestrator to run agents in parallel to minimize wall-clock time, and implemented caching to reduce API calls.

*Result:* A fully functional travel planning platform built entirely on free-tier APIs. The constraint forced creative solutions — like the parallel agent architecture which actually improved UX compared to sequential calls.

---

## LP 11: Earn Trust 🔥

**Meaning:** Leaders listen attentively, speak candidly, and treat others respectfully. They self-critically evaluate themselves.

**Questions:**

**Q1: Tell me about a time you had to earn someone's trust.**

**Answer (STAR):**

*Situation:* When I proposed expanding CloudCanva's scope to include AI-powered natural language workflows, my team needed confidence that I could deliver something this ambitious as an intern.

*Task:* Demonstrate capability through action, not just words.

*Action:* I built a working prototype within the first week — a minimal version showing natural language → Terraform generation. Instead of presenting slides, I demoed working code. I was also transparent about limitations: "The AI output needs post-processing; here's my plan for the 18-rule validation system." I shared daily progress and flagged blockers early.

*Result:* The prototype earned trust quickly. My manager gave me full autonomy to execute the expanded scope. Transparency about both progress and problems was key.

**Q2: Tell me about a time you admitted a mistake.**

**Answer:** "In IntelliQuery's early version, I didn't implement rate limiting on the NL-to-database pipeline. During testing, a rapid-fire of queries caused the Groq API to throttle. I immediately documented the issue, explained the impact, and implemented exponential backoff with user-facing feedback. Being transparent about the oversight built more trust than hiding it would have."

---

## LP 12: Dive Deep

**Meaning:** Leaders operate at all levels, stay connected to the details, audit frequently, and are skeptical when metrics and anecdotes differ.

**Questions:**

**Q1: Tell me about a time you dug deep into a problem.**

**Answer (STAR):**

*Situation:* In CloudCanva, users reported that some generated Terraform configurations would fail silently — `terraform plan` showed no errors but `terraform apply` would fail with quota exceeded errors.

*Task:* Understand why static validation passed but runtime failed.

*Action:* I dug into the root cause: AWS accounts have service quotas that are account-specific and can be modified. My static checks were comparing against default limits, but users might have lower or higher custom limits. I implemented live quota checks — actually querying the AWS Service Quotas API to get the user's real limits, not just defaults.

*Result:* The system now catches both default-exceeded AND custom-limit-exceeded scenarios. The deep dive revealed that the problem wasn't in our validation logic but in our assumption that default limits were universal.

---

## LP 13: Have Backbone; Disagree and Commit

**Meaning:** Leaders respectfully challenge decisions when they disagree. Once a decision is determined, they commit wholly.

**Questions:**

**Q1: Tell me about a time you disagreed with a decision but committed to it.**

**Answer (STAR):**

*Situation:* For IntelliQuery, I advocated for supporting only PostgreSQL initially and adding MongoDB/MySQL later. My co-developer wanted all three from day one.

*Task:* Resolve the disagreement and move forward.

*Action:* I presented my argument: supporting one database first would let us perfect the NL-to-SQL pipeline before handling MongoDB's fundamentally different query language. They argued that the multi-database support was our key differentiator. After discussion, we went with their approach. I committed fully — I designed the abstraction layer that made multi-database support clean instead of hacky.

*Result:* The multi-database support did become our strongest demo point. I was glad we went that route. My commitment to making it work well (rather than half-heartedly) made the difference.

---

## LP 14: Deliver Results 🔥

**Meaning:** Leaders focus on the key inputs for their business and deliver them with the right quality and in a timely fashion.

**Questions:**

**Q1: Tell me about your most significant delivery.**

**Answer (STAR):**

*Situation:* CloudCanva needed to be a complete, demo-ready platform by the end of my internship — not just a prototype but a working tool with AI integration, security guardrails, and exportable output.

*Task:* Deliver 15+ API endpoints, 19 Terraform templates, AI integration, IAM simulation, quota validation, React Flow canvas, and ZIP export within the internship timeframe.

*Action:* I broke the project into weekly milestones: Week 1-2: Core AI pipeline. Week 3-4: Terraform templates and Jinja2 integration. Week 5-6: Security guardrails and IAM. Week 7-8: Frontend polish and integration. I tracked blockers daily and re-prioritized when Bedrock had unexpected behaviors. When quota checking took longer than planned, I deprioritized less critical templates rather than slipping the overall deadline.

*Result:* Delivered the complete platform on schedule with all planned features functional. The key was constant reprioritization — never letting any single feature delay the overall delivery.

---

## LP 15: Strive to be Earth's Best Employer

**Meaning:** Leaders work to create a safer, more productive, and more diverse environment.

**Q1: How do you contribute to a positive team environment?**

**Answer:** "I believe in knowledge sharing. During my internship, I documented my learnings about Bedrock prompt engineering and Terraform templating patterns so future interns wouldn't start from zero. In the SIH hackathon, I pair-programmed with less experienced teammates rather than just assigning them tasks. Everyone grows more when knowledge flows freely."

---

## LP 16: Success and Scale Bring Broad Responsibility

**Meaning:** Leaders create more than they consume and always leave things better than how they found them.

**Q1: How do you think about the broader impact of your work?**

**Answer:** "In CloudCanva, I implemented security guardrails not because it was required, but because I recognized that making infrastructure generation easy without making it safe could lead to security vulnerabilities at scale. If thousands of developers use this tool, even a small percentage generating insecure configurations would be harmful. IAM least-privilege generation and quota validation are about responsible scaling."

---

**📌 KEY POINTS TO REMEMBER — Section 2:**
- Amazon values SPECIFIC examples, not generic answers
- Always use STAR format with measurable results
- Connect each answer to a DIFFERENT project/experience to show breadth
- "Customer" can be end-users, teammates, or stakeholders
- Show how you EMBODY the principle, not just understand it
- Prepare 2-3 stories per LP, have backup examples ready
- Interviewers will ask follow-up questions — have details ready

---

# SECTION 3: DEEP DIVE — CLOUDCANVA

## A) Architecture Questions

### 🔥 Q: Explain CloudCanva architecture end to end.

**Answer:**

CloudCanva has a three-layer architecture:

**1. Frontend Layer (Next.js + TypeScript + React Flow + Tailwind CSS)**
- React Flow provides the infinite canvas for drag-and-drop infrastructure design
- Custom nodes represent AWS resources (EC2, VPC, Lambda, S3, etc.)
- Edges represent connections/dependencies between resources
- State management handles cross-resource reference mapping
- Two modes: Manual (drag-drop) and AI (natural language input)
- Architecture preview panel shows live topology
- ZIP export for downloading all generated files

**2. Backend Layer (FastAPI + Python + Jinja2)**
- 15+ REST API endpoints handling:
  - Resource CRUD operations
  - AI generation requests (proxied to Bedrock)
  - Terraform template rendering via Jinja2
  - IAM policy simulation
  - Quota validation (static + live)
  - Project persistence (save/load)
  - Export and download
- Deterministic post-processing engine (18 rules)
- Cross-resource reference resolver

**3. AI/Cloud Layer**
- Amazon Bedrock (Claude Sonnet 4) for NL → infrastructure generation
- Amazon Cognito for authentication
- AWS Amplify for frontend hosting
- S3 for project file storage and export downloads
- DynamoDB for project metadata and user data
- EC2 for backend hosting

**Data Flow:**
```
User Input (NL or drag-drop) → Frontend State → API Call →
FastAPI processes request → Bedrock generates (if AI mode) →
18-rule post-processing validates → Jinja2 renders Terraform →
Response with validated config → Frontend updates canvas →
Export: Jinja2 renders all templates → ZIP → S3 presigned URL → Download
```

---

### ⭐ Q: Why did you choose Next.js + FastAPI combination?

**Answer:**

**Next.js (Frontend):**
- Server-side rendering for fast initial load
- Built-in routing and API routes for simple proxying
- TypeScript support for type safety across the complex React Flow state
- Excellent developer experience with hot reload
- Amplify deployment support out of the box

**FastAPI (Backend):**
- Native async support — critical when making Bedrock API calls that can take 5-10 seconds
- Automatic OpenAPI documentation for all 15+ endpoints
- Pydantic models for request/response validation
- Python ecosystem for Jinja2 templating and AWS SDK (boto3)
- 40% faster than Flask for I/O-bound workloads (Bedrock calls)

**Why not a single Node.js stack?**
- Jinja2 is the industry standard for Terraform templating — Python-native
- boto3 (AWS SDK for Python) is more mature than the JS SDK for IAM simulation
- FastAPI's async handling is cleaner for long-running Bedrock calls
- Separation of concerns: Frontend team can work independently from backend

---

### ⭐ Q: How does the drag-and-drop canvas work technically?

**Answer:**

React Flow is the foundation. Here's how it works:

**Nodes:** Each AWS resource (EC2, VPC, S3 bucket, Lambda, etc.) is a custom React Flow node with:
- A custom component rendering the resource icon, name, and key properties
- A data object containing all configuration (instance type, CIDR blocks, etc.)
- Connection handles (source/target ports) for establishing relationships

**Edges:** Represent dependencies between resources:
- VPC → Subnet (subnet belongs to VPC)
- Security Group → EC2 (SG attached to instance)
- Lambda → IAM Role (execution role)

**State Management:**
```typescript
// Simplified state structure
{
  nodes: [
    { id: 'vpc-1', type: 'vpc', data: { cidr: '10.0.0.0/16', ... } },
    { id: 'ec2-1', type: 'ec2', data: { instanceType: 't3.micro', vpcId: 'vpc-1', ... } }
  ],
  edges: [
    { source: 'vpc-1', target: 'ec2-1', type: 'dependency' }
  ]
}
```

**Cross-Resource Reference Mapping:**
When user adds EC2 → system automatically:
1. Checks if VPC exists → if not, suggests creating one
2. Validates security group assignments
3. Updates all references when resources are renamed/deleted

---

### Q: How does React Flow manage state for complex diagrams?

**Answer:**

React Flow uses a controlled component pattern:

1. **Centralized State:** All nodes and edges are stored in React state (or a state management solution)
2. **Event Handlers:** `onNodesChange`, `onEdgesChange`, `onConnect` handle user interactions
3. **Viewport:** Independent from node state — handles zoom/pan
4. **Performance:** React Flow uses `memo` and position-based rendering (only renders nodes in viewport)
5. **Custom Logic:** I added middleware that intercepts state changes to enforce business rules:
   - Can't delete a VPC if subnets still reference it
   - Auto-layout when new connected resources are added
   - Validation highlights (red borders on misconfigured nodes)

---

### Q: How does ZIP export work?

**Answer:**

```
1. User clicks "Export" →
2. Frontend sends current canvas state (all nodes + edges + configs) to backend →
3. Backend processes each resource:
   a. Resolves cross-references (ec2.vpc_id → actual VPC resource name)
   b. Selects appropriate Jinja2 template (from 19 templates)
   c. Renders template with resource-specific variables
4. Generates supporting files:
   - main.tf (all resources)
   - variables.tf (input variables)
   - outputs.tf (output values)
   - provider.tf (AWS provider config)
   - terraform.tfvars (default values)
5. Bundles into ZIP in memory →
6. Uploads to S3 with time-limited presigned URL →
7. Returns presigned URL to frontend →
8. Browser downloads ZIP
```

---

## B) Amazon Bedrock / AI Questions

### 🔥 Q: What is Amazon Bedrock? How did you use it?

**Answer:**

Amazon Bedrock is a fully managed AWS service that provides access to foundation models (FMs) from leading AI companies (Anthropic, Meta, Cohere, etc.) through a unified API. Key features:
- No infrastructure management — serverless
- Pay-per-token pricing
- Built-in safety/content filtering
- Model choice without code changes

**How I used it in CloudCanva:**
- Model: Claude Sonnet 4 (Anthropic via Bedrock)
- Purpose: Converting natural language descriptions into structured AWS infrastructure configurations
- Input: "I need a web application with a load balancer, two EC2 instances, and an RDS database"
- Output: Structured JSON defining all resources with their configurations and interconnections

**Integration pattern:**
```python
import boto3

bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

response = bedrock.invoke_model(
    modelId='anthropic.claude-sonnet-4-20250514-v1:0',
    body=json.dumps({
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 4096,
        "system": system_prompt,  # 18-rule prompt
        "messages": [{"role": "user", "content": user_input}]
    })
)
```

---

### 🔥 Q: Explain your 18-rule prompt engineering system.

**Answer:**

The 18 rules fall into 4 categories:

**Category 1: Output Structure Rules (Rules 1-5)**
1. Always output valid JSON — no markdown, no explanations
2. Use consistent key naming (`resource_type`, `resource_name`, `properties`)
3. Include ALL required fields for each AWS resource type
4. Group related resources together
5. Include explicit dependency declarations

**Category 2: AWS Best Practices Rules (Rules 6-10)**
6. Always use private subnets for databases
7. Apply least-privilege IAM policies
8. Include security groups with minimal required ports
9. Use multi-AZ for production workloads
10. Tag all resources with Name and Environment

**Category 3: Reference Integrity Rules (Rules 11-14)**
11. Every subnet must reference an existing VPC
12. Every security group must reference an existing VPC
13. Every EC2 instance must have a subnet and security group
14. IAM roles must be referenced by their logical name, not ARN

**Category 4: Safety & Validation Rules (Rules 15-18)**
15. Never generate resources that exceed known quota defaults
16. Never output wildcard (*) IAM permissions
17. Always include a description field for documentation
18. Flag ambiguous user requests for clarification instead of guessing

---

### ⭐ Q: What is deterministic post-processing? Why is it needed?

**Answer:**

**What:** A rule-based validation and correction layer that processes LLM output AFTER generation but BEFORE showing it to users.

**Why it's needed:**
LLMs are probabilistic — even with perfect prompts, they can:
- Generate invalid JSON (missing commas, extra brackets)
- Create circular dependencies
- Reference resources that don't exist in the design
- Use deprecated or invalid resource property names
- Exceed AWS naming constraints (64 char limit, valid characters)

**How it works:**
```python
def post_process(llm_output):
    # Phase 1: Structural validation
    parsed = validate_json(llm_output)  # Fix JSON if possible
    
    # Phase 2: Reference integrity
    for resource in parsed['resources']:
        validate_references(resource, parsed['resources'])
    
    # Phase 3: AWS constraint validation
    for resource in parsed['resources']:
        validate_naming(resource)  # Length, characters
        validate_required_fields(resource)  # No missing fields
        validate_property_values(resource)  # Valid instance types, CIDR formats
    
    # Phase 4: Security validation
    validate_iam_policies(parsed)  # No wildcard permissions
    validate_security_groups(parsed)  # No 0.0.0.0/0 on SSH
    
    return corrected_output or trigger_regeneration
```

**Key insight:** Deterministic means the same invalid input always gets the same correction — no randomness. This makes the system predictable and debuggable, unlike relying on the LLM to self-correct.

---

### Q: How do you handle AI hallucinations in infrastructure generation?

**Answer:**

Multiple layers of defense:

1. **Constrained output format:** The prompt demands JSON with a specific schema — reduces room for hallucination
2. **Enumerated valid values:** The prompt lists valid instance types, regions, etc. — LLM picks from a list rather than inventing
3. **Post-processing validation:** Every generated resource is checked against known AWS resource schemas
4. **Cross-reference verification:** If EC2 references `vpc-1`, we verify `vpc-1` exists in the output
5. **Regeneration on failure:** If post-processing finds unfixable errors, we re-invoke Bedrock with a corrective prompt rather than showing broken output
6. **User confirmation:** For ambiguous requests, the system asks for clarification instead of guessing

---

## C) Terraform Questions

### 🔥 Q: What is Terraform? How does it work?

**Answer:**

Terraform is an Infrastructure as Code (IaC) tool by HashiCorp that lets you define cloud infrastructure in declarative configuration files.

**Core Workflow:**
```
Write (.tf files) → Plan (preview changes) → Apply (create resources) → Destroy (tear down)
```

**Key Concepts:**
- **Providers:** Plugins that interact with cloud APIs (AWS, Azure, GCP)
- **Resources:** The infrastructure components (EC2 instance, S3 bucket)
- **State:** Terraform tracks what it created in a state file (terraform.tfstate)
- **Modules:** Reusable groupings of resources
- **Variables:** Parameterize configurations
- **Outputs:** Expose values after creation

**How Terraform works internally:**
1. Parses `.tf` files into a resource graph
2. Determines dependency order (a subnet depends on its VPC)
3. Creates a plan (diff between desired state and current state)
4. Executes API calls in dependency order
5. Updates state file with created resource IDs

**Example:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.main.id
  
  tags = {
    Name = "web-server"
  }
}
```

---

### ⭐ Q: What are Terraform templates? How did you create 19?

**Answer:**

In CloudCanva, "Terraform templates" are Jinja2 template files that generate valid Terraform HCL code when filled with resource-specific variables.

**Why 19 templates?** One for each supported AWS resource type:
1. VPC
2. Subnet (public/private)
3. Internet Gateway
4. NAT Gateway
5. Route Table
6. Security Group
7. EC2 Instance
8. EBS Volume
9. Elastic Load Balancer (ALB/NLB)
10. Auto Scaling Group
11. Launch Template
12. Lambda Function
13. IAM Role
14. IAM Policy
15. S3 Bucket
16. DynamoDB Table
17. RDS Instance
18. Route 53 Record
19. CloudFront Distribution

**Template example (Jinja2 → Terraform):**
```jinja2
resource "aws_instance" "{{ resource_name }}" {
  ami           = "{{ ami_id }}"
  instance_type = "{{ instance_type }}"
  subnet_id     = aws_subnet.{{ subnet_ref }}.id
  vpc_security_group_ids = [
    {% for sg in security_groups %}
    aws_security_group.{{ sg }}.id,
    {% endfor %}
  ]
  
  tags = {
    Name        = "{{ resource_name }}"
    Environment = "{{ environment }}"
  }
}
```

---

### Q: Explain Terraform plan vs apply vs destroy.

**Answer:**

| Command | What it does | Side effects |
|---------|-------------|--------------|
| `terraform plan` | Shows what WILL change (create/modify/delete) | None — read-only |
| `terraform apply` | Actually creates/modifies cloud resources | Creates real infrastructure, costs money |
| `terraform destroy` | Tears down ALL managed resources | Deletes everything in state |

**Plan output example:**
```
+ aws_instance.web (create)
    ami: "ami-0c55b159"
    instance_type: "t3.micro"
    
~ aws_security_group.web (modify)
    ingress.0.from_port: 80 → 443
    
- aws_s3_bucket.old_logs (destroy)
```

`+` = create, `~` = modify, `-` = destroy

---

## D) IAM Questions

### 🔥 Q: What is AWS IAM? Explain least privilege principle.

**Answer:**

**IAM (Identity and Access Management)** controls WHO can do WHAT on WHICH resources in AWS.

**Components:**
- **Users:** Individual identities (humans or applications)
- **Groups:** Collections of users with shared permissions
- **Roles:** Temporary permissions assumable by users/services
- **Policies:** JSON documents defining permissions (allow/deny + actions + resources)

**Least Privilege Principle:**
Give the minimum permissions necessary to perform a task — nothing more.

**Bad (overly permissive):**
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

**Good (least privilege):**
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

**How I implemented in CloudCanva:**
- When generating IAM policies, the system analyzes what each resource actually needs
- EC2 with S3 access → only s3:GetObject/PutObject on the specific bucket
- Lambda → only the specific DynamoDB table actions it uses
- Never generates wildcard (*) actions — Rule 16 of my prompt engineering system

---

### Q: How did you implement IAM policy generation?

**Answer:**

The IAM generation system works in 3 steps:

**Step 1: Dependency Analysis**
Analyze the canvas to determine what each resource needs to access:
```
EC2 instance → needs to read from S3 bucket "uploads"
Lambda function → needs to write to DynamoDB table "orders"
```

**Step 2: Policy Generation**
For each resource, generate the minimum policy:
```python
def generate_iam_policy(resource, dependencies):
    statements = []
    for dep in dependencies:
        actions = get_minimum_actions(resource.type, dep.type, dep.access_pattern)
        statements.append({
            "Effect": "Allow",
            "Action": actions,
            "Resource": dep.arn_pattern
        })
    return {"Version": "2012-10-17", "Statement": statements}
```

**Step 3: IAM Permission Simulation**
Uses AWS IAM Policy Simulator logic to verify:
- Does the generated policy actually grant the needed access?
- Does it grant anything BEYOND what's needed?
- Are there conflicting deny statements?

---

## E) AWS Services Questions

### 🔥 Q: Explain S3 in depth.

**Answer:**

**Amazon S3 (Simple Storage Service)** — Object storage with virtually unlimited scalability.

**Key Concepts:**
- **Bucket:** Container for objects (globally unique name)
- **Object:** File + metadata (key-value pairs), up to 5TB per object
- **Key:** The full path/name of an object within a bucket
- **Versioning:** Keep multiple versions of the same object
- **Storage Classes:** Standard, IA (Infrequent Access), Glacier (archival)

**How I used S3 in CloudCanva:**
1. **Project persistence:** Saving canvas state as JSON objects
2. **Export storage:** Generated ZIP files stored temporarily
3. **Presigned URLs:** Time-limited download links for exported Terraform bundles
4. **Static assets:** Terraform provider configurations and template libraries

**Presigned URL pattern:**
```python
url = s3_client.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'cloudcanva-exports', 'Key': f'{user_id}/{project_id}.zip'},
    ExpiresIn=3600  # 1 hour
)
```

---

### 🔥 Q: Explain DynamoDB in depth.

**Answer:**

**Amazon DynamoDB** — Fully managed NoSQL key-value and document database.

**Key Concepts:**
- **Table:** Collection of items
- **Item:** A single record (like a row)
- **Partition Key (PK):** Primary key for distributing data across partitions
- **Sort Key (SK):** Optional secondary key for range queries within a partition
- **GSI (Global Secondary Index):** Alternative query pattern on different attributes

**How I used DynamoDB in CloudCanva:**

Table: `cloudcanva-projects`
```
PK: USER#{userId}
SK: PROJECT#{projectId}
Attributes: projectName, canvasState, lastModified, resourceCount
```

**Why DynamoDB over RDS:**
- Serverless (no instances to manage)
- Single-digit millisecond latency at any scale
- Pay-per-request pricing (perfect for variable intern project load)
- Schema flexibility — canvas state evolves frequently

**Access patterns:**
- Get all projects for a user: `PK = USER#123`
- Get specific project: `PK = USER#123, SK = PROJECT#456`
- Recent projects GSI: Sort by `lastModified`

---

### Q: What is Amazon Cognito? How did you implement auth?

**Answer:**

**Cognito** provides authentication, authorization, and user management for web/mobile apps.

**Two components:**
1. **User Pools:** User directory with sign-up/sign-in (handles passwords, MFA, email verification)
2. **Identity Pools:** Provides temporary AWS credentials to access AWS services

**CloudCanva auth flow:**
```
1. User signs up/in via Cognito User Pool (email + password)
2. Cognito returns JWT tokens (ID token, Access token, Refresh token)
3. Frontend stores tokens and sends Access token with every API call
4. FastAPI middleware validates JWT signature and expiry
5. For AWS resource access (S3, DynamoDB), Cognito Identity Pool
   exchanges the User Pool token for temporary AWS credentials
6. These scoped credentials only allow access to that user's data
```

**Why Cognito over custom auth:**
- Handles password hashing, token rotation, MFA
- Integrates natively with other AWS services
- Scales without infrastructure management
- Compliant with security standards

---

## F) Quota & Security Questions

### ⭐ Q: Difference between static and live quota checks?

**Answer:**

| Aspect | Static Checks | Live Checks |
|--------|--------------|-------------|
| **Data source** | Hardcoded default limits | AWS Service Quotas API |
| **Speed** | Instant (no API call) | ~500ms (API call) |
| **Accuracy** | Good for defaults | Exact for the user's account |
| **When used** | During canvas design (real-time) | Before export (final validation) |
| **Example** | "Default VPC limit is 5 per region" | "Your account has 3/5 VPCs used" |

**Why both?**
- Static: Fast feedback during design — "you've added 6 VPCs, default limit is 5"
- Live: Accurate final check — "your account actually has a limit of 10 VPCs (increased), and 7 are already used"

**Implementation:**
```python
# Static check (instant)
def static_check(resource_type, count):
    defaults = {"vpc": 5, "ec2": 20, "ebs": 5000, "lambda": 1000}
    if count > defaults.get(resource_type, float('inf')):
        return Warning(f"Exceeds default {resource_type} limit")

# Live check (API call)
def live_check(resource_type, count, region):
    client = boto3.client('service-quotas', region_name=region)
    quota = client.get_service_quota(
        ServiceCode=service_codes[resource_type],
        QuotaCode=quota_codes[resource_type]
    )
    current_usage = get_current_usage(resource_type, region)
    if current_usage + count > quota['Quota']['Value']:
        return Error(f"Would exceed account limit: {current_usage + count}/{quota['Quota']['Value']}")
```

---

### Q: How did you implement security guardrails?

**Answer:**

Security guardrails operate at 3 levels:

**Level 1: Prompt-Level (Prevention)**
- Rules 15-18 in prompt engineering prevent the AI from generating insecure configs
- No wildcard IAM permissions
- No public security groups on databases
- No unencrypted storage

**Level 2: Post-Processing (Detection)**
- Validate all security groups: no 0.0.0.0/0 on port 22 (SSH)
- Verify all S3 buckets have encryption configured
- Check all RDS instances are in private subnets
- Ensure all EBS volumes have encryption enabled

**Level 3: User Warning (Notification)**
- If user explicitly designs something insecure (public database), show warning but don't block
- Color-coded nodes on canvas: green (secure), yellow (warning), red (critical issue)
- Security summary panel showing all findings before export

---

## G) FastAPI Questions

### 🔥 Q: What is FastAPI? Why use it over Flask/Django?

**Answer:**

| Feature | FastAPI | Flask | Django |
|---------|---------|-------|--------|
| Speed | Very fast (Starlette + Uvicorn) | Moderate | Slower |
| Async | Native async/await | Needs extensions | Django 3.1+ (limited) |
| Type Safety | Pydantic models | None built-in | Serializers |
| Auto Docs | Swagger + ReDoc auto-generated | Manual | DRF has it |
| Learning Curve | Low | Very low | High |
| Best For | APIs, microservices | Simple apps | Full web apps |

**Why FastAPI for CloudCanva:**
1. **Async is critical** — Bedrock API calls take 5-10 seconds. Async lets us handle other requests while waiting
2. **Pydantic validation** — Complex request bodies (resource configs) are validated automatically
3. **Auto-generated OpenAPI docs** — 15+ endpoints documented without extra effort
4. **Performance** — 40% faster than Flask for I/O-bound work
5. **Python ecosystem** — Jinja2, boto3, and AI libraries are Python-native

---

**📌 KEY POINTS TO REMEMBER — Section 3:**
- Know every technology choice and WHY you made it
- Be ready to draw the architecture diagram on a whiteboard
- Understand the data flow end-to-end
- Know failure modes: what happens when Bedrock is slow? When S3 is unavailable?
- Be ready to discuss trade-offs you made
- Security questions are critical — know IAM inside out

---

# SECTION 4: DEEP DIVE — TRIPSYNC

## A) Architecture

### 🔥 Q: Explain TripSync architecture end to end.

**Answer:**

```
┌─────────────────────────────────────────────┐
│  FRONTEND (React.js + Tailwind CSS)         │
│  - Trip configuration form                   │
│  - Live agent status dashboard              │
│  - Day-wise itinerary display               │
│  - Budget breakdown charts                  │
│  - NLP Chatbot interface                    │
│  - Destination comparison view              │
└──────────────────┬──────────────────────────┘
                   │ REST APIs + JWT Auth
┌──────────────────▼──────────────────────────┐
│  BACKEND (Node.js + Express.js)             │
│  ┌────────────────────────────────────────┐ │
│  │  ORCHESTRATOR                          │ │
│  │  - Receives trip request               │ │
│  │  - Spawns 5 agents in parallel         │ │
│  │  - Aggregates results                  │ │
│  │  - Generates day-wise itinerary        │ │
│  │  - Calculates budget                   │ │
│  └──┬─────┬─────┬─────┬─────┬────────────┘ │
│     │     │     │     │     │               │
│  ┌──▼┐ ┌──▼┐ ┌──▼┐ ┌──▼┐ ┌──▼┐            │
│  │ W │ │ F │ │ H │ │ E │ │ R │            │
│  │ e │ │ l │ │ o │ │ v │ │ o │            │
│  │ a │ │ i │ │ t │ │ e │ │ u │            │
│  │ t │ │ g │ │ e │ │ n │ │ t │            │
│  │ h │ │ h │ │ l │ │ t │ │ e │            │
│  │ e │ │ t │ │ s │ │ s │ │ s │            │
│  │ r │ │ s │ │   │ │   │ │   │            │
│  └───┘ └───┘ └───┘ └───┘ └───┘            │
│     │     │     │     │     │               │
│  External APIs:                             │
│  OpenWeatherMap │ SerpAPI │ SerpAPI │        │
│  SerpAPI │ OpenRouteService                 │
│                                             │
│  ┌────────────────────────────────────────┐ │
│  │  NLP CHATBOT (Compromise.js)           │ │
│  │  - Intent classification               │ │
│  │  - 10+ travel intents                  │ │
│  └────────────────────────────────────────┘ │
│                                             │
│  ┌────────────────────────────────────────┐ │
│  │  INTEGRATIONS                          │ │
│  │  - Google Calendar (OAuth2)            │ │
│  │  - Nodemailer (email notifications)    │ │
│  └────────────────────────────────────────┘ │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│  DATABASE (MongoDB)                         │
│  - Users, Trips, Itineraries, Chat History  │
└─────────────────────────────────────────────┘
```

---

### 🔥 Q: How does multi-agent orchestration work? How do 5 agents run in parallel?

**Answer:**

The orchestrator uses `Promise.all()` to run all 5 agents simultaneously:

```javascript
async function orchestrateTrip(tripRequest) {
  const { destination, dates, budget, preferences } = tripRequest;
  
  // All 5 agents start simultaneously
  const [weather, flights, hotels, events, routes] = await Promise.all([
    weatherAgent.fetch(destination, dates),
    flightAgent.search(tripRequest.origin, destination, dates),
    hotelAgent.search(destination, dates, budget.accommodation),
    eventsAgent.discover(destination, dates, preferences),
    routeAgent.plan(destination, preferences.placesToVisit)
  ]);
  
  // Aggregate results into day-wise itinerary
  const itinerary = generateItinerary(weather, flights, hotels, events, routes, dates);
  
  // Calculate budget breakdown
  const budgetBreakdown = calculateBudget(flights, hotels, events, routes);
  
  return { itinerary, budgetBreakdown };
}
```

**Why parallel, not sequential?**
- Sequential: 5 agents × ~2 sec each = ~10 seconds wait
- Parallel: All 5 start together = ~2-3 seconds total (limited by slowest agent)
- 70% reduction in user wait time

**Live Status UI:**
Each agent reports its status via real-time updates:
```javascript
// Agent sends status updates
weatherAgent.on('status', (status) => {
  io.emit('agent-status', { agent: 'weather', status }); // 'searching', 'found', 'error'
});
```

Frontend shows: ✅ Weather | ⏳ Flights | ✅ Hotels | ⏳ Events | ✅ Routes

---

### Q: How does live status UI update in real time?

**Answer:**

Using server-sent events or polling (depending on implementation):

**Approach: Polling with status endpoint**
```javascript
// Backend: Status tracking
const agentStatus = {};

app.get('/api/trip/:id/status', (req, res) => {
  res.json(agentStatus[req.params.id]);
});

// Each agent updates its status
weatherAgent.onStart(() => { agentStatus[tripId].weather = 'searching'; });
weatherAgent.onComplete(() => { agentStatus[tripId].weather = 'complete'; });
weatherAgent.onError(() => { agentStatus[tripId].weather = 'error'; });
```

**Frontend polling:**
```javascript
useEffect(() => {
  const interval = setInterval(async () => {
    const status = await fetch(`/api/trip/${tripId}/status`);
    setAgentStatuses(status);
    if (allComplete(status)) clearInterval(interval);
  }, 1000);
  return () => clearInterval(interval);
}, [tripId]);
```

---

### Q: Explain the budget breakdown algorithm.

**Answer:**

```javascript
function calculateBudget(flights, hotels, events, routes, userBudget) {
  const breakdown = {
    flights: {
      amount: flights.bestOption.price,
      percentage: 0,
      options: flights.allOptions // sorted by price
    },
    accommodation: {
      amount: hotels.selected.pricePerNight * nights,
      percentage: 0,
      options: hotels.allOptions
    },
    activities: {
      amount: events.reduce((sum, e) => sum + e.cost, 0),
      percentage: 0,
      details: events.map(e => ({ name: e.name, cost: e.cost }))
    },
    transportation: {
      amount: routes.estimatedCost,
      percentage: 0
    },
    miscellaneous: {
      amount: userBudget * 0.1, // 10% buffer
      percentage: 0
    }
  };
  
  const total = Object.values(breakdown).reduce((sum, cat) => sum + cat.amount, 0);
  
  // Calculate percentages
  Object.keys(breakdown).forEach(key => {
    breakdown[key].percentage = ((breakdown[key].amount / total) * 100).toFixed(1);
  });
  
  // Budget optimization suggestions
  if (total > userBudget) {
    breakdown.suggestions = generateSavingsSuggestions(breakdown, userBudget);
  }
  
  return { breakdown, total, withinBudget: total <= userBudget };
}
```

---

## B) Backend

### Q: How does JWT authentication work in TripSync?

**Answer:**

```javascript
// Sign-up: Hash password + create JWT
app.post('/api/auth/signup', async (req, res) => {
  const { email, password, name } = req.body;
  const hashedPassword = await bcrypt.hash(password, 12);
  const user = await User.create({ email, password: hashedPassword, name });
  
  const token = jwt.sign(
    { userId: user._id, email: user.email },
    process.env.JWT_SECRET,
    { expiresIn: '7d' }
  );
  res.json({ token, user: { id: user._id, name, email } });
});

// Middleware: Verify JWT on protected routes
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1]; // "Bearer <token>"
  if (!token) return res.status(401).json({ error: 'No token provided' });
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
};

// Protected route example
app.get('/api/trips', authMiddleware, async (req, res) => {
  const trips = await Trip.find({ userId: req.user.userId });
  res.json(trips);
});
```

**JWT Structure:**
```
Header.Payload.Signature
eyJhbGc.eyJ1c2Vy.SflKxwRJ

Header: { "alg": "HS256", "typ": "JWT" }
Payload: { "userId": "abc123", "email": "khushi@...", "iat": 1699000000, "exp": 1699604800 }
Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)
```

---

## C) Database

### Q: Why MongoDB for TripSync? Explain schema design.

**Answer:**

**Why MongoDB:**
- Trip data is highly nested and variable (different activities per day, different cities)
- Schema evolves rapidly during development
- Natural JSON format matches API responses from external services
- Aggregation pipeline for budget calculations

**Schema Design:**
```javascript
// User Schema
{
  _id: ObjectId,
  name: String,
  email: String (unique, indexed),
  password: String (hashed),
  preferences: {
    currency: String,
    travelStyle: String // 'budget', 'moderate', 'luxury'
  },
  createdAt: Date
}

// Trip Schema
{
  _id: ObjectId,
  userId: ObjectId (ref: User, indexed),
  origin: String,
  destination: String,
  dates: { start: Date, end: Date },
  budget: { total: Number, currency: String },
  itinerary: [{
    day: Number,
    date: Date,
    weather: { temp: Number, condition: String },
    activities: [{
      time: String,
      name: String,
      location: { lat: Number, lng: Number },
      cost: Number,
      duration: String
    }],
    transport: [{ from: String, to: String, mode: String, cost: Number }]
  }],
  budgetBreakdown: {
    flights: Number,
    accommodation: Number,
    activities: Number,
    transportation: Number
  },
  status: String, // 'planning', 'confirmed', 'completed'
  createdAt: Date,
  updatedAt: Date
}
```

**Indexes:**
```javascript
Trip.index({ userId: 1, createdAt: -1 }); // Get user's recent trips
Trip.index({ destination: 1 }); // Search by destination
```

---

## D) Integrations

### ⭐ Q: Explain Google Calendar OAuth2 flow step by step.

**Answer:**

```
Step 1: User clicks "Add to Calendar" →
Step 2: Redirect to Google's consent screen:
        https://accounts.google.com/o/oauth2/v2/auth?
          client_id=YOUR_ID&
          redirect_uri=http://localhost:3000/callback&
          scope=https://www.googleapis.com/auth/calendar.events&
          response_type=code&
          access_type=offline

Step 3: User grants permission → Google redirects back with auth code

Step 4: Backend exchanges code for tokens:
        POST https://oauth2.googleapis.com/token
        { code, client_id, client_secret, redirect_uri, grant_type: 'authorization_code' }
        
        Response: { access_token, refresh_token, expires_in }

Step 5: Use access_token to create calendar events:
        POST https://www.googleapis.com/calendar/v3/calendars/primary/events
        Authorization: Bearer <access_token>
        Body: { summary: "Flight to Paris", start: {...}, end: {...}, reminders: {...} }

Step 6: Store refresh_token for future access without re-login
```

**Adding trip events with reminders:**
```javascript
const event = {
  summary: `✈️ Flight: ${origin} → ${destination}`,
  start: { dateTime: flightDate.toISOString(), timeZone: 'Asia/Kolkata' },
  end: { dateTime: arrivalDate.toISOString(), timeZone: 'Europe/Paris' },
  reminders: {
    useDefault: false,
    overrides: [
      { method: 'popup', minutes: 1440 }, // 24 hours before
      { method: 'email', minutes: 2880 }  // 48 hours before
    ]
  }
};
```

---

## E) NLP Chatbot

### ⭐ Q: How does Compromise.js work? What are the 10+ travel intents?

**Answer:**

**Compromise.js** is a lightweight NLP library for JavaScript that provides:
- Part-of-speech tagging
- Named entity recognition
- Tokenization and sentence parsing
- Pattern matching with custom rules

**My 10+ travel intents:**

| Intent | Example Query | Action |
|--------|--------------|--------|
| `search_flights` | "Find flights to Paris" | Trigger flight agent |
| `search_hotels` | "Show hotels under $100" | Trigger hotel agent |
| `check_weather` | "What's the weather in Tokyo?" | Query weather API |
| `find_events` | "Things to do in London" | Trigger events agent |
| `get_directions` | "How to get from hotel to museum" | Query routing API |
| `set_budget` | "My budget is $2000" | Update trip budget |
| `compare_destinations` | "Compare Paris vs Rome" | Run comparison analysis |
| `add_to_calendar` | "Add this to my calendar" | Google Calendar OAuth |
| `modify_itinerary` | "Swap day 2 and day 3" | Reorder itinerary |
| `budget_check` | "Am I within budget?" | Calculate remaining |
| `recommend` | "Suggest a restaurant nearby" | Local recommendations |
| `cancel_booking` | "Cancel the hotel" | Remove from itinerary |

**Intent classification implementation:**
```javascript
const nlp = require('compromise');

function classifyIntent(userMessage) {
  const doc = nlp(userMessage);
  
  // Pattern matching rules
  if (doc.has('(flight|fly|airplane|airline)')) return 'search_flights';
  if (doc.has('(hotel|stay|accommodation|hostel)')) return 'search_hotels';
  if (doc.has('(weather|temperature|rain|sunny)')) return 'check_weather';
  if (doc.has('(event|activity|things to do|attraction)')) return 'find_events';
  if (doc.has('(direction|route|how to get|navigate)')) return 'get_directions';
  if (doc.has('(budget|cost|spend|afford)') && doc.has('#Money')) return 'set_budget';
  if (doc.has('(compare|versus|vs|or)') && doc.has('#Place')) return 'compare_destinations';
  if (doc.has('(calendar|schedule|remind)')) return 'add_to_calendar';
  if (doc.has('(swap|move|change|modify) (day|itinerary)')) return 'modify_itinerary';
  
  return 'general_query'; // fallback
}
```

**Limitations of rule-based NLP vs ML-based:**
- Rule-based: Fast, predictable, no training data needed, but brittle with unusual phrasings
- ML-based: Handles variations better but needs training data and is slower
- For a travel chatbot with known intents, rule-based is sufficient and more reliable

---

**📌 KEY POINTS TO REMEMBER — Section 4:**
- Multi-agent parallel execution is the key architectural decision — know Promise.all deeply
- Understand OAuth2 flow end-to-end (very commonly asked)
- JWT: signing, verification, expiry, refresh flow
- MongoDB schema design choices and trade-offs
- NLP chatbot: intent classification approach and limitations
- Know all external APIs and their free-tier limitations

---

# SECTION 5: DEEP DIVE — INTELLIQUERY

## A) LangChain & LangGraph

### 🔥 Q: What is LangChain? Core concepts.

**Answer:**

LangChain is a framework for building applications powered by LLMs. It provides:

**Core Components:**
- **Models:** Wrappers around LLMs (OpenAI, Groq, Bedrock, etc.)
- **Prompts:** Template management for structuring LLM inputs
- **Chains:** Sequential pipelines (prompt → LLM → output parser)
- **Memory:** Conversation history management
- **Agents:** LLMs that decide which tools to use dynamically
- **Tools:** External integrations (databases, APIs, search)

**Example chain:**
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_groq import ChatGroq
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a SQL expert. Convert natural language to SQL."),
    ("human", "{question}")
])

llm = ChatGroq(model="llama-3.1-70b-versatile")
chain = prompt | llm | StrOutputParser()

result = chain.invoke({"question": "Show all users who signed up last month"})
```

---

### 🔥 Q: What is LangGraph? How is it different from LangChain?

**Answer:**

| Aspect | LangChain | LangGraph |
|--------|-----------|-----------|
| Structure | Linear chains or simple agents | Directed graphs with nodes and edges |
| Control Flow | Sequential or basic branching | Complex conditional routing, loops, cycles |
| State | Passed through chain | Explicit state object shared across all nodes |
| Error Handling | Try-catch on chain | Node-level error handling with retry/reroute |
| Best For | Simple pipelines | Multi-step workflows with decisions |
| Persistence | Limited | Built-in checkpointing |

**LangGraph key concepts:**
- **Nodes:** Functions that process and transform state
- **Edges:** Connections between nodes (can be conditional)
- **State:** A shared object (TypedDict) that flows through the graph
- **Conditional Edges:** Route to different nodes based on state
- **Checkpointing:** Save and resume graph execution

**Why I chose LangGraph for IntelliQuery:**
LangChain chains are linear — great for simple NL→SQL. But IntelliQuery needs:
- Intent confirmation (human-in-the-loop)
- Conditional routing (SQL vs MongoDB path)
- Safety checks (destructive query blocking)
- RBAC enforcement at specific points
- Error recovery with rerouting

These require a graph, not a chain.

---

### 🔥 Q: Explain your 7-node pipeline in detail.

**Answer:**

```
┌─────────────────┐
│ 1. INTENT       │ ← User's natural language input
│ CLASSIFIER      │ → Determines: query, schema exploration, or admin action
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 2. SCHEMA       │ ← Fetches relevant table/collection schemas
│ ANALYZER        │ → Provides context for query generation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 3. QUERY        │ ← NL + schema context
│ GENERATOR       │ → Generates SQL or MongoDB query
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 4. SAFETY       │ ← Generated query
│ VALIDATOR       │ → Blocks DROP/DELETE/TRUNCATE, checks RBAC
└────────┬────────┘
         │ (if safe)
         ▼
┌─────────────────┐
│ 5. INTENT       │ ← Shows user what will be executed
│ CONFIRMER       │ → Waits for user approval (human-in-the-loop)
└────────┬────────┘
         │ (if confirmed)
         ▼
┌─────────────────┐
│ 6. QUERY        │ ← Approved query
│ EXECUTOR        │ → Runs against actual database, returns results
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 7. RESULT       │ ← Raw query results
│ VISUALIZER      │ → Auto-selects chart type, formats response
└─────────────────┘
```

**State object flowing through all nodes:**
```python
from typing import TypedDict, Optional, List

class PipelineState(TypedDict):
    user_input: str
    intent: str  # 'query', 'explore', 'admin'
    database_type: str  # 'postgresql', 'mysql', 'mongodb'
    schema_context: dict
    generated_query: str
    is_destructive: bool
    user_role: str  # from RBAC
    is_confirmed: bool
    query_results: list
    visualization_type: str  # 'table', 'bar', 'line', 'pie'
    error: Optional[str]
```

**Graph definition:**
```python
from langgraph.graph import StateGraph, END

workflow = StateGraph(PipelineState)

# Add nodes
workflow.add_node("classify_intent", classify_intent)
workflow.add_node("analyze_schema", analyze_schema)
workflow.add_node("generate_query", generate_query)
workflow.add_node("validate_safety", validate_safety)
workflow.add_node("confirm_intent", confirm_intent)
workflow.add_node("execute_query", execute_query)
workflow.add_node("visualize_results", visualize_results)

# Add edges
workflow.set_entry_point("classify_intent")
workflow.add_edge("classify_intent", "analyze_schema")
workflow.add_edge("analyze_schema", "generate_query")
workflow.add_conditional_edges("validate_safety", safety_router, {
    "safe": "confirm_intent",
    "blocked": END  # Destructive query blocked
})
workflow.add_conditional_edges("confirm_intent", confirmation_router, {
    "confirmed": "execute_query",
    "rejected": END
})
workflow.add_edge("execute_query", "visualize_results")
workflow.add_edge("visualize_results", END)

app = workflow.compile()
```

---

### Q: What is a two-phase multi-agent pipeline?

**Answer:**

**Phase 1: Understanding (Nodes 1-3)**
- Understand what the user wants (intent)
- Understand the database structure (schema)
- Generate the appropriate query

**Phase 2: Execution (Nodes 4-7)**
- Validate safety
- Get user confirmation
- Execute
- Present results

**Why two phases?**
- Phase 1 can run without any side effects — no database writes, no risk
- Phase 2 involves actual database access — needs safety checks
- Clear separation allows retrying Phase 1 without touching Phase 2
- Phase 1 results can be cached for similar future queries

---

## B) NL-to-SQL / NL-to-MongoDB

### 🔥 Q: How does NL to SQL translation work?

**Answer:**

**Step 1: Schema Context Loading**
```python
def analyze_schema(state: PipelineState) -> PipelineState:
    # Fetch table names, columns, types, relationships
    schema = get_database_schema(state['database_type'], state['connection'])
    # Format for LLM consumption
    schema_text = format_schema(schema)
    # Example output:
    # Table: users (id INT PK, name VARCHAR, email VARCHAR, created_at TIMESTAMP)
    # Table: orders (id INT PK, user_id INT FK→users.id, total DECIMAL, status VARCHAR)
    state['schema_context'] = schema_text
    return state
```

**Step 2: Query Generation with LLM**
```python
def generate_query(state: PipelineState) -> PipelineState:
    prompt = f"""Given this database schema:
    {state['schema_context']}
    
    Convert this natural language to {'SQL' if state['database_type'] != 'mongodb' else 'MongoDB query'}:
    "{state['user_input']}"
    
    Rules:
    - Use only tables and columns that exist in the schema
    - Use proper JOIN syntax for related tables
    - Add appropriate WHERE clauses
    - Return ONLY the query, no explanation
    """
    
    query = llm.invoke(prompt)
    state['generated_query'] = query
    return state
```

**Step 3: Example translations:**
```
Input: "Show me users who spent more than $1000 last month"
Output: SELECT u.name, u.email, SUM(o.total) as total_spent
        FROM users u
        JOIN orders o ON u.id = o.user_id
        WHERE o.created_at >= DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
          AND o.created_at < DATE_TRUNC('month', CURRENT_DATE)
        GROUP BY u.name, u.email
        HAVING SUM(o.total) > 1000;
```

**Challenges:**
- Ambiguous column references ("name" could be user name or product name)
- Implicit joins (user saying "their orders" implies JOIN)
- Date interpretation ("last month", "this year", "recently")
- Aggregation detection ("total", "average", "count")

---

### ⭐ Q: How did you implement destructive query blocking?

**Answer:**

```python
import sqlparse
from sqlparse.sql import Statement
from sqlparse.tokens import Keyword, DML

DESTRUCTIVE_OPERATIONS = {'DROP', 'DELETE', 'TRUNCATE', 'ALTER', 'UPDATE'}
# UPDATE is flagged for confirmation, not blocked entirely

def validate_safety(state: PipelineState) -> PipelineState:
    query = state['generated_query'].upper().strip()
    
    # Method 1: Token-level parsing (more reliable than regex)
    parsed = sqlparse.parse(query)[0]
    statement_type = parsed.get_type()
    
    if statement_type in ('DROP', 'DELETE', 'TRUNCATE'):
        state['is_destructive'] = True
        state['error'] = f"BLOCKED: {statement_type} operation detected. This query would modify/delete data."
        return state
    
    # Method 2: Check for destructive keywords in subqueries
    for token in parsed.flatten():
        if token.ttype is DML and token.value.upper() in DESTRUCTIVE_OPERATIONS:
            state['is_destructive'] = True
            state['error'] = f"BLOCKED: Contains {token.value.upper()} operation."
            return state
    
    # Method 3: MongoDB-specific checks
    if state['database_type'] == 'mongodb':
        mongo_destructive = ['deleteMany', 'deleteOne', 'drop', 'dropDatabase', 'remove']
        for op in mongo_destructive:
            if op in state['generated_query']:
                state['is_destructive'] = True
                state['error'] = f"BLOCKED: MongoDB {op} operation detected."
                return state
    
    # RBAC check: even non-destructive queries need permission
    if not has_permission(state['user_role'], statement_type):
        state['error'] = f"PERMISSION DENIED: Role '{state['user_role']}' cannot execute {statement_type}"
        return state
    
    state['is_destructive'] = False
    return state
```

**Why AST parsing over regex?**
- Regex `r'DROP'` would false-positive on: `SELECT * FROM dropship_orders`
- Regex misses: `DELETE /* hidden */ FROM users`
- sqlparse understands SQL grammar — knows the difference between keywords and data

---

## C) Security

### 🔥 Q: What is RBAC? How did you implement it?

**Answer:**

**RBAC (Role-Based Access Control):** Users are assigned roles, roles have permissions, permissions define allowed actions.

```
User → Role → Permissions → Allowed Actions
```

**IntelliQuery's RBAC model:**

| Role | SELECT | INSERT | UPDATE | DELETE | CREATE/DROP | Admin |
|------|--------|--------|--------|--------|-------------|-------|
| Viewer | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Editor | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Admin | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**Implementation:**
```python
from enum import Enum
from functools import wraps

class Role(Enum):
    VIEWER = "viewer"
    EDITOR = "editor"
    ADMIN = "admin"

ROLE_PERMISSIONS = {
    Role.VIEWER: {"SELECT"},
    Role.EDITOR: {"SELECT", "INSERT", "UPDATE"},
    Role.ADMIN: {"SELECT", "INSERT", "UPDATE", "DELETE", "CREATE", "DROP", "ALTER"}
}

def has_permission(user_role: str, operation: str) -> bool:
    role = Role(user_role)
    return operation.upper() in ROLE_PERMISSIONS[role]

# Workspace-level RBAC
class Workspace:
    def __init__(self, workspace_id):
        self.workspace_id = workspace_id
        self.members = {}  # {user_id: role}
    
    def add_member(self, user_id, role: Role):
        self.members[user_id] = role
    
    def check_access(self, user_id, operation):
        if user_id not in self.members:
            raise PermissionError("Not a workspace member")
        return has_permission(self.members[user_id].value, operation)
```

---

### ⭐ Q: What is Fernet encryption? How does it work?

**Answer:**

**Fernet** is a symmetric encryption scheme (part of Python's `cryptography` library) that guarantees a message encrypted with it cannot be read or tampered with without the key.

**Properties:**
- Symmetric: Same key encrypts and decrypts
- Authenticated: Includes HMAC — tampered ciphertext is rejected
- Timestamped: Includes creation time — enables TTL (time-to-live)
- Uses AES-128-CBC for encryption + HMAC-SHA256 for authentication

**How I used it in IntelliQuery:**
Database connection strings contain sensitive credentials. Multi-tenant workspaces mean multiple connection strings stored in our DB.

```python
from cryptography.fernet import Fernet

# Generate key once per deployment (stored in env variable)
# key = Fernet.generate_key()  # b'...' 32-byte base64-encoded

class ConnectionEncryptor:
    def __init__(self):
        self.fernet = Fernet(os.environ['ENCRYPTION_KEY'])
    
    def encrypt_connection_string(self, conn_str: str) -> str:
        """Encrypt before storing in database"""
        return self.fernet.encrypt(conn_str.encode()).decode()
    
    def decrypt_connection_string(self, encrypted: str) -> str:
        """Decrypt when user needs to connect"""
        return self.fernet.decrypt(encrypted.encode()).decode()

# Usage:
encryptor = ConnectionEncryptor()

# Storing:
encrypted = encryptor.encrypt_connection_string(
    "postgresql://user:password@host:5432/dbname"
)
# Stored in MongoDB: "gAAAAABl..." (not the raw credentials)

# Retrieving:
original = encryptor.decrypt_connection_string(encrypted)
# Back to: "postgresql://user:password@host:5432/dbname"
```

**Why Fernet over alternatives:**
- Simpler than raw AES (handles IV, padding, HMAC automatically)
- More secure than base64 encoding (which isn't encryption)
- Built into Python standard security libraries
- Authenticated — detects if someone modified the encrypted data

---

### Q: How did you implement multi-tenant isolation?

**Answer:**

Each workspace is completely isolated:

```python
# Workspace model
{
    "_id": ObjectId,
    "name": "Team Alpha Workspace",
    "owner_id": ObjectId,
    "join_code": "ABC123",  # For invitations
    "members": [
        {"user_id": ObjectId, "role": "admin"},
        {"user_id": ObjectId, "role": "editor"},
        {"user_id": ObjectId, "role": "viewer"}
    ],
    "connections": [
        {
            "name": "Production DB",
            "type": "postgresql",
            "encrypted_connection_string": "gAAAAABl...",  # Fernet encrypted
            "allowed_roles": ["admin", "editor"]  # Who can use this connection
        }
    ],
    "query_history": [...]  # Scoped to workspace
}

# Middleware ensures workspace isolation
def workspace_access_middleware(workspace_id, user_id):
    workspace = db.workspaces.find_one({"_id": workspace_id})
    member = next((m for m in workspace['members'] if m['user_id'] == user_id), None)
    if not member:
        raise HTTPException(403, "Not a member of this workspace")
    return member['role']
```

**Isolation guarantees:**
- Users only see workspaces they belong to
- Query history is workspace-scoped
- Connection strings are encrypted per-workspace
- RBAC is per-workspace (admin in one, viewer in another)

---

### Q: Explain JWT vs Google OAuth vs OTP — when to use which?

**Answer:**

| Method | How it works | Best for |
|--------|-------------|----------|
| **JWT** | Server signs a token with user claims, client sends it with requests | API authentication, stateless auth, mobile apps |
| **Google OAuth** | Delegates auth to Google, gets user profile | Social login, users who don't want another password |
| **OTP** | Time-limited code sent via email/SMS | Password reset, 2FA, high-security actions |

**In IntelliQuery, all three are used:**

1. **Primary auth: JWT** — After login (email/password or Google), server issues JWT. Every API call includes this token.

2. **Social login: Google OAuth** — Users click "Sign in with Google" → OAuth2 flow → we get their email/name → issue our own JWT.

3. **OTP: Email verification + Password reset**
```python
import random
import string
from datetime import datetime, timedelta

def generate_otp():
    otp = ''.join(random.choices(string.digits, k=6))
    expiry = datetime.now() + timedelta(minutes=10)
    return otp, expiry

# Send via Nodemailer/SMTP
# Verify: compare submitted OTP with stored OTP + check expiry
```

---

## D) Databases

### ⭐ Q: When to use PostgreSQL vs MySQL vs MongoDB?

**Answer:**

| Criteria | PostgreSQL | MySQL | MongoDB |
|----------|-----------|-------|---------|
| **Data structure** | Complex relational | Simple relational | Flexible/nested |
| **ACID compliance** | Full | Full (InnoDB) | Per-document |
| **JSON support** | Excellent (JSONB) | Basic (JSON type) | Native |
| **Performance** | Complex queries | Simple reads | Write-heavy |
| **Scalability** | Vertical | Vertical | Horizontal (sharding) |
| **Best for** | Analytics, GIS, complex logic | Web apps, CMS | Real-time, IoT, catalogs |

**How IntelliQuery supports all three:**
```python
class DatabaseAdapter:
    """Abstract interface for all database types"""
    
    def get_schema(self) -> dict:
        raise NotImplementedError
    
    def execute_query(self, query: str) -> list:
        raise NotImplementedError

class PostgreSQLAdapter(DatabaseAdapter):
    def __init__(self, connection_string):
        self.conn = psycopg2.connect(connection_string)
    
    def execute_query(self, query):
        cursor = self.conn.cursor()
        cursor.execute(query)
        return cursor.fetchall()

class MongoDBAdapter(DatabaseAdapter):
    def __init__(self, connection_string):
        self.client = MongoClient(connection_string)
    
    def execute_query(self, query):
        # query is a MongoDB aggregation pipeline (list of dicts)
        collection = self.client[self.db_name][query['collection']]
        return list(collection.aggregate(query['pipeline']))

# Factory pattern
def get_adapter(db_type, connection_string):
    adapters = {
        'postgresql': PostgreSQLAdapter,
        'mysql': MySQLAdapter,
        'mongodb': MongoDBAdapter
    }
    return adapters[db_type](connection_string)
```

---

## E) AI/LLM

### Q: What is Groq API? What is LLaMA 3.1?

**Answer:**

**Groq:** A hardware company that built the LPU (Language Processing Unit) — custom chips optimized for LLM inference. Their API provides:
- Extremely fast inference (500+ tokens/second vs ~50 for GPT-4)
- Hosts open-source models (LLaMA, Mixtral)
- Simple API compatible with OpenAI format
- Free tier available

**LLaMA 3.1 (Large Language Model Meta AI):**
- Open-source LLM by Meta
- Available in 8B, 70B, 405B parameter versions
- I used the 70B-versatile variant via Groq
- Competitive with GPT-4 on coding and reasoning tasks
- Open weights mean no vendor lock-in

**Why Groq + LLaMA 3.1 for IntelliQuery (not OpenAI):**
1. **Speed:** Query generation needs to feel instant — Groq's LPU delivers
2. **Cost:** Free tier for development, much cheaper for production
3. **Open source:** No vendor lock-in, can self-host later
4. **Quality:** LLaMA 3.1 70B is excellent for SQL generation tasks

---

### ⭐ Q: How did you implement auto-visualization?

**Answer:**

The Visualizer node analyzes query results and selects the best chart type:

```python
def visualize_results(state: PipelineState) -> PipelineState:
    results = state['query_results']
    query = state['generated_query']
    
    # Analyze result structure
    columns = get_column_names(results)
    column_types = infer_column_types(results)
    row_count = len(results)
    
    # Decision logic for chart type
    viz_type = determine_visualization(columns, column_types, query, row_count)
    state['visualization_type'] = viz_type
    return state

def determine_visualization(columns, types, query, row_count):
    has_numeric = any(t == 'numeric' for t in types.values())
    has_date = any(t == 'date' for t in types.values())
    has_categorical = any(t == 'string' for t in types.values())
    has_aggregation = any(kw in query.upper() for kw in ['SUM', 'COUNT', 'AVG', 'GROUP BY'])
    
    # Rule-based selection
    if row_count == 1:
        return 'metric_card'  # Single value display
    elif has_date and has_numeric:
        return 'line_chart'  # Time series
    elif has_aggregation and has_categorical and has_numeric:
        if row_count <= 6:
            return 'pie_chart'  # Few categories
        else:
            return 'bar_chart'  # Many categories
    elif has_numeric and len([c for c in types.values() if c == 'numeric']) >= 2:
        return 'scatter_plot'  # Two numeric columns
    elif row_count > 20:
        return 'table'  # Too much data for charts
    else:
        return 'table'  # Default fallback
```

---

**📌 KEY POINTS TO REMEMBER — Section 5:**
- Know the difference between LangChain and LangGraph clearly
- Be able to draw the 7-node pipeline on a whiteboard
- Understand state management in graph-based AI systems
- Security: RBAC + Fernet + destructive query blocking = defense in depth
- Know why you chose each database type
- Auto-visualization logic is rule-based, not AI-based (deterministic)

---

# SECTION 6: DATA STRUCTURES & ALGORITHMS

## A) Core Concepts

### Arrays & Strings

**Key Patterns:**
- Two Pointers: Start/end pointers moving inward
- Sliding Window: Fixed or variable size window over array
- Prefix Sum: Precompute cumulative sums for range queries
- Kadane's Algorithm: Maximum subarray sum in O(n)

**Two Pointers Template:**
```java
// Pattern: Find pair with given sum in sorted array
public int[] twoSum(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return new int[]{left, right};
        else if (sum < target) left++;
        else right--;
    }
    return new int[]{-1, -1};
}
// Time: O(n), Space: O(1)
```

**Sliding Window Template:**
```java
// Pattern: Longest substring with at most K distinct characters
public int longestSubstring(String s, int k) {
    Map<Character, Integer> map = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        map.put(s.charAt(right), map.getOrDefault(s.charAt(right), 0) + 1);
        
        while (map.size() > k) {
            char leftChar = s.charAt(left);
            map.put(leftChar, map.get(leftChar) - 1);
            if (map.get(leftChar) == 0) map.remove(leftChar);
            left++;
        }
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// Time: O(n), Space: O(k)
```

---

### Linked Lists

**Key Operations:**
- Reversal (iterative & recursive)
- Cycle detection (Floyd's algorithm)
- Finding middle (slow-fast pointers)
- Merge two sorted lists

```java
// Reverse a linked list
public ListNode reverse(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
// Time: O(n), Space: O(1)

// Detect cycle (Floyd's Tortoise & Hare)
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
// Time: O(n), Space: O(1)
```

---

### Stacks & Queues

**Monotonic Stack:** Finds next greater/smaller element in O(n)

```java
// Next Greater Element
public int[] nextGreater(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Stack<Integer> stack = new Stack<>();  // stores indices
    
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[i] > nums[stack.peek()]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
}
// Time: O(n), Space: O(n)
```

---

### Trees

**Traversals:**
```java
// Inorder (Left, Root, Right) — gives sorted order for BST
void inorder(TreeNode root) {
    if (root == null) return;
    inorder(root.left);
    process(root.val);
    inorder(root.right);
}

// Level-order (BFS)
List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.add(node.left);
            if (node.right != null) queue.add(node.right);
        }
        result.add(level);
    }
    return result;
}
```

**Lowest Common Ancestor:**
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode left = lowestCommonAncestor(root.left, p, q);
    TreeNode right = lowestCommonAncestor(root.right, p, q);
    if (left != null && right != null) return root;
    return left != null ? left : right;
}
// Time: O(n), Space: O(h) where h = height
```

**Diameter of Binary Tree:**
```java
int diameter = 0;
public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return diameter;
}
private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    diameter = Math.max(diameter, left + right);
    return 1 + Math.max(left, right);
}
// Time: O(n), Space: O(h)
```

---

### Graphs

**BFS Template:**
```java
public void bfs(int start, List<List<Integer>> adj) {
    boolean[] visited = new boolean[adj.size()];
    Queue<Integer> queue = new LinkedList<>();
    queue.add(start);
    visited[start] = true;
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (int neighbor : adj.get(node)) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                queue.add(neighbor);
            }
        }
    }
}
// Time: O(V + E), Space: O(V)
```

**DFS Template:**
```java
public void dfs(int node, List<List<Integer>> adj, boolean[] visited) {
    visited[node] = true;
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            dfs(neighbor, adj, visited);
        }
    }
}
```

**Dijkstra's (Shortest Path):**
```java
public int[] dijkstra(int start, List<List<int[]>> adj, int n) {
    int[] dist = new int[n];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[start] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
    pq.add(new int[]{start, 0});
    
    while (!pq.isEmpty()) {
        int[] curr = pq.poll();
        int u = curr[0], d = curr[1];
        if (d > dist[u]) continue;
        for (int[] edge : adj.get(u)) {
            int v = edge[0], w = edge[1];
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    return dist;
}
// Time: O((V + E) log V), Space: O(V)
```

**Topological Sort (Kahn's BFS):**
```java
public List<Integer> topologicalSort(int n, List<List<Integer>> adj) {
    int[] indegree = new int[n];
    for (int u = 0; u < n; u++)
        for (int v : adj.get(u))
            indegree[v]++;
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) queue.add(i);
    
    List<Integer> order = new ArrayList<>();
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order.add(node);
        for (int neighbor : adj.get(node)) {
            if (--indegree[neighbor] == 0)
                queue.add(neighbor);
        }
    }
    return order.size() == n ? order : new ArrayList<>(); // empty if cycle exists
}
// Time: O(V + E), Space: O(V)
```

**Union-Find (Disjoint Set Union):**
```java
class UnionFind {
    int[] parent, rank;
    
    UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }
    
    void union(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return;
        if (rank[px] < rank[py]) parent[px] = py;
        else if (rank[px] > rank[py]) parent[py] = px;
        else { parent[py] = px; rank[px]++; }
    }
}
// Find: O(α(n)) ≈ O(1) amortized
```

---

### Dynamic Programming

**Key Patterns:**

**1. 0/1 Knapsack:**
```java
public int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            dp[i][w] = dp[i-1][w]; // don't take
            if (weights[i-1] <= w)
                dp[i][w] = Math.max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);
        }
    }
    return dp[n][capacity];
}
// Time: O(n * capacity), Space: O(n * capacity)
```

**2. Longest Common Subsequence:**
```java
public int lcs(String s1, String s2) {
    int m = s1.length(), n = s2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1.charAt(i-1) == s2.charAt(j-1))
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
        }
    }
    return dp[m][n];
}
// Time: O(m*n), Space: O(m*n)
```

**3. Coin Change:**
```java
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i)
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
// Time: O(amount * coins), Space: O(amount)
```

**4. House Robber:**
```java
public int rob(int[] nums) {
    if (nums.length == 1) return nums[0];
    int prev2 = 0, prev1 = nums[0];
    for (int i = 1; i < nums.length; i++) {
        int curr = Math.max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Time: O(n), Space: O(1)
```

---

### Binary Search

**Template (Search on Answer):**
```java
// Find minimum capacity to ship within D days
public int shipWithinDays(int[] weights, int days) {
    int left = Arrays.stream(weights).max().getAsInt();
    int right = Arrays.stream(weights).sum();
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canShip(weights, mid, days))
            right = mid;
        else
            left = mid + 1;
    }
    return left;
}

private boolean canShip(int[] weights, int capacity, int days) {
    int daysNeeded = 1, currentLoad = 0;
    for (int w : weights) {
        if (currentLoad + w > capacity) {
            daysNeeded++;
            currentLoad = 0;
        }
        currentLoad += w;
    }
    return daysNeeded <= days;
}
// Time: O(n * log(sum)), Space: O(1)
```

---

### Backtracking

**Template:**
```java
// Generate all permutations
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
    return result;
}

private void backtrack(int[] nums, List<Integer> current, boolean[] used, List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.add(nums[i]);
        backtrack(nums, current, used, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}
// Time: O(n!), Space: O(n)
```

---

## B) Most Asked LeetCode Problems with Solutions

### 🔥 Two Sum (Easy)

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement))
            return new int[]{map.get(complement), i};
        map.put(nums[i], i);
    }
    return new int[]{};
}
// Time: O(n), Space: O(n)
```

### 🔥 Best Time to Buy and Sell Stock (Easy)

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
// Time: O(n), Space: O(1)
```

### 🔥 Valid Parentheses (Easy)

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
// Time: O(n), Space: O(n)
```

### 🔥 Merge Intervals (Medium)

```java
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
    List<int[]> result = new ArrayList<>();
    result.add(intervals[0]);
    
    for (int i = 1; i < intervals.length; i++) {
        int[] last = result.get(result.size() - 1);
        if (intervals[i][0] <= last[1])
            last[1] = Math.max(last[1], intervals[i][1]);
        else
            result.add(intervals[i]);
    }
    return result.toArray(new int[0][]);
}
// Time: O(n log n), Space: O(n)
```

### 🔥 LRU Cache (Medium)

```java
class LRUCache {
    int capacity;
    Map<Integer, Node> map;
    Node head, tail; // doubly linked list
    
    class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new HashMap<>();
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head.next = tail;
        tail.prev = head;
    }
    
    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        addToFront(node);
        return node.val;
    }
    
    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        Node node = new Node(key, value);
        map.put(key, node);
        addToFront(node);
        if (map.size() > capacity) {
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
    }
    
    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    
    private void addToFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
// Time: O(1) for get and put, Space: O(capacity)
```

### 🔥 Number of Islands (Medium)

```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                dfs(grid, i, j);
                count++;
            }
        }
    }
    return count;
}

private void dfs(char[][] grid, int i, int j) {
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] == '0')
        return;
    grid[i][j] = '0'; // mark visited
    dfs(grid, i+1, j);
    dfs(grid, i-1, j);
    dfs(grid, i, j+1);
    dfs(grid, i, j-1);
}
// Time: O(M*N), Space: O(M*N) worst case recursion
```

### 🔥 Longest Substring Without Repeating Characters (Medium)

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int left = 0, maxLen = 0;
    
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c))
            left = Math.max(left, map.get(c) + 1);
        map.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// Time: O(n), Space: O(min(n, 26))
```

### 🔥 Container With Most Water (Medium)

```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1, max = 0;
    while (left < right) {
        int area = Math.min(height[left], height[right]) * (right - left);
        max = Math.max(max, area);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return max;
}
// Time: O(n), Space: O(1)
```

### ⚠️ Trapping Rain Water (Hard)

```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    
    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = Math.max(leftMax, height[left]);
            water += leftMax - height[left];
            left++;
        } else {
            rightMax = Math.max(rightMax, height[right]);
            water += rightMax - height[right];
            right--;
        }
    }
    return water;
}
// Time: O(n), Space: O(1)
```

### 🔥 Course Schedule (Medium) — Topological Sort

```java
public boolean canFinish(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] indegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);
        indegree[pre[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++)
        if (indegree[i] == 0) queue.add(i);
    
    int count = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        count++;
        for (int next : adj.get(course))
            if (--indegree[next] == 0) queue.add(next);
    }
    return count == numCourses;
}
// Time: O(V + E), Space: O(V + E)
```

### 🔥 Word Break (Medium) — DP

```java
public boolean wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    boolean[] dp = new boolean[s.length() + 1];
    dp[0] = true;
    
    for (int i = 1; i <= s.length(); i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }
    return dp[s.length()];
}
// Time: O(n^2), Space: O(n)
```

---

**📌 KEY POINTS TO REMEMBER — Section 6:**
- Always state Time & Space complexity after coding
- Start with brute force, optimize step by step
- Practice writing code on whiteboard (no IDE autocomplete)
- Know when to use which data structure
- DP: define state, transition, base case
- Graphs: BFS for shortest path (unweighted), DFS for connectivity
- Binary Search: useful beyond sorted arrays (search on answer space)

---

# SECTION 7: OBJECT ORIENTED PROGRAMMING

## Four Pillars of OOP

### 1. Encapsulation 🔥

**Definition:** Bundling data (fields) and methods that operate on that data within a single class, and restricting direct access to some components.

```java
// BAD — no encapsulation
class BankAccount {
    public double balance; // anyone can modify directly!
}

// GOOD — encapsulated
class BankAccount {
    private double balance;
    
    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
    
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        this.balance += amount;
    }
    
    public void withdraw(double amount) {
        if (amount > balance) throw new InsufficientFundsException();
        this.balance -= amount;
    }
}
```

**Connection to my projects:** In IntelliQuery, database connection strings are encapsulated — you can't access the raw string, only use it through the `execute_query()` method which enforces RBAC checks.

---

### 2. Inheritance

**Definition:** A mechanism where a new class (child) derives properties and behaviors from an existing class (parent).

```java
// Base class
abstract class DatabaseAdapter {
    protected String connectionString;
    
    public abstract List<Map<String, Object>> executeQuery(String query);
    public abstract Map<String, Object> getSchema();
    
    // Common behavior
    public boolean testConnection() {
        try {
            getSchema();
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}

// Child classes
class PostgreSQLAdapter extends DatabaseAdapter {
    @Override
    public List<Map<String, Object>> executeQuery(String query) {
        // PostgreSQL-specific implementation
    }
}

class MongoDBAdapter extends DatabaseAdapter {
    @Override
    public List<Map<String, Object>> executeQuery(String query) {
        // MongoDB-specific implementation (aggregation pipeline)
    }
}
```

---

### 3. Polymorphism 🔥

**Definition:** Objects of different classes can be treated through the same interface, with each class providing its own implementation.

**Compile-time (Overloading):**
```java
class QueryGenerator {
    public String generate(String nlQuery, String dbType) { ... }
    public String generate(String nlQuery, String dbType, String schema) { ... }
    public String generate(String nlQuery, DatabaseAdapter adapter) { ... }
}
```

**Runtime (Overriding):**
```java
DatabaseAdapter adapter = getAdapter(userSelection); // returns PostgreSQL or MongoDB
// Same method call, different behavior based on actual type
List<Map<String, Object>> results = adapter.executeQuery(query);
```

---

### 4. Abstraction

**Definition:** Hiding complex implementation details and showing only the necessary features.

```java
// User sees this simple interface
interface InfrastructureDesigner {
    void addResource(String type, Map<String, Object> config);
    void connect(String resource1, String resource2);
    String exportTerraform();
}

// Complex implementation hidden behind it
class CloudCanvaDesigner implements InfrastructureDesigner {
    private Canvas canvas;
    private TerraformRenderer renderer;
    private SecurityValidator validator;
    private BedrockClient bedrock;
    
    @Override
    public String exportTerraform() {
        // 200 lines of complex logic: validation, template rendering,
        // cross-reference resolution, ZIP generation...
        // User doesn't need to know any of this
    }
}
```

---

## Method Overloading vs Overriding

| Aspect | Overloading | Overriding |
|--------|-------------|-----------|
| When | Compile-time | Runtime |
| Where | Same class | Parent-child classes |
| Method signature | Different parameters | Same parameters |
| Return type | Can differ | Must be same/covariant |
| Keyword | None | `@Override` annotation |
| Also called | Static polymorphism | Dynamic polymorphism |

---

## Abstract Class vs Interface ⭐

| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| Methods | Can have concrete + abstract | All abstract (Java 8+: default methods) |
| Variables | Can have instance variables | Only constants (public static final) |
| Constructor | Yes | No |
| Inheritance | Single | Multiple |
| Access modifiers | Any | Public only (methods) |
| **When to use** | "is-a" with shared code | "can-do" capability contract |

**Example:**
```java
// Abstract class: shared behavior + template
abstract class Agent {
    protected String name;
    public Agent(String name) { this.name = name; }
    
    public void run() { // Template method
        prepare();
        execute();
        cleanup();
    }
    protected abstract void execute(); // subclass implements
    protected void prepare() { /* common prep */ }
    protected void cleanup() { /* common cleanup */ }
}

// Interface: capability contract
interface Cacheable {
    String getCacheKey();
    Duration getCacheTTL();
}

// A class can extend one abstract class AND implement multiple interfaces
class WeatherAgent extends Agent implements Cacheable, Serializable {
    @Override protected void execute() { /* fetch weather */ }
    @Override public String getCacheKey() { return "weather:" + location; }
    @Override public Duration getCacheTTL() { return Duration.ofHours(1); }
}
```

---

## SOLID Principles ⭐

### S — Single Responsibility Principle
Each class has one reason to change.

```java
// BAD
class UserService {
    public void createUser() { ... }
    public void sendEmail() { ... }    // Should be separate
    public void generateReport() { ... } // Should be separate
}

// GOOD
class UserService { public void createUser() { ... } }
class EmailService { public void sendEmail() { ... } }
class ReportService { public void generateReport() { ... } }
```

### O — Open/Closed Principle
Open for extension, closed for modification.

```java
// Instead of modifying TerraformRenderer for each new resource:
interface ResourceTemplate {
    String render(Map<String, Object> config);
}

class EC2Template implements ResourceTemplate { ... }
class S3Template implements ResourceTemplate { ... }
class LambdaTemplate implements ResourceTemplate { ... }
// Adding new resource = new class, not modifying existing ones
```

### L — Liskov Substitution Principle
Subtypes must be substitutable for their base types.

### I — Interface Segregation Principle
Clients shouldn't depend on interfaces they don't use.

```java
// BAD
interface Worker {
    void code();
    void design();
    void test();
    void manage();
}

// GOOD
interface Coder { void code(); }
interface Designer { void design(); }
interface Tester { void test(); }
```

### D — Dependency Inversion Principle
High-level modules shouldn't depend on low-level modules. Both should depend on abstractions.

```java
// BAD
class QueryExecutor {
    private PostgreSQLConnection conn; // depends on concrete class
}

// GOOD
class QueryExecutor {
    private DatabaseAdapter adapter; // depends on abstraction
    public QueryExecutor(DatabaseAdapter adapter) { this.adapter = adapter; }
}
```

---

## Design Patterns

### Singleton
```java
public class ConfigManager {
    private static volatile ConfigManager instance;
    private Map<String, String> config;
    
    private ConfigManager() { config = loadConfig(); }
    
    public static ConfigManager getInstance() {
        if (instance == null) {
            synchronized (ConfigManager.class) {
                if (instance == null) instance = new ConfigManager();
            }
        }
        return instance;
    }
}
```

### Factory
```java
public class DatabaseAdapterFactory {
    public static DatabaseAdapter create(String type, String connStr) {
        switch (type) {
            case "postgresql": return new PostgreSQLAdapter(connStr);
            case "mysql": return new MySQLAdapter(connStr);
            case "mongodb": return new MongoDBAdapter(connStr);
            default: throw new UnsupportedDatabaseException(type);
        }
    }
}
// Usage: DatabaseAdapter adapter = DatabaseAdapterFactory.create("postgresql", connStr);
```

### Observer
```java
// Agent status updates in TripSync
interface AgentStatusListener {
    void onStatusChange(String agentName, String status);
}

class Orchestrator {
    private List<AgentStatusListener> listeners = new ArrayList<>();
    
    public void addListener(AgentStatusListener l) { listeners.add(l); }
    
    private void notifyListeners(String agent, String status) {
        for (AgentStatusListener l : listeners) l.onStatusChange(agent, status);
    }
}
```

### Strategy
```java
// Different visualization strategies
interface VisualizationStrategy {
    Object render(List<Map<String, Object>> data);
}

class BarChartStrategy implements VisualizationStrategy { ... }
class LineChartStrategy implements VisualizationStrategy { ... }
class PieChartStrategy implements VisualizationStrategy { ... }

class ResultVisualizer {
    private VisualizationStrategy strategy;
    public void setStrategy(VisualizationStrategy s) { this.strategy = s; }
    public Object visualize(List<Map<String, Object>> data) { return strategy.render(data); }
}
```

### Builder
```java
// Building complex Terraform configurations
class TerraformConfig {
    private String provider;
    private List<Resource> resources;
    private Map<String, String> variables;
    
    private TerraformConfig(Builder builder) { ... }
    
    public static class Builder {
        public Builder provider(String p) { ... return this; }
        public Builder addResource(Resource r) { ... return this; }
        public Builder addVariable(String k, String v) { ... return this; }
        public TerraformConfig build() { return new TerraformConfig(this); }
    }
}

// Usage:
TerraformConfig config = new TerraformConfig.Builder()
    .provider("aws")
    .addResource(vpc)
    .addResource(ec2)
    .addVariable("region", "us-east-1")
    .build();
```

---

## Memory: Stack vs Heap

| Stack | Heap |
|-------|------|
| Method calls, local variables, references | Objects, instance variables |
| LIFO order | No order (managed by GC) |
| Fast allocation/deallocation | Slower (GC overhead) |
| Thread-specific | Shared across threads |
| Size limited (StackOverflowError) | Larger (OutOfMemoryError) |
| Primitive values stored directly | Objects stored here |

```java
void example() {
    int x = 10;           // x (primitive) → STACK
    String s = "hello";   // s (reference) → STACK, "hello" (object) → HEAP
    int[] arr = new int[5]; // arr (reference) → STACK, array object → HEAP
}
```

---

**📌 KEY POINTS TO REMEMBER — Section 7:**
- Know all 4 pillars with code examples from YOUR projects
- SOLID principles — explain with real-world analogies
- Design patterns: know at least Singleton, Factory, Observer, Strategy
- Abstract class vs Interface — this is asked in 90% of interviews
- Stack vs Heap — understand garbage collection basics
- Java Collections Framework: ArrayList vs LinkedList, HashMap internals

---

# SECTION 8: OPERATING SYSTEMS

## Process vs Thread vs Goroutine 🔥

| Aspect | Process | Thread | Goroutine |
|--------|---------|--------|-----------|
| Memory | Separate address space | Shared within process | Shared (Go runtime managed) |
| Creation cost | Heavy (fork, copy) | Medium (shared memory) | Very light (~2KB stack) |
| Communication | IPC (pipes, sockets) | Shared memory | Channels |
| Context switch | Expensive | Moderate | Cheap (user-space) |
| Isolation | Full | None (crash affects all) | Managed by Go runtime |
| Example | Chrome tabs | Java threads | Go concurrent functions |

**Connection to projects:** TripSync's 5 parallel agents are like threads sharing the same process memory (the trip request context). Each agent runs concurrently, shares data through the orchestrator, and reports status back.

---

## Multithreading vs Multiprocessing

| Multithreading | Multiprocessing |
|----------------|-----------------|
| Multiple threads in one process | Multiple processes |
| Shared memory — fast communication | Separate memory — IPC needed |
| GIL in Python limits true parallelism | True parallelism |
| Best for I/O-bound tasks | Best for CPU-bound tasks |
| One crash can kill all threads | Process isolation |

**When I'd use each:**
- Bedrock API calls (I/O-bound) → Multithreading/Async (FastAPI does this)
- Heavy computation (parsing 1000 Terraform files) → Multiprocessing

---

## Context Switching

**What:** OS saving the state of one process/thread and loading another.

**Steps:**
1. Save current process state (registers, PC, stack pointer) to PCB
2. Update PCB status (running → ready/waiting)
3. Select next process (scheduling algorithm)
4. Load new process state from its PCB
5. Update memory mappings (page table base register)
6. Resume execution

**Cost:** ~1-10 microseconds. Expensive because:
- CPU cache is invalidated (cache miss storm)
- TLB flush for process switches
- Pipeline flush

---

## Synchronization 🔥

### Mutex (Mutual Exclusion)
```
- Binary lock: locked/unlocked
- Only the thread that locks it can unlock it
- Prevents race conditions on shared resources
```

### Semaphore
```
- Counter-based: allows N threads simultaneously
- Binary semaphore (N=1) ≈ mutex
- Counting semaphore (N>1) for resource pools
- Example: Connection pool with 10 slots = Semaphore(10)
```

### Monitor
```
- High-level synchronization: mutex + condition variables
- Java's synchronized keyword implements monitors
- wait(), notify(), notifyAll() for thread coordination
```

```java
// Producer-Consumer with synchronized (Monitor)
class BoundedBuffer {
    private Queue<Integer> buffer = new LinkedList<>();
    private int capacity;
    
    public synchronized void produce(int item) throws InterruptedException {
        while (buffer.size() == capacity) wait();  // buffer full
        buffer.add(item);
        notifyAll();  // wake consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) wait();  // buffer empty
        int item = buffer.poll();
        notifyAll();  // wake producers
        return item;
    }
}
```

---

## Deadlock 🔥

**Four necessary conditions (ALL must hold):**
1. **Mutual Exclusion:** Resource held by only one process at a time
2. **Hold and Wait:** Process holds resources while waiting for others
3. **No Preemption:** Resources can't be forcibly taken away
4. **Circular Wait:** Circular chain of processes waiting for each other

**Prevention (break any one condition):**
- Break circular wait: Order all resources, always acquire in order
- Break hold and wait: Acquire all resources atomically or release all before requesting
- Break no preemption: Allow OS to forcibly take resources

**Detection:** Build a resource allocation graph → if cycle exists → deadlock

**Example in my context:** If IntelliQuery's pipeline Node A waits for Node B's output AND Node B waits for Node A's output → deadlock. LangGraph prevents this with DAG (directed acyclic graph) structure — no cycles allowed.

---

## Memory Management

### Paging 🔥
- Physical memory divided into fixed-size **frames** (typically 4KB)
- Logical memory divided into same-size **pages**
- Page table maps logical pages → physical frames
- Eliminates external fragmentation

```
Logical Address: [Page Number | Offset]
Physical Address: Page Table[Page Number] → Frame Number + Offset
```

### Virtual Memory
- Allows programs to use more memory than physically available
- Pages swapped between RAM and disk (swap space)
- **Page fault:** Accessing a page not in RAM → OS loads from disk
- **Thrashing:** Too many page faults → system spends more time swapping than executing

### Page Replacement Algorithms

| Algorithm | How it works | Pros | Cons |
|-----------|-------------|------|------|
| **FIFO** | Replace oldest page | Simple | Belady's anomaly |
| **LRU** | Replace least recently used | Good performance | Expensive to implement |
| **Optimal** | Replace page not used for longest time | Best theoretically | Requires future knowledge |

**LRU Implementation:** Same as LRU Cache problem! (HashMap + Doubly Linked List)

---

## CPU Scheduling 🔥

| Algorithm | Type | Description | Pros | Cons |
|-----------|------|-------------|------|------|
| **FCFS** | Non-preemptive | First come first served | Simple, fair | Convoy effect |
| **SJF** | Non-preemptive | Shortest job first | Optimal avg wait | Starvation of long jobs |
| **SRTF** | Preemptive | Shortest remaining time | Better than SJF | Starvation |
| **Round Robin** | Preemptive | Time quantum rotation | Fair, no starvation | High context switch if quantum too small |
| **Priority** | Either | Based on priority | Important tasks first | Starvation (solved with aging) |

**Round Robin Example:**
```
Time Quantum = 4ms
Process: P1(10ms), P2(3ms), P3(7ms)

Timeline: P1[4] → P2[3] → P3[4] → P1[4] → P3[3] → P1[2]
```

---

## Inter-Process Communication (IPC)

| Method | Description | Use case |
|--------|-------------|----------|
| Pipes | Unidirectional byte stream | Parent-child communication |
| Message Queues | Structured messages with types | Decoupled processes |
| Shared Memory | Memory region accessible by multiple processes | High-speed data sharing |
| Sockets | Network-based communication | Client-server, distributed systems |
| Signals | Asynchronous notifications | Process control (SIGKILL, SIGTERM) |

---

## Race Condition

**What:** Multiple threads access shared data concurrently, and the outcome depends on the order of execution.

```java
// RACE CONDITION:
int counter = 0;
// Thread 1: counter++  →  read(0) → increment → write(1)
// Thread 2: counter++  →  read(0) → increment → write(1)
// Expected: 2, Got: 1!

// FIX: synchronized or AtomicInteger
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // thread-safe
```

**In my projects:** If two IntelliQuery users in the same workspace modify settings simultaneously without locking, race condition occurs. Solved with optimistic locking (version numbers in DynamoDB).

---

**📌 KEY POINTS TO REMEMBER — Section 8:**
- Process vs Thread — most asked OS question
- Deadlock conditions + prevention — know all 4 conditions by heart
- Paging + Virtual Memory — understand page faults
- CPU Scheduling — be able to trace through examples
- Race condition — explain with code and how to fix
- Connect OS concepts to your projects where possible

---

# SECTION 9: COMPUTER NETWORKS

## OSI Model — All 7 Layers 🔥

| Layer | Name | Function | Protocols | Data Unit |
|-------|------|----------|-----------|-----------|
| 7 | Application | User-facing services | HTTP, HTTPS, FTP, SMTP, DNS | Data |
| 6 | Presentation | Encryption, compression, format | SSL/TLS, JPEG, ASCII | Data |
| 5 | Session | Session management | NetBIOS, RPC | Data |
| 4 | Transport | Reliable delivery, flow control | TCP, UDP | Segment |
| 3 | Network | Routing, logical addressing | IP, ICMP, ARP | Packet |
| 2 | Data Link | Physical addressing, framing | Ethernet, WiFi, MAC | Frame |
| 1 | Physical | Bits on the wire | Cables, hubs, signals | Bits |

**Mnemonic:** "All People Seem To Need Data Processing" (top to bottom)

---

## TCP vs UDP 🔥

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (3-way handshake) | Connectionless |
| Reliability | Guaranteed delivery (ACK, retransmit) | Best effort (no guarantee) |
| Ordering | In-order delivery | No ordering |
| Speed | Slower (overhead) | Faster |
| Header size | 20 bytes | 8 bytes |
| Flow control | Yes (sliding window) | No |
| Use cases | HTTP, FTP, email, database queries | Video streaming, gaming, DNS, VoIP |

**In my projects:**
- CloudCanva API calls → TCP (need reliable delivery of Terraform configs)
- If I built real-time collaborative canvas → WebSockets (TCP underneath)
- Live agent status updates could use UDP for speed (loss is acceptable)

---

## TCP 3-Way Handshake ⭐

```
Client                          Server
  |                               |
  |------- SYN (seq=x) --------->|   Step 1: Client initiates
  |                               |
  |<-- SYN-ACK (seq=y, ack=x+1)-|   Step 2: Server acknowledges + sends its SYN
  |                               |
  |------- ACK (ack=y+1) ------->|   Step 3: Client acknowledges server's SYN
  |                               |
  |====== Connection Established =====|
```

**Why 3 steps (not 2)?**
- 2-way can't confirm that the client received server's initial sequence number
- Prevents stale connection requests from being accepted

**Connection Termination (4-way):**
```
Client → FIN → Server
Client ← ACK ← Server
Client ← FIN ← Server
Client → ACK → Server
```

---

## HTTP vs HTTPS 🔥

| HTTP | HTTPS |
|------|-------|
| Port 80 | Port 443 |
| Plaintext | Encrypted (TLS) |
| No authentication | Server authenticated via certificate |
| Fast | Slightly slower (TLS handshake) |
| Vulnerable to MITM | Secure against eavesdropping |

**How TLS/SSL works (simplified):**
1. Client sends "Hello" with supported cipher suites
2. Server responds with chosen cipher + its certificate (public key)
3. Client verifies certificate against trusted CAs
4. Client generates session key, encrypts with server's public key, sends it
5. Both sides now have the symmetric session key
6. All subsequent data encrypted with session key (fast symmetric encryption)

---

## REST vs GraphQL vs gRPC

| Aspect | REST | GraphQL | gRPC |
|--------|------|---------|------|
| Protocol | HTTP | HTTP | HTTP/2 |
| Data format | JSON | JSON | Protobuf (binary) |
| Endpoints | Multiple (/users, /posts) | Single (/graphql) | Service methods |
| Over/Under fetching | Common problem | Client specifies exact fields | Defined in proto |
| Best for | Simple CRUD APIs | Complex nested data | Microservice-to-microservice |

**My projects use REST because:**
- Simple request/response model
- Well-understood caching (HTTP caching)
- Easy to debug (readable JSON)
- Good tooling (Postman, Swagger)

---

## 🔥 What happens when you type a URL in browser?

```
1. Browser checks cache (browser cache → OS cache → router cache)
2. DNS Resolution:
   - Browser cache → OS hosts file → DNS resolver →
   - Root DNS → TLD DNS (.com) → Authoritative DNS →
   - Returns IP address (e.g., 93.184.216.34)
   
3. TCP Connection (3-way handshake with server at IP:443)

4. TLS Handshake (if HTTPS):
   - Exchange certificates, negotiate cipher, establish session key

5. HTTP Request:
   - GET /path HTTP/2
   - Headers: Host, User-Agent, Accept, Cookie

6. Server Processing:
   - Load balancer → Application server → Database query →
   - Generate response

7. HTTP Response:
   - Status: 200 OK
   - Headers: Content-Type, Cache-Control
   - Body: HTML content

8. Browser Rendering:
   - Parse HTML → Build DOM tree
   - Parse CSS → Build CSSOM
   - Combine → Render Tree
   - Layout (calculate positions)
   - Paint (draw pixels)
   - Composite (layer management)

9. Additional Resources:
   - CSS, JS, images trigger new requests (steps 3-8 repeat)
   - JS execution can modify DOM (reflow/repaint)
```

---

## HTTP Status Codes ⭐

| Code | Meaning | When |
|------|---------|------|
| 200 | OK | Successful GET/PUT |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 301 | Moved Permanently | URL changed, update bookmarks |
| 304 | Not Modified | Cached version is still valid |
| 400 | Bad Request | Invalid input/syntax |
| 401 | Unauthorized | Not authenticated (no/invalid token) |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource conflict (duplicate) |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server crashed |
| 502 | Bad Gateway | Upstream server error |
| 503 | Service Unavailable | Server overloaded/maintenance |

**In CloudCanva:**
- 200: Terraform generation successful
- 201: New project created
- 400: Invalid resource configuration
- 401: Cognito token expired
- 403: User doesn't own this project
- 429: Bedrock rate limit hit
- 500: Unexpected backend error

---

## WebSockets vs HTTP Polling

| Aspect | WebSockets | HTTP Polling | Long Polling |
|--------|-----------|--------------|--------------|
| Connection | Persistent, bidirectional | New request each time | Held open until data |
| Latency | Real-time | Polling interval delay | Near real-time |
| Server load | Low (one connection) | High (frequent requests) | Moderate |
| Best for | Chat, live data, gaming | Simple status checks | Notifications |

**TripSync's live agent status** uses polling (simpler), but WebSockets would be better for a production version with real-time updates.

---

## CORS (Cross-Origin Resource Sharing) 🔥

**Problem:** Browser blocks requests from `localhost:3000` (frontend) to `localhost:8000` (backend) — different origins.

**Solution:** Server sends headers allowing cross-origin requests:
```
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

**FastAPI implementation (CloudCanva):**
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://cloudcanva.amplifyapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

**Preflight request:** Browser sends OPTIONS request first for non-simple requests (POST with JSON body) to check if server allows it.

---

## CDN (Content Delivery Network)

**What:** Distributed network of servers that caches content closer to users.

**How:** 
- User in India requests `cloudcanva.com/static/logo.png`
- Instead of going to US-East-1 (origin), served from Mumbai edge server
- Reduces latency from ~200ms to ~20ms

**AWS CloudFront** (used in CloudCanva):
- Caches static assets (JS, CSS, images)
- Integrates with S3 (origin) and Amplify
- Supports HTTPS with custom domains
- Edge locations in 400+ cities globally

---

## Load Balancing Algorithms

| Algorithm | How | Best for |
|-----------|-----|----------|
| Round Robin | Rotate through servers sequentially | Equal-capacity servers |
| Weighted Round Robin | More requests to more powerful servers | Mixed-capacity servers |
| Least Connections | Send to server with fewest active connections | Variable request duration |
| IP Hash | Same client always goes to same server | Session persistence |
| Random | Random server selection | Simple, surprisingly effective |

**AWS ELB Types:**
- **ALB (Application):** Layer 7, path/host routing, WebSockets
- **NLB (Network):** Layer 4, ultra-low latency, millions of requests/sec
- **CLB (Classic):** Legacy, both Layer 4 and 7

---

**📌 KEY POINTS TO REMEMBER — Section 9:**
- "What happens when you type a URL" — top interview question, know every step
- TCP vs UDP — know use cases, not just differences
- HTTP status codes — know the common ones by heart
- CORS — every full-stack developer gets asked this
- OSI model — know which layer handles what
- REST API design principles (from your 15+ endpoints experience)

---

# SECTION 10: DATABASES

## SQL vs NoSQL 🔥

| Aspect | SQL (Relational) | NoSQL (Non-relational) |
|--------|-----------------|----------------------|
| Structure | Tables with fixed schema | Documents, key-value, graph, columnar |
| Schema | Strict, predefined | Flexible, schema-less |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |
| ACID | Full support | Varies (eventual consistency common) |
| Joins | Powerful multi-table joins | Limited or no joins |
| Best for | Complex queries, transactions | High volume, flexible data, real-time |
| Examples | PostgreSQL, MySQL | MongoDB, Redis, Cassandra, DynamoDB |

**When to choose:**
- SQL: Banking, e-commerce (need ACID), complex reporting, well-defined schema
- NoSQL: Social media feeds, IoT data, real-time analytics, rapidly evolving schema

**In my projects:**
- IntelliQuery supports BOTH (PostgreSQL + MySQL + MongoDB)
- TripSync uses MongoDB (flexible trip data, nested itineraries)
- CloudCanva uses DynamoDB (key-value access patterns, serverless)

---

## ACID Properties ⭐

| Property | Meaning | Example |
|----------|---------|---------|
| **A**tomicity | All or nothing — transaction fully completes or fully rolls back | Transfer $100: debit + credit both succeed or both fail |
| **C**onsistency | DB moves from one valid state to another | Balance can't go negative if constraint exists |
| **I**solation | Concurrent transactions don't interfere | Two transfers on same account don't corrupt balance |
| **D**urability | Once committed, data survives crashes | Power failure after commit → data still there |

---

## Indexing 🔥

**What:** Data structure (usually B-Tree or B+Tree) that speeds up lookups at the cost of extra storage and slower writes.

**Without index:** Full table scan — O(n)
**With index:** B-Tree traversal — O(log n)

**B-Tree Index (how it works):**
```
                    [50]
                   /    \
              [20,30]    [70,80]
             / |  \     / |  \
          [10][25][35] [60][75][90]  ← Leaf nodes point to actual rows
```

**Types:**
- **B-Tree:** Default, good for range queries and equality
- **Hash:** O(1) for equality, no range queries
- **Composite:** Index on multiple columns `(last_name, first_name)`
- **Partial:** Index only rows matching a condition
- **Full-text:** For text search (LIKE '%word%')

**When to index:**
- Columns in WHERE clauses
- Columns in JOIN conditions
- Columns in ORDER BY
- High cardinality columns (many distinct values)

**When NOT to index:**
- Small tables (full scan is fast enough)
- Columns with low cardinality (boolean — only 2 values)
- Frequently updated columns (index rebuild overhead)

---

## Database Normalization

| Normal Form | Rule | Example Violation |
|-------------|------|-------------------|
| **1NF** | No repeating groups, atomic values | Column "phones" = "123, 456" (should be separate rows) |
| **2NF** | 1NF + no partial dependencies (composite key) | Non-key column depends on part of composite key |
| **3NF** | 2NF + no transitive dependencies | zip_code → city (city depends on zip, not on PK directly) |
| **BCNF** | 3NF + every determinant is a candidate key | Stronger version of 3NF |

**Denormalization:** Intentionally breaking normalization for read performance.
- Example: Store `user_name` in the `orders` table to avoid JOINs on every query
- Trade-off: Faster reads, slower writes, data consistency risk

---

## Joins 🔥

```sql
-- Setup
-- users: id, name, department_id
-- departments: id, name

-- INNER JOIN: Only matching rows from both tables
SELECT u.name, d.name 
FROM users u INNER JOIN departments d ON u.department_id = d.id;
-- Users without department? Excluded. Departments without users? Excluded.

-- LEFT JOIN: All rows from left + matching from right
SELECT u.name, d.name 
FROM users u LEFT JOIN departments d ON u.department_id = d.id;
-- Users without department? Included (d.name = NULL). 

-- RIGHT JOIN: All rows from right + matching from left
SELECT u.name, d.name 
FROM users u RIGHT JOIN departments d ON u.department_id = d.id;
-- Departments without users? Included.

-- FULL OUTER JOIN: All rows from both
SELECT u.name, d.name 
FROM users u FULL OUTER JOIN departments d ON u.department_id = d.id;

-- SELF JOIN: Table joined with itself
SELECT e.name AS employee, m.name AS manager
FROM employees e JOIN employees m ON e.manager_id = m.id;

-- CROSS JOIN: Cartesian product (every row × every row)
SELECT * FROM colors CROSS JOIN sizes;
```

---

## Transactions and Isolation Levels ⭐

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|-----------|--------------------|--------------| 
| Read Uncommitted | ✅ possible | ✅ possible | ✅ possible |
| Read Committed | ❌ prevented | ✅ possible | ✅ possible |
| Repeatable Read | ❌ prevented | ❌ prevented | ✅ possible |
| Serializable | ❌ prevented | ❌ prevented | ❌ prevented |

**Definitions:**
- **Dirty Read:** Reading uncommitted data from another transaction
- **Non-repeatable Read:** Same query returns different data within one transaction
- **Phantom Read:** New rows appear that match a previous query's WHERE clause

**Trade-off:** Higher isolation = more locking = less concurrency = slower performance

---

## MongoDB Deep Dive

### Document Model
```javascript
// A single document (like a row, but nested)
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "Trip to Paris",
  user_id: ObjectId("..."),
  dates: { start: ISODate("2025-03-01"), end: ISODate("2025-03-05") },
  itinerary: [
    { day: 1, activities: ["Eiffel Tower", "Louvre"] },
    { day: 2, activities: ["Versailles"] }
  ],
  budget: { total: 2000, spent: 1500, currency: "USD" }
}
```

### Aggregation Pipeline
```javascript
// Example: Average trip budget by destination
db.trips.aggregate([
  { $match: { status: "completed" } },           // Filter
  { $group: {                                      // Group
      _id: "$destination",
      avgBudget: { $avg: "$budget.total" },
      tripCount: { $sum: 1 }
  }},
  { $sort: { avgBudget: -1 } },                   // Sort
  { $limit: 10 }                                   // Top 10
]);
```

### Sharding
- Distributes data across multiple servers (shards)
- Shard key determines which shard a document goes to
- Enables horizontal scaling for massive datasets
- Example: Shard by `user_id` — all of one user's data on same shard

### Replication
- Primary + Secondary nodes
- Primary handles writes, secondaries handle reads
- Automatic failover if primary goes down
- Read preference: primary, secondary, nearest

---

## Database Design for My Projects

### CloudCanva (DynamoDB)
```
Table: Projects
  PK: USER#<userId>
  SK: PROJECT#<projectId>
  Attributes: name, canvasState (JSON), resourceCount, lastModified

Table: Templates  
  PK: RESOURCE_TYPE#<type>
  SK: VERSION#<version>
  Attributes: templateBody, requiredFields, validProperties

Access Patterns:
- Get all user's projects: Query PK = USER#123
- Get specific project: Query PK = USER#123, SK = PROJECT#456
- Update canvas: UpdateItem with canvasState
```

### TripSync (MongoDB)
```
Collection: users     → { _id, name, email, passwordHash, preferences }
Collection: trips     → { _id, userId, destination, dates, budget, itinerary[], status }
Collection: chats     → { _id, tripId, messages: [{role, content, timestamp}] }

Indexes:
- users: { email: 1 } (unique)
- trips: { userId: 1, createdAt: -1 }
- trips: { destination: 1 } (for search)
```

### IntelliQuery (MongoDB)
```
Collection: workspaces → { _id, name, ownerId, joinCode, members[], connections[] }
Collection: queryHistory → { _id, workspaceId, userId, nlInput, generatedQuery, results, timestamp }

Indexes:
- workspaces: { "members.userId": 1 }
- queryHistory: { workspaceId: 1, timestamp: -1 }
```

---

**📌 KEY POINTS TO REMEMBER — Section 10:**
- SQL vs NoSQL — know when to choose which (with real examples)
- ACID — be able to explain each with a bank transfer example
- Indexing — B-Tree structure, when to use/not use
- Joins — draw Venn diagrams for each type
- Know your own schema designs and explain WHY you chose that structure
- MongoDB aggregation pipeline stages
- DynamoDB: partition key + sort key access patterns

---

# SECTION 11: SYSTEM DESIGN

## A) Core Concepts

### Horizontal vs Vertical Scaling 🔥

| | Vertical (Scale Up) | Horizontal (Scale Out) |
|--|--------------------|-----------------------|
| How | Bigger machine (more CPU/RAM) | More machines |
| Limit | Hardware ceiling | Practically unlimited |
| Cost | Expensive exponentially | Linear cost |
| Downtime | Required for upgrade | Zero downtime |
| Complexity | Simple | Distributed systems complexity |
| Example | Upgrade EC2 from t3.micro to c5.4xlarge | Add more EC2 instances behind ALB |

---

### CAP Theorem ⭐

**In a distributed system, you can only guarantee 2 out of 3:**
- **C**onsistency: Every read gets the most recent write
- **A**vailability: Every request gets a response (even if stale)
- **P**artition tolerance: System works despite network failures

**In practice:** Partition tolerance is non-negotiable (networks fail), so the real choice is **CP or AP**:
- **CP (Consistency + Partition tolerance):** MongoDB (primary reads), HBase — best for banking, inventory
- **AP (Availability + Partition tolerance):** DynamoDB (eventually consistent), Cassandra — best for social feeds, caching

**CloudCanva choice:** DynamoDB with eventual consistency — if a user saves a project and immediately reads it, they might see stale data for a few milliseconds. Acceptable for our use case.

---

### Caching Strategies

| Strategy | How | When to use |
|----------|-----|-------------|
| **Cache-aside** | App checks cache first; on miss, fetches from DB and writes to cache | General purpose, most common |
| **Write-through** | Write to cache AND DB simultaneously | Need consistency |
| **Write-behind** | Write to cache first, async write to DB | High write throughput |
| **Read-through** | Cache fetches from DB on miss (cache manages itself) | Simplify app code |

**Where I'd add caching in my projects:**
- CloudCanva: Cache Terraform templates (rarely change)
- TripSync: Cache weather data (same city, valid for 1 hour)
- IntelliQuery: Cache schema metadata (doesn't change often)

---

### Message Queues

**Why:** Decouple producers from consumers, handle traffic spikes, ensure delivery.

| Queue | Type | Best for |
|-------|------|----------|
| **SQS** | Pull-based, fully managed | AWS-native, simple decoupling |
| **Kafka** | Distributed log, high throughput | Event streaming, real-time analytics |
| **RabbitMQ** | Push-based, routing support | Complex routing, multiple consumers |

**Example use:** If CloudCanva has 1000 concurrent users requesting Terraform generation, instead of overwhelming Bedrock API:
```
User Request → SQS Queue → Worker Pool (5 workers) → Bedrock API
```
Queue absorbs spikes, workers process at sustainable rate.

---

### Microservices vs Monolith

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | All-or-nothing | Independent per service |
| Scaling | Scale entire app | Scale only what needs it |
| Development | Simple initially | Complex infrastructure |
| Communication | Function calls | Network calls (HTTP/gRPC) |
| Data | Shared database | Database per service |
| Best for | Small teams, MVPs | Large teams, large scale |

**My projects are monoliths** — appropriate for the team size and complexity. But at scale:
```
CloudCanva Microservices:
- Canvas Service (React Flow state management)
- AI Service (Bedrock integration)
- Terraform Service (template rendering)
- Auth Service (Cognito)
- Storage Service (S3 + DynamoDB)
```

---

## B) System Design Questions

### 🔥 Design URL Shortener (bit.ly)

**Requirements:**
- Shorten long URLs to short ones
- Redirect short URL to original
- Track click analytics
- Handle 100M URLs, 1B redirects/month

**Design:**

```
┌──────────┐      ┌──────────────┐      ┌──────────┐
│  Client  │─────→│  API Gateway │─────→│  Service │
└──────────┘      └──────────────┘      └────┬─────┘
                                              │
                  ┌───────────────────────────┼─────┐
                  │                           │     │
              ┌───▼──┐              ┌─────────▼─┐   │
              │ Cache │              │  Database │   │
              │(Redis)│              │(Cassandra)│   │
              └───────┘              └───────────┘   │
                                                    │
                                            ┌───────▼──┐
                                            │ Analytics│
                                            │  (Kafka) │
                                            └──────────┘
```

**Key decisions:**
- **ID Generation:** Base62 encoding of auto-increment ID or hash
  - `https://short.ly/abc123` → 62^6 = 56 billion possible URLs
- **Database:** Cassandra (write-heavy, eventual consistency OK)
- **Cache:** Redis for hot URLs (most redirects hit popular URLs)
- **Analytics:** Kafka stream for click tracking (async, doesn't slow redirect)

**API:**
```
POST /shorten { "url": "https://very-long-url.com/..." } → { "short": "abc123" }
GET /abc123 → 301 Redirect to original URL
```

---

### 🔥 Design Rate Limiter

**Algorithms:**

**1. Token Bucket:**
```
- Bucket holds N tokens, refills at rate R tokens/second
- Each request consumes 1 token
- No tokens → reject (429 Too Many Requests)
- Allows bursts up to bucket size
```

**2. Sliding Window Counter:**
```
- Track requests in current window + previous window
- Weighted count: prev_count × overlap_percentage + curr_count
- Smoother than fixed window
```

**Implementation:**
```python
import time
from collections import defaultdict

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.tokens = capacity
        self.last_refill = time.time()
    
    def allow_request(self):
        now = time.time()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.refill_rate)
        self.last_refill = now
        
        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False

# Per-user rate limiting
user_buckets = defaultdict(lambda: TokenBucket(capacity=10, refill_rate=1))
```

---

### Design CloudCanva at Scale (1M users)

**Current architecture (single server):**
- FastAPI + single EC2 → handles ~1000 concurrent users

**Scaled architecture:**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CloudFront CDN                                │
│   (Static assets: JS, CSS, images)                                  │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────────┐
│                    Application Load Balancer                         │
└───────┬─────────────────┬──────────────────────┬────────────────────┘
        │                 │                      │
┌───────▼───┐    ┌───────▼───┐          ┌───────▼───┐
│ FastAPI   │    │ FastAPI   │          │ FastAPI   │
│ Instance 1│    │ Instance 2│   ...    │ Instance N│
│ (ECS/Farg)│    │ (ECS/Farg)│          │ (ECS/Farg)│
└───────┬───┘    └───────┬───┘          └───────┬───┘
        │                │                      │
        └────────────────┼──────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
┌───────▼──┐    ┌───────▼───┐    ┌───────▼──────┐
│   SQS    │    │   Redis   │    │  DynamoDB    │
│(AI Queue)│    │  (Cache)  │    │(Auto-scaled) │
└─────┬────┘    └───────────┘    └──────────────┘
      │
┌─────▼──────────┐
│ AI Worker Pool │
│ (Bedrock calls)│
│ Auto-scaled    │
└────────────────┘
```

**Key scaling decisions:**
1. **Compute:** ECS Fargate (auto-scaling containers, no server management)
2. **AI calls:** SQS queue + worker pool (Bedrock has rate limits)
3. **Cache:** Redis for generated templates (same NL description → same output)
4. **Database:** DynamoDB (auto-scales, serverless)
5. **CDN:** CloudFront for frontend assets
6. **Auth:** Cognito scales automatically

---

### Design TripSync at Scale

**Challenges at scale:**
- API rate limits (OpenWeatherMap, SerpAPI)
- 5 parallel agents × 100K users = 500K concurrent API calls
- Real-time status updates for all users

**Solutions:**
1. **API Rate Limiting:** Queue + worker pool per external API
2. **Caching:** Redis cache for weather (city + date → valid 1 hour), popular destinations
3. **Real-time:** WebSocket server (separate from API server) for live updates
4. **Database:** MongoDB sharded by user_id
5. **Background Processing:** Celery/Bull queue for itinerary generation

---

**📌 KEY POINTS TO REMEMBER — Section 11:**
- Start every design with requirements (functional + non-functional)
- Estimate scale: users, requests/sec, data volume
- Draw the high-level diagram first, then dive deep into components
- Always discuss trade-offs (why this choice over alternatives)
- Know: Load Balancer, Cache, Queue, Database, CDN placement
- For YOUR projects: know how you'd scale them 100x

---

# SECTION 12: REACT & FRONTEND

## Virtual DOM 🔥

**What:** A lightweight JavaScript object representation of the actual DOM. React keeps a copy of the DOM in memory.

**How it works:**
1. State changes → React creates new Virtual DOM tree
2. React diffs new VDOM with previous VDOM (reconciliation)
3. Calculates minimum set of actual DOM changes needed
4. Batches and applies only those changes to real DOM

**Why it's fast:**
- DOM manipulation is expensive (triggers reflow/repaint)
- VDOM diffing is cheap (JavaScript object comparison)
- Batch updates: 10 state changes → 1 DOM update

---

## useState vs useReducer ⭐

```jsx
// useState: Simple state
const [count, setCount] = useState(0);
setCount(count + 1);

// useReducer: Complex state with multiple sub-values or logic
const [state, dispatch] = useReducer(reducer, initialState);

function reducer(state, action) {
  switch (action.type) {
    case 'ADD_RESOURCE':
      return { ...state, nodes: [...state.nodes, action.payload] };
    case 'REMOVE_RESOURCE':
      return { ...state, nodes: state.nodes.filter(n => n.id !== action.payload) };
    case 'UPDATE_CONNECTION':
      return { ...state, edges: updateEdges(state.edges, action.payload) };
    default:
      return state;
  }
}

// When to use useReducer:
// - Multiple sub-values that update together
// - Next state depends on previous state
// - Complex update logic (CloudCanva canvas state!)
```

**CloudCanva uses useReducer** for canvas state because adding a resource might also add edges, update references, and trigger validation — one action, multiple state changes.

---

## useEffect 🔥

```jsx
// Runs after every render
useEffect(() => { console.log('rendered'); });

// Runs once on mount
useEffect(() => { fetchProjects(); }, []);

// Runs when dependency changes
useEffect(() => { validateResources(nodes); }, [nodes]);

// Cleanup (runs before next effect or unmount)
useEffect(() => {
  const interval = setInterval(pollStatus, 1000);
  return () => clearInterval(interval); // cleanup
}, [tripId]);
```

**Common mistakes:**
- Missing dependency → stale closure (uses old state value)
- Object/array in deps → infinite loop (reference changes each render)
- No cleanup → memory leaks (intervals, event listeners, subscriptions)

---

## useMemo vs useCallback

```jsx
// useMemo: Memoize a computed VALUE
const expensiveResult = useMemo(() => {
  return nodes.filter(n => n.type === 'ec2').map(computeConfig);
}, [nodes]); // Only recomputes when nodes change

// useCallback: Memoize a FUNCTION reference
const handleNodeClick = useCallback((nodeId) => {
  dispatch({ type: 'SELECT_NODE', payload: nodeId });
}, [dispatch]); // Same function reference unless dispatch changes

// When to use:
// useMemo: Expensive calculations, preventing child re-renders with object/array props
// useCallback: Passing callbacks to optimized child components (React.memo)
```

---

## React Performance Optimization ⭐

1. **React.memo:** Prevent re-render if props haven't changed
```jsx
const ResourceNode = React.memo(({ data, selected }) => {
  return <div className={selected ? 'selected' : ''}>{data.name}</div>;
});
```

2. **Code splitting:** Load components on demand
```jsx
const HeavyEditor = React.lazy(() => import('./TerraformEditor'));
```

3. **Virtualization:** Only render visible list items (react-window)
4. **Debouncing:** Delay expensive operations (search input → API call)
5. **Key prop:** Use stable, unique keys for lists (never use array index for dynamic lists)

---

## Next.js — SSR vs SSG vs ISR vs CSR 🔥

| Strategy | Renders | When | Use case |
|----------|---------|------|----------|
| **CSR** (Client-Side) | In browser | On every visit | Dashboards, SPAs |
| **SSR** (Server-Side) | On server per request | Every request | Dynamic, personalized content |
| **SSG** (Static Site Generation) | At build time | Once | Blog, docs, marketing pages |
| **ISR** (Incremental Static) | At build + revalidates | Periodically | E-commerce, news (fresh but fast) |

**CloudCanva uses:**
- SSR for the main dashboard (personalized, user-specific projects)
- CSR for the canvas editor (highly interactive, client-heavy)
- SSG for documentation/help pages (static content)

```jsx
// SSR (runs on every request)
export async function getServerSideProps(context) {
  const projects = await fetchUserProjects(context.req.cookies.token);
  return { props: { projects } };
}

// SSG (runs at build time)
export async function getStaticProps() {
  const docs = await fetchDocs();
  return { props: { docs } };
}

// ISR (static + revalidation)
export async function getStaticProps() {
  const templates = await fetchTemplates();
  return { props: { templates }, revalidate: 3600 }; // Revalidate every hour
}
```

---

## TypeScript Benefits Over JavaScript

1. **Type safety:** Catches errors at compile time, not runtime
2. **Better IDE support:** Autocomplete, refactoring, error detection
3. **Self-documenting:** Types serve as documentation
4. **Safer refactoring:** Change a type → compiler shows all breakages
5. **Better team collaboration:** Clear contracts between components

```typescript
// CloudCanva resource type (TypeScript)
interface AWSResource {
  id: string;
  type: 'ec2' | 'vpc' | 's3' | 'lambda' | 'rds';
  name: string;
  config: Record<string, unknown>;
  connections: string[]; // IDs of connected resources
}

// This catches errors at compile time:
const resource: AWSResource = {
  id: '1',
  type: 'invalid', // ❌ Type error! Not in union type
  name: 'my-instance',
  config: {},
  connections: []
};
```

---

## React Flow — How I Used It in CloudCanva

**React Flow** is a library for building node-based editors and interactive diagrams.

```jsx
import ReactFlow, { 
  useNodesState, 
  useEdgesState, 
  addEdge,
  Background,
  Controls,
  MiniMap 
} from 'reactflow';

// Custom node types for each AWS resource
const nodeTypes = {
  ec2: EC2Node,
  vpc: VPCNode,
  s3: S3Node,
  lambda: LambdaNode,
  rds: RDSNode,
};

function CanvasEditor() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes);
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges);
  
  const onConnect = useCallback((params) => {
    // Validate connection (e.g., can't connect S3 directly to VPC)
    if (isValidConnection(params)) {
      setEdges((eds) => addEdge(params, eds));
    }
  }, []);
  
  return (
    <ReactFlow
      nodes={nodes}
      edges={edges}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      onConnect={onConnect}
      nodeTypes={nodeTypes}
      fitView
    >
      <Background />
      <Controls />
      <MiniMap />
    </ReactFlow>
  );
}
```

---

**📌 KEY POINTS TO REMEMBER — Section 12:**
- Virtual DOM → Reconciliation → Batch updates (know the flow)
- Hooks: useState, useEffect, useMemo, useCallback — know when to use each
- Next.js rendering strategies — know trade-offs
- TypeScript: explain with YOUR project examples
- React Flow: how nodes, edges, and custom components work
- Performance: React.memo, code splitting, virtualization

---

# SECTION 13: NODE.JS & BACKEND

## Event Loop in Node.js 🔥

**Node.js is single-threaded but non-blocking.** The event loop is how it handles concurrency.

```
   ┌───────────────────────────┐
┌─>│           timers          │  ← setTimeout, setInterval callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │     pending callbacks     │  ← I/O callbacks deferred from previous loop
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │       idle, prepare       │  ← internal use
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │           poll            │  ← I/O events (incoming connections, data)
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │           check           │  ← setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │      close callbacks      │  ← socket.on('close')
│  └─────────────┬─────────────┘
└─────────────────┘
```

**Key insight:** Node.js offloads I/O operations (file reads, network requests, DB queries) to the OS kernel or thread pool (libuv). When they complete, callbacks are placed in the event loop queue.

**Example:**
```javascript
console.log('1');                    // Synchronous → runs immediately
setTimeout(() => console.log('2'), 0); // Timer → queued for next loop
Promise.resolve().then(() => console.log('3')); // Microtask → runs before next phase
console.log('4');                    // Synchronous → runs immediately

// Output: 1, 4, 3, 2
// Microtasks (Promises) have priority over macrotasks (setTimeout)
```

---

## Blocking vs Non-blocking I/O ⭐

```javascript
// BLOCKING (bad for Node.js — freezes entire server)
const data = fs.readFileSync('/large-file.json'); // Server stops here
processData(data);

// NON-BLOCKING (good — server continues serving other requests)
fs.readFile('/large-file.json', (err, data) => {
  processData(data); // Called when file read completes
});
// Server continues handling other requests immediately

// ASYNC/AWAIT (non-blocking, cleaner syntax)
const data = await fs.promises.readFile('/large-file.json');
processData(data);
```

**Why this matters for TripSync:**
5 API calls (weather, flights, hotels, events, routes) — if blocking, total time = sum of all. With non-blocking (Promise.all), total time = max of all.

---

## Middleware Pattern in Express 🔥

**Middleware:** Functions that have access to `req`, `res`, and `next`. They execute in order.

```javascript
// Request flow: Client → Middleware1 → Middleware2 → Route Handler → Response

// Logging middleware
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // Pass to next middleware
};

// Auth middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// Rate limiting middleware
const rateLimit = (maxRequests, windowMs) => {
  const requests = new Map();
  return (req, res, next) => {
    const key = req.ip;
    const now = Date.now();
    const windowStart = now - windowMs;
    
    const userRequests = (requests.get(key) || []).filter(t => t > windowStart);
    if (userRequests.length >= maxRequests) {
      return res.status(429).json({ error: 'Too many requests' });
    }
    userRequests.push(now);
    requests.set(key, userRequests);
    next();
  };
};

// Apply middleware
app.use(logger);                          // All routes
app.use('/api', authenticate);            // All /api routes
app.use('/api', rateLimit(100, 60000));   // 100 req/min for API
```

---

## Error Handling in Express

```javascript
// Async error wrapper (avoids try-catch in every route)
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

// Route with error handling
app.get('/api/trips/:id', authenticate, asyncHandler(async (req, res) => {
  const trip = await Trip.findById(req.params.id);
  if (!trip) throw new AppError('Trip not found', 404);
  if (trip.userId.toString() !== req.user.userId) throw new AppError('Forbidden', 403);
  res.json(trip);
}));

// Custom error class
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

// Global error handler (must have 4 params)
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal server error';
  
  console.error(`Error: ${err.message}`, err.stack);
  res.status(statusCode).json({ error: message });
});
```

---

## Authentication: Complete JWT Flow 🔥

```
┌──────────┐                          ┌──────────┐
│  Client  │                          │  Server  │
└────┬─────┘                          └────┬─────┘
     │                                     │
     │ 1. POST /login {email, password}    │
     │────────────────────────────────────→│
     │                                     │ Verify credentials
     │                                     │ Generate JWT
     │ 2. { token: "eyJ...", user: {...} } │
     │←────────────────────────────────────│
     │                                     │
     │ Store token (localStorage/cookie)   │
     │                                     │
     │ 3. GET /api/trips                   │
     │ Authorization: Bearer eyJ...        │
     │────────────────────────────────────→│
     │                                     │ Verify JWT signature
     │                                     │ Check expiry
     │                                     │ Extract user from payload
     │ 4. { trips: [...] }                 │
     │←────────────────────────────────────│
     │                                     │
     │ TOKEN EXPIRED                       │
     │                                     │
     │ 5. POST /refresh { refreshToken }   │
     │────────────────────────────────────→│
     │                                     │ Verify refresh token
     │ 6. { newToken: "eyJ..." }           │ Generate new access token
     │←────────────────────────────────────│
```

**Token storage options:**
- `localStorage`: Easy, but vulnerable to XSS
- `httpOnly cookie`: Secure from XSS, but vulnerable to CSRF
- Best: `httpOnly` cookie with CSRF token

---

## Google OAuth2 Flow (Complete) ⭐

```javascript
const { OAuth2Client } = require('google-auth-library');
const client = new OAuth2Client(CLIENT_ID, CLIENT_SECRET, REDIRECT_URI);

// Step 1: Generate auth URL
app.get('/auth/google', (req, res) => {
  const url = client.generateAuthUrl({
    scope: ['profile', 'email'],
    access_type: 'offline', // Get refresh token
    prompt: 'consent'
  });
  res.redirect(url);
});

// Step 2: Handle callback
app.get('/auth/google/callback', asyncHandler(async (req, res) => {
  const { code } = req.query;
  
  // Exchange code for tokens
  const { tokens } = await client.getToken(code);
  client.setCredentials(tokens);
  
  // Get user info
  const { data } = await axios.get('https://www.googleapis.com/oauth2/v2/userinfo', {
    headers: { Authorization: `Bearer ${tokens.access_token}` }
  });
  
  // Find or create user
  let user = await User.findOne({ email: data.email });
  if (!user) {
    user = await User.create({ 
      name: data.name, 
      email: data.email, 
      googleId: data.id,
      avatar: data.picture 
    });
  }
  
  // Issue our own JWT
  const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET, { expiresIn: '7d' });
  res.redirect(`${FRONTEND_URL}/auth/success?token=${token}`);
}));
```

---

## WebSockets in Node.js

```javascript
const { Server } = require('socket.io');
const io = new Server(httpServer, {
  cors: { origin: 'http://localhost:3000' }
});

// Connection handling
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  // Join a trip room (for live status updates)
  socket.on('join-trip', (tripId) => {
    socket.join(`trip:${tripId}`);
  });
  
  // Emit agent status to all users watching this trip
  socket.on('agent-update', ({ tripId, agent, status }) => {
    io.to(`trip:${tripId}`).emit('agent-status', { agent, status });
  });
  
  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});

// From agent orchestrator: emit updates
function updateAgentStatus(tripId, agent, status) {
  io.to(`trip:${tripId}`).emit('agent-status', { agent, status });
}
```

---

**📌 KEY POINTS TO REMEMBER — Section 13:**
- Event loop: Know the phases and execution order
- Microtasks (Promises) execute before macrotasks (setTimeout)
- Middleware: request → middleware chain → handler → response
- JWT flow: know signing, verification, refresh token rotation
- OAuth2: understand the code → token → user info flow
- Error handling: async wrapper + global error handler pattern

---

# SECTION 14: PYTHON & FASTAPI

## Python vs JavaScript — Key Differences

| Aspect | Python | JavaScript |
|--------|--------|-----------|
| Typing | Dynamically typed (type hints optional) | Dynamically typed (TS adds types) |
| Concurrency | GIL limits threading; asyncio for async | Event loop, non-blocking by default |
| Syntax | Indentation-based, cleaner | Braces, semicolons, more verbose |
| Package manager | pip, poetry, uv | npm, yarn, pnpm |
| Best for | Data science, AI/ML, scripting, backends | Frontend, full-stack web, real-time |
| Execution | Interpreted, slower | V8 JIT compiled, faster |

---

## Async/Await in Python 🔥

```python
import asyncio
import httpx

# Synchronous (blocking) — SLOW
def fetch_all_sync():
    results = []
    for url in urls:
        response = requests.get(url)  # Blocks until response
        results.append(response.json())
    return results  # Total time = sum of all requests

# Asynchronous (non-blocking) — FAST
async def fetch_all_async():
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)  # All run concurrently
        return [r.json() for r in responses]
    # Total time = max of all requests

# FastAPI leverages this natively
@app.get("/generate")
async def generate_terraform(request: GenerateRequest):
    # Bedrock call is I/O-bound — async doesn't block the server
    result = await bedrock_client.invoke_model_async(request.prompt)
    return result
```

**Key insight:** `await` doesn't make things sequential — `asyncio.gather()` runs multiple awaitables concurrently (like Promise.all in JS).

---

## FastAPI vs Flask vs Django ⭐

| Feature | FastAPI | Flask | Django |
|---------|---------|-------|--------|
| Speed | Very fast | Moderate | Slower |
| Async | Native | Extension needed | Limited (3.1+) |
| Validation | Pydantic (automatic) | Manual | Django Forms/DRF |
| API Docs | Auto Swagger/ReDoc | Manual | DRF provides |
| ORM | None (bring your own) | SQLAlchemy | Built-in (powerful) |
| Admin panel | None | None | Built-in |
| Learning curve | Low | Very low | High |
| Best for | APIs, microservices | Simple apps, prototypes | Full web apps, admin |

**Why FastAPI for CloudCanva:**
```python
# 1. Async Bedrock calls (5-10 second responses don't block other users)
# 2. Automatic validation of complex resource configurations
# 3. Auto-generated API docs for 15+ endpoints
# 4. Native Python ecosystem (Jinja2, boto3)
```

---

## Dependency Injection in FastAPI

```python
from fastapi import Depends, HTTPException

# Database session dependency
async def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Auth dependency
async def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401)
    except JWTError:
        raise HTTPException(status_code=401)
    user = await get_user(user_id)
    return user

# Role-based dependency
def require_role(required_role: str):
    def dependency(user = Depends(get_current_user)):
        if user.role != required_role:
            raise HTTPException(status_code=403, detail="Insufficient permissions")
        return user
    return dependency

# Using dependencies in routes
@app.post("/generate-terraform")
async def generate(
    request: GenerateRequest,
    user = Depends(get_current_user),  # Must be authenticated
    db = Depends(get_db)               # Gets DB session
):
    # user and db are automatically injected
    return await generate_terraform(request, user, db)

@app.delete("/projects/{project_id}")
async def delete_project(
    project_id: str,
    user = Depends(require_role("admin"))  # Must be admin
):
    ...
```

---

## Pydantic Models and Validation 🔥

```python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from enum import Enum

class ResourceType(str, Enum):
    EC2 = "ec2"
    VPC = "vpc"
    S3 = "s3"
    LAMBDA = "lambda"
    RDS = "rds"

class ResourceConfig(BaseModel):
    name: str = Field(..., min_length=1, max_length=64, pattern=r'^[a-zA-Z][a-zA-Z0-9-]*$')
    type: ResourceType
    region: str = Field(default="us-east-1")
    properties: dict
    connections: List[str] = []
    
    @validator('name')
    def validate_name(cls, v):
        if v.startswith('aws-'):
            raise ValueError("Resource name cannot start with 'aws-'")
        return v

class GenerateRequest(BaseModel):
    prompt: str = Field(..., min_length=10, max_length=2000)
    resources: Optional[List[ResourceConfig]] = None
    include_iam: bool = True
    include_security_groups: bool = True

# FastAPI automatically validates request body
@app.post("/api/generate")
async def generate(request: GenerateRequest):
    # If validation fails, FastAPI returns 422 with detailed error message
    # e.g., "name must match pattern '^[a-zA-Z][a-zA-Z0-9-]*$'"
    return await process_generation(request)
```

**Benefits:**
- Automatic request validation (no manual if-else checking)
- Type coercion (string "123" → int 123 if field type is int)
- Auto-generated OpenAPI schema
- Serialization/deserialization

---

## My 15+ Endpoints Architecture

```python
# CloudCanva API Structure

# --- Project Management ---
@app.post("/api/projects")              # Create new project
@app.get("/api/projects")               # List user's projects
@app.get("/api/projects/{id}")          # Get specific project
@app.put("/api/projects/{id}")          # Update project (save canvas)
@app.delete("/api/projects/{id}")       # Delete project

# --- AI Generation ---
@app.post("/api/generate")              # NL → infrastructure config
@app.post("/api/generate/validate")     # Validate AI output
@app.post("/api/generate/refine")       # Refine existing config with NL

# --- Terraform ---
@app.post("/api/terraform/render")      # Render templates for resources
@app.post("/api/terraform/export")      # Generate ZIP bundle
@app.get("/api/terraform/download/{id}") # Download exported ZIP (presigned URL)

# --- IAM ---
@app.post("/api/iam/generate")          # Generate IAM policies for resources
@app.post("/api/iam/simulate")          # Simulate permissions

# --- Quota ---
@app.get("/api/quotas/static/{resource_type}")   # Get default limits
@app.post("/api/quotas/live")                     # Check live account quotas

# --- Templates ---
@app.get("/api/templates")              # List available resource templates
@app.get("/api/templates/{type}")       # Get specific template schema
```

---

## Error Handling in FastAPI

```python
from fastapi import HTTPException
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

# Custom exception
class BedrockTimeoutError(Exception):
    def __init__(self, message="AI generation timed out"):
        self.message = message

# Exception handler
@app.exception_handler(BedrockTimeoutError)
async def bedrock_timeout_handler(request, exc):
    return JSONResponse(
        status_code=504,
        content={"error": exc.message, "suggestion": "Try a simpler description"}
    )

# Validation error handler (prettier messages)
@app.exception_handler(RequestValidationError)
async def validation_handler(request, exc):
    errors = []
    for error in exc.errors():
        field = " → ".join(str(loc) for loc in error["loc"])
        errors.append({"field": field, "message": error["msg"]})
    return JSONResponse(status_code=422, content={"errors": errors})

# In route handlers
@app.post("/api/generate")
async def generate(request: GenerateRequest):
    try:
        result = await invoke_bedrock(request.prompt)
    except TimeoutError:
        raise BedrockTimeoutError()
    except BotoCoreError as e:
        raise HTTPException(status_code=502, detail=f"AWS service error: {str(e)}")
    
    if not validate_output(result):
        raise HTTPException(status_code=500, detail="AI generated invalid configuration")
    
    return result
```

---

## Python Decorators

```python
import functools
import time

# Basic decorator
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

@timer
def generate_terraform(resources):
    # ... expensive operation
    pass

# Decorator with arguments
def retry(max_attempts=3, delay=1):
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    await asyncio.sleep(delay * (2 ** attempt))  # Exponential backoff
        return wrapper
    return decorator

@retry(max_attempts=3, delay=1)
async def invoke_bedrock(prompt):
    # May fail due to rate limiting — retry with backoff
    return await bedrock_client.invoke_model(prompt)
```

---

**📌 KEY POINTS TO REMEMBER — Section 14:**
- async/await in Python — know how asyncio.gather works
- FastAPI vs Flask vs Django — clear on when to use which
- Dependency injection — FastAPI's Depends() pattern
- Pydantic — automatic validation with descriptive errors
- Decorators — know the @functools.wraps pattern
- Error handling — custom exceptions + handlers

---

# SECTION 15: SITUATIONAL & SCENARIO QUESTIONS

## ⚠️ Q: "Your Bedrock API is returning wrong infrastructure configs in production. How do you debug?"

**Answer:**

1. **Immediate triage (first 5 minutes):**
   - Check if it's all users or specific cases
   - Look at recent deployments — did we change the prompt?
   - Check Bedrock service health dashboard

2. **Isolate the issue:**
   - Log the exact prompts being sent and responses received
   - Compare a failing case against a known-working case
   - Is it the AI output that's wrong, or our post-processing?

3. **Root cause analysis:**
   - If AI output changed → check if model version was updated by AWS
   - If post-processing broke → check recent code changes
   - If intermittent → might be temperature/sampling randomness

4. **Mitigation:**
   - If critical: fall back to template-only mode (disable AI, show manual canvas)
   - If partial: increase post-processing strictness (reject and retry more aggressively)
   - Log all failures for pattern analysis

5. **Long-term fix:**
   - Add regression tests with known good prompt-response pairs
   - Pin model version if possible
   - Add monitoring/alerting on AI output validation failure rate

---

## ⚠️ Q: "A user reports IntelliQuery executed a DROP TABLE despite your guardrails. What do you do?"

**Answer:**

1. **Immediate response:**
   - Acknowledge the severity — this is a critical security bug
   - Check audit logs: what query was generated, what user, what workspace?
   - Determine if data is recoverable (backups, point-in-time recovery)

2. **Investigation:**
   - How did it bypass safety_validator node? Possible causes:
     - SQL injection through crafted natural language
     - MongoDB query that translates to dropCollection (not caught by SQL parser)
     - Edge case in sqlparse: `DROP/* comment */TABLE`
     - Pipeline skipped validation node due to a bug

3. **Immediate fix:**
   - Add a database-level safeguard: connect with a READ-ONLY user by default
   - Only use write-capable connection for explicitly confirmed write operations
   - This is defense-in-depth — even if app-level fails, DB-level blocks it

4. **Preventive measures:**
   - Add integration tests with adversarial NL inputs
   - Implement a "dry run" mode that EXPLAINs queries before executing
   - Add a secondary validation after LLM and before execution (belt + suspenders)

---

## ⚠️ Q: "TripSync's hotel agent is failing but others work. How do you handle this gracefully?"

**Answer:**

```javascript
// Graceful degradation pattern
async function orchestrateTrip(request) {
  const results = await Promise.allSettled([
    weatherAgent.fetch(request),
    flightAgent.search(request),
    hotelAgent.search(request),    // This one is failing
    eventsAgent.discover(request),
    routeAgent.plan(request)
  ]);
  
  // Process results — some may have failed
  const processedResults = results.map((result, index) => {
    if (result.status === 'fulfilled') return result.value;
    // Graceful fallback for failed agents
    return {
      agent: agentNames[index],
      status: 'unavailable',
      message: 'This data is temporarily unavailable',
      fallback: getFallbackData(agentNames[index], request)
    };
  });
  
  // Generate itinerary with available data
  // Mark hotel section as "suggestions pending"
  return generatePartialItinerary(processedResults);
}
```

**Key principles:**
- Use `Promise.allSettled` not `Promise.all` (all-or-nothing fails everything)
- Show partial results with clear indication of what's missing
- Provide fallback suggestions or cached data
- Retry failed agent in background, notify user when available
- Never show a blank page because one microservice is down

---

## 🔥 Q: "How would you scale CloudCanva to 10,000 concurrent users?"

**Answer:**

**Current bottlenecks at scale:**
1. Bedrock API rate limits (~50 requests/second)
2. Single FastAPI server (~1000 concurrent)
3. DynamoDB read/write capacity
4. Terraform generation (CPU-intensive for 19 templates)

**Solutions:**

| Bottleneck | Solution |
|-----------|----------|
| API server | Auto-scaling ECS Fargate (horizontal scaling) |
| Bedrock rate limit | SQS queue + worker pool + response caching |
| DynamoDB | On-demand capacity mode (auto-scales) |
| Terraform generation | Pre-computed template caching, async generation |
| Frontend | CloudFront CDN for static assets |

**Architecture change:**
- Add Redis cache: same NL description → same output (80% cache hit for common patterns)
- Add SQS queue: decouple generation requests from Bedrock calls
- Add WebSocket: notify users when generation completes (instead of polling)
- Rate limit per-user: max 10 generations/minute to prevent abuse

---

## Q: "A teammate disagrees with your architectural decision. How do you handle it?"

**Answer:**

"I'd take it as an opportunity, not a threat. In the IntelliQuery project, my teammate disagreed with my choice of LangGraph over a simpler LangChain approach. Here's what I did:

1. **Listen first:** Understood their concern — LangGraph was newer, less documented, riskier
2. **Acknowledge valid points:** They were right that it added complexity
3. **Use data, not opinions:** Built a quick prototype with both approaches (2 days)
4. **Let results speak:** The prototype showed LangGraph handled our edge cases (intent confirmation, safety blocking) much more cleanly
5. **Compromise where possible:** We used LangChain for the simple embedding/retrieval parts where a graph wasn't needed

If the data wasn't conclusive, I'd defer to the team's consensus and commit fully. The worst outcome is a team split between two architectures."

---

## Q: "You have 2 days deadline and 5 days of work. What do you do?"

**Answer:**

1. **Prioritize ruthlessly:** Identify the 20% of features that deliver 80% of value
2. **Communicate immediately:** Tell stakeholders "Here's what I can deliver by deadline, here's what needs more time"
3. **Cut scope, not quality:** Better to deliver 3 features well than 5 features broken
4. **Identify parallelizable work:** Can any tasks be delegated or done simultaneously?
5. **Remove blockers:** Skip nice-to-have polish, focus on functional completeness

**CloudCanva example:** If I had 2 days to demo:
- Day 1: Get NL → Terraform → Export working (core value)
- Day 2: Add security validation + polish demo flow
- Defer: Architecture preview, quota checking, advanced templates

---

## Q: "How would you add caching to IntelliQuery?"

**Answer:**

Three caching layers:

```python
# Layer 1: Schema Cache (refreshes every 5 minutes)
# Database schema rarely changes — cache it
@cache(ttl=300)  # 5 minutes
async def get_schema(connection_id):
    return await fetch_database_schema(connection_id)

# Layer 2: Query Result Cache (for repeated queries)
# Same NL input + same schema = same query
cache_key = hash(f"{nl_input}:{schema_version}")
cached_result = await redis.get(cache_key)
if cached_result:
    return json.loads(cached_result)

# Layer 3: Visualization Cache
# Same data shape → same chart type decision
# No need to re-analyze column types every time
```

**Cache invalidation:**
- Schema cache: Invalidate on DDL operations (CREATE/ALTER/DROP)
- Query cache: TTL-based (results may become stale)
- Don't cache destructive or write operations

---

## Q: "How would you make CloudCanva's Terraform generation more accurate?"

**Answer:**

1. **Fine-tuned prompts per resource type:** Instead of one generic prompt, have specialized prompts for networking, compute, storage, and serverless
2. **Few-shot examples:** Include 3-5 examples of correct output in the prompt
3. **Retrieval-Augmented Generation (RAG):** Feed relevant Terraform documentation snippets based on the user's request
4. **Feedback loop:** Log cases where post-processing had to fix output → use these as training signal for prompt improvement
5. **Constrained decoding:** Limit output to valid JSON structure using response format enforcement
6. **Version-specific validation:** Validate against specific AWS provider version's resource schemas

---

**📌 KEY POINTS TO REMEMBER — Section 15:**
- Always start with immediate triage, then root cause, then long-term fix
- Show you think about user impact first
- Demonstrate defense-in-depth thinking (multiple safety layers)
- Show graceful degradation (partial failure ≠ total failure)
- Be honest about trade-offs in time-constrained situations
- Connect technical solutions to business impact

---

# SECTION 16: QUESTIONS TO ASK THE INTERVIEWER

## For Amazon (Full-time SDE / Internship Continuation)

1. "What does a typical day look like for an SDE on this team? How much time is spent on new development vs. operational work?"
2. "How does the team balance between shipping quickly and maintaining quality? What's the review process like?"
3. "What's the most impactful project this team has shipped in the last year?"
4. "How does on-call rotation work for this team? What's the typical severity of issues?"
5. "What does career growth look like for SDEs here? How do people progress from SDE-1 to SDE-2?"

## For Product-Based Companies (Google, Microsoft, etc.)

6. "What's the team's approach to technical debt? Is there dedicated time for it?"
7. "How are engineering decisions made on the team? Is there a tech lead, or is it collaborative?"
8. "What's the most challenging engineering problem the team is currently facing?"
9. "How does the team measure success? What metrics matter most?"
10. "What development methodology does the team follow? How long are sprint cycles?"

## For Startups

11. "What's the current tech stack, and are there plans to evolve it? What drove the current choices?"
12. "How many engineers are on the team, and how do you handle code ownership?"
13. "What does the path to production look like? How fast can a new feature go from idea to users?"
14. "What's the biggest technical challenge the company faces in the next 6 months?"
15. "How do you balance feature development with infrastructure and scalability concerns at this stage?"

---

# SECTION 17: CODING INTERVIEW STRATEGY

## How to Approach a Problem You've Never Seen 🔥

**Framework (5-step process):**

```
1. UNDERSTAND (2-3 minutes)
   - Repeat the problem in your own words
   - Ask clarifying questions:
     • Input size/constraints?
     • Can there be duplicates/negatives?
     • What should I return for edge cases?
     • Is the input sorted?
   
2. EXAMPLES (2 minutes)
   - Work through 1-2 examples by hand
   - Include an edge case (empty input, single element)
   - Verify your understanding matches expected output

3. APPROACH (3-5 minutes)
   - Start with brute force (state it, don't code it)
   - Identify what's slow about brute force
   - Think: "What data structure gives me the operation I need cheaply?"
   - State your optimized approach and complexity
   
4. CODE (10-15 minutes)
   - Write clean, readable code
   - Use meaningful variable names
   - Talk through your logic as you write
   - Handle edge cases explicitly

5. VERIFY (3-5 minutes)
   - Trace through your code with the examples
   - Check edge cases
   - Verify time and space complexity
```

---

## How to Think Out Loud Effectively

**DO:**
- "I'm thinking of using a HashMap here because I need O(1) lookups..."
- "The brute force would be O(n²) because of the nested loop. Can I do better?"
- "I notice this has overlapping subproblems, so DP might work..."
- "Let me consider the edge case where the array is empty..."

**DON'T:**
- Silent coding for 10 minutes
- "I think this works..." (then move on without verifying)
- Jumping straight to code without discussing approach

---

## How to Handle Hints from the Interviewer

- Hints are NOT failure — interviewers want to see how you incorporate new information
- When you get a hint: pause, think about how it changes your approach, acknowledge it
- "Oh, that's a good point — if the array is sorted, I can use binary search instead of linear scan, bringing it from O(n) to O(log n)"
- Don't dismiss hints or get defensive

---

## Time Management in Coding Interviews

| Phase | Time | What to do |
|-------|------|------------|
| Understand + clarify | 3-5 min | Questions, examples |
| Design approach | 3-5 min | Brute force → optimize |
| Code | 15-20 min | Clean implementation |
| Test + debug | 5 min | Trace through, fix bugs |
| Discuss complexity | 2 min | Time and space |

**Total: ~30-35 minutes per problem** (typical interview has 1-2 problems in 45 min)

---

## Common Mistakes to Avoid

1. **Not asking clarifying questions** → assumptions lead to wrong solutions
2. **Jumping to code immediately** → hard to restructure mid-coding
3. **Over-engineering** → using Segment Tree when a simple loop works
4. **Ignoring edge cases** → empty array, single element, all same values
5. **Not testing your code** → trace through at least one example
6. **Silent coding** → interviewer can't evaluate your thinking
7. **Giving up too quickly** → take a step back, think of related problems
8. **Not knowing complexity** → always state Time and Space

---

# SECTION 18: SELF INTRODUCTION SCRIPTS

## 1. 30-Second Elevator Pitch

"I'm Khushi Verma, a CS student at MNNIT Allahabad, currently interning at Amazon as an AWS Support Engineering Intern. I built CloudCanva — an AI-powered platform that lets developers design AWS infrastructure visually and export production-ready Terraform code. I work at the intersection of AI and developer tools, and I love making complex systems accessible."

---

## 2. 2-Minute Introduction for Technical Interviews

"Hi, I'm Khushi Verma. I'm a pre-final year B.Tech student in Computer Science at MNNIT Allahabad with a CPI of 8.54.

I'm currently working as an AWS Support Engineering Intern at Amazon in Bangalore, where I built CloudCanva — an AI-powered platform for visual AWS infrastructure design. It integrates Amazon Bedrock for natural language workflows, generates validated Terraform code through 19 templates, and includes security guardrails like IAM least-privilege generation and quota validation. The tech stack is Next.js, React Flow, FastAPI, and several AWS services.

Before Amazon, I built two projects I'm particularly proud of. TripSync is a multi-agent travel planning platform where five agents run in parallel to plan complete trips. IntelliQuery is a natural language to database query platform with a 7-node LangGraph pipeline that includes destructive query blocking and RBAC.

On the competitive programming side, my LeetCode rating is 1759, and I was selected as an AFE-FFE Scholar among 40,000+ applicants. I'm passionate about building intelligent systems that simplify complex workflows."

---

## 3. 3-Minute Introduction for HR Rounds

*[Add to the 2-minute version:]*

"...What drives me is the challenge of making complex things simple. At Amazon, I saw developers spending hours configuring infrastructure manually. CloudCanva reduces that to minutes through AI — you describe what you want, and the system generates secure, validated code.

I'm also someone who learns quickly and goes deep. I picked up Amazon Bedrock, Terraform templating, and IAM modeling within weeks of starting my internship and built production-quality features with them.

Outside of coding, I enjoy problem-solving — both in competitive programming and in designing system architectures. The Smart India Hackathon, where our team secured 4th position, was a great experience in thinking on my feet under time pressure.

I'm looking for a role where I can continue building impactful products at the intersection of AI and developer experience."

---

## 4. Introduction Highlighting Amazon Internship

"I'm Khushi Verma, an AWS Support Engineering Intern at Amazon. My project, CloudCanva, solves a real pain point — AWS infrastructure design is complex, error-prone, and requires deep expertise. CloudCanva makes it accessible through AI.

The platform supports two workflows: visual drag-and-drop using React Flow, and natural language where users describe what they want and the system generates it. Under the hood, I integrated Amazon Bedrock with Claude Sonnet 4, built an 18-rule prompt engineering system for accuracy, and created a FastAPI backend with 15+ endpoints serving 19 Terraform templates.

What I'm most proud of is the security layer — IAM policy generation following least privilege, quota validation against actual account limits, and guardrails that prevent insecure configurations. Because making infrastructure generation easy without making it safe would be irresponsible.

The experience taught me to own problems end-to-end, deliver under ambiguity, and build systems where correctness matters."

---

## 5. Introduction for Startups

"Hi, I'm Khushi Verma. I build full-stack AI-powered products — from idea to deployment. At Amazon, I single-handedly built CloudCanva, an infrastructure design platform, using Next.js, FastAPI, and Amazon Bedrock. Before that, I built TripSync (multi-agent travel planning) and IntelliQuery (NL-to-database with security guardrails).

What makes me different: I don't just code features — I think about the whole product. Security, UX, performance, and maintainability. My LeetCode rating of 1759 reflects strong fundamentals, and my project portfolio shows I can deliver end-to-end independently.

I'm looking for a high-ownership role where I can make architectural decisions and see direct user impact."

---

# SECTION 19: WEAKNESSES & GAP AREAS

## Topics You Might Be Weak In (based on resume analysis)

| Area | Gap | How to Address |
|------|-----|----------------|
| **System Design at massive scale** | Projects are single-server; haven't dealt with millions of users | Study distributed systems: sharding, replication, consensus |
| **Testing** | No mention of test coverage, TDD, or testing frameworks | Learn Jest, Pytest, write unit tests for your projects |
| **CI/CD & DevOps** | No mention of deployment pipelines, Docker, Kubernetes | Learn basic Docker, understand pipeline concepts |
| **SQL deeply** | Resume says MongoDB primarily; SQL joins/optimization? | Practice complex SQL queries, window functions |
| **Low-level system knowledge** | High-level frameworks focus; how does the runtime work? | Study V8 engine, Python GIL, memory management |
| **Design patterns in depth** | Not explicitly mentioned in projects | Review GoF patterns, apply to your existing code |

## Questions Designed to Catch Gaps

- "What testing strategy did you use for CloudCanva?" → Be honest: mention what you tested informally, then discuss what you'd add (unit tests for post-processing, integration tests for API endpoints)
- "How would you deploy CloudCanva to production with zero downtime?" → Discuss blue-green deployment, even if you haven't done it
- "What happens if DynamoDB goes down?" → Discuss disaster recovery, even if you haven't implemented it

## How to Answer What You Don't Know

**Template:** "I haven't worked with [X] directly, but here's my understanding based on [related experience]. I'd approach learning it by [specific action]."

**Example:** "I haven't set up Kubernetes in production, but I understand the concepts — pods, services, deployments — from reading about container orchestration. My experience deploying on ECS/Fargate gives me the containerization foundation. I'd start by deploying one of my existing projects to a local K8s cluster using minikube."

---

# SECTION 20: MOCK INTERVIEW SIMULATIONS

## MOCK INTERVIEW 1: Amazon SDE Style

### Round 1: Leadership Principle (15 min)

**Interviewer:** "Tell me about a time you had to deliver a project with significant ambiguity."

**Your Answer (STAR):**

*Situation:* "When I started my Amazon internship, the project brief was vague — 'build a tool that helps with AWS infrastructure design.' There was no detailed spec, no wireframes, no architecture document."

*Task:* "I needed to define the scope, choose technologies, and deliver a working product independently within the internship timeline."

*Action:* "I spent my first 3 days researching the problem space: What tools exist? Where do developers struggle? I identified that the gap was between understanding what you need and actually writing the infrastructure code. I proposed CloudCanva with a visual canvas + AI approach, got approval, and created my own milestones. Each week had a deliverable, so if scope needed trimming, we could cut from the end, not the middle."

*Result:* "Delivered a complete platform with 15+ endpoints, 19 templates, AI integration, and security guardrails. The key was converting ambiguity into a structured plan quickly, then executing against it."

**Follow-up:** "What would you do differently?"

"I'd invest more time in user research upfront. I made assumptions about what 'non-expert developers' need based on my own experience. Talking to 5-10 actual AWS beginners would have validated or changed my feature priorities."

---

### Round 2: Coding (30 min)

**Problem:** Implement an LRU Cache with `get(key)` and `put(key, value)` operations in O(1) time.

**Approach:**
- HashMap for O(1) key lookup
- Doubly Linked List for O(1) removal and insertion
- Most recently used at head, least recently used at tail

*[See LRU Cache solution in Section 6]*

**Follow-up:** "How would you make this thread-safe?"
- Use `ConcurrentHashMap` + `ReentrantReadWriteLock`
- Read operations acquire read lock (multiple concurrent reads OK)
- Write operations acquire write lock (exclusive)
- Or use lock-free data structures (CAS operations)

---

### Round 3: Project Deep Dive (15 min)

**Interviewer:** "Walk me through how a user goes from typing 'I need a web app with a load balancer and database' to downloading Terraform code."

**Your walkthrough:**
1. User types in NL input field → frontend sends POST to `/api/generate`
2. FastAPI receives request, validates with Pydantic model
3. Constructs prompt with system context (18 rules) + user input
4. Calls Amazon Bedrock API (Claude Sonnet 4) → waits for response
5. Receives JSON with resource list (ALB, EC2s, RDS, VPC, subnets, SGs)
6. 18-rule post-processing validates: cross-references, required fields, security
7. If valid → returns to frontend; if invalid → regenerates (max 3 attempts)
8. Frontend renders resources as nodes on React Flow canvas with edges showing connections
9. User clicks "Export" → POST to `/api/terraform/export`
10. Backend: for each resource → select Jinja2 template → render with config values
11. Bundle into main.tf + variables.tf + outputs.tf + provider.tf → ZIP
12. Upload ZIP to S3 → generate presigned URL (1-hour expiry)
13. Return URL to frontend → browser downloads ZIP

**Follow-up:** "What if Bedrock takes 30 seconds to respond?"
- Set a 15-second timeout on the Bedrock call
- If exceeded, return partial response: "Generation is taking longer than expected"
- Move to async: return a job ID, client polls for completion
- Queue-based: put in SQS, worker processes, notify via WebSocket when done

---

## MOCK INTERVIEW 2: Product Company Style (Google/Microsoft)

### Problem 1 (Easy): Two Sum

*[Given sorted array, find two numbers that add to target]*
- Two pointer approach: O(n) time, O(1) space

### Problem 2 (Medium): Course Schedule II (Topological Sort)

**Problem:** Given courses and prerequisites, return the order to take all courses.

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    int[] indegree = new int[numCourses];
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    
    for (int[] pre : prerequisites) {
        adj.get(pre[1]).add(pre[0]);
        indegree[pre[0]]++;
    }
    
    Queue<Integer> queue = new LinkedList<>();
    for (int i = 0; i < numCourses; i++)
        if (indegree[i] == 0) queue.add(i);
    
    int[] order = new int[numCourses];
    int idx = 0;
    while (!queue.isEmpty()) {
        int course = queue.poll();
        order[idx++] = course;
        for (int next : adj.get(course))
            if (--indegree[next] == 0) queue.add(next);
    }
    return idx == numCourses ? order : new int[0]; // empty if cycle
}
// Time: O(V + E), Space: O(V + E)
```

### OOP Design: Design a parking lot system

```java
enum VehicleSize { SMALL, MEDIUM, LARGE }

class Vehicle {
    String licensePlate;
    VehicleSize size;
}

class ParkingSpot {
    int id;
    VehicleSize size;
    boolean occupied;
    Vehicle vehicle;
    
    boolean canFit(Vehicle v) { return !occupied && v.size.ordinal() <= this.size.ordinal(); }
    void park(Vehicle v) { this.vehicle = v; this.occupied = true; }
    void unpark() { this.vehicle = null; this.occupied = false; }
}

class ParkingLot {
    List<ParkingSpot> spots;
    Map<String, ParkingSpot> vehicleMap; // licensePlate → spot
    
    ParkingSpot findSpot(Vehicle v) {
        return spots.stream()
            .filter(s -> s.canFit(v))
            .min(Comparator.comparingInt(s -> s.size.ordinal())) // smallest available
            .orElse(null);
    }
    
    boolean park(Vehicle v) {
        ParkingSpot spot = findSpot(v);
        if (spot == null) return false;
        spot.park(v);
        vehicleMap.put(v.licensePlate, spot);
        return true;
    }
    
    boolean unpark(String licensePlate) {
        ParkingSpot spot = vehicleMap.get(licensePlate);
        if (spot == null) return false;
        spot.unpark();
        vehicleMap.remove(licensePlate);
        return true;
    }
}
```

---

## MOCK INTERVIEW 3: Startup Style

### System Design: Design a simple notification system

**Requirements:**
- Send push notifications, emails, and SMS
- Support scheduling (send at specific time)
- Handle user preferences (opt-in/out per channel)

**High-level design:**
```
API → Message Queue → Notification Workers → Channel Dispatchers
                                              ├── Push (FCM/APNs)
                                              ├── Email (SES)
                                              └── SMS (SNS)

User Preferences DB ← checked before dispatching
Scheduler Service ← for delayed/scheduled notifications
```

### Coding: Implement a function that groups anagrams

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Time: O(N * K log K) where N = num strings, K = max string length
// Space: O(N * K)
```

---

**📌 KEY POINTS TO REMEMBER — Section 20:**
- Practice full mock interviews with timer (not just individual problems)
- In project deep dives, walk through the ENTIRE data flow
- For behavioral: STAR format with specific numbers and outcomes
- For coding: talk through approach BEFORE coding
- For system design: start with requirements, then high-level, then deep dive
- Ask clarifying questions even in mock interviews — build the habit

---

## 🏆 FINAL PREPARATION CHECKLIST

- [ ] Practice self-introduction out loud (record yourself)
- [ ] Do 2-3 LeetCode problems daily (mix of Easy + Medium)
- [ ] Deep dive each project: be able to explain any component for 5 minutes
- [ ] Review all 16 Amazon Leadership Principles with YOUR stories
- [ ] Practice STAR format until it's natural
- [ ] Review time/space complexity for all major data structures
- [ ] Understand your tech stack at a level deeper than you built it
- [ ] Prepare questions for the interviewer
- [ ] Get comfortable writing code on a whiteboard/blank screen
- [ ] Mock interview with a friend at least twice

---

*Generated for Khushi Verma | Interview Preparation Guide | 2026*
