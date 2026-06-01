# Getting Started — NESsT Knowledge Navigator

This guide gets a new contributor up and running in under 10 minutes.

---

## Prerequisites

| Requirement | Version / Notes |
|-------------|-----------------|
| **Python** | 3.9 or higher |
| **pip** | Latest recommended |
| **Cisco Playground access** | JWT token for API access |
| **Git** | For cloning the repo |

No Node.js or npm required — frontend is vanilla HTML/CSS/JS served by the backend.

---

## Step 1: Clone the repository

```bash
git clone <your-repo-url>
cd Nesst_Hackathon
```

---

## Step 2: Create and activate virtual environment

```bash
python3 -m venv venv
source venv/bin/activate   # macOS/Linux
# OR
venv\Scripts\activate     # Windows
```

---

## Step 3: Install dependencies

```bash
pip install -r backend/requirements.txt
```

---

## Step 4: Configure environment

```bash
cp .env.example .env
```

Edit `.env` and set at minimum:

| Variable | Required | Description |
|----------|----------|-------------|
| `PLAYGROUND_JWT_TOKEN` | Yes | Your Cisco Playground JWT token |
| `PLAYGROUND_API_URL` | No | Default: `https://cxai-playground.cisco.com/v1` |
| `LLM_MODEL` | No | Default: `gpt-5.2` |
| `EMBEDDING_MODEL` | No | Default: `text-embedding-3-large` |

All other variables in `.env.example` are optional.

---

## Step 5: Run the application

```bash
python -m backend.main
```

Or with auto-reload (development):

```bash
uvicorn backend.main:app --reload --port 8000
```

Open **http://localhost:8000** in your browser.

---

## Step 6: Verify it works

1. **Health check** — Visit http://localhost:8000/api/health
2. **Documents tab** — Upload a PDF or DOCX (e.g., any NESsT report)
3. **Search tab** — Ask a question about the uploaded content

---

## Project layout (where things live)

| You want to… | Go to… |
|--------------|--------|
| Add or change API routes | `backend/main.py` |
| Change RAG retrieval or prompts | `backend/retrieval/rag_engine.py` |
| Add new document parsers | `backend/ingestion/parser.py` |
| Change chunking logic | `backend/ingestion/chunker.py` |
| Change embeddings model | `backend/config.py` + `.env` |
| Add or modify frontend UI | `frontend/index.html`, `frontend/app.js`, `frontend/style.css` |
| Add memo template sections | Upload via UI or `backend/template_store.py` |
| Configure environment | `.env` and `backend/config.py` |

---

## Data directories (auto-created)

| Directory | Purpose |
|-----------|---------|
| `documents/` | Uploaded knowledge-base files |
| `templates/` | Uploaded memo templates (DOCX, Excel) |
| `chroma_db/` | ChromaDB vector store (persisted) |
| `nesst_history.db` | SQLite DB for conversations and document registry |

These are in `.gitignore` — do not commit them.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError` | Run from project root; ensure `venv` is activated and `pip install -r backend/requirements.txt` ran |
| `401` or `Authentication` errors | Check `PLAYGROUND_JWT_TOKEN` in `.env`; regenerate token if expired |
| Port 8000 in use | Use `--port 8001` (or another port) with uvicorn |
| ChromaDB errors | Delete `chroma_db/` and re-upload documents to re-index |
| Document upload fails | Ensure file is PDF, DOCX, PPTX, TXT, CSV, or MD; check file size limits |

---

## Next steps

- [Architecture (HLD/LLD)](ARCHITECTURE.md) — Design and module dependencies
- [Solution Documentation](SOLUTION_DOCUMENTATION.md) — API reference, config, flows
- [CONTRIBUTING.md](../CONTRIBUTING.md) — Development workflow and conventions
