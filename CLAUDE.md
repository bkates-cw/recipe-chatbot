# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An AI evaluations course built around a Recipe Chatbot. The chatbot is a FastAPI app backed by LiteLLM; the course has 5 progressive homework assignments (HW1-HW5) teaching practical AI evaluation techniques: prompt engineering, error analysis, LLM-as-judge, RAG/retrieval evaluation, and agent failure analysis.

## Commands

```bash
# Install dependencies (uses uv, not pip)
uv sync

# Run the chatbot server (auto-reloads on .py and .md changes)
uv run uvicorn backend.main:app --reload --reload-include '*.md'
# App at http://127.0.0.1:8000

# Bulk test queries against the chatbot
uv run python scripts/bulk_test.py                          # uses data/sample_queries.csv
uv run python scripts/bulk_test.py --csv path/to/file.csv   # custom CSV (columns: id, query)

# Run annotation tool (FastHTML-based, separate from main app)
uv run python annotation/annotation.py

# Run homework walkthrough notebooks
cd homeworks/hw2 && jupyter notebook hw2_walkthrough.ipynb
```

There is no test suite or linter configured in the project.

## Architecture

**Request flow:** Frontend (`frontend/index.html`) → `POST /chat` (`backend/main.py`) → `get_agent_response()` (`backend/utils.py`) → LiteLLM completion → response saved as JSON trace in `annotation/traces/`.

Key details:
- `backend/utils.py` loads `backend/system_prompt.md` at import time and prepends it as the system message. Editing `system_prompt.md` changes chatbot behavior (server reloads pick up changes via `--reload-include '*.md'`).
- LLM calls go through **LiteLLM** — model is configured via `MODEL_NAME` env var using LiteLLM's `provider/model` format (e.g., `openai/gpt-4.1-mini`, `anthropic/claude-3-sonnet-20240229`).
- Every `/chat` request saves a full request/response trace as timestamped JSON in `annotation/traces/`.

**Retrieval system** (`backend/retrieval.py`): BM25-based recipe search using `rank_bm25`. Used in HW4. `RecipeRetriever` class loads recipes from JSON, builds/caches a pickle index. The `backend/query_rewrite_agent.py` provides LLM-powered query rewriting with three strategies: keywords, rewrite, expand.

**Evaluation utilities** (`backend/evaluation_utils.py`): `BaseRetrievalEvaluator` class computing IR metrics (Recall@k, MRR) for retrieval evaluation.

**Lesson directories** (`lesson-4/`, `lesson-7/`, `lesson-8/`): Standalone scripts for in-class exercises (log cleaning, LLM judge substantiation, labeling tools, model cascades). Not part of the main app.

## Environment

Requires Python >=3.13. Copy `env.example` to `.env` and set:
- `MODEL_NAME` — chatbot model (LiteLLM format)
- `MODEL_NAME_JUDGE` — judge model for evals (can be cheaper)
- API keys: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc.

## Key Libraries

- **LiteLLM** for model-agnostic LLM calls
- **judgy** for LLM-as-judge evaluation (HW3)
- **rank_bm25** for BM25 retrieval (HW4)
- **FastAPI + uvicorn** for the backend
- **pandas, scikit-learn, matplotlib, seaborn, plotly** for analysis in notebooks
