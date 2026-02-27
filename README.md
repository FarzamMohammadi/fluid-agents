# customizable-agents

An agentic application exploring how far CopilotKit and assistant-ui can be pushed for dynamic, customized agent interactions.

**Backend:** PydanticAI + AG-UI — orchestrator agent, two sub-agents, tools, one SSE endpoint
**Frontend:** CopilotKit and assistant-ui — two pages, same backend
**Goal:** Understand which frontend library fits which use case — and how far you can push each for deeply customized, unbounded agentic interactions

---

## What's Inside

```
server/     Python backend — FastAPI + PydanticAI + AG-UI
web/        Vite + React frontend — two pages, two libraries
docs/       Architecture and build roadmap
```

## Docs

- [`docs/architecture.md`](./docs/architecture.md) — system design, agent topology, event flow, integration points
- [`docs/roadmap.md`](./docs/roadmap.md) — phased build plan

## Stack

| Layer | Technology |
|-------|-----------|
| Agent framework | PydanticAI |
| Agent-UI protocol | AG-UI (SSE event stream) |
| LLM gateway | OpenRouter |
| Web search | DuckDuckGo (free, no API key) |
| Frontend lib A | CopilotKit |
| Frontend lib B | assistant-ui |
| Frontend meta | Vite + React + TypeScript |

## Setup

```bash
# Backend
cd server && uv sync
cp .env.example .env   # add OPENROUTER_API_KEY
uv run uvicorn src.main:app --reload --port 8000

# Frontend
cd web && pnpm install
pnpm dev
```
