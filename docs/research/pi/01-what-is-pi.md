# What Is PI?

## Identification

"PI" in the context of AI agent hosting is **[pi-mono](https://github.com/badlogic/pi-mono)** — a TypeScript monorepo by **Mario Zechner**, the creator of the libGDX game framework.

It is **not**:
- **pi.ai** (Inflection AI's consumer chatbot — effectively dead after Microsoft acquired Inflection's talent in 2024)
- **Pipecat** (Daily.co's real-time voice agent framework)
- **Phidata / Agno** (Python enterprise agentic platform)
- **Microsoft Phi** (language model family)

---

## Origin

Mario Zechner built PI after years of frustration with existing agent frameworks. His core observation: **modern LLMs trained with RL on coding tasks already understand how to be coding agents** — they don't need 10,000-token system prompts telling them what code is. Every token you inject is a performance tax.

He's known in game dev circles for libGDX (10K+ stars, cross-platform Java game framework). PI applies the same engineering rigor — extreme specificity, minimal surface area, deliberate constraints — to AI agents.

The project was published in late 2025 and immediately spawned a viral ecosystem.

---

## OpenClaw Connection

**[OpenClaw](https://openclaw.ai/)** is built on top of PI. It wraps `pi-coding-agent` in a persistent daemon that adds:
- 12+ messaging platform integrations (Slack, Discord, Teams, etc.)
- Heartbeat scheduler
- Session management across platforms
- Industrial deployment (ThunderSoft deployed it on RUBIK Pi 3 and AIBOX hardware)

OpenClaw hit **145,000+ GitHub stars in its first week**. Most people who've "heard about PI" heard about it through OpenClaw.

---

## Stats

| Metric | Value |
|--------|-------|
| **pi-mono GitHub stars** | 8,900+ |
| **OpenClaw GitHub stars** | 145,000+ (week 1) |
| **Contributors** | 128 |
| **Releases** | 151+ |
| **@mariozechner/pi-ai npm downloads** | 1,263,991 |
| **@mariozechner/pi-agent-core npm downloads** | 1,344,181 |
| **@mariozechner/pi-tui npm downloads** | 1,580,835 |
| **License** | MIT |
| **Cost** | Free (you pay only for LLM API calls) |

---

## Sources

- https://github.com/badlogic/pi-mono
- https://mariozechner.at/posts/2025-11-30-pi-coding-agent/ — Zechner's own writeup on what he learned building PI
- https://lucumr.pocoo.org/2026/1/31/pi/ — Armin Ronacher (creator of Flask) on PI inside OpenClaw
- https://openclaw.ai/
- https://medium.com/@shivam.agarwal.in/agentic-ai-pi-anatomy-of-a-minimal-coding-agent-powering-openclaw-5ecd4dd6b440
