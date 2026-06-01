# NESsT Knowledge Navigator — Architecture (HLD & LLD)

This document provides High-Level Design (HLD) and Low-Level Design (LLD) diagrams to support long-term maintenance after the hackathon.

---

## 1. High-Level Design (HLD)

### 1.1 System Context

```mermaid
flowchart TB
    subgraph External["External Services"]
        CP[Cisco Playground API<br/>LLM + Embeddings]
    end

    subgraph NKN["NESsT Knowledge Navigator"]
        FE[Frontend<br/>HTML/JS/CSS]
        API[FastAPI Backend]
    end

    subgraph Storage["Local Storage"]
        CHROMA[(ChromaDB<br/>Vector Store)]
        SQL[(SQLite<br/>Conversations, Documents)]
        FILES[(Filesystem<br/>documents/, templates/)]
    end

    User([User / Admin]) --> FE
    FE -->|REST API| API
    API --> CP
    API --> CHROMA
    API --> SQL
    API --> FILES
```

### 1.2 Top-Level Component View

```mermaid
flowchart LR
    subgraph Frontend
        UI[UI Layer]
    end

    subgraph Backend
        API[API Layer<br/>main.py]
        RAG[RAG Pipeline<br/>rag_engine]
        ING[Ingestion Pipeline<br/>parser, chunker, embedder]
        MEMO[Memo Generator<br/>memo_generator]
        TPL[Template Store<br/>template_store]
    end

    subgraph Data
        VS[Vector Store<br/>ChromaDB]
        DB[(SQLite)]
        DOCS[Documents]
        TPLS[Templates]
    end

    UI --> API
    API --> RAG
    API --> ING
    API --> MEMO
    API --> TPL
    RAG --> VS
    ING --> VS
    ING --> DOCS
    MEMO --> RAG
    MEMO --> TPL
    TPL --> TPLS
    API --> DB
```

### 1.3 Data Flow — Query (RAG)

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant API as FastAPI
    participant RAG as RAG Engine
    participant EMB as Embedder
    participant VS as Vector Store
    participant LLM as Cisco Playground

    U->>FE: Query + mode
    FE->>API: POST /api/query
    API->>RAG: query_knowledge()
    RAG->>EMB: embed(query)
    EMB->>LLM: embeddings API
    LLM-->>EMB: vector
    RAG->>VS: query(vector, filters, audience)
    VS-->>RAG: top-k chunks
    RAG->>LLM: chat completion (context + query)
    LLM-->>RAG: answer
    RAG-->>API: {answer, sources}
    API->>API: Save to SQLite
    API-->>FE: QueryResponse
    FE-->>U: Display answer
```

### 1.4 Data Flow — Document Ingestion

```mermaid
sequenceDiagram
    participant U as Admin
    participant FE as Frontend
    participant API as FastAPI
    participant Parser as Parser
    participant Chunker as Chunker
    participant Embedder as Embedder
    participant VS as Vector Store
    participant DB as SQLite

    U->>FE: Upload file/ZIP
    FE->>API: POST /api/documents/upload
    API->>API: Save to documents/
    API->>DB: Insert Document (processing)
    API->>Parser: parse_document()
    Parser-->>API: {pages/sections, full_text}
    API->>Chunker: chunk_document()
    Chunker-->>API: chunks
    API->>Embedder: generate_embeddings()
    Embedder-->>API: embeddings
    API->>VS: add_documents()
    VS-->>API: count
    API->>DB: Update Document (completed)
    API-->>FE: UploadResponse
```

### 1.5 Data Flow — Memo Generation

```mermaid
sequenceDiagram
    participant U as Admin
    participant FE as Frontend
    participant API as FastAPI
    participant TPL as Template Store
    participant MEMO as Memo Generator
    participant RAG as RAG Engine

    U->>FE: Select template, company, Generate
    FE->>API: POST /api/memo/generate
    API->>TPL: get_template(id)
    TPL-->>API: {sections}
    API->>MEMO: generate_from_template_sections()
    loop For each section
        MEMO->>RAG: query_knowledge(section)
        RAG-->>MEMO: answer
    end
    MEMO->>MEMO: Build DOCX + TXT
    MEMO-->>API: (docx_bytes, txt)
    API->>API: ZIP output
    API-->>FE: ZIP download
```

---

## 2. Low-Level Design (LLD)

### 2.1 Backend Module Dependencies

```mermaid
flowchart TB
    subgraph main["main.py - API Layer"]
        routes[Routes]
    end

    subgraph models
        database[database.py<br/>Conversation, Message, Document]
        schemas[schemas.py<br/>Pydantic models]
    end

    subgraph ingestion
        parser[parser.py<br/>parse_document]
        chunker[chunker.py<br/>chunk_document]
        embedder[embedder.py<br/>generate_embeddings]
    end

    subgraph retrieval
        vector_store[vector_store.py<br/>add, query, delete]
        rag_engine[rag_engine.py<br/>query_knowledge]
    end

    subgraph memo
        template_store[template_store.py<br/>save, list, get]
        memo_generator[memo_generator.py<br/>generate_from_template_sections]
    end

    routes --> database
    routes --> schemas
    routes --> parser
    routes --> chunker
    routes --> embedder
    routes --> vector_store
    routes --> rag_engine
    routes --> template_store
    routes --> memo_generator

    rag_engine --> embedder
    rag_engine --> vector_store
    memo_generator --> rag_engine
    memo_generator --> template_store
```

### 2.2 Directory & File Map

```mermaid
flowchart LR
    subgraph backend
        main[main.py]
        config[config.py]
        parser[ingestion/parser.py]
        chunker[ingestion/chunker.py]
        embedder[ingestion/embedder.py]
        vs[retrieval/vector_store.py]
        rag[retrieval/rag_engine.py]
        tpl[template_store.py]
        memo[memo_generator.py]
        db[models/database.py]
        schemas[models/schemas.py]
    end

    main --> config
    main --> parser
    main --> chunker
    main --> embedder
    main --> vs
    main --> rag
    main --> tpl
    main --> memo
    main --> db
    main --> schemas
```

### 2.3 Database Schema (SQLite)

```mermaid
erDiagram
    Conversation ||--o{ Message : has
    Document }o--|| ChromaDB : "chunks stored"

    Conversation {
        string id PK
        string title
        datetime created_at
        datetime updated_at
    }

    Message {
        int id PK
        string conversation_id FK
        string role
        text content
        datetime created_at
        text metadata_json
    }

    Document {
        string id PK
        string filename UK
        string file_path
        string file_type
        int file_size_bytes
        string status
        int num_chunks
        string language
        int num_pages
        string audience
        datetime created_at
    }
```

### 2.4 ChromaDB Metadata (per chunk)

| Field | Type | Description |
|-------|------|-------------|
| `source_file` | str | Original filename |
| `page` | int | Page number (PDF) |
| `section` | str | Section heading (DOCX/PPTX) |
| `slide` | int | Slide number (PPTX) |
| `audience` | str | `public` or `internal` |

### 2.5 Template Store Structure

```mermaid
flowchart TB
    subgraph TemplatesDir["templates/"]
        T1[template_xxx.docx]
        T2[template_yyy.xlsx]
        meta[templates_meta.json]
    end

    meta -->|entries| T1
    meta -->|entries| T2

    meta_content["templates_meta.json:\n  - id, filename, stored_name\n  - path, sections"]
```

### 2.6 Key Functions Reference

| Module | Function | Purpose |
|--------|----------|---------|
| `main.py` | `handle_query` | Query → RAG → save conversation |
| `main.py` | `upload_document` | Parse → chunk → embed → store |
| `main.py` | `upload_zip` | Extract ZIP, ingest supported files |
| `main.py` | `upload_template` | Save DOCX/Excel, parse sections |
| `main.py` | `generate_memo` | Call memo_generator, return ZIP |
| `rag_engine.py` | `query_knowledge` | Embed → retrieve → LLM → answer |
| `vector_store.py` | `query` | ChromaDB similarity search |
| `vector_store.py` | `add_documents` | Add chunks with embeddings |
| `template_store.py` | `parse_template_sections` | DOCX headings or Excel col A |
| `memo_generator.py` | `generate_from_template_sections` | RAG per section, build DOCX |

---

## 3. Deployment Considerations

### 3.1 Single-Process Deployment

- **Uvicorn** runs FastAPI in a single process
- **ChromaDB** uses file-based persistence; one writer
- **SQLite** single-writer; sufficient for hackathon scale

### 3.2 Future Scope & Azure-Ready Adjustments

*For full future scope (auth, analytics, integrations, etc.), see `docs/SOLUTION_DOCUMENTATION.md` §10.*

| Component | Current | Azure-Ready |
|-----------|---------|-------------|
| Auth | Header-based role | Azure AD / OAuth2 |
| Secrets | `.env` | Azure Key Vault |
| DB | SQLite | Azure SQL / PostgreSQL |
| Vector Store | ChromaDB file | Azure AI Search / Pinecone |
| Static | Local mount | Azure Blob / CDN |

---

## 4. Quick Reference for Maintainers

| Task | Files to Edit |
|------|---------------|
| Add API endpoint | `backend/main.py` |
| Change RAG behavior | `backend/retrieval/rag_engine.py` |
| Support new document type | `backend/ingestion/parser.py` |
| Change chunking | `backend/ingestion/chunker.py` |
| Add template format | `backend/template_store.py` |
| Change memo layout | `backend/memo_generator.py` |
| Add DB table/column | `backend/models/database.py` |
| Change API schemas | `backend/models/schemas.py` |
| Frontend behavior | `frontend/app.js` |
| Styling | `frontend/style.css` |
| Config / env vars | `backend/config.py`, `.env` |

---

*Last updated: March 2025*
