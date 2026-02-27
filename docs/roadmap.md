# Roadmap

Phases are sequential. Each has a concrete done state — don't move on until you hit it.

---

## Phase 1 — Backend: Server + One Agent + One Tool

**Goal:** `POST /agent` streams AG-UI events. Verify with curl or Postman — no frontend yet.

- Set up FastAPI + `pydantic_ai.ext.ag_ui`
- Wire OpenRouter as the LLM provider (OpenAI-compat with `base_url` override)
- Create `main_agent` with a system prompt — no tools yet
- Mount the AG-UI adapter on `POST /agent`
- Confirm: sending a message returns `RUN_STARTED → TEXT_MESSAGE_* → RUN_FINISHED` over SSE

**Done when:** You can curl the endpoint and see AG-UI events in your terminal.

---

## Phase 2 — Backend: Add Tools

**Goal:** `main_agent` has all 5 tools. Tool call events appear in the stream.

- Implement `web_search` with DuckDuckGo
- Implement `show_data_card` — returns a `DataCard` Pydantic model
- Implement `get_current_time` — trivial, good for testing the tool lifecycle
- Register all three on `main_agent`
- Confirm: ask "what time is it?" → see `TOOL_CALL_START → TOOL_CALL_ARGS → TOOL_CALL_END → TOOL_CALL_RESULT`

**Done when:** All 5 tools registered, tool calls visible in the SSE stream.

---

## Phase 3 — Backend: Sub-agents

**Goal:** `researcher_agent` and `analyst_agent` exist and are callable from `main_agent` tools.

- Implement `researcher_agent` — runs `web_search`, returns `ResearchResult`
- Implement `analyst_agent` — no external tools, returns `AnalysisResult`
- Add `delegate_research` tool on `main_agent` (calls `researcher_agent.run()`)
- Add `analyze_findings` tool on `main_agent` (calls `analyst_agent.run()`)
- Explore: do sub-agent events appear in the parent stream? Read the adapter source.
- Explore: can tools emit `STATE_SNAPSHOT` or `CUSTOM` events? If yes, wire it.

**Done when:** Ask "research Python async frameworks" → see research tool call + analyst tool call in stream.

---

## Phase 4 — Frontend: Base Setup

**Goal:** Vite + React app running with two routes and a nav bar.

- Init Vite React TS project
- Add React Router — routes `/copilotkit` and `/assistant-ui`
- Add Tailwind + shadcn/ui (optional but makes it look good)
- Create `Nav` component with links to both pages
- Both pages render placeholder text for now

**Done when:** `pnpm dev` → can navigate between two pages.

---

## Phase 5 — Frontend: assistant-ui Page

**Goal:** Full assistant-ui chat talking to the backend. Layer 1 + Layer 2.

- Install `@assistant-ui/react`, `@assistant-ui/react-ag-ui`, `@ag-ui/client`, `rxjs`
- Create `HttpAgent` pointing at `http://localhost:8000/agent`
- Wire `useAgUiRuntime` + `AssistantRuntimeProvider`
- Build chat UI using `ThreadPrimitive.*` and `MessagePrimitive.*` — build it from primitives, don't use pre-built components, you want to understand what each primitive does
- Add thinking block rendering (`useMessagePartReasoning`)
- Add custom tool components for `show_data_card`, `delegate_research`, `analyze_findings`
- Add Layer 2: Zustand store + `useAgentEventLayer` hook + 3 side panels

**Done when:** Chat works, thinking blocks render, tool cards render, Layer 2 panels update.

---

## Phase 6 — Frontend: CopilotKit Page

**Goal:** Same backend, CopilotKit frontend. Understand how it compares.

- Install `@copilotkit/react-core`, `@copilotkit/react-ui`
- Figure out how CopilotKit connects to a raw AG-UI endpoint — read their docs carefully
  (the connection mode, the right prop, whether you need their Python SDK on the backend)
- Get a basic chat working with `useCopilotChat`
- Try `useCopilotAction` for custom tool rendering
- Try `useCoAgent` for state
- Notice what works, what doesn't, what events get dropped

**Done when:** Chat works. You've felt the difference between CopilotKit and assistant-ui firsthand.

---

## Phase 7 — Polish

- Make the two pages visually distinct so the comparison is obvious at a glance
- Add a raw event inspector panel next to each chat (log every AG-UI event as it arrives)
- Wire real STATE/ACTIVITY/CUSTOM events on the backend if Phase 3 exploration paid off
- Deploy: server on Railway or Fly.io, web on Vercel or Cloudflare Pages
- Add setup instructions to README

---

## Reference

| | |
|--|--|
| PydanticAI | https://ai.pydantic.dev/ |
| PydanticAI AG-UI adapter | https://ai.pydantic.dev/ag-ui/ |
| AG-UI protocol | https://docs.ag-ui.com/ |
| CopilotKit | https://docs.copilotkit.ai/ |
| assistant-ui | https://www.assistant-ui.com/docs |
| @ag-ui/client | https://github.com/ag-ui-protocol/ag-ui |
