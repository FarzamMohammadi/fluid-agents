# Architecture

## System Overview

```
┌─────────────────────────────────────┐
│  Browser                            │
│                                     │
│  /copilotkit   →  CopilotKit        │
│  /assistant-ui →  assistant-ui      │
└──────────────┬──────────────────────┘
               │  POST /agent  (AG-UI SSE)
┌──────────────▼──────────────────────┐
│  Server  (FastAPI)                  │
│                                     │
│  main_agent (orchestrator)          │
│    ├── researcher_agent (sub)       │
│    └── analyst_agent    (sub)       │
│                                     │
│  Tools: web search, data card,      │
│         delegate research,          │
│         analyze findings            │
└─────────────────────────────────────┘
```

One endpoint: `POST /agent`. No auth. No database.

---

## Backend

### Stack
- Python + FastAPI
- PydanticAI for agents
- AG-UI adapter (`pydantic_ai.ext.ag_ui`) to stream events over SSE
- DuckDuckGo for free web search
- OpenRouter as the LLM gateway (OpenAI-compat, any model)

### Agents

**`main_agent`** — the orchestrator. The one the frontend talks to directly.
Coordinates the other two. Has tools. Streams text and tool calls back to the client.

**`researcher_agent`** — sub-agent 1. Called by `main_agent` via a tool.
Does focused web research: searches DuckDuckGo, returns structured findings.

**`analyst_agent`** — sub-agent 2. Called by `main_agent` via a tool.
Takes research findings, synthesizes them, returns a structured recommendation.
The one that ideally emits STATE events so the frontend can show live structured data.

### Tools (on `main_agent`)

| Tool | What it does | Frontend effect |
|------|-------------|-----------------|
| `web_search` | Direct DuckDuckGo search | TOOL_CALL events — frontend shows raw results |
| `delegate_research` | Runs `researcher_agent` | TOOL_CALL — frontend renders a research card |
| `analyze_findings` | Runs `analyst_agent` | TOOL_CALL — frontend renders an analysis card |
| `show_data_card` | Returns a structured card | TOOL_CALL — frontend renders a rich info card (explicit UI side effect) |
| `get_current_time` | Returns current time | Simple TOOL_CALL to see the full lifecycle clearly |

The sub-agent tools (`delegate_research`, `analyze_findings`) are where you discover:
- Do sub-agent events propagate into the parent stream?
- Can tools emit STATE or CUSTOM events mid-execution?
Read the PydanticAI + AG-UI adapter source to find out.

### AG-UI Event Flow

When a user sends a message, the stream looks like this:

```
RUN_STARTED
THINKING_START / THINKING_* (if model supports it — depends on OpenRouter model)
TEXT_MESSAGE_START / TEXT_MESSAGE_CONTENT / TEXT_MESSAGE_END
TOOL_CALL_START → TOOL_CALL_ARGS (streaming JSON) → TOOL_CALL_END → TOOL_CALL_RESULT
... (more text or more tools)
RUN_FINISHED
```

Optional (if you implement them):
```
STATE_SNAPSHOT   ← structured agent state (drives Layer 2 panel in assistant-ui)
STATE_DELTA      ← JSON patch update to existing state
ACTIVITY_SNAPSHOT ← sub-agent timeline event
CUSTOM           ← domain events (e.g. "search_completed")
```

---

## Frontend

### Stack
- Vite + React + TypeScript
- React Router (two pages)
- Tailwind + shadcn/ui

### Page 1 — `/copilotkit`

Uses **CopilotKit** to connect to the backend and render the chat.

Key things to figure out:
- How `<CopilotKit>` connects to a raw AG-UI endpoint (vs needing CopilotKit's own Python SDK)
- `useCopilotChat` — the core hook for sending messages and reading the stream
- `useCopilotAction` — registering custom React components for specific tool calls
- `useCoAgent` — reading shared agent state (STATE events)
- Built-in `<CopilotChat>` vs building your own UI with the headless hooks

### Page 2 — `/assistant-ui`

Uses **assistant-ui** to connect to the same backend endpoint.

**Layer 1** (text, thinking, tool calls):
- `HttpAgent` from `@ag-ui/client` — points at `POST /agent`
- `useAgUiRuntime({ agent, showThinking: true })` — bridges agent to assistant-ui
- `AssistantRuntimeProvider` + `ThreadPrimitive.*` + `MessagePrimitive.*` — the UI primitives
- Custom tool components for `show_data_card`, `delegate_research`, `analyze_findings`

**Layer 2** (state, activity, custom — the events CopilotKit drops):
- `agent.subscribe()` — attach a parallel subscriber alongside the Layer 1 runtime
- Zustand store to hold: `agentState`, `activities` (Map), `domainEvents`, `isRunning`
- Hook (`useAgentEventLayer`) that routes STATE/ACTIVITY/CUSTOM events into the store
- Three side panels: `AgentStatePanel`, `SubAgentTimeline`, `CustomEventLog`

---

## Integration Points

| Concern | CopilotKit page | assistant-ui page |
|---------|----------------|-------------------|
| Connect to backend | `<CopilotKit runtimeUrl="...">` | `new HttpAgent({ url: "..." })` |
| Send a message | `useCopilotChat().append(...)` | `ThreadPrimitive.Composer` |
| Read streamed text | automatic in `<CopilotChat>` | `useMessagePartText()` |
| Render thinking blocks | limited / broken (REASONING_* mismatch) | `useMessagePartReasoning()` + ThinkingBlock component |
| Custom tool UI | `useCopilotAction({ name, render })` | `makeAssistantToolUI()` or tool component map |
| Agent state (STATE events) | `useCoAgent` | Zustand store via `useAgentEventLayer` |
| Sub-agent activity | not exposed | `SubAgentTimeline` via ACTIVITY events |
| Domain events | not exposed | `CustomEventLog` via CUSTOM events |

assistant-ui exposes all 17 AG-UI event types. CopilotKit surfaces fewer — the right column is more work to build, and that's where the real learning is.
