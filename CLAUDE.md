# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Run the app:**
```bash
./run.sh
# or manually:
uv sync
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Environment setup:** Copy `.env.example` to `.env` and add `ANTHROPIC_API_KEY`.

- Web UI: `http://localhost:8000`
- API docs: `http://localhost:8000/docs`

No test suite is currently configured.

## Architecture

This is a RAG (Retrieval-Augmented Generation) chatbot that answers questions about course materials using semantic search + Claude AI.

**Stack:** Python 3.13 / FastAPI backend, vanilla JS frontend, ChromaDB vector store, Anthropic Claude API.

**Query flow:**
1. Frontend sends query + session ID → `POST /api/query`
2. `RAGSystem` coordinates: loads conversation history from `SessionManager`, invokes `AIGenerator`
3. `AIGenerator` runs an agentic loop with Claude (`claude-sonnet-4-20250514`): Claude calls the `search_course_content` tool
4. Tool call hits `VectorStore` (ChromaDB, SentenceTransformer `all-MiniLM-L6-v2` embeddings) for semantic search
5. Results returned to Claude, which generates the final answer
6. Response + cited sources sent back to frontend

**Key backend files:**
- `backend/rag_system.py` — top-level orchestrator; entry point for `query()` and document loading
- `backend/ai_generator.py` — Claude API integration and agentic tool loop
- `backend/vector_store.py` — ChromaDB wrapper; two collections: `course_catalog` and `course_content`
- `backend/document_processor.py` — parses `.txt`/`.pdf`/`.docx` course files into chunks (800 chars, 100 overlap)
- `backend/search_tools.py` — `search_course_content` tool definition and `ToolManager`
- `backend/session_manager.py` — per-session conversation history (last 2 messages kept)
- `backend/config.py` — all tuneable parameters (model, chunk size, search limits, etc.)

**Document format:** Files in `docs/` follow a structured format with course title, link, and instructor on the first lines, then lessons as `Lesson #: Title` headers. `DocumentProcessor` parses this structure to build metadata alongside content chunks.
