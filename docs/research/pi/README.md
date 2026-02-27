# PI Research

Research into **pi-mono** — a minimal TypeScript AI agent toolkit — and whether it belongs in this stack.

---

## What PI Is (TL;DR)

**PI** is [pi-mono](https://github.com/badlogic/pi-mono) by Mario Zechner (creator of libGDX). A **TypeScript monorepo of 7 packages** for building AI agents with a radical minimalism philosophy: 4 tools, <1K token system prompt, self-extending agents. It powers **OpenClaw** (145K+ GitHub stars). MIT licensed. Free.

---

## Contents

| File | What's inside |
|------|---------------|
| [01-what-is-pi.md](./01-what-is-pi.md) | Identification, origin, OpenClaw connection, stats |
| [02-architecture.md](./02-architecture.md) | 7 packages, operating modes, SDK usage |
| [03-philosophy.md](./03-philosophy.md) | Minimalism doctrine, design decisions, tradeoffs |
| [04-integration-analysis.md](./04-integration-analysis.md) | Fits/gaps vs this stack, concrete integration paths |
| [05-agent-hosting-landscape.md](./05-agent-hosting-landscape.md) | Broader landscape: Agno, LangGraph, E2B, and others |
| [06-verdict.md](./06-verdict.md) | Final recommendation and action items |

---

## Verdict (TL;DR)

**Don't replace the stack with PI. Do borrow from it.**

1. **PI's philosophy** → apply inside PydanticAI agents: fewer tools, leaner prompts, full visibility
2. **`pi-coding-agent` in RPC mode** → optional specialized coding sub-agent sidecar
3. **PI's web components** → skip, they're Lit not React
4. **AG-UI + PydanticAI** → stays as the backbone, backed by Google/AWS/Microsoft/LangChain

See [06-verdict.md](./06-verdict.md) for full reasoning.
