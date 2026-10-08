# ORACLE

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Next.js](https://img.shields.io/badge/Next.js-16-000000?style=flat&logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Orchestration-1C3C3C?style=flat)](https://github.com/langchain-ai/langgraph)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-Queue%20%26%20Cache-DC382D?style=flat&logo=redis&logoColor=white)](https://redis.io/)
[![Celery](https://img.shields.io/badge/Celery-Distributed%20Tasks-37814A?style=flat&logo=celery&logoColor=white)](https://docs.celeryq.dev/)
[![LangSmith](https://img.shields.io/badge/LangSmith-Tracing-0052CC?style=flat)](https://smith.langchain.com/)

A multi-agent research intelligence system. Given a complex user query, ORACLE plans research subtasks, executes them in parallel across specialized agents, verifies claims through an automated fact-checking pipeline, resolves academic citations, and generates structured research reports with full human-in-the-loop oversight.

---

## Architecture

```
                              ┌──────────────────┐
   User Query ───────────────▶│    Supervisor    │
                              └────────┬─────────┘
                                       │ Decomposes query into subtasks
                                       ▼
                              ┌──────────────────┐
                              │   Human Review   │◀── User approves / edits plan
                              │  (interrupt())   │    (LangGraph HITL)
                              └────────┬─────────┘
                       ┌───────────────┼───────────────┬───────────────┐
                       ▼               ▼               ▼               ▼
                  Web Search      PDF Reader       Code Exec      Fact Checker
                   (Tavily)       (PyMuPDF +      (Sandboxed     (Explicit plan
                                  pdfplumber)     subprocess)     claims)
                       └───────────────┼───────────────┴───────────────┘
                                       ▼
                              ┌──────────────────┐
                              │ Synthesis Agent  │
                              └────────┬─────────┘
                                       ▼
                              ┌──────────────────┐
                              │ Fact-Check Pass  │──▶ Automated claim verification
                              └────────┬─────────┘
                                       ▼
                              ┌──────────────────┐
                              │Citation Formatter│──▶ CrossRef metadata resolution
                              └────────┬─────────┘    & confidence scoring
                                       ▼
                         Final Structured Report
```

Every agent decision, tool execution, token usage, and latency metric is traced end-to-end via LangSmith.

---

## Agent Pipeline

- **Supervisor**: Analyzes the query, splits research requirements into discrete subtasks, and revises execution plans based on user feedback.
- **Human Review**: Implements LangGraph `interrupt()` checkpoints to allow interactive approval or modification of research plans before agent execution.
- **Web Search Agent**: Executes targeted web queries using Tavily and extracts relevant excerpts.
- **PDF Extraction Agent**: Parses local or remote research documents with PyMuPDF (fast layout/text parsing) and `pdfplumber` (table extraction).
- **Code Execution Agent**: Runs generated Python snippets inside a constrained subprocess sandbox (memory limits, timeout, and import restrictions) for quantitative calculations.
- **Fact-Checking Pass**: Cross-verifies synthesized claims against external evidence before report finalization.
- **Citation Formatter**: Resolves publication metadata through CrossRef and generates a calibrated confidence score.

---

## Prerequisites

The system requires three API keys:
- **OpenRouter**: Model inference with automated fallback handling.
- **Tavily**: Search engine designed for LLM agents.
- **LangSmith**: Execution tracing and evaluation tracking.

Refer to [`docs/GET_API_KEYS.md`](docs/GET_API_KEYS.md) for step-by-step key acquisition instructions.

---

## Quick Start

### 1. Terminal / CLI Mode (Agent Graph Only)

Run the research pipeline directly from the command line without setting up background workers:

```bash
cd backend
python3.12 -m venv .venv
source .venv/bin/activate    # On Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt

cp .env.example .env         # Add your OPENROUTER_API_KEY, TAVILY_API_KEY, and LANGSMITH_API_KEY
python run_local.py "What are the primary architectures for retrieval-augmented generation?"
```

### 2. Full Stack (API, PostgreSQL, Redis, Celery)

Deploy the backend services via Docker Compose:

```bash
cp backend/.env.example backend/.env
docker compose up -d postgres redis
docker compose run --rm api alembic upgrade head
docker compose up -d
```

- **Interactive API Docs (Swagger UI)**: `http://localhost:8000/docs`
- **Health Check**: `http://localhost:8000/health`
- See [`docs/RUNNING_BACKEND.md`](docs/RUNNING_BACKEND.md) for API usage and curl examples.

### 3. Frontend Web Interface

The frontend runs on Next.js 16 with React 19:

```bash
cd frontend
cp .env.example .env.local
npm install
npm run dev
```

Open `http://localhost:3000` in your browser.

---

## Evaluation

To evaluate report quality and factual consistency against benchmark datasets using deterministic metrics and LLM-as-a-judge:

```bash
cd backend
python -m eval.run_eval
```

For detailed benchmark specifications, see [`docs/RUNNING_EVALUATION.md`](docs/RUNNING_EVALUATION.md).

---

## Documentation

- [`docs/PROJECT_OVERVIEW.md`](docs/PROJECT_OVERVIEW.md) — Architectural specifications and system design rationale.
- [`docs/GET_API_KEYS.md`](docs/GET_API_KEYS.md) — API credential setup guide.
- [`docs/RUNNING_LOCALLY.md`](docs/RUNNING_LOCALLY.md) — Local development and environment configuration.
- [`docs/RUNNING_BACKEND.md`](docs/RUNNING_BACKEND.md) — Backend API documentation and testing commands.
- [`docs/RUNNING_FRONTEND.md`](docs/RUNNING_FRONTEND.md) — Frontend architecture and UI state flow.
- [`docs/RUNNING_EVALUATION.md`](docs/RUNNING_EVALUATION.md) — LangSmith evaluation harness and rubric.
- [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) — Production deployment guides for Render and Supabase.
