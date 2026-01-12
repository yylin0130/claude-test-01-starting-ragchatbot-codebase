# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a RAG (Retrieval-Augmented Generation) chatbot for querying course materials. Users ask natural language questions and receive AI-generated responses powered by semantic search over course content.

## Commands

This project uses `uv` for dependency management. Always use `uv run` to execute Python commands.

**Start the application:**
```bash
./run.sh
# or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

**Install/sync dependencies:**
```bash
uv sync
```

**Add a new dependency:**
```bash
uv add <package-name>
```

**Run a Python file:**
```bash
uv run python <file.py>
```

**Access points:**
- Web UI: http://localhost:8000
- API docs: http://localhost:8000/docs

## Architecture

### Backend Flow (Query Processing)

```
POST /api/query (app.py)
    → RAGSystem.query() (rag_system.py)
        → AIGenerator.generate_response() (ai_generator.py)
            → Claude API with tool definitions
            → If tool_use: execute CourseSearchTool
                → VectorStore.search() (vector_store.py)
                    → ChromaDB semantic search
            → Return final response with sources
```

### Key Components

- **rag_system.py**: Central orchestrator that coordinates all components
- **ai_generator.py**: Claude API wrapper with tool-calling support; handles the agentic loop where Claude decides when to search
- **vector_store.py**: ChromaDB integration with two collections: `course_catalog` (metadata) and `course_content` (chunks)
- **search_tools.py**: Tool definitions for Claude's tool-use; `CourseSearchTool` performs semantic search and tracks sources
- **document_processor.py**: Parses course files (Title on line 1, optional Link/Instructor, then content with "Lesson N:" markers)
- **session_manager.py**: Conversation history per session

### Frontend

Vanilla HTML/CSS/JS in `frontend/`. Uses Marked.js for markdown rendering. Communicates via `/api/query` endpoint.

### Data Flow

1. Course TXT files in `docs/` are parsed and chunked on startup
2. Chunks stored in ChromaDB with embeddings (all-MiniLM-L6-v2)
3. User queries trigger semantic search via Claude's tool-calling
4. Claude synthesizes search results into responses

## Configuration

Settings in `backend/config.py`:
- `CHUNK_SIZE`: 800 chars
- `CHUNK_OVERLAP`: 100 chars
- `MAX_RESULTS`: 5 search results
- `MAX_HISTORY`: 2 conversation turns

Requires `ANTHROPIC_API_KEY` in `.env` file.
