# NESsT Knowledge Navigator — Solution Documentation

## 1. Overview

**NESsT Knowledge Navigator** is an AI-powered RAG (Retrieval-Augmented Generation) application that makes NESsT’s 28 years of institutional knowledge (reports, case studies, investment memos, donor documents) searchable and usable for structured outputs such as loan memos.

### 1.1 Key Capabilities


| Capability               | Description                                                   |
| ------------------------ | ------------------------------------------------------------- |
| **Search**               | Natural-language queries with source citations                |
| **Analyze**              | Cross-document analysis, patterns, cause-effect relationships |
| **Summarize**            | Executive briefings and synthesized summaries                 |
| **Document Ingestion**   | PDF, DOCX, PPTX, TXT, CSV, MD; single-file or ZIP upload      |
| **Memo Generator**       | Fill DOCX/Excel templates with RAG-generated content          |
| **Audience Control**     | Public vs internal documents; server-side enforcement         |
| **Conversation History** | Chat sessions with export to DOCX                              |


### 1.2 Tech Stack


| Layer            | Technology                                  |
| ---------------- | ------------------------------------------- |
| Backend          | Python 3.9+, FastAPI                        |
| Vector Store     | ChromaDB (persistent)                       |
| LLM & Embeddings | Cisco Playground (OpenAI-compatible)        |
| Database         | SQLite (conversations, document registry)   |
| Frontend         | Vanilla JS, HTML5, CSS3                     |
| Document Parsing | PyMuPDF, python-docx, python-pptx, openpyxl |


---

## 2. Project Structure

```
Nesst_Hackathon/
├── backend/
│   ├── main.py              # FastAPI app, API routes
│   ├── config.py            # Settings (env vars, pydantic-settings)
│   ├── models/
│   │   ├── database.py      # SQLAlchemy models (Conversation, Message, Document)
│   │   └── schemas.py       # Pydantic request/response schemas
│   ├── ingestion/
│   │   ├── parser.py        # Parse PDF, DOCX, PPTX, TXT, CSV, MD
│   │   ├── chunker.py       # Chunk documents with section/page metadata
│   │   └── embedder.py      # Generate embeddings via Cisco Playground
│   ├── retrieval/
│   │   ├── vector_store.py  # ChromaDB wrapper (add, query, delete, audience)
│   │   └── rag_engine.py    # RAG pipeline: embed → retrieve → LLM synthesize
│   ├── template_store.py    # Template upload, parse sections (DOCX/Excel)
│   ├── memo_generator.py    # Generate filled memo from template sections
│   └── requirements.txt     # Python dependencies
├── frontend/
│   ├── index.html           # SPA shell, all tabs
│   ├── app.js               # Query, documents, templates, memo generation
│   └── style.css            # Styling
├── documents/               # Uploaded knowledge-base documents
├── templates/               # Uploaded memo templates (DOCX, Excel)
├── chroma_db/               # ChromaDB persisted vectors
├── nesst_history.db         # SQLite (conversations, documents)
├── .env                     # API keys, model config (not committed)
└── docs/
    ├── SOLUTION_DOCUMENTATION.md   # This file
    ├── ARCHITECTURE.md             # HLD/LLD diagrams
    └── HACKATHON_WINNING_CASE.md   # Pitch / winning case
```

---

## 3. Configuration

### 3.1 Environment Variables


| Variable               | Description                                | Default                                       |
| ---------------------- | ------------------------------------------ | --------------------------------------------- |
| `PLAYGROUND_API_URL`   | Cisco Playground OpenAI-compatible API URL | `https://cxai-playground.cisco.com/openai/v1` |
| `PLAYGROUND_JWT_TOKEN` | JWT for Cisco Playground                   | Required                                      |
| `LLM_MODEL`            | LLM model name                             | `gpt-5.2`                                     |
| `EMBEDDING_MODEL`      | Embedding model                            | `text-embedding-ada-002`                      |
| `EMBEDDING_DIMENSIONS` | Embedding vector size                      | `1536`                                        |
| `CHUNK_SIZE`           | Chunk size (chars)                         | `1000`                                        |
| `CHUNK_OVERLAP`        | Overlap between chunks                     | `200`                                         |
| `CHROMA_PERSIST_DIR`   | ChromaDB data directory                    | `./chroma_db`                                 |
| `DOCUMENTS_DIR`        | Uploaded documents directory               | `./documents`                                 |
| `TEMPLATES_DIR`        | Memo templates directory                   | `./templates`                                 |


### 3.2 Run Locally

```bash
# Install dependencies
pip install -r backend/requirements.txt

# Set .env (PLAYGROUND_JWT_TOKEN, etc.)
# Start server
uvicorn backend.main:app --reload --port 8000

# Open http://localhost:8000
```

---

## 4. API Reference

### 4.1 Query


| Endpoint     | Method | Description                                                                                                                                               |
| ------------ | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/api/query` | POST   | Natural language query. Body: `query`, `mode` (search/analyze/summarize), `top_k`, `filters`, `model`. Header: `X-User-Role: public` for audience filter. |


### 4.2 Conversations


| Endpoint                         | Method | Description                    |
| -------------------------------- | ------ | ------------------------------ |
| `/api/conversations`             | GET    | List conversations             |
| `/api/conversations/{id}`        | GET    | Get conversation with messages |
| `/api/conversations/{id}`        | DELETE | Delete conversation            |
| `/api/conversations/{id}`        | PATCH  | Rename (body: `title`)         |
| `/api/conversations/{id}/export` | GET    | Export conversation as DOCX    |


### 4.3 Documents


| Endpoint                         | Method | Description                        |
| -------------------------------- | ------ | ---------------------------------- |
| `/api/documents`                 | GET    | List ingested documents            |
| `/api/documents/upload`          | POST   | Upload single document             |
| `/api/documents/upload-zip`      | POST   | Upload ZIP; ingest supported files |
| `/api/documents/{id}/audience`   | PATCH  | Set audience (public/internal)     |
| `/api/documents/{id}`            | DELETE | Delete document from KB            |
| `/api/documents/file/{filename}` | GET    | Download document file             |


### 4.4 Templates


| Endpoint                | Method | Description                   |
| ----------------------- | ------ | ----------------------------- |
| `/api/templates/upload` | POST   | Upload DOCX or Excel template |
| `/api/templates`        | GET    | List templates                |


### 4.5 Memo Generation


| Endpoint             | Method | Description                                                                                                                                          |
| -------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/api/memo/generate` | POST   | Generate memo. Body: `company_name`, `template_id` (required), `source_filter` (optional). Returns ZIP with `.docx` memo + `Descriptive_Answer.txt`. |


### 4.6 Health & Models


| Endpoint      | Method | Description                     |
| ------------- | ------ | ------------------------------- |
| `/api/health` | GET    | Health check                    |
| `/api/models` | GET    | Available LLM models            |
| `/api/stats`  | GET    | Collection stats (chunks, docs) |


---

## 5. Data Flow

### 5.1 Document Ingestion

1. File uploaded (single or extracted from ZIP)
2. **Parser** extracts text and structure (pages, sections, slides)
3. **Chunker** splits into chunks with metadata (source_file, page, section, audience)
4. **Embedder** generates embeddings via Cisco Playground
5. **Vector Store** stores chunks in ChromaDB
6. **Document** record saved in SQLite

### 5.2 Query (RAG)

1. User query + mode (search/analyze/summarize)
2. **Embedder** embeds query
3. **Vector Store** retrieves top-k chunks (with audience filter if `X-User-Role: public`)
4. **RAG Engine** builds prompt with context, calls LLM
5. LLM returns synthesized answer with citations
6. Conversation + messages saved in SQLite

### 5.3 Memo Generation

1. Admin uploads template (DOCX headings or Excel column A as sections)
2. **Template Store** parses sections, saves file and metadata
3. User selects template, enters company name, clicks Generate
4. **Memo Generator** for each section: RAG query → fill content
5. Builds DOCX with header table, Executive Summary, sections
6. Returns ZIP: memo.docx + Descriptive_Answer.txt

---

## 6. Audience & Security

- **Audience**: Each document has `audience` = `public` or `internal`
- **Filtering**: When `X-User-Role: public`, only chunks with `audience=public` are retrieved
- **Admin**: Full access; no audience filter applied
- **Enforcement**: Server-side via header; client cannot override

---

## 7. Template Formats


| Format            | Sections From                                 |
| ----------------- | --------------------------------------------- |
| **DOCX**          | Heading 1, Heading 2, etc. (paragraph styles) |
| **Excel (.xlsx)** | First column (column A) of first sheet        |


Output memo is always DOCX. Excel templates provide section structure only; output is Word.

---

## 8. Maintenance Notes

- **ChromaDB**: Persisted in `chroma_db/`; survives restarts
- **SQLite**: `nesst_history.db` holds conversations and document registry
- **Documents**: Physical files in `documents/`; deleting via API removes from ChromaDB and DB
- **Templates**: Stored in `templates/`; metadata in `templates/templates_meta.json`
- **Embeddings**: If switching embedding model, re-ingest all documents
- **Logs**: Check uvicorn/FastAPI logs for errors; ChromaDB and SQLite are file-based

---

## 9. Troubleshooting


| Issue                 | Likely Cause                         | Action                                                                    |
| --------------------- | ------------------------------------ | ------------------------------------------------------------------------- |
| 401 / Auth errors     | Invalid or missing JWT               | Check `PLAYGROUND_JWT_TOKEN` in `.env`                                    |
| No results for query  | Empty ChromaDB or audience filter    | Verify documents ingested; check `X-User-Role`                            |
| Memo generation fails | Template not found or empty sections | Re-upload template; ensure DOCX has headings or Excel has column A labels |
| Duplicate filename    | Document already in KB               | Delete existing or use different filename                                 |
| Embedding rate limit  | Cisco Playground limits              | Embedder uses retry with exponential backoff                              |


---

## 10. Future Scope

### 10.1 Near-Term (3–6 months)


| Area                | Capability                  | Notes                                                       |
| ------------------- | --------------------------- | ----------------------------------------------------------- |
| **Auth & Security** | Azure AD / SSO              | Replace header-based role with OAuth2; per-user permissions |
| **Auth**            | API keys / service accounts | Support programmatic access for integrations                |
| **Audit**           | Query and access logging    | Log who queried what, when; retention policy                |
| **Templates**       | L1-style Excel output       | Generate filled Excel workbooks for L1 workflow             |
| **Templates**       | Multi-sheet Excel parsing   | Use sheet names or multiple columns as sections             |


### 10.2 Medium-Term (6–12 months)


| Area               | Capability                           | Notes                                               |
| ------------------ | ------------------------------------ | --------------------------------------------------- |
| **Deployment**     | Azure App Service / Container        | Production hosting on Azure                         |
| **Data**           | Azure SQL / PostgreSQL               | Replace SQLite for scale and HA                     |
| **Vector Store**   | Azure AI Search or managed vector DB | Scale beyond ChromaDB file storage                  |
| **Analytics**      | Usage dashboard                      | Queries by mode, document popularity, user activity |
| **Feedback**       | Thumbs up/down on answers            | Store and use for model or prompt tuning            |
| **Multi-language** | Language detection & prioritization  | Surface results by language preference              |
| **Scheduling**     | Batch memo generation                | Queue memos for multiple companies                  |


### 10.3 Long-Term


| Area             | Capability                         | Notes                                            |
| ---------------- | ---------------------------------- | ------------------------------------------------ |
| **Integrations** | SharePoint / OneDrive sync         | Ingest from NESsT’s document repositories        |
| **Integrations** | Microsoft 365 / Teams              | Native add-in or bot for inline queries          |
| **RAG**          | Hybrid search (keyword + semantic) | Combine BM25 with vector search                  |
| **RAG**          | Re-ranking                         | Two-stage retrieval for higher precision         |
| **Generation**   | Streaming responses                | Stream answer tokens for lower perceived latency |
| **Governance**   | Data retention, deletion           | Policy-driven retention and compliance           |


### 10.4 Technical Debt & Cleanup

- Migrate frontend to a framework (e.g. React/Vue) for maintainability
- Add API versioning (`/api/v1/...`)
- Introduce integration and E2E tests
- Add OpenAPI/Swagger examples for all endpoints
- Document deployment runbook and rollback procedure

---

## 11. References

- **Architecture diagrams**: See `docs/ARCHITECTURE.md`
- **Hackathon pitch**: See `docs/HACKATHON_WINNING_CASE.md`
- **Cisco Playground**: [Cisco AI documentation](https://developer.cisco.com/)
- **ChromaDB**: [ChromaDB docs](https://docs.trychroma.com/)
- **FastAPI**: [FastAPI docs](https://fastapi.tiangolo.com/)

