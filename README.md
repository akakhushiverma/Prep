# RepoRover – Complete Interview Study Guide
## A Deep-Dive Technical Reference for Interview Preparation

---

# TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Why This Project – Motivation & Problem Statement](#2-why-this-project)
3. [Architecture & System Design](#3-architecture--system-design)
4. [Tech Stack Deep Dive](#4-tech-stack-deep-dive)
5. [Data Flow & Pipelines](#5-data-flow--pipelines)
6. [Database Design & Schema](#6-database-design--schema)
7. [API Design & Endpoints](#7-api-design--endpoints)
8. [Core Algorithms & Data Structures](#8-core-algorithms--data-structures)
9. [LLM Integration & RAG Pipeline](#9-llm-integration--rag-pipeline)
10. [Frontend Architecture](#10-frontend-architecture)
11. [Design Patterns Used](#11-design-patterns-used)
12. [Challenges & Problem Solving](#12-challenges--problem-solving)
13. [Performance & Optimization](#13-performance--optimization)
14. [Security Considerations](#14-security-considerations)
15. [Testing Strategy](#15-testing-strategy)
16. [Deployment & DevOps](#16-deployment--devops)
17. [Impact & Results](#17-impact--results)
18. [STAR Stories](#18-star-stories)
19. [Interview Questions & Answers](#19-interview-questions--answers)
20. [Tech Stack Interview Questions](#20-tech-stack-interview-questions)
21. [System Design Interview Angles](#21-system-design-interview-angles)
22. [Questions You Can Ask Interviewers](#22-questions-you-can-ask-interviewers)

---

# 1. PROJECT OVERVIEW

## What is RepoRover?

RepoRover is a **GraphRAG-powered codebase intelligence system** that allows developers to ingest entire code repositories, automatically build a knowledge graph of the code structure, create semantic embeddings, and then ask natural language questions about the codebase and get accurate, context-rich answers.

## One-Liner for Interviews

> "I built a GraphRAG system that ingests code repositories, constructs a Neo4j knowledge graph of functions/classes/calls/imports, creates vector embeddings per symbol, and enables natural language Q&A about codebases using hybrid retrieval (graph + vector) combined with LLM-powered answers."

## Business Problem It Solves

1. **Developer Onboarding** – New team members can understand large codebases quickly by asking questions like "How does authentication work?" or "What functions handle payment processing?"
2. **Code Discovery** – Finding where specific functionality lives across thousands of files
3. **Architecture Understanding** – Understanding call flows, dependencies, and relationships without reading every file
4. **Documentation Gap** – Many codebases lack documentation; this auto-generates understanding from code structure
5. **Code Review Aid** – Understanding impact of changes by exploring the code graph

## Key Features

| Feature | Description |
|---------|-------------|
| Multi-language AST Parsing | Python, JavaScript, TypeScript, Go, Java, C/C++, Ruby, Rust, PHP |
| Knowledge Graph | Neo4j graph with Files, Functions, Classes, Imports + DEFINES/CALLS/MENTIONS/IMPORTS edges |
| Semantic Search | ChromaDB vector store with per-symbol embeddings (not whole-file) |
| Hybrid Retrieval | Combines vector similarity + graph traversal for rich context |
| LLM Query Rewriting | Transforms questions into optimized technical search queries |
| Intent-Aware Enrichment | Different retrieval strategies for "flow", "usage", and "general" queries |
| Git Integration | Clone remote repos or index local folders |
| Modern UI | Streamlit-based dark-mode dashboard with chat interface |
| Graph Explorer | Visual exploration of code relationships |


---

# 2. WHY THIS PROJECT

## Why GraphRAG for Code?

### The Problem with Traditional RAG for Code

Traditional RAG (Retrieval-Augmented Generation) simply splits documents into chunks, embeds them, and retrieves similar chunks. For code, this fails because:

1. **Code has structure** – Functions call other functions, classes inherit, files import modules. Simple text similarity misses these relationships.
2. **Context is relational** – To understand "how login works," you need to follow the call chain: `login_route → authenticate_user → verify_password → hash_compare`.
3. **Semantic similarity ≠ Code similarity** – Two functions with similar descriptions may do completely different things; two functions with different descriptions may be tightly coupled.

### Why Graph + RAG (GraphRAG)?

GraphRAG combines:
- **Vector Search** (semantic) – Find code that's conceptually similar to the question
- **Graph Traversal** (structural) – Expand to neighboring symbols via CALLS/DEFINES edges to get complete execution context
- **LLM Reasoning** – Synthesize the combined context into a coherent answer

This is significantly better than either approach alone because:
- Vector alone misses the "how things connect" aspect
- Graph alone doesn't understand natural language queries
- Together, they provide both semantic relevance AND structural completeness

## Why This Tech Stack?

| Choice | Reasoning |
|--------|-----------|
| **Neo4j** (graph DB) | Purpose-built for traversals, Cypher query language is expressive for code relationships, supports variable-length path queries |
| **ChromaDB** (vector store) | Lightweight, persistent, no server needed, good LangChain integration |
| **tree-sitter** (AST parser) | Multi-language support in a single library, fast C-based parsing, byte-accurate ranges |
| **FastAPI** (backend) | Async-first, automatic OpenAPI docs, type-safe with Pydantic, high performance |
| **LangChain** (LLM framework) | LCEL chains for composable pipelines, many integrations, output parsers |
| **Streamlit** (frontend) | Rapid prototyping, session state for chat, built-in components |
| **sentence-transformers** (embeddings) | Local inference (no API cost), good quality for code, normalized embeddings |

## Why Not Alternatives?

| Alternative | Why Not Used |
|-------------|-------------|
| OpenAI embeddings | Costs money per call, slower (network), privacy concerns for code |
| Pinecone/Weaviate | Overkill for single-machine use; ChromaDB is simpler |
| LlamaIndex | More opinionated, less control over retrieval pipeline |
| Full AST compiler | tree-sitter is faster and multi-language; we don't need full compilation |
| React/Next.js frontend | Would require separate build tooling; Streamlit is faster to iterate |


---

# 3. ARCHITECTURE & SYSTEM DESIGN

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    FRONTEND (Streamlit)                         │
│  ┌──────┐ ┌──────────┐ ┌─────────┐ ┌───────────┐ ┌────────┐ │
│  │ Home │ │ Ingest   │ │ Query   │ │ Graph     │ │Settings│ │
│  │ Page │ │ Page     │ │ Chat    │ │ Explorer  │ │ Page   │ │
│  └──────┘ └──────────┘ └─────────┘ └───────────┘ └────────┘ │
│                         │ HTTP API Client │                     │
└─────────────────────────┼────────────────┼─────────────────────┘
                          │                │
                   HTTP (REST/JSON)         │
                          │                │
┌─────────────────────────┼────────────────┼─────────────────────┐
│              BACKEND (FastAPI + Uvicorn)                        │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              API Layer (Routers)                          │  │
│  │  /health  │  /ingest  │  /query  │  /graph/explore       │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Service Layer                                │  │
│  │  IngestService  │  QueryService                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Domain Layer                                 │  │
│  │  Query Chain (LCEL)  │  Indexer  │  AST Extract          │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Infrastructure Layer                         │  │
│  │  Neo4jClient  │  VectorStore  │  ChatModel  │  Registry  │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────┐    ┌──────────────────┐    ┌─────────────┐
│   Neo4j     │    │    ChromaDB      │    │  LLM API    │
│  (Graph DB) │    │  (Vector Store)  │    │ (Groq/etc.) │
└─────────────┘    └──────────────────┘    └─────────────┘
```

## Architecture Type

This is a **Layered Monolith** with clear separation of concerns:

1. **API Layer** – HTTP routing, request/response validation (Pydantic schemas)
2. **Service Layer** – Orchestration logic, business workflows
3. **Domain Layer** – Core algorithms (AST parsing, query chains, indexing)
4. **Infrastructure Layer** – External system clients (Neo4j, ChromaDB, LLM)

## Key Architectural Decisions

### Decision 1: Monolith vs Microservices
- **Chose: Monolith** – Single deployable unit is simpler for an MVP, avoids network overhead between services
- **Trade-off**: Harder to scale individual components independently
- **Mitigation**: Clean layering means easy future extraction into services

### Decision 2: Per-Symbol vs Per-File Embeddings
- **Chose: Per-Symbol** – Each function/class gets its own embedding(s)
- **Why**: More precise retrieval; a file may have 20 functions but only 1 is relevant
- **Trade-off**: More storage, more embedding computation
- **Impact**: Dramatically better retrieval precision

### Decision 3: Async Query Chain, Sync Ingestion
- **Query: Async** – Multiple I/O operations (vector search, graph queries, LLM calls)
- **Ingest: Sync** – Sequential pipeline (parse → graph → embed), blocking is acceptable
- **Why**: Queries need responsiveness; ingestion is a one-time batch operation

### Decision 4: Local Embeddings vs API Embeddings
- **Chose: Local (sentence-transformers)** – No API cost, no network latency, data privacy
- **Trade-off**: Requires GPU for speed (CPU works but slower), model quality may be lower than OpenAI
- **Mitigation**: MiniLM-L6-v2 is surprisingly good for code; normalized embeddings help


---

# 4. TECH STACK DEEP DIVE

## 4.1 FastAPI (Backend Framework)

**What it is**: Modern Python async web framework built on Starlette + Pydantic.

**Why used here**:
- Automatic request/response validation via Pydantic models
- Async support (critical for LLM + DB I/O)
- Auto-generated OpenAPI/Swagger docs
- Dependency injection system
- Lifespan context managers for startup/shutdown

**Key patterns used in this project**:
```python
# Lifespan for Neo4j connection management
@asynccontextmanager
async def lifespan(app: FastAPI):
    neo4j = Neo4jClient.from_settings()
    neo4j.init_schema()
    app.state.neo4j = neo4j  # Attach to app state
    yield
    neo4j.close()  # Cleanup on shutdown

# Router-based organization
app.include_router(health_router)
app.include_router(ingest_router)
app.include_router(query_router)
```

**Interview Points**:
- FastAPI uses ASGI (vs Flask's WSGI) – supports async/await natively
- Pydantic models provide runtime type validation + serialization
- The lifespan pattern replaced the deprecated `@app.on_event("startup")`
- `app.state` is a shared mutable namespace accessible from any request handler

---

## 4.2 Neo4j (Graph Database)

**What it is**: Native graph database using the property graph model. Nodes have labels and properties; relationships have types and properties.

**Why used here**:
- Code relationships are inherently graph-shaped (calls, defines, imports, mentions)
- Cypher query language is optimized for traversals
- Variable-length path queries (`*1..4`) are trivial
- Constraint system ensures data integrity

**Schema in this project**:
```cypher
-- Nodes
(:File {path, repo_id})
(:Function {repo_id, qualified_name, name})
(:Class {repo_id, qualified_name, name})
(:Import {repo_id, value})

-- Relationships
(File)-[:DEFINES]->(Function|Class)
(Function)-[:CALLS]->(Function)
(File)-[:IMPORTS]->(Import)
(File)-[:MENTIONS]->(Function|Class)
```

**Key Cypher queries used**:
```cypher
-- Multi-hop neighbor expansion (for query enrichment)
MATCH (s {repo_id: $repo_id})
WHERE s.qualified_name IN $qns
CALL (s) {
  MATCH (s)-[:CALLS|DEFINES*1..2]-(n)
  RETURN DISTINCT n.qualified_name AS qn
}
RETURN DISTINCT qn

-- Call flow extraction (for "how does X work?" questions)
MATCH path = (s:Function {repo_id: $repo_id})-[:CALLS*1..4]->(n:Function)
WHERE s.qualified_name IN $qns
RETURN [node in nodes(path) | node.name] AS call_chain
```

**Interview Points**:
- Neo4j uses index-free adjacency (each node stores direct pointers to neighbors)
- This makes traversals O(1) per hop, unlike SQL JOINs which are O(n)
- Compound uniqueness constraints: `REQUIRE (fn.repo_id, fn.qualified_name) IS UNIQUE`
- MERGE operation is idempotent (upsert behavior)
- `DETACH DELETE` removes nodes and all their relationships

---

## 4.3 ChromaDB (Vector Store)

**What it is**: Open-source embedding database. Stores vectors + metadata, supports similarity search.

**Why used here**:
- Persistent storage (survives restarts)
- No separate server needed (embedded mode)
- Supports metadata filtering
- Good LangChain integration

**How used**:
```python
# One collection per repository
client = chromadb.PersistentClient(path=".chroma")
collection = Chroma(
    client=client,
    collection_name=f"reporover_{repo_id}",
    embedding_function=HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
)

# Upsert documents with metadata
collection.add_texts(
    texts=[f"{qualified_name}\n\n{code_chunk}"],
    metadatas=[{"qualified_name": qn, "kind": "function", "file_path": path}],
    ids=["sym_abc123"]
)

# Retrieve by similarity
retriever = collection.as_retriever(search_kwargs={"k": 24})
docs = await retriever.ainvoke("authentication and password hashing")

# Retrieve by metadata (qualified name lookup)
results = collection.get(where={"qualified_name": {"$in": qn_list}})
```

**Interview Points**:
- ChromaDB uses HNSW (Hierarchical Navigable Small World) algorithm for ANN search
- Cosine similarity is used (embeddings are normalized)
- Each repo gets its own collection to avoid cross-contamination
- `PersistentClient` stores data on disk (vs `Client()` which is in-memory only)

---

## 4.4 tree-sitter (AST Parser)

**What it is**: Incremental parsing library written in C. Supports 40+ programming languages. Used by GitHub, VS Code, Neovim.

**Why used here**:
- Parse multiple languages with one library
- Get byte-accurate ranges for each symbol (critical for per-symbol embedding)
- Fast (C-based, can parse megabytes in milliseconds)
- Doesn't require the code to compile (works on partial/broken code)

**How used**:
```python
from tree_sitter_languages import get_parser

parser = get_parser("python")
tree = parser.parse(source_bytes)
root = tree.root_node

# Extract function definitions
func_nodes = _collect(root, {"function_definition", "function_declaration", "method_definition"})

# For each function, get its name and byte range
for fn in func_nodes:
    name_node = fn.named_children[0]  # identifier node
    name = source_bytes[name_node.start_byte:name_node.end_byte].decode()
    code = source_bytes[fn.start_byte:fn.end_byte].decode()
    
    # Extract calls within this function's scope
    call_nodes = _collect(fn, {"call", "call_expression"})
```

**Interview Points**:
- tree-sitter produces a concrete syntax tree (CST), not abstract syntax tree (AST) – it preserves all tokens
- Nodes have types (e.g., "function_definition"), named children, and byte ranges
- The parser is language-agnostic; grammar files define each language
- Supports JavaScript/TypeScript arrow functions via `variable_declarator` + `arrow_function` pattern
- Qualified names include byte offset to handle multiple anonymous functions: `path::name::byte_offset`

---

## 4.5 LangChain (LLM Framework)

**What it is**: Framework for building LLM-powered applications with composable chains.

**Key concepts used**:
- **LCEL (LangChain Expression Language)** – Pipe operator (`|`) for chaining operations
- **RunnableLambda** – Wrap any Python function as a chain step
- **ChatPromptTemplate** – Template system for LLM prompts
- **PydanticOutputParser** – Parse LLM output into structured Pydantic models
- **ChatOpenAI** – Client for OpenAI-compatible APIs

**How the query chain is built**:
```python
# LCEL composition
chain = (
    RunnableLambda(_rewrite_step)    # LLM rewrites question → search query
    | RunnableLambda(_retrieve_step)  # Vector search
    | RunnableLambda(_enrich_step)    # Graph expansion
    | RunnableLambda(_assemble_step)  # Join context
    | RunnableLambda(_llm_step)       # Generate answer
)

# Each step receives a state dict and returns an augmented state dict
result = await chain.ainvoke({"question": q, "repo_id": id, "top_k": 8, "neo4j": client})
```

**Interview Points**:
- LCEL chains are async-first and support streaming
- The `|` operator creates a `RunnableSequence` that passes output of each step as input to next
- `PydanticOutputParser` generates format instructions for the LLM and validates output
- Error handling is per-step (each step catches exceptions and provides fallbacks)

---

## 4.6 sentence-transformers (Embeddings)

**What it is**: Library for computing dense vector representations of text.

**Model used**: `all-MiniLM-L6-v2`
- Dimension: 384
- Speed: Very fast (22ms per sentence on CPU)
- Quality: Good for semantic similarity tasks
- Size: 80MB

**How used**:
```python
embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2",
    encode_kwargs={"normalize_embeddings": True},  # Unit vectors for cosine similarity
)
```

**Interview Points**:
- Normalized embeddings mean cosine similarity = dot product (faster computation)
- The model is downloaded once and cached locally (~/.cache/huggingface)
- 384 dimensions is a good trade-off between quality and storage/speed
- For code, the model understands function names, docstrings, and code patterns

---

## 4.7 Streamlit (Frontend)

**What it is**: Python library that turns scripts into interactive web apps.

**Key features used**:
- `st.session_state` – Persistent state across reruns (chat history, config)
- `st.chat_input` – Chat-style input widget
- `st.markdown(unsafe_allow_html=True)` – Custom HTML/CSS injection
- `st.radio` – Navigation menu
- `st.status` – Progress indicator
- `st.dataframe` – Data display

**Architecture pattern**: Single-page app with manual page routing:
```python
# Router pattern
page = st.radio("Navigate", ["🏠 Home", "📥 Ingest", "💬 Query", ...])
if page == "🏠 Home":
    from frontend.pages.home import render
    render()
elif page == "📥 Ingest":
    ...
```

**Interview Points**:
- Streamlit reruns the entire script on every interaction (top-to-bottom)
- `st.session_state` persists data across reruns
- `st.rerun()` forces a page refresh (used after adding chat messages)
- Custom CSS is injected via markdown with `unsafe_allow_html=True`
- No virtual DOM or component lifecycle – simpler mental model than React


---

# 5. DATA FLOW & PIPELINES

## 5.1 Ingestion Pipeline (Complete Flow)

```
USER INPUT: "source": "/path/to/repo" or "https://github.com/user/repo.git"
            "repo_id": "my-app"
            "branch": "main" (optional)

STEP 1: Source Resolution [app/repos/sources.py]
├── If source is a local directory path → use directly (is_temp=False)
├── If source is a Git URL → clone to .work/repo_{hash} (is_temp=True)
├── Uses GitPython Repo.clone_from()
├── Force cleanup if target exists (handles locked files on Windows)
└── Register repo_id → root_dir mapping in .work/repos.json

STEP 2: File Discovery [app/ingestion/indexer.py]
├── Walk directory tree with os.walk()
├── Filter by supported extensions: .py, .js, .jsx, .ts, .tsx
├── Skip directories: .work, .chroma, node_modules, .git, .venv, dist, build
└── Return list of Path objects

STEP 3: AST Extraction (per file) [app/ingestion/ast_extract.py]
├── Read file as bytes
├── Select tree-sitter parser based on extension
├── Parse → get root node
├── Extract FUNCTIONS:
│   ├── function_definition (Python)
│   ├── function_declaration (JS/TS)
│   ├── method_definition (JS/TS class methods)
│   └── variable_declarator with arrow_function value (JS/TS)
├── Extract CLASSES:
│   ├── class_definition (Python)
│   └── class_declaration (JS/TS)
├── Extract IMPORTS:
│   ├── import_statement, import_from_statement (Python)
│   └── import_declaration (JS/TS)
├── For each FUNCTION, extract CALLS within its scope:
│   ├── call (Python)
│   └── call_expression (JS/TS)
├── Build qualified_name: "{rel_path}::{name}::{start_byte}"
└── Return (symbols: list[Symbol], file_text: str, imports: list[str])

STEP 4: Graph Construction [app/services/ingest_service.py]
├── For each file: MERGE (:File {path, repo_id})
├── For each file: Create IMPORTS edges to (:Import) nodes
├── For each symbol: MERGE (:Function/:Class {repo_id, qualified_name, name})
├── Create DEFINES edges: (:File)-[:DEFINES]->(:Function/:Class)
├── For each function with calls: Create CALLS edges
│   └── Callees get qualified_name: "{repo_id}::external::{callee_name}"
└── For each file: Tokenize identifiers → Create MENTIONS edges

STEP 5: Embedding Generation [app/services/ingest_service.py]
├── Group symbols by file
├── For each symbol:
│   ├── Extract code: raw_bytes[start_byte:end_byte]
│   ├── Select language-aware text splitter (LangChain)
│   ├── Split into chunks (chunk_size=2000, overlap=200)
│   └── For each chunk:
│       ├── doc_id = sha256(repo_id|qualified_name)[:24] + "_ch{i}"
│       ├── document = "{qualified_name}\n\n{chunk_text}"
│       └── metadata = {repo_id, qualified_name, kind, file_path, name, chunk_index}
├── Batch upsert to ChromaDB collection: reporover_{repo_id}
└── HuggingFace embeddings computed automatically by ChromaDB

STEP 6: Return Result
└── IngestResult(repo_id, files_indexed, symbols_indexed)
```

## 5.2 Query Pipeline (Complete Flow)

```
USER INPUT: "question": "How does authentication work?"
            "repo_id": "my-app"
            "top_k": 8

STEP 1: Query Rewriting [app/query/chain.py → _rewrite_step]
├── Send question to LLM with structured prompt
├── LLM returns QueryPlan:
│   ├── search_vector: "Implementation of user authentication, login routes,
│   │                    password verification, JWT token generation, and session management"
│   ├── intent: "flow" (because user asked "how does X work?")
│   └── target_symbol: None
├── If no LLM_API_KEY → fallback: use original question as search_vector
└── Output: state with "plan" added

STEP 2: Vector Retrieval [app/query/chain.py → _retrieve_step]
├── Create retriever with top_k * 3 = 24 results
├── Search ChromaDB with plan.search_vector
├── For each result, extract:
│   ├── document (code chunk)
│   ├── metadata (qualified_name, file_path, kind, name)
│   └── distance score
├── Deduplicate and cap at top_k = 8 results
├── Extract seed_qns = [qualified_names from top hits]
├── Extract context_parts = [document[:2500] for each hit]
└── Output: state with "hits", "seed_qns", "context_parts" added

STEP 3: Graph Enrichment [app/query/chain.py → _enrich_step]
├── ALWAYS: Graph expansion
│   ├── Query Neo4j: from seed symbols, traverse CALLS|DEFINES edges 2 hops deep
│   ├── Get all neighbor qualified_names
│   ├── Separate into internal vs external (filter "::external::")
│   ├── Priority: internal symbols first, then external
│   ├── Deduplicate + remove already-seen seeds
│   └── Cap at 50 neighbors
│
├── IF intent == "usage":
│   ├── Find all files that MENTION the target symbol
│   ├── For each file (up to 5): read source, extract snippet around symbol usage
│   └── Add file paths + code snippets to context_parts
│
├── IF intent == "flow":
│   ├── Query Neo4j for call chains starting from seed symbols (depth 4)
│   ├── Format as: "func_a -> func_b -> func_c -> func_d"
│   └── Add top 20 call flows to context_parts
│
├── ALWAYS: Pull source code for top 10 neighbors from ChromaDB
│   ├── Use get_documents_by_qns() with neighbor qualified_names
│   └── Add full source code to context_parts
└── Output: state with enriched "context_parts"

STEP 4: Context Assembly [app/query/chain.py → _assemble_step]
├── Join all context_parts with "\n\n---\n\n" separators
├── Count total context items
└── Output: state with "context" string and "context_items" count

STEP 5: LLM Answer Generation [app/query/chain.py → _llm_step]
├── System prompt: "You are RepoRover, an assistant that answers questions about a codebase.
│                   Use ONLY the provided context. If unsure, say what is missing."
├── Human prompt: "Question:\n{question}\n\nContext:\n{context}"
├── Invoke ChatOpenAI (Groq/OpenAI-compatible)
│   ├── model: llama-3.3-70b-versatile
│   ├── temperature: 0.2 (low creativity, high accuracy)
│   ├── max_tokens: 800
│   └── timeout: 60s
├── Parse response as string
└── Output: state with "answer" added

STEP 6: Return Result
└── (answer: str, context_items: int)
```

## 5.3 Graph Exploration Flow

```
USER INPUT: "qualified_name": "src/auth.py::login::42"
            "repo_id": "my-app"
            "depth": 2

STEP 1: Query Neo4j
├── Find the seed node by qualified_name
├── Traverse ALL relationships up to {depth} hops
├── Collect all nodes (with id, name, qualified_name, labels)
├── Collect all edges (with source, target, type)
└── Return combined set

STEP 2: Return to Frontend
├── nodes: [{id, name, qualified_name, labels}, ...]
├── edges: [{source, target, type}, ...]
└── Frontend renders as Mermaid flowchart
```


---

# 6. DATABASE DESIGN & SCHEMA

## 6.1 Neo4j Graph Schema

### Node Types

| Label | Properties | Constraints | Purpose |
|-------|-----------|-------------|---------|
| `File` | path, repo_id | path IS UNIQUE | Represents a source code file |
| `Function` | repo_id, qualified_name, name | (repo_id, qualified_name) IS UNIQUE | Represents a function/method |
| `Class` | repo_id, qualified_name, name | (repo_id, qualified_name) IS UNIQUE | Represents a class |
| `Import` | repo_id, value | (repo_id, value) IS UNIQUE | Represents an import statement |

### Relationship Types

| Relationship | From | To | Meaning |
|-------------|------|-----|---------|
| `DEFINES` | File | Function/Class | "File X defines function Y" |
| `CALLS` | Function | Function | "Function A calls function B" |
| `IMPORTS` | File | Import | "File X has import statement Y" |
| `MENTIONS` | File | Function/Class | "File X references/uses symbol Y" |

### Qualified Name Format

```
{relative_file_path}::{symbol_name}::{start_byte_offset}

Examples:
- src/auth/login.py::authenticate_user::156
- components/Button.tsx::handleClick::892
- utils/helpers.js::formatDate::42
```

The byte offset ensures uniqueness even when multiple functions share the same name (e.g., nested functions, overloads).

### External Symbols

When a function calls something not defined in the repo (e.g., library functions):
```
{repo_id}::external::{callee_expression}

Examples:
- my-app::external::print
- my-app::external::requests.get
- my-app::external::os.path.join
```

### Graph Traversal Patterns

```cypher
-- "What does function X call?" (1 hop)
MATCH (f:Function {qualified_name: $qn})-[:CALLS]->(callee)
RETURN callee.name, callee.qualified_name

-- "What calls function X?" (reverse)
MATCH (caller)-[:CALLS]->(f:Function {name: $name})
RETURN caller.name, caller.qualified_name

-- "Show me the full call chain" (variable depth)
MATCH path = (s:Function)-[:CALLS*1..4]->(n:Function)
WHERE s.qualified_name IN $start_qns
RETURN [node in nodes(path) | node.name] AS chain

-- "Which files use this symbol?"
MATCH (f:File)-[:MENTIONS]->(sym {name: $symbol_name})
RETURN f.path

-- "What does this file define?"
MATCH (f:File {path: $path})-[:DEFINES]->(sym)
RETURN sym.name, labels(sym), sym.qualified_name
```

## 6.2 ChromaDB Schema

### Collection Naming
```
Collection per repo: "reporover_{repo_id}"
Examples: "reporover_my-app", "reporover_flask-backend"
```

### Document Structure
```
ID: "sym_{sha256(repo_id|qualified_name)[:24]}" or "sym_{hash}_ch{i}" for multi-chunk symbols
Document: "{qualified_name}\n\n{source_code_chunk}"
Vector: 384-dimensional (MiniLM-L6-v2)
Metadata: {
    "repo_id": "my-app",
    "qualified_name": "src/auth.py::login::156",
    "kind": "function",
    "file_path": "src/auth.py",
    "name": "login",
    "chunk_index": 0
}
```

### Why Per-Symbol (not Per-File)?

| Approach | Pros | Cons |
|----------|------|------|
| Per-file | Fewer documents, simpler | Low precision (returns 20 functions when 1 is relevant) |
| Per-symbol | High precision, targeted | More documents, more embedding computation |
| Per-chunk (fixed-size) | Uniform sizes | Cuts across function boundaries, loses context |

**This project uses per-symbol chunking**: Each function/class is extracted by its exact AST byte range, then split with language-aware splitters if it's too long (>2000 chars).

## 6.3 Repository Registry

```json
// .work/repos.json
{
  "my-app": "/absolute/path/to/my-app",
  "flask-backend": "/path/to/.work/repo_a1b2c3d4e5f6"
}
```

This JSON file maps repo_id to filesystem path, enabling:
- Cross-session persistence (survives server restarts)
- Source code lookup during query time (for "usage" intent)
- Cleanup of cloned repos


---

# 7. API DESIGN & ENDPOINTS

## 7.1 API Architecture

- **Framework**: FastAPI (ASGI)
- **Server**: Uvicorn
- **Serialization**: Pydantic v2 models
- **Pattern**: Router-based organization (one file per resource)
- **State Management**: `app.state.neo4j` attached during lifespan
- **Error Handling**: HTTPException with status codes + traceback logging

## 7.2 Endpoint Reference

### GET /health
```json
// Response 200
{
    "ok": true,
    "neo4j": true
}
```
- No authentication required
- Used by frontend sidebar for status display
- Returns false for neo4j if connection was never established

### POST /ingest
```json
// Request
{
    "repo_id": "my-app",           // Required: stable alias
    "source": "/path/to/repo",      // Required: local path or Git URL
    "branch": "main"                // Optional: for remote repos
}

// Response 200
{
    "repo_id": "my-app",
    "files_indexed": 47,
    "symbols_indexed": 312
}

// Response 500
{
    "detail": "Error message with traceback"
}
```
- Synchronous endpoint (blocking; ingestion takes 10-60 seconds)
- Timeout: 300s on frontend client
- Idempotent: re-ingesting overwrites existing data (MERGE semantics)

### POST /query
```json
// Request
{
    "repo_id": "my-app",
    "question": "How does authentication work?",
    "top_k": 8                      // Default: 8
}

// Response 200
{
    "answer": "Authentication in this codebase works by...",
    "context_items": 12
}
```
- Async endpoint (awaits LLM + DB calls)
- Timeout: 120s on frontend client
- Requires LLM_API_KEY to be set for full answers
- `context_items` indicates how many context pieces were assembled

### POST /query/context
```json
// Request (same as /query)
{
    "repo_id": "my-app",
    "question": "How does authentication work?",
    "top_k": 8
}

// Response 200
{
    "context": "src/auth.py::login::156\n\ndef login(username, password):\n    ...",
    "context_items": 12
}
```
- Returns raw retrieved context WITHOUT LLM processing
- Useful for debugging retrieval quality
- No LLM_API_KEY needed (only retrieval + graph enrichment)

### POST /graph/explore
```json
// Request
{
    "repo_id": "my-app",
    "qualified_name": "src/auth.py::login::156",
    "depth": 2                      // 1-3
}

// Response 200
{
    "nodes": [
        {"id": "4:abc:0", "name": "login", "qualified_name": "src/auth.py::login::156", "labels": ["Function"]},
        {"id": "4:def:1", "name": "verify_password", "qualified_name": "...", "labels": ["Function"]}
    ],
    "edges": [
        {"source": "4:abc:0", "target": "4:def:1", "type": "CALLS"}
    ]
}
```
- Returns graph neighborhood for visualization
- Depth is clamped to 1-3 to prevent explosive expansion
- Empty result if symbol doesn't exist

### GET /test/peek/{repo_id}
```json
// Response 200
{
    "repo_id": "my-app",
    "symbol_count": 312
}
```
- Debug endpoint: checks ChromaDB collection size
- Returns -1 if collection doesn't exist

### POST /test/reset
```json
// Response 200
{
    "status": "success",
    "message": "Neo4j wiped. Local folders cleared.",
    "directories_deleted": [".chroma", ".work"],
    "directories_failed_locks": []
}
```
- **DESTRUCTIVE**: Wipes everything (Neo4j + ChromaDB + work directory)
- Development-only endpoint
- Handles Windows file locks gracefully

## 7.3 API Design Principles

1. **Resource-oriented URLs**: `/ingest`, `/query`, `/graph/explore`
2. **POST for operations with side effects** (even queries, because they're complex)
3. **GET for safe, idempotent reads** (`/health`, `/test/peek/{id}`)
4. **Consistent error format**: `{"detail": "error message"}` or `{"error": "..."}`
5. **Pydantic validation**: Invalid requests automatically return 422 with field-level errors
6. **Timeout awareness**: Different operations have different timeout expectations


---

# 8. CORE ALGORITHMS & DATA STRUCTURES

## 8.1 AST Traversal (Tree Walking)

**Algorithm**: Iterative depth-first search using an explicit stack.

```python
def _collect(node: Node, types: set[str]) -> list[Node]:
    """Collect all descendant nodes matching given types."""
    out: list[Node] = []
    stack = [node]
    while stack:
        cur = stack.pop()
        if cur.type in types:
            out.append(cur)
        stack.extend(reversed(cur.named_children))  # Maintain order
    return out
```

**Complexity**: O(n) where n = number of nodes in the subtree.
**Why not recursion?** Avoids stack overflow for deeply nested code (Python default recursion limit is 1000).

## 8.2 Identifier Tokenization

**Algorithm**: Regex-based extraction of all identifiers from source text.

```python
IDENT_RE = re.compile(r"\b[A-Za-z_][A-Za-z0-9_]*\b")

def extract_identifiers(text: str) -> set[str]:
    return set(IDENT_RE.findall(text))
```

**Purpose**: Build "MENTIONS" edges by finding which files reference which symbol names.
**Complexity**: O(n) where n = text length.
**Trade-off**: Simple but imprecise (matches keywords, variable names, etc. – not just function calls).

## 8.3 Content-Based Hashing (Document IDs)

```python
def _doc_id(repo_id: str, qualified_name: str) -> str:
    h = hashlib.sha256(f"{repo_id}|{qualified_name}".encode("utf-8")).hexdigest()[:24]
    return f"sym_{h}"
```

**Purpose**: Generate deterministic, collision-resistant IDs for ChromaDB documents.
**Why SHA256?** Deterministic (same input → same ID) enables idempotent upserts.
**Why truncate to 24 chars?** Balance between collision resistance and readability.

## 8.4 Language-Aware Text Splitting

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter, Language

# Language-specific splitting respects code structure
if lang:  # e.g., Language.PYTHON
    splitter = RecursiveCharacterTextSplitter.from_language(
        language=lang, chunk_size=2000, chunk_overlap=200
    )
else:
    splitter = RecursiveCharacterTextSplitter(chunk_size=2000, chunk_overlap=200)
```

**How it works**:
- For Python: splits on `\nclass `, `\ndef `, `\n\n`, `\n`, ` ` (in priority order)
- For JavaScript: splits on `\nfunction `, `\nconst `, `\n\n`, `\n`, ` `
- This ensures splits happen at natural boundaries rather than mid-statement

**Parameters**:
- `chunk_size=2000`: Target chunk length in characters
- `chunk_overlap=200`: Overlap between consecutive chunks (prevents losing context at boundaries)

## 8.5 Graph Expansion (Multi-hop BFS)

```cypher
MATCH (s {repo_id: $repo_id})
WHERE s.qualified_name IN $qns
CALL (s) {
  MATCH (s)-[:CALLS|DEFINES*1..{depth}]-(n)
  RETURN DISTINCT n.qualified_name AS qn
}
RETURN DISTINCT qn
```

**What this does**: Starting from seed symbols, traverse CALLS and DEFINES edges up to `depth` hops in either direction, collecting all reachable symbols.

**Why bidirectional (`-` not `->`)**: A function might be called BY something interesting, not just call others.

**Limiting factors**:
- Depth capped at 3 (prevents explosion)
- Results capped at 50 symbols
- Internal symbols prioritized over external

## 8.6 Call Flow Extraction (Path Enumeration)

```cypher
MATCH path = (s:Function {repo_id: $repo_id})-[:CALLS*1..4]->(n:Function)
WHERE s.qualified_name IN $qns
RETURN [node in nodes(path) | node.name] AS call_chain
```

**What this does**: Find all directed paths from seed functions following CALLS edges, up to 4 hops.
**Output format**: `["login", "authenticate", "verify_password", "bcrypt_compare"]`
**Use case**: "How does authentication work?" → show the execution flow.

## 8.7 Priority-Based Context Assembly

```python
# 1. Direct vector hits (most semantically relevant)
context_parts = [h["document"][:2500] for h in hits]

# 2. Intent-specific context
if plan.intent == "usage":
    context_parts.append(f"Files mentioning `{symbol_name}`:\n...")
    context_parts.append(f"{path}\n\n{code_snippet}")

if plan.intent == "flow":
    context_parts.append("Function Call Flows:\n" + "\n".join(flows))

# 3. Graph-expanded neighbors (structural context)
context_parts.append("Graph-expanded symbols:\n" + "\n".join(neighbors))
context_parts.append("Source code for key neighbors:\n" + "\n".join(neighbor_code))
```

**Priority order**:
1. Direct semantic hits (from vector search)
2. Intent-specific enrichment (usage/flow)
3. Graph-expanded neighbor code

This ensures the LLM always sees the most relevant information first, even if context window is limited.

## 8.8 Data Structures Summary

| Structure | Type | Purpose |
|-----------|------|---------|
| `Symbol` | Frozen dataclass | Represents a parsed code symbol (function/class) |
| `IndexedFile` | Frozen dataclass | File + its symbols + text + imports |
| `IndexedRepo` | Frozen dataclass | Collection of all indexed files |
| `QueryPlan` | Pydantic model | LLM-generated search strategy |
| `IngestResult` | Frozen dataclass | Ingestion outcome metrics |
| `QueryResult` | Frozen dataclass | Query answer + metadata |
| `ResolvedRepo` | Frozen dataclass | Resolved filesystem location |


---

# 9. LLM INTEGRATION & RAG PIPELINE

## 9.1 What is RAG?

**Retrieval-Augmented Generation (RAG)** is a pattern where:
1. **Retrieve** relevant documents from a knowledge base
2. **Augment** the LLM prompt with those documents as context
3. **Generate** an answer grounded in the retrieved context

This prevents hallucination because the LLM answers based on actual data, not just training knowledge.

## 9.2 What is GraphRAG?

**GraphRAG** extends traditional RAG by:
1. **Vector retrieval** – Semantic similarity search (same as regular RAG)
2. **Graph traversal** – Follow relationships to find structurally connected information
3. **Combined context** – Both semantically similar AND structurally related content

For code, this means:
- Vector search finds functions with similar descriptions/code
- Graph traversal finds functions that are called by or call those functions
- Together, you get a complete "execution context" for answering questions

## 9.3 The Two-Phase Retrieval Strategy

### Phase 1: Semantic (Vector)
```
Question → LLM Rewrite → Vector Search → Top-K Hits
```

The LLM rewrites "How does login work?" into:
> "Implementation of user authentication, login routes, password verification, JWT token generation, and session management"

This rewritten query is much better for vector similarity because it's descriptive and technical.

### Phase 2: Structural (Graph)
```
Top-K Hits → Extract qualified_names → Graph Expand (2 hops) → Neighbor Symbols → Pull Code
```

From the vector hits, we extract their qualified names, then traverse the graph to find connected symbols that might not have appeared in vector search but are structurally essential.

## 9.4 Query Planning (Intent Detection)

The LLM classifies each question into one of three intents:

| Intent | Example Questions | Enrichment Strategy |
|--------|-------------------|---------------------|
| `flow` | "How does X work?", "Explain the execution path" | Extract call chains, show function flow |
| `usage` | "Where is X used?", "Who calls X?" | Find files mentioning symbol, show surrounding code |
| `general` | "What does X do?", "Explain this function" | Standard vector + graph expansion |

## 9.5 Prompt Engineering

### Planner Prompt (Query Rewriting)
```
System: You are a senior technical architect. Your goal is to rewrite the user's question 
into a highly effective vector search query.

For the 'search_vector' field, do NOT just repeat the user's words. Instead, generate a 
detailed technical description or a 'hypothetical code snippet' that describes what we are 
looking for.

Example: If the user asks 'How does login work?', rewrite it as 
'Implementation of user authentication, login routes, password verification, and session management.'
```

This is a **HyDE (Hypothetical Document Embeddings)** technique – generating what the answer would look like, then searching for similar documents.

### Answer Prompt
```
System: You are RepoRover, an assistant that answers questions about a codebase. 
Use ONLY the provided context. If unsure, say what is missing.

Human: Question:
{question}

Context:
{context}
```

Key design choices:
- **"Use ONLY the provided context"** – Prevents hallucination
- **"If unsure, say what is missing"** – Honest about gaps rather than making things up
- **Low temperature (0.2)** – Factual, deterministic responses over creative ones
- **max_tokens=800** – Enough for detailed answers, prevents runaway generation

## 9.6 LLM Configuration

```python
ChatOpenAI(
    model="llama-3.3-70b-versatile",  # Groq-hosted Llama 3.3 70B
    base_url="https://api.groq.com/openai/v1",
    temperature=0.2,
    max_tokens=800,
    timeout=60,
    max_retries=1,
)
```

**Why Groq?**
- Free tier available
- Fast inference (uses custom LPU hardware)
- OpenAI-compatible API (works with `langchain-openai`)
- Llama 3.3 70B is excellent for code understanding

**Why temperature 0.2?**
- We want factual, repeatable answers
- Not creative writing – code explanations should be precise
- Still >0 to avoid degenerate repetition

## 9.7 Error Handling & Graceful Degradation

```python
# If no LLM API key → return context without answer
if not settings.llm_api_key:
    return {**state, "answer": "LLM_API_KEY is missing. Set it in .env to enable answers."}

# If LLM call fails → return error message (not crash)
try:
    answer = await (prompt | model | parser).ainvoke(...)
except Exception as e:
    answer = f"LLM error: {e}"

# If query rewriting fails → use original question as search vector
try:
    plan = parser.invoke(output)
except Exception:
    plan = QueryPlan(search_vector=question, intent="general", target_symbol=None)
```

This ensures the system is **always usable**:
- Without LLM key → still get raw context (vector + graph retrieval works)
- With LLM failure → still get context + error message
- With rewrite failure → still search with original question

## 9.8 The LCEL (LangChain Expression Language) Chain

```python
chain = (
    RunnableLambda(_rewrite_step)     # async: question → QueryPlan
    | RunnableLambda(_retrieve_step)   # async: plan → vector hits
    | RunnableLambda(_enrich_step)     # sync: hits → enriched context
    | RunnableLambda(_assemble_step)   # sync: parts → joined context string
    | RunnableLambda(_llm_step)        # async: context → LLM answer
)

result = await chain.ainvoke(initial_state_dict)
```

**How LCEL works**:
- Each step receives the full state dict from the previous step
- Each step returns an augmented state dict (adds new keys, preserves existing)
- The `|` operator creates a sequential pipeline
- `ainvoke()` runs async steps with `await`

**Why LCEL over plain functions?**
- Standardized interface (invoke/ainvoke/batch/stream)
- Built-in retry, fallback, and error handling
- Easy to compose, test, and extend
- Supports streaming (for future chat-style responses)

---

# 10. FRONTEND ARCHITECTURE

## 10.1 Streamlit Application Structure

```
frontend/
├── streamlit_app.py       # Entry point + page router
├── theme.py               # Custom CSS (dark mode)
├── api_client.py          # HTTP wrapper for backend
├── components/
│   └── sidebar.py         # Navigation + status
└── pages/
    ├── home.py            # Landing page
    ├── ingest.py          # Repo ingestion form
    ├── query.py           # Chat Q&A interface
    ├── graph_explorer.py  # Graph visualization
    └── settings_page.py   # Admin + health check
```

## 10.2 Page Routing Pattern

Streamlit doesn't have built-in multi-page routing (in the traditional sense), so this project uses a **manual router pattern**:

```python
# streamlit_app.py
current_page = render_sidebar()  # Returns selected page name

if current_page == "🏠 Home":
    from frontend.pages.home import render
    render()
elif current_page == "📥 Ingest Repository":
    from frontend.pages.ingest import render
    render()
# ... etc
```

Each page is a module with a `render()` function that draws its UI.

## 10.3 State Management

Streamlit reruns the entire script on every user interaction. State is preserved using `st.session_state`:

```python
# Initialize state (only on first run)
if "chat_messages" not in st.session_state:
    st.session_state.chat_messages = []

# Add a message
st.session_state.chat_messages.append({"role": "user", "content": question})

# Read state
for msg in st.session_state.chat_messages:
    display(msg)
```

**State used in this project**:
- `chat_messages`: List of chat messages (role + content + context_items)
- `query_repo_id`: Currently selected repo for queries
- `api_url`: Backend URL (configurable)
- `ingested_repos`: History of ingested repo IDs

## 10.4 Custom Theming

The project uses a premium dark-mode theme with:
- **CSS Variables** for consistent colors
- **Custom fonts**: Inter (UI), JetBrains Mono (code)
- **Animations**: Pulse dot for status indicators
- **Gradient effects**: Primary buttons, brand text
- **Card components**: Glassmorphism style with backdrop-filter

Key CSS features:
```css
/* Glassmorphism cards */
.rover-card {
    background: rgba(17, 24, 39, 0.7);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(99, 102, 241, 0.15);
    border-radius: 12px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Gradient text (brand) */
.gradient-text {
    background: linear-gradient(135deg, #6366f1, #8b5cf6, #a855f7);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}
```

## 10.5 API Client Pattern

All backend communication goes through `frontend/api_client.py`:

```python
def query_codebase(repo_id: str, question: str, top_k: int = 8) -> dict:
    """POST /query – returns {"answer": str, "context_items": int}."""
    payload = {"repo_id": repo_id, "question": question, "top_k": top_k}
    try:
        r = requests.post(f"{_base_url()}/query", json=payload, timeout=120)
        r.raise_for_status()
        return r.json()
    except requests.exceptions.HTTPError as e:
        return {"error": f"HTTP {e.response.status_code}: {e.response.text}"}
    except Exception as e:
        return {"error": str(e)}
```

**Design choices**:
- Returns dict (not raises) – caller checks for "error" key
- Timeout per endpoint type (300s ingest, 120s query, 5s health)
- Base URL from session state (configurable without restart)
- Consistent error format across all functions


---

# 11. DESIGN PATTERNS USED

## 11.1 Creational Patterns

### Factory Method
```python
# Neo4jClient.from_settings() – creates client from config
@classmethod
def from_settings(cls) -> "Neo4jClient":
    driver = GraphDatabase.driver(settings.neo4j_uri, auth=(...))
    return cls(driver=driver)

# VectorStore.from_settings(repo_id) – creates per-repo store
@classmethod
def from_settings(cls, repo_id: str) -> "VectorStore":
    return cls(_chroma=_chroma_for_repo(repo_id))
```

### Singleton (via LRU Cache)
```python
@lru_cache(maxsize=1)
def get_chat_model() -> ChatOpenAI:
    return ChatOpenAI(model=settings.llm_model, ...)

@lru_cache(maxsize=1)
def _embeddings() -> HuggingFaceEmbeddings:
    return HuggingFaceEmbeddings(model_name=settings.embed_model, ...)

@lru_cache(maxsize=32)
def _chroma_for_repo(repo_id: str) -> Chroma:
    ...
```

**Why `@lru_cache` instead of module-level globals?**
- Lazy initialization (created on first use, not import time)
- Thread-safe
- Easy to clear/reset for testing

## 11.2 Structural Patterns

### Repository Pattern (Data Access Abstraction)
```python
class Neo4jClient:
    def upsert_file(self, repo_id, path) -> None
    def upsert_symbol(self, repo_id, kind, qualified_name, ...) -> None
    def add_calls(self, repo_id, caller_qn, callees) -> None
    def add_mentions(self, repo_id, file_path, symbol_qns) -> None

class VectorStore:
    def upsert_documents(self, ids, docs, metadatas) -> None
    def as_retriever(self, top_k) -> VectorStoreRetriever
    def get_documents_by_qns(self, qns) -> list[str]
```

The service layer doesn't know about Cypher queries or ChromaDB internals.

### Facade Pattern
```python
# ingest_service.py is a facade over multiple subsystems
def ingest_repo(neo4j, repo_id, source, branch) -> IngestResult:
    resolved = resolve_repo_source(...)     # Repo management
    indexed = index_repo(...)                # AST parsing
    neo4j.upsert_file(...)                  # Graph DB
    vs.upsert_documents(...)                # Vector DB
```

### Adapter Pattern
```python
# LangChain wrappers adapt different APIs to a common interface
# HuggingFaceEmbeddings adapts sentence-transformers to LangChain's embedding interface
# ChatOpenAI adapts OpenAI-compatible APIs to LangChain's chat model interface
# Chroma adapts chromadb to LangChain's vector store interface
```

## 11.3 Behavioral Patterns

### Chain of Responsibility (LCEL Pipeline)
```python
chain = rewrite | retrieve | enrich | assemble | llm_step
```
Each step processes the request and passes it to the next. If a step fails, it provides a fallback value and the chain continues.

### Strategy Pattern (Intent-Based Enrichment)
```python
if plan.intent == "usage":
    # Strategy: find files mentioning the symbol
    paths = files_mentioning_symbol(...)
    
elif plan.intent == "flow":
    # Strategy: extract call chains
    flows = graph_get_call_flows(...)
    
else:  # general
    # Strategy: standard graph expansion only
    pass
```

### Template Method (Page Rendering)
Each frontend page follows the same pattern:
```python
def render() -> None:
    # 1. Header markdown
    st.markdown(...)
    # 2. Input controls
    col1, col2 = st.columns([2, 1])
    # 3. Action button
    if st.button("Submit"):
        # 4. Call API
        result = api_call(...)
        # 5. Display results
        st.metric(...)
```

## 11.4 Architectural Patterns

### Layered Architecture
```
API Layer → Service Layer → Domain Layer → Infrastructure Layer
```

### Pipeline Pattern (Ingestion)
```
Source Resolution → File Discovery → AST Extraction → Graph Construction → Embedding Generation
```

### CQRS-lite (Command/Query Separation)
- **Commands** (write): `POST /ingest`, `POST /test/reset`
- **Queries** (read): `POST /query`, `POST /graph/explore`, `GET /health`
- Separate services: `ingest_service` vs `query_service`

### Event-Driven Startup (Lifespan)
```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: initialize resources
    neo4j = Neo4jClient.from_settings()
    neo4j.init_schema()
    app.state.neo4j = neo4j
    yield
    # Shutdown: cleanup
    neo4j.close()
```

## 11.5 Data Patterns

### Immutable Data Transfer Objects
All dataclasses use `frozen=True`:
```python
@dataclass(frozen=True)
class Symbol:
    kind: str
    name: str
    qualified_name: str
    ...
```
**Why frozen?** Thread safety, hashability, prevents accidental mutation.

### Registry Pattern (Repo Mapping)
```python
# Persistent registry for repo_id → filesystem path
def set_repo_root(repo_id, root_dir): ...  # Write to repos.json
def get_repo_root(repo_id): ...             # Read from repos.json
```


---

# 12. CHALLENGES & PROBLEM SOLVING

## Challenge 1: Handling Multi-Language AST Parsing

**Problem**: Each language has different AST node types for the same concept. A Python function is `function_definition`, a JS function might be `function_declaration`, `arrow_function`, or `method_definition`.

**Solution**: 
- Created a unified extraction layer that handles language-specific node types
- Used a set-based approach: `_collect(root, {"function_definition", "function_declaration", "method_definition"})`
- Special handling for JS/TS arrow functions (detected via `variable_declarator` → `arrow_function` pattern)

**Trade-off**: Some language-specific constructs might be missed (e.g., Python decorators, TypeScript generics), but the approach covers 90% of symbols.

## Challenge 2: Call Graph Accuracy

**Problem**: Static analysis can't perfectly determine what a function call refers to. `self.method()` could be any class in the hierarchy. `callback()` is a variable, not a function name.

**Solution**:
- Best-effort approach: capture the callee expression text (e.g., `"self.authenticate"`, `"os.path.join"`)
- Store callees as external references with qualified name: `{repo_id}::external::{callee_text}`
- During graph traversal, these connect to a "virtual" node representing the called function
- Limit callees to 200 per function to prevent explosion

**Trade-off**: The call graph has false positives (non-function calls captured) and false negatives (dynamic dispatch missed). But for RAG context gathering, precision matters less than recall.

## Challenge 3: Vector Search Quality for Code

**Problem**: `all-MiniLM-L6-v2` is trained on natural language, not code. Searching for "authentication" might not find a function named `verify_jwt_token`.

**Solution**: LLM-driven query rewriting (HyDE technique):
- Question: "How does authentication work?"
- Rewritten: "Implementation of user authentication, login routes, password verification, JWT token generation, and session management"
- The rewritten query bridges the vocabulary gap

**Impact**: Dramatically improved retrieval relevance because the search query now contains the same technical vocabulary as the code documentation/naming.

## Challenge 4: Context Window Limits

**Problem**: The LLM has a finite context window. With 50 graph neighbors + 8 vector hits + call flows, context could exceed limits.

**Solution**: 
- Cap vector hit text at 2500 chars each
- Cap graph neighbors at 50 symbols
- Cap call flows at 20 paths
- Pull source code only for top 10 neighbors
- Priority ordering ensures most relevant info comes first (LLMs pay more attention to beginning of context)

## Challenge 5: Windows File Locks (ChromaDB)

**Problem**: On Windows, ChromaDB holds file locks on its SQLite database. `shutil.rmtree()` fails when the server is running.

**Solution**:
```python
def _rmtree_force(path: Path) -> None:
    def _onerror(func, p, exc_info):
        try:
            Path(p).chmod(0o777)
        except Exception:
            pass
        try:
            func(p)
        except Exception:
            pass
    shutil.rmtree(path, onerror=_onerror)
```
- Also: `VectorStore.clear_all()` clears LRU caches before deletion
- Reset endpoint reports which directories failed with locks

## Challenge 6: Qualified Name Collisions

**Problem**: Multiple functions can have the same name in different files (or even the same file – nested functions, overloads).

**Solution**: Include byte offset in qualified name:
```
{file_path}::{name}::{start_byte}
```
This ensures every symbol has a globally unique identifier, even anonymous functions or multiple functions with identical names.

## Challenge 7: Ingestion Performance

**Problem**: Large repos with thousands of files take a long time to ingest (AST parsing + embedding generation).

**Solution**:
- Skip non-code directories early (`.git`, `node_modules`, `dist`, etc.)
- Use LRU-cached embedding model (one-time load)
- Batch upsert to ChromaDB (all documents at once, not one-by-one)
- Language-aware splitting reduces redundant chunk computation
- ChromaDB `PersistentClient` avoids re-initialization overhead

**Future improvements**: Parallel file processing, incremental re-indexing (only changed files).


---

# 13. PERFORMANCE & OPTIMIZATION

## 13.1 Caching Strategies

| What | How | Cache Size | Purpose |
|------|-----|-----------|---------|
| Embedding model | `@lru_cache(maxsize=1)` | 1 instance | Avoid reloading 80MB model on every request |
| ChromaDB client per repo | `@lru_cache(maxsize=32)` | 32 repos | Avoid recreating client/collection objects |
| LLM client | `@lru_cache(maxsize=1)` | 1 instance | Reuse connection pool |

**Why LRU cache?**
- Thread-safe in CPython (GIL protects dict operations)
- Automatic eviction when maxsize reached
- `cache_clear()` for testing/reset

## 13.2 Database Optimization

### Neo4j
- **Uniqueness constraints** act as indexes (automatic B-tree index on constrained properties)
- **MERGE** (not CREATE) prevents duplicate nodes on re-ingestion
- **Parameterized queries** prevent Cypher injection and enable query plan caching
- **UNWIND** for batch operations (process list in single query, not N separate queries)
- **Depth limits** on variable-length paths (cap at 3-4 hops)
- **LIMIT** clauses on all queries returning lists

### ChromaDB
- **Persistent storage** avoids re-embedding on restart
- **Collection per repo** enables efficient isolation and deletion
- **HNSW index** provides approximate nearest neighbor in O(log n)
- **Metadata filtering** (`$in` operator) for direct symbol lookup

## 13.3 Network Optimization

- **Local embeddings**: No network round-trip for embedding generation
- **Connection reuse**: Neo4j driver maintains a connection pool
- **LLM timeout**: 60s prevents hanging on slow API responses
- **Chunked context**: Cap at 2500 chars per hit prevents sending megabytes to LLM
- **Frontend timeouts**: Appropriate per-endpoint (5s health, 120s query, 300s ingest)

## 13.4 Memory Optimization

- **Frozen dataclasses**: No mutation overhead, enables hash-based deduplication
- **Streaming file reads**: Files read as bytes (not loaded into memory as strings until needed)
- **Generator-style processing**: Files processed one at a time during ingestion
- **Capped result sets**: Never return unbounded lists (always `.[:50]`, `LIMIT 30`)

## 13.5 What Could Be Improved (Good Interview Discussion Points)

| Improvement | Impact | Complexity |
|-------------|--------|-----------|
| **Incremental indexing** (only changed files) | Faster re-ingestion | Medium |
| **Parallel AST parsing** (multiprocessing) | 4-8x faster ingestion | Medium |
| **Streaming LLM responses** | Better UX (see answer as it generates) | Low |
| **Embedding batch API** (if using cloud embeddings) | Fewer API calls | Low |
| **Neo4j connection pooling tuning** | Better concurrent query performance | Low |
| **Result caching** (same question → cached answer) | Instant repeat queries | Medium |
| **Async Neo4j driver** | Non-blocking graph queries in LCEL chain | Medium |
| **HNSW parameter tuning** (ef, M values) | Better recall/speed trade-off | Low |

---

# 14. SECURITY CONSIDERATIONS

## 14.1 Current Security Measures

| Area | Implementation |
|------|---------------|
| **API Key Storage** | Environment variables via `.env` file (not in code) |
| **Input Validation** | Pydantic models enforce types/constraints on all API inputs |
| **Path Traversal Prevention** | `resolve()` + `relative_to()` prevents escaping repo root |
| **Query Parameterization** | All Cypher queries use `$parameters` (not string interpolation) |
| **Error Sanitization** | Traceback logged server-side; generic errors returned to client |
| **CORS** | Default FastAPI (same-origin only unless explicitly configured) |
| **File Permissions** | `.env` should be chmod 600 (user-readable only) |

## 14.2 Potential Vulnerabilities & Mitigations

### Path Traversal (Ingest Source)
**Risk**: User provides `source: "../../../../etc/passwd"` as local path.
**Current mitigation**: The code checks `p.exists() and p.is_dir()` – files are rejected. Only directories are accepted.
**Additional mitigation**: Could add an allowlist of permitted base directories.

### Cypher Injection
**Risk**: Malicious `qualified_name` or `repo_id` in requests.
**Mitigation**: All queries use parameterized queries (`$repo_id`, `$qn`), never string formatting. Neo4j parameters are type-safe.

### LLM Prompt Injection
**Risk**: If ingested code contains text like "Ignore previous instructions and...", the LLM might follow it.
**Mitigation**: 
- Context is clearly delimited in the prompt
- System message explicitly says "Use ONLY the provided context"
- Low temperature reduces susceptibility
- Could add output filtering for sensitive patterns

### Denial of Service
**Risk**: Ingesting a massive repo (millions of files) could exhaust memory/disk.
**Mitigation**: 
- Skip non-code directories
- Cap callees at 200 per function
- Cap mentions at 500 per file
- Timeouts on all operations
- Could add: file count limits, total size limits

### Sensitive Code Exposure
**Risk**: If deployed as a shared service, one user could query another's ingested repos.
**Mitigation**: 
- Currently single-tenant (no auth)
- `repo_id` isolation (can only query repos you know the ID of)
- For production: add authentication + authorization per repo

## 14.3 .env File Security

```bash
# .env.example (committed to git)
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password123  # Placeholder only!
LLM_API_KEY=PUT_YOUR_KEY_HERE

# .env (NOT committed – in .gitignore)
NEO4J_PASSWORD=actual_secret_here
LLM_API_KEY=gsk_actual_groq_key_here
```

**Best practices applied**:
- `.env` in `.gitignore`
- `.env.example` shows structure without real secrets
- `pydantic-settings` loads from environment (12-factor app principle)


---

# 15. TESTING STRATEGY

## 15.1 Current Test Files

| File | Purpose |
|------|---------|
| `test/test_query.py` | End-to-end query test against live backend |
| `test/test_chroma_setup.py` | Ingestion + ChromaDB verification |
| `test/test_rewrite.py` | Query rewriting chain test |
| `test/peek.py` | ChromaDB inspection utility |
| `test/reset_all.py` | Database reset utility |
| `test_ingest.py` | Remote repo ingestion test |

## 15.2 Testing Approaches for This Project

### Unit Testing (What Could Be Tested)
```python
# AST extraction
def test_extract_python_function():
    code = b"def hello(): pass"
    symbols, text, imports = extract_symbols(Path("test.py"), "test-repo", Path("."))
    assert len(symbols) == 1
    assert symbols[0].name == "hello"
    assert symbols[0].kind == "function"

# Identifier extraction
def test_extract_identifiers():
    result = extract_identifiers("def foo(): bar()")
    assert "foo" in result
    assert "bar" in result
    assert "def" in result  # Keywords are included

# Document ID generation
def test_doc_id_deterministic():
    id1 = _doc_id("repo", "path::func::0")
    id2 = _doc_id("repo", "path::func::0")
    assert id1 == id2  # Same input → same output
```

### Integration Testing
```python
# Neo4j operations
def test_upsert_and_query():
    neo4j = Neo4jClient.from_settings()
    neo4j.upsert_symbol(repo_id="test", kind="function", ...)
    # Verify with Cypher query

# End-to-end ingestion
def test_ingest_local_repo():
    result = ingest_repo(neo4j, "test-repo", "/path/to/small/repo", None)
    assert result.files_indexed > 0
    assert result.symbols_indexed > 0
```

### API Testing
```python
# FastAPI TestClient
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json()["ok"] == True

def test_ingest():
    response = client.post("/ingest", json={
        "repo_id": "test",
        "source": "/path/to/test/repo"
    })
    assert response.status_code == 200
```

## 15.3 Testing Challenges for This Project

| Challenge | Reason | Approach |
|-----------|--------|----------|
| **External dependencies** | Neo4j, ChromaDB, LLM API | Mock or use test containers |
| **Non-deterministic LLM output** | Different answers each run | Test for structure, not exact text |
| **Large test data** | Need real repos to test meaningfully | Use small fixture repos |
| **Embedding model loading** | Takes 5-10 seconds on first load | Cache in test fixtures |
| **Async code** | Query chain is async | Use `pytest-asyncio` |

## 15.4 Interview Answer: "How Would You Test This?"

> "I'd use a layered testing approach:
> 1. **Unit tests** for pure functions (AST extraction, identifier parsing, ID generation) – no mocking needed
> 2. **Integration tests** with Neo4j test containers and in-memory ChromaDB for data pipeline verification
> 3. **API tests** using FastAPI's TestClient with mocked service layer
> 4. **LLM tests** that verify the query plan structure (intent, search_vector fields exist) without checking exact text
> 5. **End-to-end smoke tests** against a small test repository to verify the full pipeline works
> 
> For CI, I'd skip LLM-dependent tests (mark with `@pytest.mark.llm`) and run them only in nightly builds with a real API key."

---

# 16. DEPLOYMENT & DEVOPS

## 16.1 Current Deployment Setup

This project runs locally with a simple batch script:

```batch
@echo off
echo Starting Backend...
start cmd /k ".\venv\Scripts\python.exe -m uvicorn app.main:app --port 8080"
echo Starting Frontend...
start cmd /k ".\venv\Scripts\python.exe -m streamlit run frontend\streamlit_app.py"
```

**Components**:
- Backend: Uvicorn on port 8080
- Frontend: Streamlit on port 8501 (default)
- Neo4j: External service (Neo4j Aura cloud or local Docker)
- ChromaDB: Embedded (file-based, `.chroma/` directory)
- LLM: External API (Groq cloud)

## 16.2 Production Deployment (How You'd Scale)

### Containerization (Docker)
```dockerfile
# Backend
FROM python:3.11-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app/ app/
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]

# Frontend
FROM python:3.11-slim
COPY requirements.txt .
RUN pip install streamlit requests
COPY frontend/ frontend/
CMD ["streamlit", "run", "frontend/streamlit_app.py", "--server.port=8501"]
```

### Architecture for Scale
```
Load Balancer (ALB)
├── Frontend (Streamlit) → ECS/Fargate, 2 replicas
└── Backend API (FastAPI) → ECS/Fargate, auto-scaling
    ├── Neo4j → Aura Enterprise (managed)
    ├── ChromaDB → Replace with Pinecone/Weaviate (managed vector DB)
    └── LLM → OpenAI/Anthropic API (or self-hosted vLLM)
```

### Key Scaling Considerations
1. **Vector store**: ChromaDB is single-process; for multi-worker, use Pinecone/Weaviate
2. **Neo4j**: Neo4j Aura Enterprise handles clustering automatically
3. **Embeddings**: For GPU-accelerated embeddings, run sentence-transformers on GPU instances
4. **LLM**: Rate limits are the bottleneck; implement request queuing
5. **Ingestion**: Can be moved to a background worker (Celery/SQS) for non-blocking

## 16.3 CI/CD Pipeline (Ideal)

```
Code Push → GitHub Actions
├── Lint (ruff/flake8)
├── Type check (mypy/pyright)
├── Unit tests (pytest)
├── Build Docker image
├── Integration tests (with Neo4j test container)
├── Push to ECR
└── Deploy to staging → smoke tests → deploy to production
```

## 16.4 Monitoring & Observability

| Layer | Tool | What to Monitor |
|-------|------|-----------------|
| Application | Structured logging (Python logging) | Ingestion counts, query latency, errors |
| API | FastAPI middleware | Request count, response times, status codes |
| Infrastructure | CloudWatch/Prometheus | CPU, memory, disk usage |
| Neo4j | Neo4j monitoring | Query times, connection pool, node/edge counts |
| LLM | Custom metrics | Token usage, latency, error rate |
| Vector Store | ChromaDB metrics | Collection sizes, query times |


---

# 17. IMPACT & RESULTS

## 17.1 Quantifiable Metrics

| Metric | Value | Context |
|--------|-------|---------|
| Languages supported | 5+ (Python, JS, TS, JSX, TSX) | Extensible to any tree-sitter grammar |
| Ingestion speed | ~50-100 files/minute | Depends on file size + embedding computation |
| Query response time | 3-8 seconds | Includes vector search + graph traversal + LLM |
| Context richness | 8-15 context items per query | Combines vector hits + graph neighbors + call flows |
| Graph depth | Up to 4-hop call chains | Deep enough for most architectural questions |
| Embedding dimension | 384 | Good balance of quality vs. storage |

## 17.2 Before vs After (Business Value)

| Without RepoRover | With RepoRover |
|-------------------|----------------|
| Read thousands of files manually to understand flow | Ask "How does X work?" and get an answer in seconds |
| grep for function names across files | Ask "Where is X used?" with graph-aware results |
| Draw architecture diagrams from memory | Explore the graph visually with actual call relationships |
| New dev onboarding takes 2-4 weeks | Ask questions and understand patterns in hours |
| Documentation is always outdated | Code graph is always current (re-ingest to update) |

## 17.3 Technical Achievements

1. **Hybrid retrieval outperforms pure vector**: By combining graph expansion with vector search, queries that ask about flows or relationships get significantly better context than vector-only approaches.

2. **LLM query rewriting improves relevance**: The HyDE technique (generating a hypothetical answer to search for) bridges vocabulary gaps between questions and code.

3. **Per-symbol embeddings beat file-level**: Retrieving the exact function needed (not a 500-line file containing it) means the LLM gets focused, relevant context.

4. **Intent-aware enrichment**: Different question types get different retrieval strategies, leading to more targeted answers.

---

# 18. STAR STORIES

## Story 1: Building the Hybrid Retrieval Pipeline

**Situation**: The initial version used only vector search (ChromaDB) to find relevant code. Questions about code flow ("How does authentication work?") returned individual functions but missed the connections between them.

**Task**: Improve retrieval quality so that flow-based questions return complete execution paths, not just isolated function snippets.

**Action**: 
- Designed a multi-hop graph expansion step that follows CALLS/DEFINES edges from vector search results
- Implemented intent detection (flow vs usage vs general) to apply different enrichment strategies
- Added call chain extraction (path enumeration in Neo4j) for flow questions
- Prioritized internal symbols over external library calls to keep context focused

**Result**: Flow-based queries now return complete execution paths (e.g., `login → authenticate → verify_password → bcrypt_compare`) with source code for each step. Context richness improved from 5 items to 12-15 items, and LLM answers became significantly more accurate for architectural questions.

---

## Story 2: Solving the AST Multi-Language Challenge

**Situation**: The project needed to parse Python, JavaScript, and TypeScript files, but each language has different syntax tree structures. A Python function is `function_definition`, while JS has `function_declaration`, `arrow_function`, and `method_definition`.

**Task**: Build a unified extraction layer that handles all supported languages without language-specific code paths for every case.

**Action**:
- Used tree-sitter as a language-agnostic parser (same API for all languages)
- Created a set-based node type collection: pass all relevant types at once
- Added special handling for JS/TS arrow functions (detected via `variable_declarator` pattern)
- Implemented iterative DFS (not recursion) for tree traversal to handle deeply nested code
- Used byte ranges from AST for exact code extraction

**Result**: Successfully parsing 5 language variants with a single 100-line extraction function. The iterative approach handles arbitrarily deep nesting without stack overflow. New languages can be added by just extending the `EXT_TO_LANG` mapping.

---

## Story 3: Designing the Query Rewriting Strategy

**Situation**: Direct vector search for user questions like "How does login work?" returned poor results because the embedding model wasn't trained on code-specific vocabulary. The question "login" didn't match functions named `authenticate_user` or `verify_credentials`.

**Task**: Bridge the vocabulary gap between natural language questions and technical code terminology.

**Action**:
- Researched the HyDE (Hypothetical Document Embeddings) technique
- Implemented an LLM-driven query rewriting step as the first stage of the pipeline
- Designed a structured output (QueryPlan) with intent classification and search vector
- Added graceful degradation: if rewriting fails, fall back to original question
- Prompt engineering: instructed the LLM to "generate a detailed technical description" rather than just keywords

**Result**: Retrieval relevance improved dramatically. "How does login work?" gets rewritten to "Implementation of user authentication, login routes, password verification, and session management" – which matches many more relevant functions in vector search. The intent classification also enables targeted enrichment strategies.

---

## Story 4: Handling Windows File Lock Issues

**Situation**: During development on Windows, the reset functionality kept failing because ChromaDB held SQLite file locks. Users couldn't reset their databases without restarting the server.

**Task**: Implement a robust reset mechanism that works even when files are locked.

**Action**:
- Implemented `VectorStore.clear_all()` to clear LRU caches before file deletion
- Created `_rmtree_force()` with an `onerror` handler that attempts to change permissions before retrying
- Added graceful reporting: the reset endpoint returns which directories were successfully deleted vs. which had lock issues
- Frontend shows both successes and failures clearly

**Result**: Reset works cleanly in most cases. When locks persist, users get clear feedback about what failed and can restart the server for a full clean. No more cryptic "PermissionError" crashes.

---

## Story 5: Scaling Context Assembly for Large Codebases

**Situation**: For large repos with thousands of functions, the graph expansion step could return hundreds of neighbors, creating contexts too large for the LLM's input window.

**Task**: Ensure context stays within LLM limits while preserving the most relevant information.

**Action**:
- Implemented priority-based context assembly (direct hits first, then intent-specific, then graph neighbors)
- Added caps at every level: 8 top hits, 50 graph neighbors, 20 call flows
- Text truncation: max 2500 chars per hit, only pull source for top 10 neighbors
- Separated internal vs external symbols, prioritizing project code over library calls
- Added deduplication to prevent the same symbol appearing multiple times

**Result**: Context stays under 15 items even for massive repos. The LLM receives focused, non-redundant context with the most relevant information positioned first. Answer quality improved because the LLM isn't overwhelmed with noise.


---

# 19. INTERVIEW QUESTIONS & ANSWERS

## Project-Specific Questions

### Q1: "Tell me about your project."

> "I built RepoRover, a GraphRAG-powered codebase intelligence system. It ingests code repositories by parsing ASTs with tree-sitter, builds a knowledge graph in Neo4j capturing function calls, class definitions, imports, and file relationships, creates per-symbol vector embeddings in ChromaDB, and then enables natural language Q&A.
>
> What makes it unique is the hybrid retrieval: when a user asks 'How does authentication work?', the system first rewrites the question using an LLM for better vector search, retrieves relevant symbols, then expands through the graph to find connected functions in the call chain. This gives much richer context than pure vector search.
>
> The backend is FastAPI with async support, the frontend is a Streamlit chat interface, and it supports Python, JavaScript, and TypeScript out of the box."

### Q2: "Why did you choose Neo4j over a relational database?"

> "Code relationships are inherently graph-shaped. A function calls another function, which is defined in a file, which imports modules. In SQL, representing these as joins would be expensive – a 4-hop call chain query would be a 4-way JOIN with exponential cost. In Neo4j, it's a simple variable-length path query: `(a)-[:CALLS*1..4]->(b)`.
>
> Neo4j uses index-free adjacency – each node stores direct pointers to its neighbors – making traversals O(1) per hop regardless of database size. For a code graph where we constantly traverse relationships, this is the right tool."

### Q3: "How does your retrieval pipeline work?"

> "It's a 5-step LCEL chain:
> 1. **Rewrite**: LLM transforms the question into a technical search vector (HyDE technique)
> 2. **Retrieve**: Vector search in ChromaDB finds semantically similar code
> 3. **Enrich**: Graph expansion follows CALLS/DEFINES edges 2 hops deep to find connected symbols
> 4. **Assemble**: All context is joined with priority ordering
> 5. **Generate**: LLM synthesizes the answer from combined context
>
> The enrichment step is what makes it GraphRAG – we don't just find similar code, we find structurally connected code that completes the picture."

### Q4: "What's the difference between your approach and just using ChatGPT with code pasted in?"

> "Three key differences:
> 1. **Scale**: ChatGPT can't process a 10,000-file repo. We index it once and query any part.
> 2. **Structure awareness**: We know the call graph. 'How does login work?' returns the actual execution path, not just similar-looking code.
> 3. **Precision**: Per-symbol embeddings mean we retrieve the exact function needed, not a 500-line file that happens to contain it.
>
> Think of it as giving ChatGPT a perfectly curated context window every time, with exactly the code it needs to answer correctly."

### Q5: "How do you handle different programming languages?"

> "I use tree-sitter, which provides a uniform parsing API across 40+ languages. Each language has a grammar that produces an AST, and while node types differ (Python's `function_definition` vs JS's `function_declaration`), I handle this with a set-based approach – pass all relevant types at once.
>
> For JavaScript/TypeScript, there's special handling for arrow functions (detected via the `variable_declarator` → `arrow_function` AST pattern). Adding a new language is just extending the `EXT_TO_LANG` mapping and potentially adding its node types to the collection set."

### Q6: "What would you change if you could start over?"

> "Three things:
> 1. **Incremental indexing** – Currently re-ingestion processes all files. I'd add a hash-based change detection to only re-process modified files.
> 2. **Streaming LLM responses** – The 3-8 second wait for answers hurts UX. Streaming would show the answer as it generates.
> 3. **Better call resolution** – Static analysis misses dynamic dispatch. I'd explore type inference (e.g., pytype for Python) to resolve `self.method()` to the correct class."

### Q7: "How do you ensure the LLM doesn't hallucinate?"

> "Multiple layers:
> 1. The system prompt says 'Use ONLY the provided context. If unsure, say what is missing.'
> 2. Low temperature (0.2) reduces creative generation
> 3. Context is grounded in actual source code (not just descriptions)
> 4. The 'Context Only' mode lets users see exactly what the LLM received
> 5. Answers include a context_items count so users know how much evidence was found
>
> It can still hallucinate, but the grounding significantly reduces it compared to asking an LLM with no context."

### Q8: "How would you scale this for 1000 concurrent users?"

> "Several changes:
> 1. Replace ChromaDB with a managed vector database (Pinecone/Weaviate) that supports concurrent reads
> 2. Add a Redis/Memcached layer for caching frequent queries
> 3. Move ingestion to background workers (Celery/SQS) so it doesn't block the API
> 4. Use Neo4j Aura Enterprise with read replicas for query scaling
> 5. Add authentication + per-user/per-org repo isolation
> 6. Implement rate limiting on LLM calls (most expensive bottleneck)
> 7. Deploy on auto-scaling container infrastructure (ECS/K8s)
>
> The architecture already separates concerns cleanly, so each component can be scaled independently."

### Q9: "What's the most complex piece of code in this project?"

> "The `_enrich_step` in the query chain. It handles three different intent strategies, performs graph expansion with deduplication, priority ordering (internal vs external symbols), and pulls source code for neighbors. It's where vector search meets graph traversal meets intent-aware logic.
>
> The complexity comes from balancing completeness (more context = better answers) with constraints (LLM context window, response time). Every list is capped, every path is bounded, and results are deduplicated and priority-sorted."

### Q10: "How do you handle errors gracefully?"

> "At every level:
> - **No LLM key**: System still works – returns raw context without generating answers
> - **LLM failure**: Returns error message, doesn't crash the pipeline
> - **Query rewrite failure**: Falls back to using the original question as search vector
> - **Neo4j unreachable**: Caught at startup with a clear error message about common causes
> - **Ingestion failure**: Returns HTTP 500 with traceback (logged server-side)
> - **ChromaDB file locks**: Graceful reporting of what succeeded vs failed
>
> The philosophy is: never crash, always degrade gracefully, always give the user actionable information."


---

# 20. TECH STACK INTERVIEW QUESTIONS

## 20.1 Python Questions

### Q: "What's the difference between `dataclass` and Pydantic `BaseModel`?"
> "Both generate boilerplate (init, repr, eq), but:
> - `dataclass` is standard library, zero overhead, no validation
> - Pydantic validates types at runtime, supports serialization/deserialization, environment loading
> - In this project, I use `dataclass(frozen=True)` for internal DTOs (Symbol, IndexedFile) and Pydantic for API schemas (IngestRequest, QueryResponse) that need validation"

### Q: "What's `@lru_cache` and when would you use it?"
> "`lru_cache` memoizes function results. It caches return values keyed by arguments. I use it for:
> - Singletons (maxsize=1): embedding model, LLM client
> - Per-key caching (maxsize=32): ChromaDB client per repo_id
> - It's thread-safe in CPython and supports `cache_clear()` for testing"

### Q: "Explain async/await in Python."
> "Async enables cooperative multitasking. When a coroutine hits `await` (I/O, network), it yields control to the event loop which runs other tasks. This project uses async for:
> - Vector search (`retriever.ainvoke()`)
> - LLM calls (`model.ainvoke()`)
> - The query chain steps
> 
> Ingestion is sync because it's CPU-bound (AST parsing) and sequential, so async wouldn't help."

### Q: "What's a context manager and how does `asynccontextmanager` work?"
> "Context managers handle setup/teardown with `with` blocks. `asynccontextmanager` enables `async with`:
> ```python
> @asynccontextmanager
> async def lifespan(app):
>     neo4j = Neo4jClient.from_settings()  # Setup
>     yield                                  # App runs
>     neo4j.close()                         # Teardown
> ```
> FastAPI's lifespan manages the app lifecycle – Neo4j connects at startup, disconnects at shutdown."

### Q: "How does Python's `hashlib` work?"
> "It provides cryptographic hash functions. I use SHA-256 to generate deterministic document IDs:
> ```python
> hashlib.sha256(f'{repo_id}|{qn}'.encode()).hexdigest()[:24]
> ```
> Same input always produces same hash, collision probability is negligible for 24 hex chars (96 bits)."

## 20.2 FastAPI Questions

### Q: "How does FastAPI's dependency injection work?"
> "You define functions or classes as dependencies, and FastAPI calls them per-request:
> ```python
> def get_db(request: Request):
>     return request.app.state.neo4j
>
> @router.post('/query')
> async def query(req: QueryRequest, db = Depends(get_db)):
>     ...
> ```
> In this project, I access Neo4j via `request.app.state.neo4j` directly (simpler for a single shared resource)."

### Q: "How does Pydantic validation work?"
> "Pydantic models validate input at construction time:
> - Type coercion (str '8' → int 8)
> - Required/optional fields
> - Custom validators
> - Automatic 422 response for invalid API requests
> - In this project: `top_k: int = 8` means optional with default, validated as integer"

### Q: "What's the difference between ASGI and WSGI?"
> "WSGI (Flask/Django) is synchronous – one thread per request. ASGI (FastAPI/Starlette) is async – one event loop handles many concurrent requests. For this project, async matters because:
> - LLM API calls take 2-5 seconds (don't want to block a thread)
> - Neo4j queries are I/O-bound
> - Multiple users can query simultaneously"

## 20.3 Neo4j / Graph Database Questions

### Q: "What's the property graph model?"
> "Nodes have labels (types) and properties (key-value pairs). Relationships have types, direction, and properties. Unlike RDF (triples), property graphs are more intuitive for application developers.
> Example: `(:Function {name: 'login'})-[:CALLS]->(:Function {name: 'hash_password'})`"

### Q: "What's the difference between BFS and DFS in graph traversal?"
> "BFS explores level by level (uses queue), DFS goes deep first (uses stack). Neo4j's variable-length patterns (`*1..4`) use BFS internally to find shortest paths. For my AST traversal, I use iterative DFS because order doesn't matter and it uses less memory."

### Q: "What are graph indexes?"
> "In Neo4j, constraints automatically create indexes. My constraints:
> - `File.path IS UNIQUE` → B-tree index on path
> - `(Function.repo_id, Function.qualified_name) IS UNIQUE` → compound index
> - These make MERGE and MATCH operations O(log n) instead of O(n)"

### Q: "Explain MERGE vs CREATE in Cypher."
> "CREATE always creates a new node (might create duplicates). MERGE is 'find or create' – it matches an existing node or creates one if not found. I use MERGE exclusively for idempotent ingestion – re-ingesting a repo updates existing nodes instead of creating duplicates."

## 20.4 Vector Database / Embeddings Questions

### Q: "What are vector embeddings?"
> "Dense numerical representations of text in a high-dimensional space (384D for MiniLM-L6). Similar texts have vectors that are close together (high cosine similarity). I embed code symbols so that 'authenticate_user' and 'verify_credentials' are close in embedding space, even though they have different names."

### Q: "What's cosine similarity vs euclidean distance?"
> "Cosine similarity measures the angle between vectors (ignores magnitude). Euclidean measures absolute distance. I use cosine because:
> 1. Normalized embeddings mean cosine = dot product (fast)
> 2. Document length doesn't affect similarity (magnitude is 1)
> 3. Works better for text similarity in practice"

### Q: "What's HNSW and why is it used?"
> "Hierarchical Navigable Small World – an algorithm for approximate nearest neighbor (ANN) search. Instead of comparing against every vector (O(n)), HNSW builds a multi-layer graph where each layer has skip connections. Search is O(log n). ChromaDB uses HNSW internally.
> Trade-off: Not exact (approximate), but 95-99% recall with 100x faster search."

### Q: "Why per-symbol embeddings instead of per-file?"
> "Precision. A 500-line file might have 20 functions. If I search for 'password hashing', I want the specific `hash_password()` function, not the entire auth.py file. Per-symbol embedding means each retrieval result is exactly one logical unit (function or class)."

## 20.5 LLM / RAG Questions

### Q: "What's the difference between fine-tuning and RAG?"
> "Fine-tuning changes the model's weights (training). RAG doesn't change the model – it provides relevant context at inference time.
> - Fine-tuning: expensive, needs data, can't update dynamically
> - RAG: cheap, real-time updates (just re-index), no training needed
> For a codebase that changes daily, RAG is the right approach."

### Q: "What's prompt engineering?"
> "Designing instructions for LLMs to get desired behavior. In this project:
> - System prompt: defines role ('You are RepoRover'), constraints ('Use ONLY provided context')
> - Few-shot examples: showing the LLM how to rewrite queries
> - Structured output: format instructions for JSON parsing
> - Temperature control: 0.2 for factual, 0.8 for creative"

### Q: "What's HyDE (Hypothetical Document Embeddings)?"
> "Instead of searching with the user's question directly, generate what the answer would look like, then search for documents similar to that hypothetical answer. In this project:
> - User asks: 'How does login work?'
> - LLM generates: 'Implementation of user authentication, login routes, password verification...'
> - We search for code similar to that description
> This works because the hypothetical answer contains the technical vocabulary that matches the actual code."

### Q: "How do you prevent hallucination in RAG?"
> "Multiple techniques:
> 1. Explicit instruction: 'Use ONLY provided context'
> 2. Low temperature (reduces creative generation)
> 3. Source attribution (context_items count shows evidence)
> 4. Fallback messaging: 'If unsure, say what is missing'
> 5. Context-only mode for verification
> Can't eliminate hallucination completely, but these significantly reduce it."

## 20.6 Streamlit Questions

### Q: "How does Streamlit's execution model work?"
> "Streamlit reruns the entire script top-to-bottom on every user interaction. This is different from React (virtual DOM, selective re-renders). To preserve state across reruns, you use `st.session_state`. The trade-off is simplicity over performance – fine for data apps, not ideal for high-interactivity UIs."

### Q: "How do you handle chat state in Streamlit?"
> "Chat messages are stored in `st.session_state.chat_messages` (a list of dicts with role, content, context_items). On each rerun, we iterate and render all messages. After adding a new message, `st.rerun()` forces a fresh render to show the update."

## 20.7 Git / GitPython Questions

### Q: "How does Git cloning work programmatically?"
> "Using GitPython: `Repo.clone_from(url, target_dir, branch=branch)`. It's equivalent to `git clone --branch X url target`. I handle:
> - Cleanup of existing target directory (force delete with permission changes)
> - Branch selection (optional)
> - Error handling for network failures
> - Registry mapping for later access to the cloned path"


---

# 21. SYSTEM DESIGN INTERVIEW ANGLES

## 21.1 "Design a Code Search System" (Classic System Design)

If asked to design a code search system, you can draw from this project:

### Requirements
- Ingest repositories (local/remote)
- Search by natural language queries
- Understand code relationships (who calls whom)
- Return relevant code with explanations
- Multi-language support
- Sub-5-second query latency

### High-Level Design
```
┌─────────────┐     ┌────────────────┐     ┌─────────────┐
│  Ingestion  │────▶│  Storage Layer │◀────│   Query     │
│  Pipeline   │     │                │     │   Engine    │
└─────────────┘     └────────────────┘     └─────────────┘
       │                    │                      │
  ┌────┴────┐        ┌─────┴─────┐         ┌─────┴─────┐
  │AST Parse│        │Graph DB   │         │Vector DB   │
  │Embed    │        │Vector DB  │         │Graph DB    │
  │Graph    │        │File Store │         │LLM API     │
  └─────────┘        └───────────┘         └───────────┘
```

### Scale Estimation
- 1000 repos × 1000 files × 10 symbols = 10M symbols
- 384D embeddings × 4 bytes × 10M = ~15GB vector storage
- Graph: 10M nodes + 50M edges ≈ 5GB in Neo4j
- Query rate: 100 QPS (if shared service)

### Key Design Decisions (from this project)
1. **Per-symbol vs per-file embedding**: Per-symbol for precision
2. **Graph model**: Property graph with typed relationships
3. **Retrieval strategy**: Hybrid (vector + graph)
4. **Chunking**: Language-aware text splitting
5. **Caching**: Model instances, per-repo stores

## 21.2 "Design a RAG System" (Common ML System Design)

### Components
1. **Document Processing Pipeline**
   - Ingestion → Parsing → Chunking → Embedding → Storage
2. **Retrieval Engine**
   - Query understanding → Vector search → Re-ranking → Context assembly
3. **Generation Engine**
   - Prompt construction → LLM invocation → Output parsing

### Metrics to Discuss
- **Retrieval**: Precision@K, Recall@K, MRR (Mean Reciprocal Rank)
- **Generation**: Faithfulness (grounded in context), relevance, completeness
- **System**: Latency (p50, p99), throughput, cost per query

### Trade-offs to Discuss
| Decision | Option A | Option B | This Project |
|----------|----------|----------|--------------|
| Embedding location | Cloud API | Local model | Local (MiniLM) |
| Chunk size | Small (500) | Large (2000) | 2000 (code needs more context) |
| Re-ranking | Cross-encoder | Graph-based | Graph expansion |
| Context limit | Fixed window | Dynamic | Priority-ordered with caps |

## 21.3 "Design a Knowledge Graph System"

### Graph Modeling Principles (from this project)
1. **Nodes represent entities** (File, Function, Class, Import)
2. **Edges represent relationships** (DEFINES, CALLS, MENTIONS, IMPORTS)
3. **Properties store attributes** (name, qualified_name, repo_id)
4. **Constraints ensure integrity** (uniqueness on key combinations)

### Graph Queries for Code Intelligence
```cypher
-- Dependency analysis: What does module X depend on?
MATCH (f:File {path: $path})-[:IMPORTS]->(i:Import)
RETURN i.value

-- Impact analysis: What would break if I change function X?
MATCH (caller)-[:CALLS]->(f:Function {name: $name})
RETURN caller.name, caller.file_path

-- Architecture view: Which modules are tightly coupled?
MATCH (f1:File)-[:DEFINES]->(s1)-[:CALLS]->(s2)<-[:DEFINES]-(f2:File)
WHERE f1 <> f2
RETURN f1.path, f2.path, count(*) AS coupling
ORDER BY coupling DESC
```

## 21.4 "How Would You Make This Production-Ready?"

### Checklist
- [ ] **Authentication**: JWT/OAuth2 for API access
- [ ] **Authorization**: Per-user/per-org repo isolation
- [ ] **Rate limiting**: Token bucket on LLM calls
- [ ] **Monitoring**: Prometheus metrics + Grafana dashboards
- [ ] **Logging**: Structured JSON logs with correlation IDs
- [ ] **Error tracking**: Sentry or equivalent
- [ ] **Health checks**: Deep health (check Neo4j + ChromaDB + LLM connectivity)
- [ ] **Graceful shutdown**: Drain connections, finish in-flight requests
- [ ] **Horizontal scaling**: Stateless API servers + managed databases
- [ ] **CI/CD**: Automated testing + deployment pipeline
- [ ] **Data backup**: Neo4j snapshots, ChromaDB exports
- [ ] **Cost optimization**: Embedding caching, query deduplication

## 21.5 Concurrency & Parallelism Discussion

### Current Model
- **FastAPI**: Single async event loop (handles concurrent requests via await)
- **Ingestion**: Synchronous, sequential (one file at a time)
- **Query**: Async steps interleaved on event loop

### Scaling Options
| Component | Current | Scaled |
|-----------|---------|--------|
| API Server | 1 Uvicorn worker | Multiple workers (--workers 4) |
| Ingestion | Sequential | Parallel with multiprocessing/thread pool |
| Embedding | Sync, one at a time | Batch embedding (sentence-transformers supports batches) |
| Neo4j writes | One-at-a-time session | Transaction batching |
| Vector search | Single-threaded | Multiple readers (managed vector DB) |

### Bottleneck Analysis
1. **LLM API** (3-5s per call) – Rate limited by provider
2. **Embedding generation** (during ingestion) – CPU/GPU bound
3. **Neo4j writes** (during ingestion) – Network + transaction overhead
4. **Vector search** (during query) – Fast (<100ms for ChromaDB)
5. **Graph traversal** (during query) – Fast (<50ms for bounded queries)

The **LLM call is always the bottleneck** for query latency. Everything else is fast.


---

# 22. QUESTIONS YOU CAN ASK INTERVIEWERS

## About Their Architecture
1. "How do you handle code search/discovery internally? Do you use any graph-based approaches?"
2. "Do you use any vector databases or embedding-based retrieval in your stack?"
3. "How do you manage knowledge graphs at scale? What's your node/edge count like?"
4. "What's your experience with LLM integration in production? How do you handle latency and cost?"

## About Their RAG/ML Systems
5. "How do you handle RAG retrieval quality? Do you have evaluation pipelines for relevance?"
6. "What's your approach to prompt management? Do you version-control prompts?"
7. "How do you handle LLM hallucination in production? What guardrails do you use?"

## About Their Code/Development
8. "How do you onboard new developers to your codebase? Is there tooling for code understanding?"
9. "What's your approach to code documentation? Is any of it auto-generated?"
10. "Do you use graph databases for any use cases? What's been your experience with Neo4j vs alternatives?"

## About Scale & Operations
11. "What's your concurrent user count for similar ML-powered features?"
12. "How do you handle model updates and embedding re-computation at scale?"
13. "What does your CI/CD look like for ML/AI services?"

---

# APPENDIX A: FILE-BY-FILE REFERENCE

## Backend Files

| File | Lines | Purpose |
|------|-------|---------|
| `app/main.py` | ~45 | FastAPI app creation, lifespan, router registration |
| `app/core/settings.py` | ~30 | Pydantic settings (env loading), all config in one place |
| `app/api/schemas.py` | ~35 | Pydantic models for all request/response types |
| `app/api/routers/health.py` | ~12 | GET /health endpoint |
| `app/api/routers/ingest.py` | ~22 | POST /ingest endpoint with error handling |
| `app/api/routers/query.py` | ~20 | POST /query and /query/context endpoints |
| `app/api/routers/graph.py` | ~50 | POST /graph/explore with Cypher query |
| `app/api/routers/test.py` | ~60 | Peek + reset endpoints (dev-only) |
| `app/infrastructure/neo4j_client.py` | ~100 | All Neo4j operations (CRUD + schema init) |
| `app/infrastructure/vector_store.py` | ~65 | ChromaDB wrapper (upsert, search, lookup) |
| `app/ingestion/ast_extract.py` | ~140 | tree-sitter AST parsing for all languages |
| `app/ingestion/indexer.py` | ~50 | Directory walking + file indexing |
| `app/ingestion/text_scan.py` | ~10 | Regex identifier extraction |
| `app/llm/chat_model.py` | ~15 | LLM client factory (cached singleton) |
| `app/query/chain.py` | ~170 | The full LCEL query pipeline (5 steps) |
| `app/query/graph_context.py` | ~80 | Graph traversal helpers (expand, mentions, flows) |
| `app/repos/registry.py` | ~35 | JSON-based repo_id → path registry |
| `app/repos/sources.py` | ~60 | Git clone + local path resolution |
| `app/services/ingest_service.py` | ~110 | Orchestrates full ingestion pipeline |
| `app/services/query_service.py` | ~30 | Thin wrapper around query chain |

## Frontend Files

| File | Lines | Purpose |
|------|-------|---------|
| `frontend/streamlit_app.py` | ~40 | Entry point, page config, page router |
| `frontend/theme.py` | ~280 | Full custom CSS (dark mode) |
| `frontend/api_client.py` | ~95 | HTTP client for all backend endpoints |
| `frontend/components/sidebar.py` | ~100 | Navigation + backend status display |
| `frontend/pages/home.py` | ~95 | Landing page with feature cards |
| `frontend/pages/ingest.py` | ~95 | Ingest form + peek utility |
| `frontend/pages/query.py` | ~100 | Chat interface for Q&A |
| `frontend/pages/graph_explorer.py` | ~130 | Graph visualization + Mermaid rendering |
| `frontend/pages/settings_page.py` | ~120 | Health check + danger zone + about |

---

# APPENDIX B: QUICK REFERENCE CARD

## One-Liners for Common Interview Questions

| Question | Quick Answer |
|----------|-------------|
| "What does your project do?" | "GraphRAG for codebases – ingest repos, build knowledge graph, ask questions in natural language" |
| "What's the hardest part?" | "Hybrid retrieval: combining vector similarity with graph traversal to get complete execution context" |
| "What technology impressed you?" | "tree-sitter – one library parses 40+ languages with byte-accurate ranges, no compilation needed" |
| "What's your biggest technical decision?" | "Per-symbol embeddings (not per-file) – 10x better retrieval precision for code" |
| "How do you ensure quality?" | "LLM query rewriting bridges vocabulary gaps; graph expansion provides structural completeness" |
| "What would you do differently?" | "Add incremental indexing, streaming LLM responses, and better type inference for call resolution" |
| "What's the scale?" | "Handles repos with thousands of files; query latency 3-8s; designed for single-tenant but architecture supports multi-tenant" |

---

# APPENDIX C: CONCEPT GLOSSARY

| Term | Definition | Used In This Project |
|------|-----------|---------------------|
| **RAG** | Retrieval-Augmented Generation – provide context to LLM at query time | Core architecture pattern |
| **GraphRAG** | RAG enhanced with knowledge graph traversal | The hybrid retrieval approach |
| **AST** | Abstract Syntax Tree – parsed code structure | tree-sitter extracts functions/classes |
| **LCEL** | LangChain Expression Language – composable chain syntax | Query pipeline built with `\|` operator |
| **HyDE** | Hypothetical Document Embeddings – generate answer to search for it | Query rewriting step |
| **HNSW** | Hierarchical Navigable Small World – ANN algorithm | ChromaDB's internal search |
| **ANN** | Approximate Nearest Neighbor – fast similarity search | Vector retrieval |
| **Cypher** | Neo4j's query language | All graph operations |
| **ASGI** | Async Server Gateway Interface | FastAPI/Uvicorn |
| **MERGE** | Cypher upsert operation | Idempotent node creation |
| **Qualified Name** | Unique identifier for a code symbol | `path::name::byte_offset` |
| **Cosine Similarity** | Angle-based distance metric for vectors | Embedding comparison |
| **Frozen Dataclass** | Immutable Python data structure | All DTOs in this project |
| **Lifespan** | FastAPI startup/shutdown hook | Neo4j connection management |
| **Session State** | Streamlit's persistent storage mechanism | Chat history, config |

---

# APPENDIX D: COMMON FOLLOW-UP QUESTIONS

## "Walk me through a request..."

### Ingestion Request Flow:
```
HTTP POST /ingest {"repo_id": "x", "source": "/path"}
  → FastAPI validates with Pydantic (IngestRequest)
  → Router calls ingest_service.ingest_repo()
  → resolve_repo_source() → local path or git clone
  → index_repo() → os.walk → tree-sitter parse per file
  → neo4j.upsert_file/symbol/calls/mentions/imports
  → VectorStore.upsert_documents() → ChromaDB
  → Return IngestResult → serialize to IngestResponse JSON
```

### Query Request Flow:
```
HTTP POST /query {"repo_id": "x", "question": "How does login work?"}
  → FastAPI validates (QueryRequest)
  → Router calls query_service.answer_question()
  → run_repo_query_chain() starts LCEL chain
  → _rewrite_step: LLM → QueryPlan(search_vector=..., intent="flow")
  → _retrieve_step: ChromaDB search → 8 hits
  → _enrich_step: Neo4j expand + call flows
  → _assemble_step: join 12 context parts
  → _llm_step: LLM generates answer
  → Return (answer, 12) → serialize to QueryResponse JSON
```

## "What happens when X fails?"

| Failure | Behavior |
|---------|----------|
| Neo4j unreachable at startup | RuntimeError with helpful message (check URI, VPN, etc.) |
| Neo4j unreachable during query | Graph enrichment returns empty list; answer still generated from vector hits |
| LLM API timeout | Returns "LLM error: timeout" as the answer |
| LLM returns unparseable output (rewrite) | Falls back to original question as search vector |
| ChromaDB collection doesn't exist | Empty retrieval results; answer says "no context found" |
| Git clone fails | HTTP 500 with the clone error message |
| File permission error during ingestion | Skips that file, continues with others |
| Frontend can't reach backend | Shows "API Offline" badge in sidebar |

---

# APPENDIX E: TECHNOLOGY COMPARISON TABLE

## Why X over Y?

| Our Choice | Alternative | Why We Chose Ours |
|------------|-------------|-------------------|
| Neo4j | PostgreSQL + JOINs | O(1) per hop traversal; Cypher is expressive for paths |
| Neo4j | Amazon Neptune | Neptune is RDF-focused; Neo4j has better developer experience |
| ChromaDB | Pinecone | Free, local, no network dependency; good for MVP |
| ChromaDB | FAISS | ChromaDB has metadata filtering, persistence, LangChain integration |
| tree-sitter | Python `ast` module | Multi-language; `ast` is Python-only |
| tree-sitter | Babel (JS) | tree-sitter handles all languages; Babel is JS-only |
| FastAPI | Flask | Async support, Pydantic validation, auto-docs |
| FastAPI | Django | Too heavy for an API-only service |
| Streamlit | React + Next.js | 10x faster to build; Python ecosystem alignment |
| Streamlit | Gradio | Better customization, multi-page support |
| sentence-transformers | OpenAI embeddings | Free, local, fast, privacy |
| Groq (LLM) | OpenAI GPT-4 | Free tier, fast inference, good Llama models |
| LangChain | Custom code | LCEL composability, output parsers, many integrations |
| LangChain | LlamaIndex | More control over retrieval pipeline with LangChain |

---

*End of Study Guide*

**Total Technical Concepts Covered**: 50+
**Interview Questions Prepared**: 30+
**STAR Stories Ready**: 5
**Design Patterns Identified**: 12
**Architecture Decisions Documented**: 8

**Tip**: Read this guide section by section over 3-4 days. Practice explaining each concept out loud. For each section, think about:
1. What would I ask if I were the interviewer?
2. Can I draw this on a whiteboard?
3. What's the trade-off I made and why?
