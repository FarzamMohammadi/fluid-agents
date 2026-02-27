# PI Integration Analysis

How PI fits (and doesn't fit) with this stack.

**Our stack:** PydanticAI + AG-UI + FastAPI (Python) + CopilotKit + assistant-ui + OpenRouter

---

## What Fits

### `pi-ai` supports OpenRouter natively
PI's unified LLM abstraction layer lists OpenRouter as a first-class provider alongside OpenAI, Anthropic, Groq, etc. The same multi-provider flexibility we already get from OpenRouter directly.

### RPC mode enables Python ↔ TypeScript bridging
`pi-coding-agent` in RPC mode runs as a subprocess communicating via JSON over stdin/stdout. A FastAPI endpoint could spawn it as a sidecar, send prompts, and receive streaming events — without caring that it's TypeScript.

### TypeScript aligns with the frontend
PI's packages are TypeScript. Our frontend is TypeScript (Vite + React). If we wanted to use `pi-ai` or `pi-agent-core` directly in a Node.js backend or at the edge, it fits.

### Minimal prompt philosophy applies everywhere
The core insight — keep system prompts lean, trust model training, give fewer broader tools — applies directly to our PydanticAI agents regardless of whether we use PI's code.

---

## What Doesn't Fit

### No AG-UI support
PI has no concept of AG-UI events. It doesn't emit `RUN_STARTED`, `TEXT_MESSAGE_CONTENT`, `TOOL_CALL_START`, `STATE_SNAPSHOT`, or any of the 17 AG-UI event types.

To expose a PI agent via AG-UI, you'd need to write a custom adapter that:
1. Subscribes to PI's event stream
2. Maps PI events → AG-UI events
3. Streams them as SSE

This is non-trivial, especially for STATE and ACTIVITY events.

### TypeScript-only backend
Our backend is Python + FastAPI. PI's packages are TypeScript/Node.js. Running the full PI stack on the backend means either:
- Running a Node.js sidecar alongside FastAPI (operational complexity)
- Rewriting the backend in TypeScript (not the goal)

### No React components
`pi-web-ui` uses Lit web components. Our frontend uses React. You can technically embed Lit elements in React with a wrapper, but it adds friction and breaks the component model we've built with CopilotKit and assistant-ui primitives.

### No CopilotKit or assistant-ui integration
Neither CopilotKit nor assistant-ui has a PI integration. CopilotKit integrates with PydanticAI, LangChain, LangGraph, and Mastra. assistant-ui integrates with AG-UI and the AI SDK. PI is outside this ecosystem.

### Optimized for coding specifically
PI's 4-tool philosophy (Read, Write, Edit, Bash) is built for coding agents. Our agents do web research, analysis, and general-purpose tasks. The 4-tool constraint would need to be relaxed — at which point we're using PI's architecture but not its core design.

### Session model conflicts with AG-UI
PI persists sessions as JSONL files with tree-based branching. AG-UI is stateless event streaming per request. These models don't naturally compose. PI's session compaction, branching, and persistence would all need custom handling to work through an AG-UI endpoint.

---

## Integration Paths

Three realistic options, from highest to lowest value:

### Option 1: Adopt the philosophy (no code change)
Apply PI's minimalism principles to PydanticAI agent design:

- Cap system prompts at ~800-1,000 tokens
- Give sub-agents fewer, broader tools
- Avoid over-specifying behavior — trust the model
- Make context visible: log exactly what enters the agent on each run

**Effort:** Zero code changes. **Value:** High — better agent performance, lower cost.

### Option 2: PI as a coding sub-agent sidecar
Run `pi-coding-agent` in RPC mode as a subprocess from FastAPI. Use it as a specialized coding sub-agent that `main_agent` can delegate to via a `run_coding_agent` tool.

```python
# Conceptual FastAPI tool
async def run_coding_agent(task: str) -> str:
    # Spawn pi-coding-agent --mode rpc as subprocess
    # Send JSON {"type": "prompt", "text": task} to stdin
    # Read streaming JSON events from stdout
    # Return final result
    pass
```

**Effort:** Medium — subprocess management, JSON protocol implementation, error handling.
**Value:** Medium-high — best-in-class coding agent at minimal context cost for code-specific tasks.
**Risk:** Node.js runtime dependency on Python server, subprocess lifecycle management.

### Option 3: Use `pi-ai` as LLM layer
Replace direct OpenRouter calls with `pi-ai` for its unified multi-provider abstraction and built-in cost tracking.

**Effort:** Medium — requires a small Node.js service or switching the backend to TypeScript.
**Value:** Low — we already have this via OpenRouter. `pi-ai` adds provider-switching flexibility we don't need right now.
**Verdict:** Skip unless there's a specific multi-provider need that OpenRouter doesn't cover.

---

## Summary

| Integration | Effort | Value | Verdict |
|-------------|--------|-------|---------|
| Adopt PI philosophy in PydanticAI agents | None | High | **Do it** |
| PI coding sub-agent via RPC | Medium | Medium-High | **Worth exploring in a later phase** |
| `pi-ai` as LLM layer | Medium | Low | Skip |
| `pi-web-ui` React components | High | None | Skip |
| Full PI backend replacement | Very High | Negative | Hard no |
