# Verdict: Should PI Be Integrated?

**Short answer: Apply the methodology. Skip the library. Revisit the coding sub-agent as a potential complement later.**

---

## The Parallel

PI and this project are solving the same problem from different directions.

**PI** is an agent hosting runtime: it receives tasks, runs an agent loop, executes tools, and streams results back — built in TypeScript with a minimalist philosophy.

**This project's server** is also an agent hosting runtime: FastAPI receives requests, PydanticAI runs the agent loop, tools execute, and AG-UI events stream back to the frontend.

The stacks are parallel. PI is not a dependency to adopt — it's a reference implementation in a different language for the same class of problem. The value is in studying its design decisions and applying the sound ones here.

---

## What PI Gets Right

PI's core insight is well-evidenced: **modern LLMs are trained on agentic behavior and don't need verbose scaffolding**. The minimalism approach produces competitive results with a fraction of the context overhead.

The 4-tool design, the <1,000 token system prompt, and the full context visibility principle are all worth applying regardless of whether PI's code is used.

---

## Why PI Is Not a Replacement

Our stack (PydanticAI + AG-UI + FastAPI) is the right choice for this project:

1. **AG-UI has serious industry backing.** Google, AWS, Microsoft, LangChain, and Mastra all support it. It's the emerging standard for agent-UI communication. PI has no AG-UI support at all.

2. **PydanticAI is Python-native.** Our backend is Python. PydanticAI gives us type-safe agents with built-in AG-UI streaming and OpenTelemetry. Switching to PI would mean TypeScript throughout or running a language-boundary sidecar.

3. **CopilotKit and assistant-ui expect AG-UI.** Both frontends are built around the AG-UI event stream. PI doesn't speak this protocol.

4. **PI is optimized for coding agents.** Our agents do web research, analysis, and general-purpose tasks. PI's 4-tool design is well-suited for coding — less so for research and analysis agents that need web search, structured output, and custom tools.

---

## Action Items

### Immediate: Apply PI's methodology to PydanticAI agents

When building the agents in Phases 2 and 3 of the roadmap:

- **Keep system prompts under ~800 tokens.** State the agent's role, constraints, and output format. Modern models understand the rest.
- **Give sub-agents fewer, broader tools.** `researcher_agent` should have web search and file access. Not a long list of specialized tools.
- **Trust the model's training.** Claude, GPT-4, and similar models already know how to search, reason, and synthesize. Over-specifying behavior in the system prompt works against this.
- **Make context observable.** Build a debug mode that logs the full context on each run — the exact messages, tool schemas, and state entering the model. This is how you debug agent behavior reliably.

### Medium-term: PI coding sub-agent (Phase 7 or later)

Once the base stack is solid, consider running `pi-coding-agent` in RPC mode as a specialized sub-agent for code-related tasks.

**How it would work:**
- `main_agent` gets a `run_coding_agent(task: str)` tool
- That tool spawns `pi-coding-agent --mode rpc` as a subprocess
- FastAPI manages the subprocess lifecycle
- Events from PI stream back and get wrapped as AG-UI `TOOL_CALL_RESULT` or `CUSTOM` events

**Why worth considering:** PI is a highly capable coding agent at minimal context cost. If the project expands into tasks that involve writing or executing code, this is a natural integration point.

**Prerequisites:**
- Phases 1-6 complete and working
- A concrete coding task use case that warrants a dedicated sub-agent
- Willingness to manage a Node.js runtime dependency alongside the Python server

### Skip: `pi-ai`, `pi-web-ui`, `pi-pods`

- **`pi-ai`**: Already covered by OpenRouter. Adds no value and introduces a language boundary.
- **`pi-web-ui`**: Lit web components. The frontend is React. Incompatible without a wrapper layer.
- **`pi-pods`**: GPU pod orchestration. OpenRouter handles model access. Not needed.

---

## Positioning

PI is a well-designed reference implementation for a minimal, high-performance agent runtime. It is most useful to this project as a source of design principles — not as a library to depend on.

The recommended approach: read the author's original post (linked below), apply the minimalism principles when writing PydanticAI agents, and revisit the coding sub-agent integration if a concrete use case emerges.

---

## Sources

- https://mariozechner.at/posts/2025-11-30-pi-coding-agent/ — read this in full
- https://github.com/badlogic/pi-mono
- https://lucumr.pocoo.org/2026/1/31/pi/ — Armin Ronacher's take on PI
- https://docs.ag-ui.com/ — why AG-UI is the right protocol bet
