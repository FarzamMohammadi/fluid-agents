# PI Architecture

## Monorepo Structure

PI is organized as a TypeScript monorepo with 7 packages that layer on each other from low to high level.

```
pi-mono/
├── packages/
│   ├── pi-ai            ← LLM abstraction layer
│   ├── pi-agent-core    ← Agent runtime loop
│   ├── pi-coding-agent  ← Full CLI/SDK coding agent
│   ├── pi-tui           ← Terminal UI
│   ├── pi-web-ui        ← Web components
│   ├── pi-mom           ← Messaging routing (Slack, etc.)
│   └── pi-pods          ← GPU pod orchestration
```

---

## Package Breakdown

### `@mariozechner/pi-ai` — Unified LLM API

Single interface across all major providers. Addresses real incompatibilities pragmatically (Cerebras parameter quirks, Mistral naming differences, etc.) rather than pretending all providers are identical.

**Supported providers:**
- OpenRouter, OpenAI, Anthropic, Google, xAI, Groq, Cerebras, Ollama, Azure, Bedrock
- Any OpenAI-compatible endpoint

**Features:**
- Streaming
- Tool calling with TypeBox schemas
- Reasoning/thinking support
- Token and cost tracking per request

### `@mariozechner/pi-agent-core` — Agent Runtime

Implements the agent loop: call LLM → execute tools → feed results back → repeat.

**Features:**
- Tool validation and execution
- Event streaming (subscribe to turn events)
- State management across turns
- Stateful `Agent` wrapper class

### `@mariozechner/pi-coding-agent` — The Main Product

A full interactive coding agent built on top of `pi-agent-core`. This is what most people interact with.

**Built-in tools (exactly 4):**
- `Read` — read files
- `Write` — write files
- `Edit` — apply targeted edits
- `Bash` — execute shell commands

**Session management:**
- JSONL persistence on disk
- Tree-based sessions with branching support
- Context compaction when context gets long (lossy but necessary)

**See [03-philosophy.md](./03-philosophy.md) for why only 4 tools.**

### `@mariozechner/pi-tui` — Terminal UI

Retained-mode terminal UI with differential rendering (only redraws changed lines — efficient for slow terminals).

**Features:**
- Markdown rendering
- Multi-line editing
- Autocomplete
- Loading spinners
- Native scrollback buffer

### `@mariozechner/pi-web-ui` — Web Components

Lit-based (not React) web components for embedding PI in browser contexts.

**Features:**
- `ChatPanel` component with streaming
- File attachment support
- Artifact rendering: HTML, SVG, Markdown in sandboxed iframes

**Note:** These are Lit elements, not React components. They won't drop into a CopilotKit or assistant-ui setup without a wrapper layer.

### `@mariozechner/pi-mom` — Messaging Routing

Routes messages to `pi-coding-agent` from Slack and other messaging platforms. Used by OpenClaw.

### `@mariozechner/pi-pods` — GPU Pod Orchestration

CLI for spinning up open-source models on GPU cloud providers via vLLM.

**Supports:**
- DataCrunch
- RunPod
- Vast.ai
- Bare metal

Exposes OpenAI-compatible endpoints. Useful if you want to run your own model rather than going through OpenRouter.

---

## Operating Modes

`pi-coding-agent` has 4 modes — this is key to integration:

| Mode | Description | Integration use |
|------|-------------|-----------------|
| **Interactive** | Full terminal chat, user controls everything | Local dev, direct use |
| **Print / JSON** | Runs non-interactively, outputs results to stdout | Scripting, CI |
| **RPC** | JSON protocol over stdin/stdout | **Embed in FastAPI as subprocess** |
| **SDK** | TypeScript/Node.js programmatic embedding | Build on top of PI in JS |

**RPC mode is the integration key for Python backends.** You spawn `pi-coding-agent` as a subprocess, write JSON messages to stdin, read events from stdout. Language-agnostic. Process-isolated.

---

## SDK Usage Example

```typescript
import { createAgentSession } from '@mariozechner/pi-coding-agent';

const { session } = await createAgentSession({
  sessionManager: SessionManager.inMemory(),
  authStorage,
  modelRegistry,
});

// Subscribe to streaming events
session.subscribe((event) => {
  if (event.type === 'message_update') {
    process.stdout.write(event.assistantMessageEvent.delta);
  }
});

// Send a prompt, await completion
await session.prompt("Refactor this function to be async");

// Interrupt mid-operation
await session.steer("Actually, leave the function sync");

// Queue follow-up
await session.followUp("Now add tests for it");
```

---

## Sources

- https://github.com/badlogic/pi-mono
- https://mariozechner.at/posts/2025-11-30-pi-coding-agent/
- https://nader.substack.com/p/how-to-build-a-custom-agent-framework
