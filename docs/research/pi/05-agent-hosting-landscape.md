# Agent Hosting Landscape

What "agent hosting" means and who the real players are.

---

## What Is "Agent Hosting"?

Agent hosting is distinct from LLM hosting or model serving. It refers to managed infrastructure for **stateful, long-running agents** — including:

- **Persistent memory**: agents that remember across sessions
- **Tool execution**: sandboxed environments for code execution, file access, web calls
- **Multi-step workflow management**: agents that take many turns over minutes or hours
- **Human-in-the-loop**: pausing execution for approval, correction, or input
- **Observability**: tracing tool calls, state snapshots, latency, cost
- **Scaling**: running many agents concurrently without managing servers

Running `uvicorn main:app` on a VPS is not agent hosting. It's server hosting. A hosted agent runtime handles the agent lifecycle itself.

---

## Platform Overview

### Agno (formerly Phidata) — `agno.com`

Python-native. The most relevant platform for our stack.

**Three-layer product:**
- **Agent Framework** (open-source Python SDK) — build agents, teams, and workflows
- **AgentOS Runtime** — turns agents into scalable API services
- **Control Plane UI** — monitoring, tracing, session management

**Key claims:**
- 529× faster agent instantiation than LangGraph
- 24× lower memory footprint
- Private-by-default (data stays in your cloud)

**Deployment:** AWS, GCP, Railway, or self-hosted.

**Relevance:** High. Python. Deployment-focused. But it's a different agent framework (not PydanticAI), so adopting AgentOS would mean migrating off PydanticAI.

### LangGraph Platform

The hosting layer for LangGraph agents. Generally Available as of 2025.

**Three deployment tiers:**
- **Cloud (SaaS)** — managed, Plus/Enterprise plans
- **Hybrid** — SaaS control plane + self-hosted data plane (Enterprise only)
- **Fully Self-Hosted** — VPC deployment

**Features:** persistence layer, LangGraph Studio IDE, checkpointing, human-in-the-loop, memory management.

**Relevance:** High for teams already on LangGraph. We're on PydanticAI. Switching would be a major migration. Not the right move.

### E2B — `e2b.dev`

Firecracker microVM sandboxes for code execution. 125ms startup time.

Not an agent framework or hosting platform — it's a **code execution sandbox** for when agents need to run untrusted code safely. If your agents execute arbitrary code (code interpreter, data analysis), E2B is the right isolation layer.

**Relevance:** Medium. Worth adding if agents start doing code execution tasks.

### Modal — `modal.com`

gVisor-based sandboxes covering inference, training, and batch compute. More general than E2B.

**Relevance:** Medium. Good alternative to E2B. Also useful for running expensive model inference serverlessly.

### Pipecat (Daily.co) — `pipecat.ai`

Real-time voice and multimodal conversational agent framework. Pipeline-based architecture.

**Relevance:** Low. Different use case — real-time voice/video agents. Not relevant for text-based web agents.

### Azure AI Foundry

Microsoft's enterprise unified AI platform with hosted agents service.

**Relevance:** Low. Enterprise-only, heavy, vendor lock-in. Interesting for reference but not appropriate for this project.

### OpenAI Frontier

OpenAI's enterprise platform for building, deploying, and managing agents.

**Relevance:** Low. Vendor lock-in to OpenAI models. We're using OpenRouter specifically to stay model-agnostic.

### Prismatic (iPaaS)

Embedded integration platform that exposes integration workflows as structured tools for AI agents. MCP and on-premises agent support.

**Relevance:** Low. B2B SaaS integration use case. Not general-purpose agent hosting.

---

## Deployment for This Project

For this stack specifically, the right deployment is straightforward:

| Component | Where to deploy | Why |
|-----------|----------------|-----|
| FastAPI backend | Railway or Fly.io | Already in roadmap; simple container deploy |
| Frontend | Vercel or Cloudflare Pages | Static Vite build |
| Agent runtime | In-process (FastAPI handles it) | PydanticAI agents run in the same process |
| Code sandboxing | E2B (if needed later) | If agents start running code |
| Observability | OpenTelemetry → Logfire or LangSmith | PydanticAI has built-in OTel support |

**No dedicated "agent hosting platform" is needed right now.** PydanticAI agents run in-process with FastAPI. Session state can be held in memory or persisted to a simple database when needed. The complexity of a full agent hosting platform (Agno's AgentOS, LangGraph Platform) is not justified until:
- Agents need to run for minutes/hours across multiple HTTP requests
- Human-in-the-loop workflows require pausing and resuming
- Scale requires running hundreds of concurrent agent sessions

---

## Sources

- https://www.agno.com/
- https://docs.phidata.com/introduction
- https://github.com/agno-agi/agno
- https://blog.langchain.com/langgraph-platform-ga/
- https://e2b.dev/
- https://northflank.com/blog/e2b-vs-modal — E2B vs Modal comparison 2026
- https://www.pipecat.ai/
- https://learn.microsoft.com/en-us/azure/ai-foundry/
- https://openai.com/index/introducing-openai-frontier/
- https://prismatic.io/
