# IntelliQuery — Complete Technical Deep Dive & Interview Preparation Guide

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Tech Stack Deep Dive](#3-tech-stack-deep-dive)
4. [Two-Phase Query Pipeline](#4-two-phase-query-pipeline)
5. [Backend Architecture (FastAPI)](#5-backend-architecture-fastapi)
6. [LangGraph & AI Orchestration](#6-langgraph--ai-orchestration)
7. [LangChain & Prompt Engineering](#7-langchain--prompt-engineering)
8. [Database Layer](#8-database-layer)
9. [Authentication & Authorization](#9-authentication--authorization)
10. [Frontend Architecture (React + Vite)](#10-frontend-architecture-react--vite)
11. [MCP Integration (Claude Desktop)](#11-mcp-integration-claude-desktop)
12. [Security & Safety](#12-security--safety)
13. [API Reference](#13-api-reference)
14. [State Management & Data Flow](#14-state-management--data-flow)
15. [Deployment & DevOps](#15-deployment--devops)
16. [Design Patterns Used](#16-design-patterns-used)
17. [Alternative Technologies & Why These Were Chosen](#17-alternative-technologies--why-these-were-chosen)
18. [System Design Interview Questions](#18-system-design-interview-questions)
19. [Technical Interview Questions & Answers](#19-technical-interview-questions--answers)
20. [Scalability & Performance](#20-scalability--performance)
21. [Future Enhancements](#21-future-enhancements)

---

## 1. Project Overview

### What is IntelliQuery?

IntelliQuery is a **full-stack AI-powered database querying platform** that allows users to interact with their databases using **natural language**. Instead of writing MongoDB queries or SQL statements manually, users type questions like "Show me all sci-fi movies released after 2020 sorted by rating" and the system automatically generates, validates, explains, and executes the corresponding database query.

### Core Value Proposition

| Problem | Solution |
|---------|----------|
| Non-technical users can't write database queries | Natural language interface |
| Query mistakes can damage data | Two-phase confirmation + destructive query blocking |
| Teams need to share database access | Multi-tenant workspace system with role-based permissions |
| Security of database credentials | Encrypted URI storage with Fernet encryption |
| Understanding query results | AI-generated explanations + automatic chart visualization |
| Integration with AI tools | MCP server for Claude Desktop integration |

### Key Features

- **Natural Language → Database Query**: Supports MongoDB (find, aggregate, insert, update) and SQL (SELECT)
- **Two-Phase Pipeline**: Phase 1 extracts user intent → User confirms/edits → Phase 2 generates precise query
- **Multi-Tenant Workspaces**: Team collaboration with admin/user roles and read_only/read_write permissions
- **Authentication**: Email OTP verification + Google OAuth
- **Query Safety**: Destructive operations (delete, drop, remove) are blocked at multiple levels
- **AI-Powered Visualization**: Automatic chart type selection based on query results
- **MCP Integration**: Works as a tool inside Claude Desktop via Model Context Protocol
- **Real-Time Schema Inspection**: Sidebar shows live database schema with field names
- **Query History**: Persistent chat history per user per workspace
- **CSV Export**: One-click download of results as CSV

---

## 2. High-Level Architecture

### System Architecture Diagram (Text)

```
┌────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND (React + Vite)                         │
│  ┌─────────┐  ┌────────────┐  ┌──────────┐  ┌───────────────┐             │
│  │  SignIn  │  │ Dashboard  │  │ Sidebar  │  │   ChatInput   │             │
│  └─────────┘  └────────────┘  └──────────┘  └───────┬───────┘             │
│                                                       │                     │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────────┴──────────┐         │
│  │  IntentCard   │  │ ResultsTable  │  │     ChatMessage        │         │
│  │ (Phase 1 UI)  │  │ + CSV Export  │  │ (renders all types)    │         │
│  └───────────────┘  └───────────────┘  └────────────────────────┘         │
│                                                                             │
│  ┌────────────────────┐  ┌────────────────────────────┐                    │
│  │VisualizationChart  │  │   WorkspaceSettings        │                    │
│  │(Recharts/ECharts)  │  │   (Member management)      │                    │
│  └────────────────────┘  └────────────────────────────┘                    │
└───────────────────────────────────┬─────────────────────────────────────────┘
                                    │ HTTP (Axios/fetch)
                                    ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                         BACKEND (FastAPI + Uvicorn)                          │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    API Layer (api/main.py)                         │      │
│  │  /health  /schema  /query  /extract-intent  /visualize            │      │
│  │  /install-mcp                                                     │      │
│  └────────────────────────────────────┬─────────────────────────────┘      │
│                                        │                                    │
│  ┌─────────────────┐   ┌──────────────┴──────────────────────────┐        │
│  │  Auth Routes    │   │        Router Agent (LangGraph)           │        │
│  │  /auth/*        │   │                                          │        │
│  │  - signup       │   │  schema → relevance → intent (Phase 1)   │        │
│  │  - login        │   │  schema → query → validate → explain     │        │
│  │  - google       │   │          → execute (Phase 2)             │        │
│  │  - OTP          │   │                                          │        │
│  └─────────────────┘   └─────────┬──────────────────┬────────────┘        │
│                                    │                  │                      │
│  ┌─────────────────┐   ┌─────────┴──────┐  ┌───────┴──────────┐          │
│  │ Workspace Routes│   │ LangChain LCEL │  │  Prompt Templates │          │
│  │ /workspaces/*   │   │  Chains        │  │  (intent, query,  │          │
│  │ - CRUD          │   │ (find, agg,    │  │   explain, viz)   │          │
│  │ - members       │   │  insert, upd)  │  │                   │          │
│  │ - join code     │   └────────┬───────┘  └───────────────────┘          │
│  └─────────────────┘            │                                           │
│                                  ▼                                           │
│  ┌──────────────────────────────────────────────────────────────────┐      │
│  │                    External Services                               │      │
│  │  ┌─────────┐  ┌─────────────┐  ┌──────────┐  ┌──────────────┐  │      │
│  │  │ Groq API│  │ MongoDB     │  │ SQL DBs  │  │ SendGrid     │  │      │
│  │  │(LLama 3)│  │ (PyMongo)   │  │(SQLAlch) │  │ (Email OTP)  │  │      │
│  │  └─────────┘  └─────────────┘  └──────────┘  └──────────────┘  │      │
│  └──────────────────────────────────────────────────────────────────┘      │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│                     MCP SERVER (Claude Desktop Integration)                  │
│  mcp_server.py → FastMCP → ask_database() tool                             │
│  mcp_proxy.py → SSE async proxy for remote deployment                      │
└────────────────────────────────────────────────────────────────────────────┘
```

### Data Flow Summary

1. **User types natural language** → Frontend sends to `/extract-intent`
2. **Phase 1**: Backend runs LangGraph (schema → relevance → intent extraction)
3. **IntentCard displayed**: User reviews/edits extracted intent
4. **User clicks "Confirm & Run"** → Frontend sends to `/query` with confirmed_intent
5. **Phase 2**: Backend runs LangGraph (schema → query generation → validation → explanation → execution)
6. **Results returned**: Table + explanation + optional chart visualization

---

## 3. Tech Stack Deep Dive

### Backend Stack

| Technology | Version | Purpose | Why Chosen |
|-----------|---------|---------|------------|
| **Python** | 3.8+ | Primary backend language | Rich AI/ML ecosystem, async support |
| **FastAPI** | Latest | Web framework | Async, auto-docs, Pydantic validation, high performance |
| **Uvicorn** | Latest | ASGI server | Production-grade async server for FastAPI |
| **LangChain** | Latest | LLM framework | Chain composition (LCEL), prompt templates, output parsers |
| **LangGraph** | Latest | Multi-agent orchestration | Stateful graph workflows, conditional routing |
| **langchain_groq** | Latest | Groq LLM integration | Bridge between LangChain and Groq API |
| **Groq API** | - | LLM inference | Ultra-fast inference for LLaMA 3.1 (8B) |
| **LLaMA 3.1 8B Instant** | - | Language model | Fast, capable, open-source model |
| **PyMongo** | Latest | MongoDB driver | Official driver with connection pooling |
| **SQLAlchemy** | Latest | SQL ORM/toolkit | Multi-database support (Postgres, MySQL, SQLite) |
| **psycopg2-binary** | Latest | PostgreSQL adapter | Required by SQLAlchemy for Postgres |
| **pymysql** | Latest | MySQL adapter | Pure Python MySQL driver |
| **Pydantic** | v2 | Data validation | Request/response model validation |
| **PyJWT** | Latest | JWT tokens | Stateless authentication |
| **bcrypt** | Latest | Password hashing | Industry-standard one-way hashing |
| **cryptography (Fernet)** | Latest | Symmetric encryption | Database URI encryption at rest |
| **SendGrid** | Latest | Email service | OTP email delivery (HTTP API, no SMTP) |
| **google-auth** | Latest | Google OAuth | Server-side Google token verification |
| **python-dotenv** | Latest | Environment variables | Load .env configuration |
| **certifi** | Latest | TLS certificates | MongoDB Atlas TLS connection |
| **FastMCP** | Latest | MCP server framework | Claude Desktop integration |

### Frontend Stack

| Technology | Version | Purpose | Why Chosen |
|-----------|---------|---------|------------|
| **React** | 18.2.0 | UI framework | Component model, hooks, ecosystem |
| **Vite** | 5.0.0 | Build tool | Fast HMR, ESM-native bundling |
| **React Bootstrap** | 2.10.0 | UI components | Pre-built accessible components |
| **Bootstrap** | 5.3.0 | CSS framework | Responsive grid, utility classes |
| **Recharts** | 3.8.1 | Charts (primary) | React-native, composable, responsive |
| **ECharts** | 6.1.0 | Charts (secondary) | Rich chart types, fallback |
| **Axios** | 1.6.0 | HTTP client | Interceptors, request/response transforms |
| **react-markdown** | 10.1.0 | Markdown rendering | Display AI-generated explanations |
| **@react-oauth/google** | 0.12.2 | Google Login UI | Pre-built Google sign-in button |
| **ogl** | 1.0.11 | WebGL library | Aurora animation effects |

### Infrastructure / Services

| Service | Purpose |
|---------|---------|
| **Groq Cloud** | LLM inference (free tier available) |
| **MongoDB Atlas** | Primary application database + user databases |
| **Vercel** | Frontend deployment |
| **SendGrid** | Transactional email |
| **Google Cloud** | OAuth client credentials |

---

### Detailed Technology Explanations

#### FastAPI — Why Not Flask/Django/Express?

**FastAPI** was chosen over Flask, Django, and Express.js for several reasons:

1. **Async by Default**: FastAPI is built on Starlette (ASGI), supporting async/await natively. This matters because LLM API calls (to Groq) are I/O-bound and benefit from non-blocking execution.

2. **Automatic OpenAPI Docs**: Every endpoint gets auto-generated Swagger UI at `/docs`. No extra configuration needed.

3. **Pydantic Integration**: Request bodies are validated automatically using Python type hints. Invalid payloads return structured error responses without manual validation code.

4. **Performance**: FastAPI is one of the fastest Python frameworks, comparable to Node.js/Go in benchmarks due to ASGI.

5. **Dependency Injection**: The `Depends()` system makes middleware (like JWT auth) composable and testable.

```python
# Example: How Depends() works for auth
@app.get("/schema")
async def get_db_schema(
    x_workspace_id: str = Header(..., alias="X-Workspace-Id"),
    current_user: dict = Depends(get_current_user)  # ← Automatically validates JWT
):
```

#### LangGraph — Why Not Simple Function Calls?

**LangGraph** provides stateful, graph-based orchestration for the AI pipeline:

1. **Conditional Routing**: Different paths based on state (e.g., skip relevance check in Phase 2)
2. **State Machine Pattern**: Each node receives and returns typed state (PipelineState TypedDict)
3. **Composability**: Easy to add/remove nodes without restructuring
4. **Built-in Error Handling**: Nodes that fail don't crash the entire pipeline
5. **Visualization**: The graph can be rendered for debugging

```python
# The graph structure:
graph.add_conditional_edges("schema", after_schema, {
    "relevance": "relevance",  # Phase 1: check if query is relevant
    "query": "query"           # Phase 2: skip to query generation
})
```

#### Groq + LLaMA 3.1 8B — Why Not OpenAI/Anthropic?

1. **Speed**: Groq's custom LPU (Language Processing Unit) hardware provides inference at ~500 tokens/second — 10x faster than GPU-based inference
2. **Cost**: Free tier available; paid tier is significantly cheaper than OpenAI
3. **Open-Source Model**: LLaMA 3.1 is open-source (Meta), no vendor lock-in
4. **Sufficient Quality**: For structured JSON generation (query building), 8B parameter model is sufficient
5. **Low Latency**: Critical for interactive chat UX — users expect sub-2-second responses

#### PyMongo + Connection Pooling

```python
@lru_cache(maxsize=100)
def _get_mongo_client(uri: str) -> MongoClient:
    return MongoClient(
        uri,
        tlsCAFile=certifi.where(),      # TLS for Atlas
        serverSelectionTimeoutMS=10000,  # Fail fast
        connectTimeoutMS=10000,
    )
```

The `lru_cache` ensures only one `MongoClient` per unique URI. PyMongo's built-in connection pool (default: 100 connections) handles concurrent requests.

---

## 4. Two-Phase Query Pipeline

### Why Two Phases?

The two-phase design solves the fundamental problem of **AI hallucination in database queries**:

- **Without confirmation**: LLM might query the wrong collection, apply wrong filters, or misinterpret the user's intent → bad results or data corruption
- **With Phase 1 (Intent)**: Extract WHAT the user wants in structured form → user verifies
- **With Phase 2 (Query)**: Generate precise syntax from CONFIRMED intent → much higher accuracy

### Phase 1: Intent Extraction

```
User Query: "Show me top 5 sci-fi movies with rating above 8"
                    │
                    ▼
┌──────────────────────────────────────────┐
│           Schema Node                     │
│  - Fetches database schema               │
│  - Returns: {movies: [title, genre,      │
│              rating, year, ...]}          │
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│         Relevance Node                    │
│  - Checks if query matches schema        │
│  - If irrelevant → suggest alternatives  │
│  - If relevant → continue                │
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│         Intent Extraction Node            │
│  - LLM extracts structured intent        │
│  - Returns:                              │
│    {                                     │
│      operation: "find",                  │
│      collection: "movies",               │
│      goal: "Find top 5 sci-fi movies...",│
│      filters: [                          │
│        {field:"genre", op:"regex",       │
│         value:"sci.fi"},                 │
│        {field:"rating", op:"gt",         │
│         value: 8}                        │
│      ],                                  │
│      sort: {field:"rating",dir:"desc"},  │
│      limit: 5                            │
│    }                                     │
└──────────────────┬───────────────────────┘
                    │
                    ▼
          [PAUSE — Return to Frontend]
          IntentCard displayed to user
```

### Phase 2: Query Generation & Execution

```
User clicks "Confirm & Run" with (possibly edited) intent
                    │
                    ▼
┌──────────────────────────────────────────┐
│           Schema Node                     │
│  (re-fetches for safety, uses 5min cache)│
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│          Query Node                       │
│  - Routes to operation-specific chain    │
│  - find_chain generates:                 │
│    {                                     │
│      operation: "find",                  │
│      collection: "movies",               │
│      filter: {                           │
│        genre: {$regex:"sci.fi",$options:"i"},│
│        rating: {$gt: 8}                  │
│      },                                  │
│      sort: {rating: -1},                 │
│      limit: 5                            │
│    }                                     │
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│        Validation Node                    │
│  - Scans for forbidden words             │
│  - Blocks: delete, drop, remove,         │
│            $out, $merge                   │
│  - If unsafe → return error immediately  │
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│       Explanation Node                    │
│  - LLM explains query in plain English   │
│  - "This finds movies whose genre        │
│     contains 'sci-fi' (case-insensitive) │
│     with a rating above 8, sorted by     │
│     rating descending, limited to 5."    │
└──────────────────┬───────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────┐
│       Execution Node                      │
│  - Connects to user's database           │
│  - Executes the MongoDB/SQL query        │
│  - Sanitizes BSON types (ObjectId,       │
│    Decimal128, datetime → JSON)          │
│  - Hard cap: 200 documents maximum       │
│  - Permission check: read_only blocks    │
│    insert/update/delete                  │
└──────────────────┬───────────────────────┘
                    │
                    ▼
          [Return full response to Frontend]
          {query, explanation, result}
```

### Error & Fallback Flow

```
At any point if error occurs:
                    │
                    ▼
┌──────────────────────────────────────────┐
│        Suggestion Node                    │
│  - LLM generates 4 alternative queries   │
│  - Based on schema + error message       │
│  - Returns: ["suggestion1", ...,         │
│    "None of these (I will rephrase)"]    │
└──────────────────────────────────────────┘
```

---

## 5. Backend Architecture (FastAPI)

### Project Structure

```
api/
├── main.py              # FastAPI app, core endpoints, CORS, error handling
├── dependencies.py      # JWT auth dependency (get_current_user)
├── models/
│   └── user.py          # Pydantic request/response models
├── routes/
│   ├── auth.py          # Auth endpoints (signup, login, OTP, Google OAuth)
│   └── workspaces.py    # Workspace CRUD, member management
└── utils/
    ├── jwt_handler.py   # JWT token creation/verification
    ├── encryption.py    # Fernet URI encryption/decryption
    ├── cache_utils.py   # TTL cache decorator (thread-safe)
    ├── email_sender.py  # SendGrid email integration
    └── password_handler.py  # bcrypt hash/verify
```

### Middleware Configuration

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173", "http://localhost:3000", 
                   "https://intelliquery-five.vercel.app"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
    expose_headers=["*"],
)
```

**CORS** is configured to allow:
- Local development (Vite on port 5173)
- Alternative local port (3000)
- Production Vercel deployment

### Global Exception Handler

```python
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    traceback.print_exc()
    return JSONResponse(status_code=500, content={"detail": str(exc) or "Internal server error"})
```

This ensures no unhandled exceptions crash the server — they return structured JSON errors.

### Request/Response Models (Pydantic)

```python
class QueryRequest(BaseModel):
    query: str                                    # Natural language query
    history: Optional[List[HistoryItem]] = []     # Conversation history
    confirmed: bool = False                       # Update confirmation flag
    confirmed_intent: Optional[Any] = None        # Structured intent from IntentCard

class QueryResponse(BaseModel):
    query: Any                                    # Generated MongoDB/SQL query
    explanation: str                              # Plain-English explanation
    result: Any                                   # Query execution results
    suggestions: Optional[List[str]] = None       # Error fallback suggestions
    error: Optional[str] = None                   # Error message if failed
    requires_confirmation: Optional[bool] = None  # True for update operations
```

### Dependency Injection Pattern

The `get_current_user` dependency extracts and validates JWT from the `Authorization: Bearer <token>` header:

```python
async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    payload = verify_token(token)          # Decode JWT
    user_id = payload.get("user_id")
    db = get_db()
    user = db["users"].find_one({"_id": ObjectId(user_id)})  # Verify user exists
    user["id"] = str(user["_id"])
    return user
```

Every protected endpoint includes `current_user: dict = Depends(get_current_user)`.

### Workspace Access Verification Pattern

Every query endpoint verifies workspace membership:

```python
member = app_db["workspace_members"].find_one({
    "workspace_id": x_workspace_id,
    "user_id": current_user["id"]
})
if not member:
    raise HTTPException(status_code=403, detail="Not a member of this workspace")
permission_level = member.get("permission", "read_only")
```

---

## 6. LangGraph & AI Orchestration

### What is LangGraph?

LangGraph is a library from LangChain for building **stateful, multi-step AI workflows** as directed graphs. Each node is a function that:
1. Receives the current state (a TypedDict)
2. Performs some operation (call LLM, query DB, validate)
3. Returns updated state

Edges between nodes can be **conditional** — the next node depends on state values.

### Pipeline State Definition

```python
class PipelineState(TypedDict):
    db:                  Any              # MongoDB database object
    user_query:          str              # Raw natural language input
    history:             Optional[list]   # Conversation history
    schema:              Optional[Any]    # Database schema dict
    # Phase 1 outputs
    extracted_intent:    Optional[dict]   # Structured intent from LLM
    # Phase 2 inputs
    confirmed_intent:    Optional[dict]   # User-confirmed intent
    intent_confirmed:    Optional[bool]   # Flag: True = Phase 2 mode
    # Pipeline outputs
    query_dict:          Optional[dict]   # Generated MongoDB query
    is_valid:            Optional[bool]   # Validation result
    validation_msg:      Optional[str]    # Validation message
    explanation:         Optional[str]    # Plain-English explanation
    result:              Optional[Any]    # Execution results
    error:               Optional[str]    # Error message
    suggestions:         Optional[list]   # Fallback suggestions
    is_relevant:         Optional[bool]   # Relevance check result
    relevance_reason:    Optional[str]    # Why irrelevant
    confirmed:           Optional[bool]   # Update confirmation
    requires_confirmation: Optional[bool] # Needs user confirmation
    permission_level:    Optional[str]    # "read_only" or "read_write"
```

### Graph Construction

```python
def build_graph():
    graph = StateGraph(PipelineState)

    # Add all nodes
    graph.add_node("schema",    schema_node)
    graph.add_node("relevance", relevance_node)
    graph.add_node("intent",    intent_node)
    graph.add_node("query",     query_node)
    graph.add_node("validate",  validation_node)
    graph.add_node("explain",   explanation_node)
    graph.add_node("execute",   execution_node)
    graph.add_node("suggest",   suggestion_node)

    # Entry point
    graph.set_entry_point("schema")

    # Conditional edges
    graph.add_conditional_edges("schema", after_schema, {
        "relevance": "relevance",   # Phase 1
        "query": "query"            # Phase 2 (skip relevance)
    })
    
    graph.add_conditional_edges("relevance", after_relevance, {
        "intent":  "intent",    # Phase 1: extract intent
        "query":   "query",     # Phase 2: generate query
        "suggest": "suggest",   # Irrelevant: suggest alternatives
    })
    
    graph.add_edge("intent", END)  # Phase 1 always stops here
    
    graph.add_conditional_edges("query", after_query, {
        "validate": "validate",  # Success → validate
        "suggest":  "suggest",   # Error → suggest
        "end":      END,         # Destructive → hard stop
    })
    
    graph.add_conditional_edges("validate", after_validation, {
        "explain": "explain",    # Valid → explain
        "end":     END,          # Invalid → stop
    })
    
    graph.add_edge("explain", "execute")
    graph.add_edge("execute", END)
    graph.add_edge("suggest", END)

    return graph.compile()
```

### Conditional Edge Functions

```python
def after_schema(state: PipelineState) -> str:
    """Phase 2 skips relevance check (intent already confirmed)"""
    if state.get("intent_confirmed"):
        return "query"
    return "relevance"

def after_relevance(state: PipelineState) -> str:
    """If irrelevant → suggest, else → intent or query"""
    if not state.get("is_relevant", True):
        return "suggest"
    if state.get("intent_confirmed"):
        return "query"
    return "intent"

def after_query(state: PipelineState) -> str:
    """On error → suggest; on destructive → end; else → validate"""
    if state.get("error"):
        if state.get("is_valid") is False:
            return "end"      # Destructive — no suggestions
        return "suggest"
    return "validate"
```

### BSON Sanitization

MongoDB returns BSON types (ObjectId, Decimal128, datetime) that aren't JSON-serializable. The sanitizer recursively converts them:

```python
def _sanitize(value):
    if isinstance(value, ObjectId):    return str(value)
    if isinstance(value, Decimal128):  return float(value.to_decimal())
    if isinstance(value, datetime):    return value.isoformat()
    if isinstance(value, dict):        return {k: _sanitize(v) for k, v in value.items()}
    if isinstance(value, list):        return [_sanitize(v) for v in value]
    return value
```

---

## 7. LangChain & Prompt Engineering

### LCEL (LangChain Expression Language)

Every AI operation uses the LCEL pattern: `prompt | llm | parser`

```python
# Example: Intent extraction chain
_intent_chain = intent_prompt | get_llm(json_mode=True) | JsonOutputParser()
```

This composes three steps:
1. **prompt**: ChatPromptTemplate formats the input variables into a prompt
2. **llm**: ChatGroq calls Groq API with the formatted prompt
3. **parser**: JsonOutputParser extracts structured JSON from the response

### Prompt Templates

#### Intent Extraction Prompt (Phase 1)

The system prompt for intent extraction is carefully structured:

```
System: You are a database query intent extractor.
- Do NOT generate any MongoDB query syntax
- Just extract the intent
- Schema: {schema}
- History: {history}
- Return JSON: {operation, collection, goal, filters, projection, sort, limit, aggregate_stages}

Human: {user_query}
```

Key design decisions:
- **No MongoDB syntax in output** — prevents hallucination
- **Schema provided** — forces LLM to use real collection/field names
- **History included** — enables multi-turn queries ("now filter those by rating")
- **Strict JSON format** — JsonOutputParser can parse it reliably

#### Query Generation Prompts (Phase 2)

Four separate prompts for four operations:

**Find Chain:**
```
- Collection: {collection}
- Available fields: {schema_fields}
- Goal: {goal}
- Confirmed filters: {filters}
- Projection: {projection}
- Sort: {sort}
- Limit: {limit}
- Rules: Use $regex for text, $and for multiple filters
- Return: {operation, collection, filter, projection, sort, limit}
```

**Aggregate Chain:**
```
- Collection: {collection}
- Available fields: {schema_fields}
- Goal: {goal}
- Required stages: {aggregate_stages}
- Pre-match filters: {filters}
- Rules: Build pipeline using only listed stages
- Return: {operation, collection, pipeline}
```

**Insert Chain:**
```
- Collection: {collection}
- Available fields: {schema_fields}
- Goal: {goal}
- Rules: Use only existing fields, infer values from goal
- Return: {operation, collection, data}
```

**Update Chain:**
```
- Collection: {collection}
- Available fields: {schema_fields}
- Goal: {goal}
- Confirmed filters: {filters}
- Rules: Use $set/$inc/$push, NEVER use $out/$merge/delete/drop
- Return: {operation, collection, filter, update_data}
```

### Chain Router

```python
_CHAIN_MAP = {
    "find":      find_chain,
    "aggregate": aggregate_chain,
    "insert":    insert_chain,
    "update":    update_chain,
}

def get_query_chain(operation: str):
    chain = _CHAIN_MAP.get(operation)
    if chain is None:
        raise ValueError(f"Unsupported operation: '{operation}'")
    return chain
```

### JSON Mode

The LLM is configured with `json_mode=True` for all structured output chains:

```python
def get_llm(json_mode: bool = False):
    llm = ChatGroq(groq_api_key=GROQ_API_KEY, model_name="llama-3.1-8b-instant")
    if json_mode:
        return llm.bind(response_format={"type": "json_object"})
    return llm
```

This forces the model to output valid JSON, preventing parsing failures.

### Explanation Chain (Plain Text Mode)

The explanation node uses `json_mode=False` because it outputs prose:

```python
_explanation_chain = _explanation_prompt | get_llm(json_mode=False) | StrOutputParser()
```

Rules in the explanation prompt:
- Avoid technical jargon
- Explain $and, regex, empty filters
- Explain projection fields
- Keep 2-5 sentences

### Visualization Chain

The visualization agent decides the best chart type for query results:

```python
_viz_chain = _viz_prompt | get_llm(json_mode=True) | JsonOutputParser()
```

Output format:
```json
{
  "visualizable": true,
  "chart_type": "bar|line|scatter|pie|area",
  "x_axis": "exact_field_name",
  "y_axis": "exact_field_name",
  "title": "Chart Title"
}
```

Heuristics embedded in the prompt:
- pie: distribution/percentage, ≤7 categories
- line/area: time-series data
- scatter: both axes numeric
- bar: default for comparing quantities

---

## 8. Database Layer

### MongoDB Connection Management

```python
@lru_cache(maxsize=100)
def _get_mongo_client(uri: str) -> MongoClient:
    """Cached MongoClient factory — one client per unique URI."""
    return MongoClient(
        uri,
        tlsCAFile=certifi.where(),         # TLS certificates for Atlas
        serverSelectionTimeoutMS=10000,    # 10s timeout for server selection
        connectTimeoutMS=10000,            # 10s timeout for initial connection
    )
```

**Design Decisions:**
- `lru_cache(maxsize=100)`: Caches up to 100 MongoClient instances (one per unique URI)
- Connection pooling is handled internally by PyMongo (default pool size: 100)
- TLS is mandatory for MongoDB Atlas connections
- Timeout settings prevent hanging on unreachable servers

### Schema Extraction (MongoDB)

```python
def extract_full_schema(db, sample_size=5):
    schema = {}
    for collection_name in db.list_collection_names():
        samples = list(db[collection_name].find().limit(sample_size))
        fields = set()
        for doc in samples:
            for key in doc.keys():
                fields.add(key)
        schema[collection_name] = list(fields)
    return schema
```

**How it works:**
1. Lists all collections in the database
2. Samples 5 documents from each collection
3. Extracts all unique field names across the sample
4. Returns: `{collection_name: [field1, field2, ...]}`

**Limitation:** Only discovers fields present in the first 5 documents. Schema-less collections may have fields that don't appear in the sample.

### Schema Caching (TTL Cache)

```python
@ttl_cache(ttl_seconds=300)  # 5-minute cache
def get_schema(db):
    return extract_full_schema(db)
```

The custom TTL cache implementation is thread-safe:

```python
def ttl_cache(ttl_seconds: int = 300):
    def decorator(func):
        cache = {}
        lock = threading.Lock()
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = str(args) + str(kwargs)
            with lock:
                if key in cache:
                    result, timestamp = cache[key]
                    if time.time() - timestamp < ttl_seconds:
                        return result
                result = func(*args, **kwargs)
                cache[key] = (result, time.time())
                return result
        return wrapper
    return decorator
```

### SQL Database Layer

```python
@lru_cache(maxsize=100)
def get_sql_engine(uri: str) -> Engine:
    """Create and cache a SQLAlchemy engine."""
    return create_engine(
        uri,
        pool_pre_ping=True,   # Verify connection is alive before using
        pool_size=5,           # Keep 5 connections in pool
        max_overflow=10        # Allow up to 10 extra connections under load
    )

def execute_sql(engine: Engine, sql: str) -> list[dict]:
    """Execute raw SQL and return list of dicts."""
    with engine.connect() as conn:
        result = conn.execute(text(sql))
        cols = list(result.keys())
        return [dict(zip(cols, row)) for row in result.fetchall()]
```

### SQL Schema Extraction

```python
@ttl_cache(ttl_seconds=300)
def extract_sql_schema(engine) -> dict:
    inspector = inspect(engine)
    schema = {}
    for table_name in inspector.get_table_names():
        cols = [col["name"] for col in inspector.get_columns(table_name)]
        schema[table_name] = cols
    return schema
```

Uses SQLAlchemy's `inspect()` to introspect table structure — works with PostgreSQL, MySQL, and SQLite.

### Multi-Database Support

The system supports:

| Database | URI Format | Driver |
|----------|-----------|--------|
| MongoDB | `mongodb+srv://user:pass@cluster/dbname` | PyMongo |
| PostgreSQL | `postgresql+psycopg2://user:pass@host:5432/db` | psycopg2 |
| MySQL | `mysql+pymysql://user:pass@host:3306/db` | pymysql |
| SQLite | `sqlite:////absolute/path/to/database.db` | built-in |

### Database URI Security

Database URIs are **encrypted at rest** using Fernet symmetric encryption:

```python
# Derive key from JWT_SECRET using SHA-256
secret = os.getenv("JWT_SECRET", "super_secret_fallback_key")
key = base64.urlsafe_b64encode(hashlib.sha256(secret.encode()).digest())
fernet = Fernet(key)

def encrypt_uri(uri: str) -> str:
    return fernet.encrypt(uri.encode()).decode()

def decrypt_uri(encrypted_uri: str) -> str:
    try:
        return fernet.decrypt(encrypted_uri.encode()).decode()
    except Exception:
        # Legacy fallback: plain URIs that weren't encrypted
        if encrypted_uri.startswith(("mongodb://", "postgresql", "mysql", "sqlite")):
            return encrypted_uri
        raise ValueError("Unable to decrypt the database URI.")
```

---

## 9. Authentication & Authorization

### Authentication Flow

#### Email + Password Registration (OTP Verification)

```
1. User fills signup form (name, email, password)
2. Frontend calls POST /auth/send-signup-otp
3. Backend:
   - Checks email doesn't already exist
   - Generates 6-digit OTP
   - Stores OTP in MongoDB (with 10-minute TTL index)
   - Sends OTP via SendGrid email
4. User enters OTP from email
5. Frontend calls POST /auth/signup (with name, email, password, otp)
6. Backend:
   - Verifies OTP matches stored OTP
   - Hashes password with bcrypt (72-byte truncation)
   - Creates user document in MongoDB
   - Deletes used OTP
   - Returns JWT token + user info
```

#### Email + Password Login

```
1. User enters email + password
2. Frontend calls POST /auth/login
3. Backend:
   - Finds user by email
   - Checks if stored password is bcrypt hash ($2...) or legacy plaintext
   - Verifies password
   - Returns JWT token + user info
```

#### Google OAuth Flow

```
1. User clicks "Sign in with Google" (via @react-oauth/google)
2. Google returns credential JWT (client-side)
3. Frontend calls POST /auth/google with {token: credential}
4. Backend:
   - Verifies Google JWT using google-auth library
   - Extracts email and name from claims
   - If user exists → login
   - If user doesn't exist → auto-register (password = "OAUTH_USER")
   - Returns JWT token + user info
```

#### Password Reset Flow

```
1. User clicks "Forgot Password"
2. Frontend calls POST /auth/forgot-password with {email}
3. Backend sends OTP to email (same as signup OTP)
4. User enters OTP + new password
5. Frontend calls POST /auth/reset-password with {email, otp, new_password}
6. Backend verifies OTP, hashes new password, updates user
```

### JWT Implementation

```python
SECRET_KEY = os.getenv("JWT_SECRET", "fallback-secret-key")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=60)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def verify_token(token: str):
    return jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
```

**Token Payload:** `{user_id, email, exp}`

### Password Security

```python
def get_safe_password(pwd: str) -> bytes:
    """Bcrypt has a strict 72-byte limit."""
    return pwd.encode('utf-8')[:72]

hashed = bcrypt.hashpw(get_safe_password(password), bcrypt.gensalt())
```

**Why 72-byte truncation?** Bcrypt silently ignores bytes beyond 72. By explicitly truncating, we ensure deterministic behavior.

### Authorization — Workspace Permissions

| Role | Permission | Can Read | Can Write | Can Manage Members |
|------|-----------|----------|-----------|-------------------|
| admin | read_write | ✅ | ✅ | ✅ |
| user | read_write | ✅ | ✅ | ❌ |
| user | read_only | ✅ | ❌ | ❌ |

**Permission enforcement in execution_node:**
```python
if operation in ["insert", "update", "delete", "drop"] and state.get("permission_level") == "read_only":
    return {**state, "error": "Permission Denied: You have read-only access."}
```

### OTP Storage with TTL

```python
# TTL index auto-deletes OTP documents after 10 minutes
otps_collection.create_index("createdAt", expireAfterSeconds=600)
```

MongoDB's TTL index automatically garbage-collects expired OTPs — no cron job needed.

### Join Code System

```python
def generate_join_code(length=6):
    chars = string.ascii_uppercase + string.digits
    return ''.join(random.choice(chars) for _ in range(length))
```

- 6 characters, alphanumeric uppercase
- 36^6 = 2.17 billion possible codes
- Checked for uniqueness before assignment
- Only visible to admin

---

## 10. Frontend Architecture (React + Vite)

### Component Hierarchy

```
App.jsx (Root — state management)
├── Home.jsx (Landing page — unauthenticated)
├── SignIn.jsx (Login/Signup/OTP/Reset flows)
├── Dashboard.jsx (Workspace list + create/join)
└── [Main Chat Interface — authenticated + workspace selected]
    ├── Sidebar.jsx (Schema browser + "New Chat")
    ├── Top Navbar (Status, Settings, Profile)
    ├── Chat Area
    │   └── ChatMessage.jsx (renders different message types)
    │       ├── IntentCard.jsx (Phase 1 — editable intent review)
    │       ├── ResultsTable.jsx (Table + CSV download)
    │       └── VisualizationChart.jsx (Recharts charts)
    ├── ChatInput.jsx (Text input + Enter to send)
    ├── WorkspaceSettings.jsx (Member management modal)
    └── McpModal.jsx (Claude Desktop config)
```

### State Management

The app uses **React's built-in useState + useEffect** — no Redux or Zustand. This is appropriate because:
- State is localized (most state lives in App.jsx and flows down)
- No complex cross-component state sharing
- sessionStorage/localStorage for persistence

#### Key State Variables in App.jsx

```javascript
const [authData, setAuthData] = useState()       // {user, token} from sessionStorage
const [currentWorkspace, setCurrentWorkspace] = useState(null)  // Active workspace
const [messages, setMessages] = useState([])     // Chat history array
const [loading, setLoading] = useState(false)    // Loading indicator
const [backendStatus, setBackendStatus] = useState('checking')  // Health check
const [schema, setSchema] = useState({})         // DB schema for IntentCard
const [signInView, setSignInView] = useState(null)  // null=home, 'login', 'signup'
```

### Message Types & Rendering

```javascript
// Message objects in the messages[] array:
{ role: 'user', content: "Show me all movies" }
{ role: 'intent', extracted_intent: {...}, collections: [...], userQuery: "..." }
{ role: 'assistant', content: "explanation", data: {query, result}, userQuery: "..." }
{ role: 'confirm', content: "explanation", data: {query, confirmed_intent} }
{ role: 'error', content: "error msg", suggestions: ["..."] }
```

The `ChatMessage.jsx` component renders different UI for each role.

### Data Persistence Strategy

| Data | Storage | Scope | Lifetime |
|------|---------|-------|----------|
| Auth token + user | sessionStorage | Per tab | Until tab closes |
| Chat history | localStorage | Per user + workspace | Permanent (until cleared) |
| Schema | React state | Per session | Refetched on workspace change |

```javascript
// Persist chat history scoped to user and workspace
useEffect(() => {
    localStorage.setItem(
        `iq_history_${user.email}_ws_${currentWorkspace.id}`, 
        JSON.stringify(messages)
    )
}, [messages, user?.email, currentWorkspace?.id])
```

### IntentCard Component — The Confirmation UI

The IntentCard is the critical UX element that makes the two-phase pipeline work:

```javascript
// Editable fields:
- Operation: find | aggregate | insert | update (tab buttons)
- Collection: dropdown populated from schema
- Goal: textarea (plain English description)
- Filters: dynamic list of {field, operator, value}
  - Operators: eq, ne, gt, gte, lt, lte, regex, in, nin
  - Add/remove filter rows dynamically
- Sort: field name + direction (asc/desc)
- Limit: number input
- Aggregate Stages: toggleable chips ($match, $group, $sort, $limit, $unwind, $project, $lookup)
```

### Vite Configuration

```javascript
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',  // FastAPI backend
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
})
```

The proxy configuration means:
- Frontend: `http://localhost:5173`
- API calls: `/api/query` → proxied to `http://localhost:8000/query`
- No CORS issues in development

### ResultsTable with CSV Export

```javascript
const downloadCSV = (data) => {
    const cols = Object.keys(data[0])
    const escape = (val) => {
        const str = typeof val === 'object' ? JSON.stringify(val) : String(val ?? '')
        return `"${str.replace(/"/g, '""')}"`  // Escape quotes
    }
    const rows = data.map(row => cols.map(col => escape(row[col])).join(','))
    const csv = [cols.join(','), ...rows].join('\n')
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
    // Trigger download
    const a = document.createElement('a')
    a.href = URL.createObjectURL(blob)
    a.download = `intelliquery_results_${Date.now()}.csv`
    a.click()
}
```

### Visualization Chart Component

The VisualizationChart renders dynamically based on AI-recommended chart type:

- **Bar**: `<BarChart>` with `<Bar>` 
- **Line**: `<LineChart>` with `<Line>`
- **Scatter**: `<ScatterChart>` with `<Scatter>`
- **Pie**: `<PieChart>` with colored `<Cell>` elements
- **Area**: `<AreaChart>` with `<Area>`

Smart fallback logic handles when AI returns column names that don't match the data:
```javascript
// If AI guessed wrong columns, try to auto-fix for 2-column aggregation results
if (!cols.includes(x_axis) || !cols.includes(y_axis)) {
    if (cols.length === 2) {
        const numCol = cols.find(c => typeof resultData[0][c] === 'number')
        const strCol = cols.find(c => c !== numCol)
        if (numCol && strCol) { x_axis = strCol; y_axis = numCol }
    }
}
```

---

## 11. MCP Integration (Claude Desktop)

### What is MCP (Model Context Protocol)?

MCP is Anthropic's open protocol that allows AI assistants (like Claude Desktop) to connect to external data sources and tools. IntelliQuery exposes itself as an MCP "tool" that Claude can call.

### MCP Server Implementation

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("IntelliQuery LangGraph Engine")

@mcp.tool()
def ask_database(user_query: str) -> str:
    """
    Ask a natural language question about the database.
    Triggers Phase 1 + Phase 2 pipeline automatically.
    """
    # Phase 1: Extract Intent
    intent_result = extract_intent_pipeline(db, user_query=user_query)
    if "error" in intent_result:
        return f"❌ Pipeline Error:\n{intent_result['error']}"
    
    extracted_intent = intent_result.get("extracted_intent", {})
    
    # Phase 2: Execute with extracted intent (no user confirmation needed in MCP)
    pipeline_state = run_pipeline(db, user_query=user_query, confirmed_intent=extracted_intent)
    
    if "error" in pipeline_state:
        return f"❌ Pipeline Error:\n{pipeline_state['error']}"
    
    return f"""✅ Query Executed Successfully
--- Generated MongoDB Query ---
{json.dumps(pipeline_state['query'], indent=2)}

--- Agent Explanation ---
{pipeline_state['explanation']}

--- Execution Results ---
{json.dumps(pipeline_state['result'], indent=2, default=str)}"""
```

**Key difference from web flow:** In MCP mode, there's no IntentCard confirmation step — Phase 1 and Phase 2 run sequentially. This is because Claude (the caller) is itself an AI that can interpret results and retry.

### MCP Proxy (Remote Deployment)

```python
async def proxy(url: str, token: str):
    """Bridges Claude's stdio with remote HTTP SSE endpoint."""
    async with sse_client(url, headers={"Authorization": f"Bearer {token}"}) as (read_stream, write_stream):
        async def read_sse():    # SSE → stdout
            async for message in read_stream:
                print(message.model_dump_json(), flush=True)
        
        async def write_sse():   # stdin → SSE
            while True:
                line = await loop.run_in_executor(None, sys.stdin.readline)
                msg = JSONRPCMessage.model_validate(json.loads(line))
                await write_stream.send(msg)
        
        await asyncio.gather(read_sse(), write_sse())
```

### Installation Script

The `install_mcp.py` script:
1. Asks user for Groq API Key + MongoDB URI
2. Detects OS (Windows/macOS)
3. Finds Claude Desktop config file path
4. Locates Python virtual environment
5. Writes MCP server configuration to Claude's config JSON
6. User restarts Claude to see the `ask_database` tool

### SSE Mount in FastAPI

```python
# Mount MCP SSE transport for remote deployment
app.mount("/mcp", mcp.sse_app())
```

This allows the MCP server to be accessed remotely via Server-Sent Events at `/mcp`.

---

## 12. Security & Safety

### Multi-Layer Query Safety

IntelliQuery implements **defense in depth** with multiple safety layers:

#### Layer 1: Destructive Intent Check (Raw Query)

Before any LLM processing, the raw user query is scanned:

```python
_DESTRUCTIVE_PATTERNS = re.compile(
    r"\b(delete|drop|remove|truncate|wipe|purge|erase|clear all|destroy|"
    r"merge into|insert into|\$merge|\$out)\b",
    re.IGNORECASE
)

def check_destructive_intent(user_query: str):
    match = _DESTRUCTIVE_PATTERNS.search(user_query)
    if match:
        return False, f"⚠️ Unsafe query detected: '{match.group(1)}' operations are not permitted."
    return True, "Safe"
```

#### Layer 2: Generated Query Validation

After the LLM generates a query dict, it's scanned for forbidden words:

```python
FORBIDDEN_WORDS = ["delete", "drop", "remove", "$out", "$merge"]

def validate_query(query):
    query_str = str(query).lower()
    for word in FORBIDDEN_WORDS:
        if word in query_str:
            return False, f"⚠️ Unsafe query blocked: '{word}' operations are not permitted."
    return True, "Valid query"
```

#### Layer 3: Permission Enforcement

```python
# In execution_node:
if operation in ["insert", "update", "delete", "drop"] and permission_level == "read_only":
    return {**state, "error": "Permission Denied: You have read-only access."}
```

#### Layer 4: Result Size Cap

```python
# Hard safety cap: never return more than 200 documents
cursor = cursor.limit(200)
```

#### Layer 5: MongoDB Projection Sanitization

```python
# MongoDB cannot mix inclusion (1) and exclusion (0) except for _id
if isinstance(projection_query, dict):
    has_inclusion = any(bool(v) for k, v in projection_query.items() if k != "_id")
    if has_inclusion:
        projection_query = {
            k: v for k, v in projection_query.items()
            if bool(v) or (k == "_id" and not bool(v))
        }
```

#### Layer 6: Update Confirmation Gate

For update operations, the pipeline pauses and asks the user to confirm:

```python
if operation == "update":
    if not state.get("confirmed"):
        return {**state, "requires_confirmation": True}
```

### Credential Security

| Asset | Protection |
|-------|-----------|
| Database URIs | Fernet encryption (AES-128-CBC) |
| Passwords | bcrypt (cost factor 12, salted) |
| JWT tokens | HMAC-SHA256 signed, 60-min expiry |
| API keys | Environment variables (.env not committed) |
| OTPs | MongoDB TTL index (auto-delete after 10 min) |

### Fernet Encryption Details

```python
# Key derivation: SHA-256 hash of JWT_SECRET → 32-byte key → base64 encoded
secret = os.getenv("JWT_SECRET")
key = base64.urlsafe_b64encode(hashlib.sha256(secret.encode()).digest())
fernet = Fernet(key)  # Uses AES in CBC mode with PKCS7 padding + HMAC-SHA256
```

**Properties of Fernet:**
- Symmetric encryption (same key encrypts and decrypts)
- Includes timestamp — could support time-based expiry
- HMAC guarantees integrity — tampered ciphertext is rejected
- Key is derived from JWT_SECRET — if server changes JWT_SECRET, old encrypted URIs become unreadable

### SQL Injection Prevention

For SQL queries, the system:
1. Only allows SELECT statements (for read_only users)
2. Uses SQLAlchemy's `text()` for parameterized execution
3. The LLM generates the SQL — no user input is directly concatenated

```python
if permission_level == "read_only":
    if not sql.upper().startswith("SELECT"):
        return {"error": "Permission Denied: Only SELECT queries are allowed."}
```

**Note:** Since the LLM generates SQL, there's inherent risk if the LLM is manipulated. The permission check and SELECT-only enforcement provide a safety net.

---

## 13. API Reference

### Core Endpoints

#### `GET /health`
- **Auth**: None
- **Response**: `{"status": "ok", "message": "IntelliQuery API is running"}`

#### `GET /schema`
- **Auth**: JWT + X-Workspace-Id header
- **Response**: `{"schema": {"collection_name": ["field1", "field2", ...]}}`
- **Logic**: Verifies workspace membership → gets workspace URI → extracts schema

#### `POST /extract-intent` (Phase 1)
- **Auth**: JWT + X-Workspace-Id header
- **Body**: `{"query": "string", "history": [{"user": "...", "query": "..."}]}`
- **Success Response**: `{"extracted_intent": {operation, collection, goal, filters, ...}}`
- **Error Response**: `{"error": "string", "suggestions": ["..."]}`

#### `POST /query` (Phase 2)
- **Auth**: JWT + X-Workspace-Id header
- **Body**: 
```json
{
  "query": "original natural language query",
  "history": [{"user": "...", "query": "..."}],
  "confirmed": false,
  "confirmed_intent": {
    "operation": "find",
    "collection": "movies",
    "goal": "Find top 5 sci-fi movies",
    "filters": [{"field": "genre", "operator": "regex", "value": "sci.fi"}],
    "projection": ["title", "rating"],
    "sort": {"field": "rating", "direction": "desc"},
    "limit": 5
  }
}
```
- **Success Response**:
```json
{
  "query": {"operation": "find", "collection": "movies", "filter": {...}},
  "explanation": "This query finds movies...",
  "result": [{"title": "Interstellar", "rating": 8.6}, ...]
}
```
- **Confirmation Response**: `{"query": {...}, "explanation": "...", "result": [], "requires_confirmation": true}`
- **Error + Suggestions**: `{"query": null, "explanation": "", "result": [], "error": "...", "suggestions": [...]}`

#### `POST /visualize`
- **Auth**: None (called after query results are obtained)
- **Body**: `{"user_query": "string", "result_data": [...]}`
- **Response**: `{"visualizable": true, "chart_type": "bar", "x_axis": "genre", "y_axis": "count", "title": "..."}`

#### `POST /install-mcp`
- **Auth**: None
- **Body**: `{"mongo_uri": "mongodb+srv://..."}`
- **Response**: `{"success": true, "config_path": "/path/to/config.json"}`

### Auth Endpoints

| Endpoint | Method | Body | Response |
|----------|--------|------|----------|
| `/auth/send-signup-otp` | POST | `{email}` | `{message}` |
| `/auth/signup` | POST | `{name, email, password, otp}` | `{token, user}` |
| `/auth/login` | POST | `{email, password}` | `{token, user}` |
| `/auth/forgot-password` | POST | `{email}` | `{message}` |
| `/auth/reset-password` | POST | `{email, otp, new_password}` | `{message}` |
| `/auth/google` | POST | `{token}` (Google JWT) | `{token, user}` |

### Workspace Endpoints

| Endpoint | Method | Auth | Body | Response |
|----------|--------|------|------|----------|
| `/workspaces/` | POST | JWT | `{name, db_type, db_uri, db_name}` | `{id, name, join_code, role, permission}` |
| `/workspaces/` | GET | JWT | - | `[{id, name, join_code, role, permission, db_type}]` |
| `/workspaces/join` | POST | JWT | `{join_code}` | `{id, name, role, permission, db_type}` |
| `/workspaces/{id}/members` | GET | JWT (admin) | - | `[{user_id, name, email, role, permission}]` |
| `/workspaces/{id}/members/{uid}` | PUT | JWT (admin) | `{permission}` | `{status, message}` |
| `/workspaces/{id}` | DELETE | JWT (admin) | - | `{status, message}` |
| `/workspaces/{id}/members/{uid}` | DELETE | JWT (admin/self) | - | `{status, message}` |

---
