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

## 14. State Management & Data Flow

### Frontend Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    App.jsx (Central State)                        │
│                                                                   │
│  authData ─────────────► SignIn/Home (if null)                   │
│  currentWorkspace ─────► Dashboard (if null)                     │
│  messages[] ───────────► ChatMessage[] (mapped)                  │
│  schema {} ────────────► IntentCard (collections dropdown)       │
│  loading ──────────────► ChatInput (disabled) + typing indicator │
│                                                                   │
│  ┌──────────────────────────────────────────────────────┐       │
│  │              Message Flow                              │       │
│  │                                                        │       │
│  │  handleSend(query)                                     │       │
│  │    → add {role:'user', content:query}                 │       │
│  │    → POST /extract-intent                             │       │
│  │    → add {role:'intent', extracted_intent:{...}}      │       │
│  │                                                        │       │
│  │  handleIntentConfirm(query, confirmedIntent)          │       │
│  │    → mark intent as resolved                          │       │
│  │    → POST /query                                      │       │
│  │    → add {role:'assistant', data:{query,result,...}}   │       │
│  │                                                        │       │
│  │  handleIntentReset()                                   │       │
│  │    → remove last unresolved intent + preceding user    │       │
│  │                                                        │       │
│  │  handleConfirmUpdate(query, intent)                    │       │
│  │    → POST /query with confirmed:true                  │       │
│  │    → add {role:'assistant', ...}                      │       │
│  └──────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

### Conversation History Format for LLM

```javascript
// The last 5 successful queries are sent as history
const llmHistory = messages
    .filter(m => m.role === 'assistant' && m.data)
    .slice(-5)
    .map(m => ({ user: m.userQuery, query: JSON.stringify(m.data.query) }))
```

This enables multi-turn queries like:
- Turn 1: "Show me all movies" → generates find all
- Turn 2: "Now filter those by year > 2020" → LLM sees previous query and adds filter

### Backend State Flow (LangGraph)

```
Initial State:
{
    db: <MongoDB Database Object>,
    user_query: "Show me top 5 sci-fi movies",
    history: [{user:"...", query:"..."}],
    schema: null,                    ← filled by schema_node
    extracted_intent: null,          ← filled by intent_node (Phase 1)
    confirmed_intent: {...},         ← provided by caller (Phase 2)
    intent_confirmed: true/false,    ← determines Phase 1 vs Phase 2
    query_dict: null,               ← filled by query_node
    is_valid: null,                 ← filled by validation_node
    explanation: null,              ← filled by explanation_node
    result: null,                   ← filled by execution_node
    error: null,                    ← filled on failure
    suggestions: null,              ← filled by suggestion_node
    permission_level: "read_only"
}
```

Each node reads relevant fields, performs its operation, and returns the state with updated fields.

---

## 15. Deployment & DevOps

### Local Development Setup

```bash
# Backend
cd Project
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt

# Create .env file
echo "GROQ_API_KEY=your_key
MONGO_URI=mongodb+srv://...
JWT_SECRET=your_secret
GOOGLE_CLIENT_ID=your_client_id
SENDGRID_API_KEY=your_key
SENDGRID_FROM_EMAIL=noreply@yourdomain.com" > .env

# Start backend
uvicorn api.main:app --reload --host 0.0.0.0 --port 8000

# Frontend (separate terminal)
cd frontend
npm install
npm run dev
```

### Production Deployment Architecture

```
┌─────────────────────┐         ┌─────────────────────────┐
│   Vercel (Frontend) │ ──────► │  Backend (FastAPI)       │
│   React + Vite      │  HTTPS  │  (Railway/Render/EC2)   │
│   CDN + Edge        │         │  Uvicorn workers        │
└─────────────────────┘         └──────────┬──────────────┘
                                            │
                    ┌───────────────────────┬┴─────────────────┐
                    │                       │                   │
          ┌────────▼─────┐     ┌──────────▼────┐    ┌────────▼────────┐
          │ MongoDB Atlas│     │  Groq Cloud   │    │   SendGrid      │
          │ (App DB +    │     │  (LLM API)    │    │   (Email)       │
          │  User DBs)   │     └───────────────┘    └─────────────────┘
          └──────────────┘
```

### Frontend Deployment (Vercel)

The `frontend/vercel.json` handles SPA routing:
```json
{
  "rewrites": [{"source": "/(.*)", "destination": "/index.html"}]
}
```

Build command: `npm run build` (Vite outputs to `dist/`)

### Environment Variables Required

| Variable | Service | Required |
|----------|---------|----------|
| `GROQ_API_KEY` | Groq Cloud | Yes |
| `MONGO_URI` | MongoDB Atlas (app DB) | Yes |
| `JWT_SECRET` | Internal | Yes |
| `GOOGLE_CLIENT_ID` | Google Cloud Console | For Google OAuth |
| `SENDGRID_API_KEY` | SendGrid | For email OTP |
| `SENDGRID_FROM_EMAIL` | SendGrid | For email OTP |

### Production Considerations

1. **Multiple Uvicorn Workers**: `uvicorn api.main:app --workers 4`
2. **Gunicorn as process manager**: `gunicorn api.main:app -w 4 -k uvicorn.workers.UvicornWorker`
3. **HTTPS**: Reverse proxy (Nginx/Caddy) handles TLS termination
4. **Rate Limiting**: Should be added (not currently implemented)
5. **Logging**: Python logging module configured per-node
6. **Health Monitoring**: `/health` endpoint for uptime monitoring

---

## 16. Design Patterns Used

### 1. Chain of Responsibility (LangGraph Pipeline)

Each node in the pipeline handles one specific concern and passes state to the next:
- Schema → Relevance → Intent → Query → Validate → Explain → Execute

If any node determines the request can't proceed, it routes to a different path (suggest, end).

### 2. Strategy Pattern (Query Chain Router)

```python
_CHAIN_MAP = {
    "find":      find_chain,
    "aggregate": aggregate_chain,
    "insert":    insert_chain,
    "update":    update_chain,
}

def get_query_chain(operation: str):
    return _CHAIN_MAP.get(operation)
```

Different operations are handled by different strategies (chains), selected at runtime.

### 3. Factory Pattern (Database Connections)

```python
@lru_cache(maxsize=100)
def _get_mongo_client(uri: str) -> MongoClient:
    return MongoClient(uri, ...)
```

The factory creates and caches MongoClient instances based on URI.

### 4. Decorator Pattern (TTL Cache, Auth Dependency)

```python
@ttl_cache(ttl_seconds=300)
def get_schema(db):
    return extract_full_schema(db)
```

Cross-cutting concerns (caching, authentication) are applied via decorators without modifying core logic.

### 5. Observer Pattern (React useEffect)

```javascript
useEffect(() => {
    chatEndRef.current?.scrollIntoView({ behavior: 'smooth' })
}, [messages, loading])  // Re-runs when messages or loading change
```

### 6. Mediator Pattern (App.jsx as Central Coordinator)

App.jsx mediates all communication between components — no component talks directly to another.

### 7. Command Pattern (Message Types)

Each message in the `messages[]` array is a command object:
```javascript
{ role: 'intent', extracted_intent: {...}, userQuery: "..." }
```

The `ChatMessage` component dispatches rendering based on the `role` field.

### 8. Singleton Pattern (LLM Instance)

```python
@lru_cache(maxsize=1)
def get_llm(json_mode: bool = False):
    return ChatGroq(...)
```

Only one LLM client instance per configuration — prevents redundant connections.

### 9. Facade Pattern (Router Agent Public API)

```python
def extract_intent_pipeline(db, user_query, history):
    """Simple interface hiding complex LangGraph internals."""
    ...

def run_pipeline(db, user_query, history, confirmed, confirmed_intent, permission_level):
    """Simple interface for Phase 2."""
    ...
```

External callers don't need to know about PipelineState, nodes, or graph compilation.

### 10. Template Method Pattern (LCEL Chains)

```python
_intent_chain = intent_prompt | get_llm(json_mode=True) | JsonOutputParser()
```

The template (prompt → LLM → parser) is the same for all chains; only the prompt content varies.

---

## 17. Alternative Technologies & Why These Were Chosen

### Backend Framework Alternatives

| Alternative | Pros | Cons | Why Not Chosen |
|------------|------|------|----------------|
| **Flask** | Simple, lightweight, large ecosystem | No async, no built-in validation, no auto-docs | LLM calls are async I/O; Flask would block |
| **Django** | Batteries-included, ORM, admin panel | Heavy, opinionated, overkill for API-only app | No ORM needed (MongoDB), too much overhead |
| **Express.js (Node)** | Fast, huge NPM ecosystem | Python AI ecosystem is better (LangChain, PyMongo) | LangChain/LangGraph are Python-native |
| **Go (Gin/Fiber)** | Extremely fast, compiled | Poor AI/ML ecosystem, no LangChain equivalent | AI orchestration tools aren't available in Go |

**Winner: FastAPI** — Best combination of async performance, Python AI ecosystem access, and automatic documentation.

### LLM Provider Alternatives

| Alternative | Speed | Cost | Quality | Why Not Chosen |
|------------|-------|------|---------|----------------|
| **OpenAI GPT-4** | ~30 tok/s | $30/1M tokens | Excellent | Too slow for interactive chat, expensive |
| **OpenAI GPT-3.5** | ~80 tok/s | $2/1M tokens | Good | Still slower than Groq, cost adds up |
| **Anthropic Claude** | ~50 tok/s | $15/1M tokens | Excellent | Not optimized for speed |
| **Ollama (local)** | Varies | Free | Model-dependent | Requires GPU, not portable |
| **AWS Bedrock** | ~40 tok/s | Pay-per-use | Good | Slower, vendor lock-in |

**Winner: Groq (LLaMA 3.1 8B)** — 500+ tokens/second, free tier available, sufficient quality for structured JSON generation.

### Database Alternatives

| Alternative | Pros | Cons | Why Not Chosen (for app DB) |
|------------|------|------|------|
| **PostgreSQL** | ACID, relational, mature | Requires schema migration for evolving data | User/workspace data is document-shaped |
| **Firebase** | Real-time, easy auth | Vendor lock-in, pricing at scale | Need multi-tenant dynamic DB connections |
| **Supabase** | Postgres + auth + realtime | Limited to PostgreSQL | Need to support arbitrary user databases |
| **DynamoDB** | Scalable, serverless | Complex, expensive for development | Overkill for this use case |

**Winner: MongoDB** — Schema-less (good for evolving data), same driver for app DB and user databases, Atlas is free tier.

### Frontend Framework Alternatives

| Alternative | Pros | Cons | Why Not Chosen |
|------------|------|------|----------------|
| **Next.js** | SSR, file routing, API routes | Overkill for SPA chat app, extra complexity | No SEO needs, pure SPA is simpler |
| **Vue.js** | Simple, good DX | Smaller ecosystem than React | React has better component libraries |
| **Svelte** | Fast, no virtual DOM | Smaller ecosystem, fewer chart libraries | Recharts/Bootstrap are React-native |
| **Angular** | Enterprise, TypeScript-first | Heavy, steep learning curve | Too much boilerplate for a chat app |

**Winner: React + Vite** — Large ecosystem (Recharts, React Bootstrap, react-markdown), simple SPA architecture, fast Vite bundling.

### Chart Library Alternatives

| Alternative | Pros | Cons | Why Not Chosen |
|------------|------|------|----------------|
| **Chart.js** | Lightweight, popular | Not React-native, imperative API | Recharts is more declarative |
| **D3.js** | Maximum flexibility | Very complex, low-level | Overkill for standard chart types |
| **Plotly** | Interactive, scientific | Heavy (1MB+), complex | Too heavy for a chat app |
| **Victory** | React-native | Smaller community | Recharts has better docs |

**Winner: Recharts** — React-native, composable, declarative, responsive, good defaults. ECharts kept as secondary for flexibility.

### Auth Alternatives

| Alternative | Pros | Cons | Why Not Chosen |
|------------|------|------|----------------|
| **Auth0** | Managed, enterprise-grade | Cost, vendor lock-in | Custom auth gives full control |
| **Firebase Auth** | Free, easy | Tied to Google ecosystem | Need independent MongoDB user store |
| **Passport.js** | Flexible | Node.js only | Backend is Python |
| **Supabase Auth** | Free, Postgres-backed | Tied to Supabase | Need MongoDB as app DB |

**Winner: Custom JWT + bcrypt + OTP** — Full control, no external dependencies, free, learns well for interviews.

### Orchestration Alternatives

| Alternative | Pros | Cons | Why Not Chosen |
|------------|------|------|----------------|
| **Simple function calls** | Easy to understand | No conditional routing, hard to maintain | Pipeline needs branching logic |
| **Celery** | Task queue, distributed | Too heavy for real-time chat | Not a background task — synchronous flow |
| **CrewAI** | Multi-agent, simple | Less flexible than LangGraph | LangGraph offers finer control |
| **AutoGen** | Microsoft, multi-agent | Complex setup, conversation-heavy | Not suited for structured pipelines |

**Winner: LangGraph** — Typed state, conditional edges, composable nodes, excellent for the two-phase pipeline pattern.

---

## 18. System Design Interview Questions

### Q1: "How would you design a system that converts natural language to database queries?"

**Answer Framework:**

1. **Requirements Gathering:**
   - Functional: NL → query, multiple DB types, multi-tenant, safe execution
   - Non-functional: <2s latency, 99.9% uptime, secure credentials

2. **High-Level Design:**
   - Frontend (React) → API Gateway → Query Service → LLM Service → DB Connector
   - Two-phase approach: Intent extraction → Confirmation → Query generation

3. **Deep Dive on Query Pipeline:**
   - Schema extraction provides context to LLM
   - Structured intent avoids hallucination (LLM doesn't guess schema)
   - Validation layer blocks destructive operations
   - Execution layer handles type coercion (BSON → JSON)

4. **Scaling Considerations:**
   - LLM calls are the bottleneck (~500ms per call via Groq)
   - Schema caching (5-min TTL) reduces DB introspection load
   - Connection pooling (lru_cache) reuses DB clients
   - Stateless API → horizontal scaling with load balancer

### Q2: "How do you handle AI hallucination in database queries?"

**Answer:**

IntelliQuery uses a **multi-layer anti-hallucination strategy**:

1. **Two-Phase Pipeline**: Phase 1 extracts intent WITHOUT generating query syntax. The LLM can't hallucinate query syntax because it's not asked to produce it yet.

2. **Schema Grounding**: Both phases receive the actual database schema. The prompt says "use ONLY fields in this schema" — constraining the LLM's options.

3. **User Confirmation (IntentCard)**: Before Phase 2, the user sees and can edit:
   - Collection name (dropdown from real collections)
   - Filters (field names, operators, values)
   - Sort/limit parameters

4. **Validated Input to Phase 2**: Phase 2 receives CONFIRMED intent — not raw NL. The LLM's job narrows from "understand everything" to "convert confirmed intent to syntax."

5. **Post-Generation Validation**: The generated query is scanned for forbidden patterns.

6. **Result Cap**: Even if everything else fails, only 200 documents max are returned.

### Q3: "How would you scale this to 10,000 concurrent users?"

**Answer:**

Current bottlenecks and solutions:

| Bottleneck | Current | Scaled Solution |
|-----------|---------|-----------------|
| LLM inference | Single Groq API key | Multiple API keys + load balancing across providers |
| Database connections | lru_cache (100 clients) | Redis-based connection registry + pooling |
| Schema extraction | 5-min TTL in-memory | Redis cache shared across workers |
| State management | In-process | External state store (Redis) for multi-worker |
| API server | Single Uvicorn | Multiple Gunicorn workers + Kubernetes HPA |
| Frontend | Single Vercel deploy | CDN + edge caching |

Additional changes:
- **Queue-based architecture**: For heavy queries, use Celery + Redis to dequeue
- **WebSocket for streaming**: Stream LLM tokens in real-time instead of waiting for full response
- **Rate limiting**: Per-user rate limits on LLM calls
- **Caching repeated queries**: If same query was asked before for same schema, return cached result

### Q4: "Explain the security considerations of this system."

**Answer:**

1. **Credential Protection:**
   - DB URIs encrypted at rest (Fernet)
   - Passwords hashed (bcrypt, 72-byte limit handled)
   - JWT tokens expire in 60 minutes
   - OTPs auto-expire (MongoDB TTL index, 10 minutes)

2. **Authorization:**
   - Every API call verifies JWT + workspace membership
   - Role-based: admin vs user
   - Permission-based: read_only vs read_write
   - read_only users cannot execute insert/update/delete

3. **Query Safety:**
   - Multi-layer destructive query blocking
   - Update operations require explicit confirmation
   - Result size capped at 200 documents
   - SQL: Only SELECT allowed for read_only

4. **Network Security:**
   - CORS restricted to known origins
   - TLS for MongoDB Atlas connections (certifi)
   - SendGrid uses HTTP API (not SMTP — works on serverless)

5. **Risks & Mitigations:**
   - LLM prompt injection → Schema grounding + validation + confirmation
   - JWT secret compromise → All tokens invalid, re-key
   - Database URI leak → Encrypted at rest, decrypted only in memory

### Q5: "What would you change if you were rebuilding this from scratch?"

**Answer:**

1. **TypeScript backend**: Better type safety, especially for the complex state management
2. **WebSocket for streaming**: Show LLM thinking in real-time
3. **Redis for caching**: Shared schema cache across workers (current in-memory cache doesn't scale)
4. **Proper migration system**: For the MongoDB app database structure changes
5. **Rate limiting from day 1**: Per-user rate limits on expensive LLM operations
6. **Observability**: OpenTelemetry traces through the pipeline (currently just logging)
7. **E2E tests**: Playwright tests for the full Phase 1 → Phase 2 flow
8. **Vector search for schema matching**: Instead of LLM relevance check, use embeddings

---

## 19. Technical Interview Questions & Answers

### FastAPI & Python Questions

**Q: What is ASGI and how is it different from WSGI?**

A: WSGI (Web Server Gateway Interface) is synchronous — one request blocks one thread. ASGI (Asynchronous Server Gateway Interface) supports async/await — one event loop handles many concurrent requests without blocking. FastAPI uses ASGI through Uvicorn, which means I/O-bound operations (like calling the Groq API or MongoDB) don't block other requests.

**Q: How does FastAPI's dependency injection work?**

A: FastAPI uses `Depends()` to declare function dependencies. When an endpoint declares `current_user = Depends(get_current_user)`, FastAPI:
1. Calls `get_current_user` before the endpoint handler
2. Injects the return value as the parameter
3. If the dependency raises an exception, the endpoint is never called
4. Dependencies can have their own dependencies (chaining)

```python
# Chain: HTTPBearer → get_current_user → endpoint
security = HTTPBearer()
async def get_current_user(credentials = Depends(security)):
    # security already extracted the Bearer token
    payload = verify_token(credentials.credentials)
    return db["users"].find_one({"_id": ObjectId(payload["user_id"])})
```

**Q: Why use `@lru_cache` for the LLM instance?**

A: `@lru_cache(maxsize=1)` ensures only one `ChatGroq` instance exists per configuration. Without it, every chain invocation would create a new HTTP session to Groq. The cache means connection reuse, lower memory, and faster subsequent calls.

**Q: How do you handle errors gracefully in the pipeline?**

A: Every node in the pipeline follows the same pattern:
```python
try:
    result = do_something()
    return {**state, "result": result}
except Exception as exc:
    return {**state, "error": str(exc)}
```

No node ever raises — they set `state["error"]` and the conditional edges route to the suggestion node or END. This means the user always gets a response (even if it's an error + suggestions).

---

### LangChain & LangGraph Questions

**Q: What is LCEL and why use it?**

A: LCEL (LangChain Expression Language) uses the `|` operator to compose chains:
```python
chain = prompt | llm | parser
```

Benefits:
- Declarative: Easy to read what the chain does
- Streaming support: Each step can stream output to the next
- Async support: `.ainvoke()` for concurrent execution
- Batch support: `.batch([inputs])` for parallel processing
- Fallbacks: `.with_fallbacks([alternative_chain])`

**Q: How does LangGraph differ from a simple sequential pipeline?**

A: LangGraph adds:
1. **State**: Typed state object (TypedDict) flows through the graph
2. **Conditional edges**: Next node depends on runtime state values
3. **Cycles**: Nodes can loop back (not used here, but possible for retries)
4. **Parallel execution**: Multiple branches can run concurrently
5. **Checkpointing**: State can be saved/restored (useful for long-running agents)

In IntelliQuery, conditional edges are critical:
- Phase 1 stops at intent_node → END
- Phase 2 skips relevance → goes directly to query_node
- Errors route to suggestion_node
- Destructive queries route directly to END (no suggestions)

**Q: How does `json_mode=True` work with Groq/LLaMA?**

A: Setting `response_format={"type": "json_object"}` in the API call instructs the model to:
1. Only output valid JSON (no markdown, no prose)
2. Stop generation at the closing `}`
3. This makes `JsonOutputParser()` reliable — it won't fail on invalid JSON

Without json_mode, the model might output:
```
Here's the query:
```json
{"operation": "find", ...}
```
```

With json_mode:
```
{"operation": "find", ...}
```

**Q: What happens if the LLM returns unexpected JSON structure?**

A: The `extract_intent` function merges with defaults:
```python
_INTENT_DEFAULTS = {
    "operation": "find",
    "collection": "",
    "goal": "",
    "filters": [],
    ...
}
intent = {**_INTENT_DEFAULTS, **result}  # Missing keys get defaults
```

Type normalization ensures `limit` is int and list fields are actually lists.

---

### MongoDB Questions

**Q: Explain the MongoDB aggregation pipeline.**

A: An aggregation pipeline processes documents through sequential stages. Each stage transforms the data:

```javascript
// Example: Count movies by genre, sorted by count
db.movies.aggregate([
    { $match: { year: { $gte: 2020 } } },     // Filter first (uses index)
    { $unwind: "$genres" },                     // Flatten arrays
    { $group: { _id: "$genres", count: { $sum: 1 } } },  // Group + count
    { $sort: { count: -1 } },                  // Sort by count
    { $limit: 10 }                             // Top 10
])
```

Common stages: `$match`, `$group`, `$sort`, `$project`, `$unwind`, `$lookup` (join), `$limit`, `$skip`

**Q: How does MongoDB's `$regex` work?**

A: `$regex` performs pattern matching on string fields:
```javascript
{ genre: { $regex: "sci.fi", $options: "i" } }
// "i" = case-insensitive
// "." matches any character (so "sci-fi", "sci fi" both match)
```

IntelliQuery uses regex for text matching because:
- Users don't know exact values in the database
- Case-insensitive matching is more user-friendly
- Pattern matching handles variations (sci-fi, Sci-Fi, science fiction)

**Q: What is connection pooling in MongoDB?**

A: PyMongo's `MongoClient` maintains a pool of TCP connections to the MongoDB server. Instead of creating a new connection per query (expensive: TCP handshake + TLS negotiation), it reuses existing connections from the pool. Default pool size is 100.

In IntelliQuery, `@lru_cache` ensures one `MongoClient` per unique URI, and each client maintains its own connection pool.

**Q: How does the TTL index work for OTPs?**

A: 
```python
otps_collection.create_index("createdAt", expireAfterSeconds=600)
```

MongoDB's TTL index runs a background thread (every 60 seconds) that:
1. Checks documents where `createdAt` field is older than 600 seconds
2. Deletes them automatically
3. No application code needed for cleanup

---

### React & Frontend Questions

**Q: Why useState + useEffect instead of Redux/Zustand?**

A: For this application:
- State is localized: Most state lives in App.jsx
- No complex cross-component communication needed
- No need for time-travel debugging or middleware
- Only ~10 state variables total
- Performance is fine (no unnecessary re-renders affecting UX)

Redux/Zustand would add complexity without benefit. If the app grew (e.g., real-time collaboration, complex caching), a state library would make sense.

**Q: Explain the controlled component pattern in ChatInput.**

A:
```javascript
const [value, setValue] = useState('')
<textarea value={value} onChange={e => setValue(e.target.value)} />
```

React controls the input's value at all times. Benefits:
- Predictable state (value always matches `value` state)
- Can validate/transform input before rendering
- Easy to clear after submit: `setValue('')`

**Q: How does the Vite proxy work?**

A:
```javascript
proxy: {
    '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
    }
}
```

In development, Vite intercepts requests starting with `/api`:
1. Browser requests `http://localhost:5173/api/query`
2. Vite rewrites to `http://localhost:8000/query`
3. Response returned to browser as if it came from same origin
4. No CORS issues in development

In production, the frontend's Vercel deployment proxies differently (or uses absolute URLs to the backend).

**Q: What is createPortal and why is it used?**

A:
```javascript
{profileOpen && createPortal(
    <div className="profile-dropdown">...</div>,
    document.body
)}
```

`createPortal` renders a React component outside its parent DOM hierarchy. Used for:
- Dropdown menus that need to escape `overflow: hidden` parents
- Modals that should be on top of everything
- Tooltips that shouldn't be clipped

**Q: How do you handle the CSV download without a server?**

A: Client-side CSV generation:
```javascript
const blob = new Blob([csvString], { type: 'text/csv' })
const url = URL.createObjectURL(blob)
const a = document.createElement('a')
a.href = url
a.download = 'filename.csv'
a.click()
URL.revokeObjectURL(url)
```

No server round-trip needed — the data is already in browser memory.

---

### Security Questions

**Q: What is Fernet encryption?**

A: Fernet is a symmetric encryption scheme from the `cryptography` library:
- Algorithm: AES-128-CBC + HMAC-SHA256
- Key: 32-byte URL-safe base64 encoded
- Ciphertext includes: version + timestamp + IV + ciphertext + HMAC
- HMAC ensures integrity (tampered data is rejected)
- Timestamp enables optional time-based expiry

In IntelliQuery, the Fernet key is derived from `JWT_SECRET` via SHA-256, meaning both JWT signing and URI encryption use the same root secret.

**Q: How do you prevent SQL injection?**

A: Multiple layers:
1. **Permission check**: read_only users can only run SELECT
2. **LLM generates SQL**: User input never directly concatenated
3. **SQLAlchemy's text()**: Provides a layer of parameterization
4. **No user-provided values in SQL**: The LLM constructs the full query

**Remaining risk**: If an adversary crafts a prompt that tricks the LLM into generating malicious SQL. Mitigation: SELECT-only enforcement for read_only users.

**Q: How does bcrypt prevent rainbow table attacks?**

A: bcrypt:
1. Generates a random salt (22 characters)
2. Hashes password + salt with configurable cost factor (default: 12 = 2^12 iterations)
3. Stores: `$2b$12$<salt><hash>`
4. Each hash is unique even for identical passwords (due to random salt)
5. Cost factor makes brute-force exponentially expensive
A rainbow table is a precomputed database of common passwords and their hashes, allowing attackers to quickly reverse unsalted password hashes. bcrypt prevents rainbow table attacks by adding a unique random salt to every password before hashing, so even identical passwords produce different hashes. This makes precomputed rainbow tables useless.

**Q: What's the security risk of storing JWT_SECRET in .env?**

A: If `.env` is compromised (e.g., leaked to version control), an attacker can:
- Forge JWTs (impersonate any user)
- Decrypt all stored database URIs
- Reset passwords (generate valid OTPs)

Mitigations:
- `.env` in `.gitignore`
- Production uses environment variables (not files)
- Secret rotation capability (change JWT_SECRET → invalidates all sessions)

---

### Architecture Questions

**Q: Why a two-phase pipeline instead of single-shot?**

A: Single-shot approach: User types → LLM generates query → Execute → Hope it's right.

Problems with single-shot:
1. LLM might query wrong collection (e.g., "movies" instead of "films")
2. LLM might add phantom filters not in the original question
3. LLM might misinterpret intent (aggregate when user wanted find)
4. No way to catch errors before execution

Two-phase approach gives the user a checkpoint:
1. Phase 1: LLM extracts WHAT the user wants (structured, editable)
2. User verifies: "Yes, I want to find in movies collection with genre=sci-fi"
3. Phase 2: LLM generates precise syntax from CONFIRMED intent

Result: Much higher accuracy because Phase 2 has clear, verified instructions.

**Q: How would you add streaming responses?**

A: Currently, the user waits for the full pipeline to complete. To add streaming:

1. **WebSocket endpoint**: Replace REST with WebSocket
2. **Stream Phase 1**: Send tokens as they arrive from intent extraction
3. **Stream Phase 2**: Send query generation tokens, then explanation tokens
4. **Use LangChain streaming**: `.astream()` on LCEL chains
5. **Frontend**: Show typing effect as tokens arrive

```python
# Example: Streaming explanation
async for chunk in _explanation_chain.astream({"query": query_str}):
    await websocket.send_json({"type": "token", "content": chunk})
```

**Q: How would you add support for a new database type (e.g., Elasticsearch)?**

A: The architecture is extensible by design:

1. Add `database/elasticsearch_client.py` (connection + pooling)
2. Add `database/elasticsearch_schema_extractor.py` (index → field mapping)
3. Add `prompts/elasticsearch_prompt.py` (ES query DSL generation)
4. Add `nodes/elasticsearch.py` (similar to `nodes/sql.py`)
5. Add `elasticsearch` to workspace `db_type` enum
6. Add routing in `api/main.py` based on `db_type`

The workspace system already supports `db_type` field, so the multi-tenant model works without changes.

---

### Performance Questions

**Q: What are the latency bottlenecks and how do you optimize them?**

| Operation | Typical Latency | Optimization |
|-----------|----------------|--------------|
| LLM call (Groq) | 300-800ms | json_mode (shorter output), 8B model (fast) |
| Schema extraction | 50-200ms | TTL cache (5 minutes) |
| MongoDB query | 10-100ms | Connection pooling (lru_cache) |
| Network (frontend→backend) | 5-50ms | Same-origin proxy in dev |
| Total Phase 1 | ~1-2 seconds | Schema cached, single LLM call |
| Total Phase 2 | ~2-4 seconds | 2 LLM calls (query + explain) |

**Q: How does the TTL cache prevent stale data issues?**

A: The 5-minute TTL is a deliberate trade-off:
- **Benefit**: Avoids re-extracting schema on every query (saves 50-200ms)
- **Risk**: If schema changes (new collection added), it won't appear for up to 5 minutes
- **Acceptable because**: Database schemas change infrequently, and wrong schema just means the query might fail (user retries after cache expires)

**Q: How would you handle 200+ concurrent LLM requests?**

A: Groq has rate limits. Strategies:
1. **Queue with backpressure**: Redis queue, process at max rate
2. **Multiple API keys**: Distribute load across keys
3. **Caching identical queries**: If same query + same schema → cached result
4. **Graceful degradation**: Show "System busy" message instead of timeout
5. **Priority queue**: Paying users get priority

---

### Behavioral / Project Questions

**Q: Walk me through how you built this project.**

A: I built IntelliQuery iteratively:

1. **MVP (Backend only)**: CLI app that connects to MongoDB, calls LLM, returns query results. Validated the core concept works.

2. **Added LangGraph**: Structured the pipeline as a graph instead of sequential functions. This enabled conditional routing (error handling, different paths for different operations).

3. **Two-Phase Split**: Initially had single-shot generation. Accuracy was ~60%. After splitting into intent extraction + confirmed query generation, accuracy improved to ~90%+ because the LLM receives verified inputs.

4. **FastAPI + React**: Built the web interface. Started with simple chat, added IntentCard for the confirmation step.

5. **Multi-Tenancy**: Added workspaces, encrypted URIs, join codes, member permissions. This makes it shareable.

6. **Authentication**: JWT + bcrypt + OTP verification + Google OAuth.

7. **Visualization**: Added Recharts + AI-powered chart type selection.

8. **MCP Integration**: Built Claude Desktop integration as a bonus feature.

**Q: What was the hardest technical challenge?**

A: **Preventing LLM hallucination while maintaining accuracy.** The LLM would:
- Add filters that the user never mentioned (e.g., auto-adding `status: "active"`)
- Use field names that don't exist in the schema
- Generate valid-looking but semantically wrong queries

The solution was the two-phase pipeline:
- Phase 1: Extract intent in a STRUCTURED format (not query syntax)
- IntentCard: User explicitly confirms what they want
- Phase 2: Generate syntax from CONFIRMED, structured input

This narrowed the LLM's job from "understand intent AND write query" to just "convert structured data to syntax" — much more reliable.

**Q: What would you do differently if starting over?**

A: 
1. **Start with TypeScript**: Type safety across the full stack would catch bugs earlier
2. **WebSocket from the start**: Streaming responses dramatically improve perceived performance
3. **Redis for caching**: In-memory TTL cache doesn't work with multiple workers
4. **Testing from day 1**: End-to-end tests for the critical query flow
5. **Vector embeddings for relevance**: Instead of LLM-based relevance check, use cosine similarity between query embedding and schema embedding — faster and cheaper

**Q: How do you ensure the system is production-ready?**

A: Current production-readiness checklist:
- ✅ Error handling (no unhandled exceptions)
- ✅ Authentication + authorization
- ✅ Input validation (Pydantic models)
- ✅ CORS configured for specific origins
- ✅ Connection pooling for databases
- ✅ Global exception handler
- ⚠️ Missing: Rate limiting, request logging, APM, automated tests
- ⚠️ Missing: Database migration strategy
- ⚠️ Missing: Health check on external dependencies (Groq, MongoDB Atlas)

---

### Advanced Questions

**Q: How would you implement real-time collaboration (multiple users querying same workspace simultaneously)?**

A: 
1. **WebSocket per session**: Each user gets a WebSocket connection
2. **Shared query history**: Store in MongoDB instead of localStorage
3. **Live activity feed**: Broadcast "User X is querying..." to workspace members
4. **Query result sharing**: Allow pinning results for team visibility
5. **Conflict handling**: Concurrent updates → last-write-wins with version field

**Q: How would you add query result caching?**

A: Implement a query cache layer:
```python
# Key: hash(workspace_id + schema_version + confirmed_intent)
# Value: {query, explanation, result, timestamp}
# TTL: 30 seconds (data might change)

import hashlib
def cache_key(workspace_id, intent):
    raw = f"{workspace_id}:{json.dumps(intent, sort_keys=True)}"
    return hashlib.sha256(raw.encode()).hexdigest()
```

This avoids re-running expensive LLM + DB calls for identical queries.

**Q: How would you implement query scheduling / saved queries?**

A: 
1. New collection: `saved_queries` → {workspace_id, intent, schedule_cron, last_run, results}
2. Background scheduler (APScheduler or Celery Beat)
3. On schedule: run_pipeline with saved intent → store results
4. Frontend: "Saved Queries" tab showing latest results with timestamps
5. Notifications: Email/webhook when results change

**Q: How would you add natural language to GraphQL?**

A: Similar to the SQL pipeline:
1. New schema extractor: Parse GraphQL SDL → extract types + fields
2. New prompt: Generate GraphQL query syntax instead of MongoDB/SQL
3. New execution engine: Use `gql` library to execute against GraphQL endpoint
4. Same two-phase flow: intent → confirm → generate → validate → execute

**Q: How would you handle very large result sets (10,000+ documents)?**

A: Current hard cap is 200. For larger sets:
1. **Server-side pagination**: Add `page` and `pageSize` to QueryRequest
2. **Cursor-based pagination**: Use MongoDB cursor ID for efficient next-page
3. **Streaming results**: WebSocket chunks of 50 documents
4. **Aggregation for large sets**: Suggest `$count`, `$group`, or `$sample` instead of returning all
5. **Download option**: Generate CSV/JSON file and return download link

**Q: How would you add query explanation with lineage (which LLM call produced what)?**

A: Implement observability tracing:
```python
# Add trace_id to each LLM call
import uuid

def explain_query(query, trace_id=None):
    trace_id = trace_id or str(uuid.uuid4())
    logger.info(f"[Trace:{trace_id}] explanation_chain input: {query}")
    result = _explanation_chain.invoke({"query": query})
    logger.info(f"[Trace:{trace_id}] explanation_chain output: {result}")
    return result, trace_id
```

In production: OpenTelemetry integration with LangSmith or Langfuse for full LLM observability.

---

### Coding Questions They Might Ask You to Implement

**Q: "Write a function that validates a MongoDB query dict is safe to execute."**

```python
import re

FORBIDDEN_PATTERNS = [
    r'\$out\b', r'\$merge\b', r'\bdelete\b', r'\bdrop\b', 
    r'\bremove\b', r'\$unset\b.*\*',
]

def is_safe_query(query: dict) -> tuple[bool, str]:
    """Returns (is_safe, reason)."""
    if not isinstance(query, dict):
        return False, "Query must be a dictionary"
    
    query_str = str(query).lower()
    
    for pattern in FORBIDDEN_PATTERNS:
        if re.search(pattern, query_str, re.IGNORECASE):
            return False, f"Forbidden pattern detected: {pattern}"
    
    # Check operation type
    operation = query.get("operation", "").lower()
    if operation in ("delete", "drop", "remove"):
        return False, f"Destructive operation: {operation}"
    
    return True, "Safe"
```

**Q: "Write a TTL cache decorator that's thread-safe."**

```python
import time
import threading
from functools import wraps

def ttl_cache(ttl_seconds=300):
    def decorator(func):
        cache = {}
        lock = threading.Lock()
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            
            with lock:
                if key in cache:
                    value, timestamp = cache[key]
                    if time.time() - timestamp < ttl_seconds:
                        return value
                    del cache[key]  # Expired
            
            # Compute outside the lock (don't hold lock during expensive ops)
            result = func(*args, **kwargs)
            
            with lock:
                cache[key] = (result, time.time())
            
            return result
        
        wrapper.cache_clear = lambda: cache.clear()
        return wrapper
    return decorator
```

**Q: "Write a function that sanitizes MongoDB BSON documents for JSON serialization."**

```python
from bson import ObjectId, Decimal128
from datetime import datetime, date

def sanitize_document(doc):
    """Recursively convert BSON types to JSON-serializable Python types."""
    if isinstance(doc, ObjectId):
        return str(doc)
    elif isinstance(doc, Decimal128):
        return float(doc.to_decimal())
    elif isinstance(doc, datetime):
        return doc.isoformat()
    elif isinstance(doc, date):
        return doc.isoformat()
    elif isinstance(doc, dict):
        return {key: sanitize_document(value) for key, value in doc.items()}
    elif isinstance(doc, list):
        return [sanitize_document(item) for item in doc]
    elif isinstance(doc, bytes):
        return doc.decode('utf-8', errors='replace')
    return doc
```

---

## 20. Scalability & Performance

### Current Architecture Limitations

| Aspect | Current State | Bottleneck At |
|--------|--------------|---------------|
| Concurrent users | ~50-100 | Single Uvicorn process, Groq rate limits |
| Schema cache | In-memory (per process) | Multi-worker deployment (no sharing) |
| DB connections | lru_cache (100 max) | Memory pressure with many workspaces |
| LLM calls | 2-3 per query flow | Groq free tier: 30 RPM |
| Chat history | localStorage | No cross-device sync |
| File storage | None | No document upload capability |

### Scaling Strategy (Short-term)

1. **Horizontal API scaling**: 
   - Gunicorn with 4+ Uvicorn workers
   - Stateless API design allows load balancing
   - Schema cache moves to Redis (shared across workers)

2. **LLM rate limit management**:
   - Multiple Groq API keys with round-robin
   - Request queue with exponential backoff
   - Fallback to slower provider (OpenAI) when Groq is rate-limited

3. **Database optimization**:
   - MongoDB indexes on `workspace_members.workspace_id + user_id`
   - Connection pool per worker (not per process)
   - Read replicas for schema extraction

### Scaling Strategy (Long-term / Enterprise)

```
                    ┌─────────────────────┐
                    │   Load Balancer     │
                    │   (Nginx / ALB)     │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
┌────────▼───────┐  ┌────────▼───────┐  ┌─────────▼──────┐
│  API Server 1  │  │  API Server 2  │  │  API Server N  │
│  (4 workers)   │  │  (4 workers)   │  │  (4 workers)   │
└────────┬───────┘  └────────┬───────┘  └────────┬───────┘
         │                   │                    │
         └─────────┬─────────┴────────┬──────────┘
                   │                   │
         ┌────────▼───────┐   ┌───────▼────────┐
         │  Redis Cluster  │   │  LLM Gateway   │
         │  (cache + queue)│   │  (rate limit)  │
         └────────┬───────┘   └───────┬────────┘
                   │                   │
    ┌──────────────┼───────────────────┼──────────────┐
    │              │                   │              │
┌───▼──┐  ┌───────▼───────┐  ┌───────▼──┐  ┌──────▼─────┐
│MongoDB│  │MongoDB Replica│  │  Groq    │  │  OpenAI    │
│Primary│  │   (reads)     │  │  (fast)  │  │  (fallback)│
└──────┘  └───────────────┘  └──────────┘  └────────────┘
```

### Performance Optimizations Already Implemented

1. **Schema caching (5-min TTL)**: Avoids expensive DB introspection on every query
2. **Connection pooling (lru_cache)**: Reuses MongoClient/SQLAlchemy Engine instances
3. **json_mode for LLM**: Shorter outputs, no parsing failures
4. **Fast model (8B)**: LLaMA 3.1 8B instead of 70B — 3x faster, sufficient quality
5. **Lazy evaluation**: Schema only fetched when needed (not on login)
6. **Frontend caching**: localStorage for chat history (no server round-trip)
7. **Vite dev proxy**: Zero CORS overhead in development

### Monitoring & Observability (Recommended)

```
┌──────────────────────────────────────────────────┐
│                 Observability Stack                │
│                                                    │
│  ┌─────────┐  ┌──────────┐  ┌────────────────┐  │
│  │Prometheus│  │  Grafana  │  │   LangSmith    │  │
│  │(metrics) │  │(dashboard)│  │(LLM traces)    │  │
│  └─────────┘  └──────────┘  └────────────────┘  │
│                                                    │
│  Metrics to track:                                │
│  - LLM latency (p50, p95, p99)                   │
│  - Query success rate                            │
│  - Phase 1 → Phase 2 conversion rate            │
│  - Error rate by type (validation, LLM, DB)      │
│  - Active workspaces                             │
│  - Concurrent users                              │
└──────────────────────────────────────────────────┘
```

---

## 21. Future Enhancements

### Planned Features

| Feature | Complexity | Impact | Description |
|---------|-----------|--------|-------------|
| WebSocket streaming | Medium | High | Stream LLM tokens to show real-time thinking |
| Query history (server) | Low | Medium | Store queries in MongoDB instead of localStorage |
| Saved queries & scheduling | Medium | High | Schedule recurring queries with email alerts |
| Multi-model support | Medium | Medium | Let users choose GPT-4, Claude, Mistral |
| Vector-based relevance | Medium | High | Replace LLM relevance check with embeddings |
| Admin dashboard | High | Medium | Analytics: queries/day, popular collections, errors |
| Export to BI tools | Low | Medium | Export results to Google Sheets, Notion |
| Natural language to GraphQL | High | Medium | Support GraphQL databases |
| Query templates | Low | High | Saveable, parameterized query templates |
| Collaborative annotations | Medium | Medium | Team members comment on query results |
| API keys (programmatic access) | Medium | High | Allow headless API usage without web UI |
| Rate limiting | Low | Critical | Per-user rate limits on LLM calls |
| Audit log | Low | High | Track who queried what and when |

### Technical Debt to Address

1. **Test coverage**: Currently 0% — needs unit tests for nodes, integration tests for pipeline
2. **Error messages**: Some raw Python exceptions leak to frontend
3. **TypeScript migration**: Frontend would benefit from type safety
4. **Database migrations**: No formal schema versioning for app DB
5. **Logging standardization**: Mix of print() and logging module
6. **Environment validation**: No startup check that all env vars are present
7. **API versioning**: No `/v1/` prefix on endpoints

---

## Appendix A: Complete File-by-File Reference

### Backend Files

| File | Lines | Purpose |
|------|-------|---------|
| `api/main.py` | ~250 | FastAPI app, all core endpoints |
| `api/dependencies.py` | ~35 | JWT auth dependency |
| `api/models/user.py` | ~30 | Pydantic request models |
| `api/routes/auth.py` | ~180 | Auth endpoints (signup, login, OAuth, OTP) |
| `api/routes/workspaces.py` | ~200 | Workspace CRUD + members |
| `api/utils/jwt_handler.py` | ~25 | JWT create/verify |
| `api/utils/encryption.py` | ~30 | Fernet encrypt/decrypt |
| `api/utils/cache_utils.py` | ~30 | Thread-safe TTL cache |
| `api/utils/email_sender.py` | ~100 | SendGrid HTML email template |
| `api/utils/password_handler.py` | ~15 | bcrypt hash/verify |
| `agents/router_agent.py` | ~300 | LangGraph pipeline definition |
| `nodes/intent_extraction.py` | ~80 | Phase 1 intent extraction |
| `nodes/query.py` | ~90 | Phase 2 query generation |
| `nodes/validation.py` | ~40 | Destructive query blocking |
| `nodes/execution.py` | ~15 | Legacy (unused — execution is in router_agent) |
| `nodes/explanation.py` | ~40 | Plain-English explanation |
| `nodes/relevance.py` | ~50 | Schema relevance check |
| `nodes/schema.py` | ~10 | Schema extraction (cached) |
| `nodes/suggestion.py` | ~50 | Error fallback suggestions |
| `nodes/visualization.py` | ~60 | Chart config generation |
| `nodes/sql.py` | ~90 | Full SQL pipeline (generate + execute + explain) |
| `prompts/intent_prompt.py` | ~60 | Intent extraction prompt template |
| `prompts/query_prompt.py` | ~150 | Per-operation query prompts |
| `prompts/explanation_prompt.py` | ~15 | Explanation prompt (legacy) |
| `database/mongo_client.py` | ~30 | MongoDB connection factory |
| `database/sql_client.py` | ~25 | SQLAlchemy engine factory |
| `database/schema_extractor.py` | ~15 | MongoDB schema introspection |
| `database/sql_schema_extractor.py` | ~15 | SQL schema introspection |
| `app/config.py` | ~25 | LLM configuration |
| `app/main.py` | ~10 | CLI entry point |
| `mcp_server.py` | ~70 | Claude Desktop MCP tool |
| `mcp_proxy.py` | ~60 | SSE proxy for remote MCP |
| `install_mcp.py` | ~80 | Interactive MCP setup script |

### Frontend Files

| File | Purpose |
|------|---------|
| `App.jsx` | Root component, state management, routing |
| `main.jsx` | React entry point, Google OAuth provider |
| `Home.jsx` | Landing page (unauthenticated) |
| `SignIn.jsx` | Login/signup/OTP/reset flows |
| `Dashboard.jsx` | Workspace list + create/join modals |
| `Sidebar.jsx` | Schema browser + new chat button |
| `ChatInput.jsx` | Text input with Enter to send |
| `ChatMessage.jsx` | Renders all message types |
| `IntentCard.jsx` | Phase 1 editable confirmation card |
| `ResultsTable.jsx` | Data table + CSV download |
| `ResultsDisplay.jsx` | Generic results formatter |
| `VisualizationChart.jsx` | Recharts chart renderer |
| `WorkspaceSettings.jsx` | Member management modal |
| `McpModal.jsx` | Claude Desktop config modal |
| `Aurora.jsx` | WebGL background animation |
| `SpotlightCard.jsx` | Hover spotlight effect card |
| `QueryInput.jsx` | Legacy query input (replaced by ChatInput) |
| `QueryResult.jsx` | Legacy result display (replaced by ChatMessage) |

---

## Appendix B: Key Concepts to Review Before Interview

### Must-Know Concepts

1. **REST API Design** — HTTP methods, status codes, headers, CORS
2. **JWT Authentication** — Token structure, HMAC signing, claims, expiry
3. **MongoDB** — CRUD, aggregation pipeline, indexing, connection pooling
4. **SQL** — JOINs, GROUP BY, indexes, SQLAlchemy basics
5. **React Hooks** — useState, useEffect, useRef, custom hooks
6. **Python async/await** — Event loop, coroutines, ASGI
7. **LLM Concepts** — Tokens, temperature, prompting, json_mode
8. **Encryption** — Symmetric (Fernet/AES) vs asymmetric (RSA), hashing (bcrypt)
9. **OAuth 2.0** — Authorization code flow, implicit flow, token verification
10. **WebSocket** — Full-duplex communication, vs HTTP polling, vs SSE

### Design Pattern Keywords

- **Separation of Concerns**: Each node handles one thing
- **Single Responsibility**: Each file has one purpose
- **Don't Repeat Yourself**: Shared utilities (cache, encryption, auth)
- **Fail Gracefully**: Every node catches exceptions, returns structured errors
- **Defense in Depth**: Multiple safety layers for destructive queries
- **Principle of Least Privilege**: read_only by default for new members

### Buzzwords You Should Be Comfortable With

- LCEL (LangChain Expression Language)
- LangGraph (stateful graph orchestration)
- ASGI (Asynchronous Server Gateway Interface)
- BSON (Binary JSON — MongoDB's wire format)
- TTL (Time To Live — for cache and OTP expiry)
- MCP (Model Context Protocol — Anthropic's tool standard)
- SSE (Server-Sent Events — for streaming)
- Fernet (Symmetric encryption scheme)
- HMAC (Hash-based Message Authentication Code)
- Connection Pooling
- Multi-tenancy
- Two-phase commit (conceptually similar to two-phase pipeline)

---

## Appendix C: Quick Reference Cheat Sheet

### Start the Project

```bash
# Backend
cd Project && source venv/bin/activate
uvicorn api.main:app --reload --port 8000

# Frontend
cd frontend && npm run dev
```

### Test an API Call

```bash
# Health check
curl http://localhost:8000/health

# Login
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"test123"}'

# Extract Intent (requires token + workspace ID)
curl -X POST http://localhost:8000/extract-intent \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -H "X-Workspace-Id: <workspace_id>" \
  -d '{"query":"Show me all movies","history":[]}'
```

### Key Environment Variables

```
GROQ_API_KEY=gsk_...
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/sample_mflix
JWT_SECRET=your-256-bit-secret
GOOGLE_CLIENT_ID=xxxxx.apps.googleusercontent.com
SENDGRID_API_KEY=SG.xxxxx
SENDGRID_FROM_EMAIL=noreply@intelliquery.app
```

### MongoDB Collections (App Database)

```
users:              {_id, name, email, password}
otps:               {_id, email, type, otp, createdAt} + TTL index
workspaces:         {_id, name, admin_id, db_type, db_uri, db_name, join_code, created_at}
workspace_members:  {_id, workspace_id, user_id, role, permission, joined_at}
```

---

## Appendix D: Common Interview Scenarios

### Scenario 1: "The LLM is generating wrong queries"

**Diagnosis steps:**
1. Check if schema is being passed correctly (log schema_node output)
2. Check if intent extraction is correct (inspect IntentCard)
3. Check if prompt has conflicting instructions
4. Check if json_mode is enabled (invalid JSON causes parse failures)

**Solutions:**
- Add more examples to the prompt (few-shot learning)
- Make prompt instructions more explicit
- Add post-processing validation
- Use a larger model (70B instead of 8B) for complex queries

### Scenario 2: "Users are getting 401 errors"

**Diagnosis steps:**
1. Check if token is expired (60-min expiry)
2. Check if JWT_SECRET matches between token creation and verification
3. Check if user was deleted from database
4. Check if Authorization header format is correct (`Bearer <token>`)

### Scenario 3: "Schema is not showing new collections"

**Diagnosis:**
- Schema is cached for 5 minutes (TTL cache)
- Wait for cache expiry or restart server
- Check if MongoDB user has `listCollections` permission

### Scenario 4: "Connection to user's database is failing"

**Diagnosis:**
1. Is the URI encrypted correctly? (Check JWT_SECRET hasn't changed)
2. Is the MongoDB Atlas allowlist configured for server IP?
3. Is the database name correct?
4. Is TLS working? (certifi handles this)

---

*This document covers the complete IntelliQuery system from architecture to implementation details, designed for comprehensive interview preparation. Review each section based on the expected interview focus.*
