# PI Philosophy

The most important thing about PI is not the code. It's the constraints.

---

## The Core Thesis

> "What you leave out matters more than what you put in."

Modern LLMs trained with RL on coding tasks already understand how to be agents. They've seen thousands of coding sessions during training. They know what a file system is. They know what a terminal is. They know what "read this file and fix the bug" means.

**You don't need to explain any of this to them.**

Every token in your system prompt is a performance tax:
- It consumes context window
- It increases latency and cost
- It can contradict what the model already knows
- It can confuse the model when instructions collide

PI's system prompt is under 1,000 tokens. Claude Code's is over 10,000. According to PI's benchmarks, PI is competitive with Claude Code on coding tasks — at a fraction of the context cost.

---

## The 4 Tools

PI gives agents exactly 4 tools:

1. **Read** — read a file
2. **Write** — write a file
3. **Edit** — apply a targeted edit to a file
4. **Bash** — execute a shell command

That's it.

**Why only 4?** Because Bash is a meta-tool. Anything else the agent needs — run tests, install packages, browse the web, call an API, spawn a subprocess — it can do through Bash. If it needs a persistent capability it doesn't have, it can write a script and add it. The agent *is* the extension mechanism.

This is the opposite of frameworks that give agents 40 tools "so they have everything they need." More tools = more context = more confusion = worse performance.

---

## No Plan Mode

PI has no plan mode. No "first I'll think step by step, then I'll act."

The reasoning: plan mode adds tokens. Modern models reason implicitly during generation. Explicit plan mode can actually *reduce* quality by forcing the model into a structured reasoning path that doesn't match how it naturally thinks.

If the task is complex enough to need explicit planning, the model will plan. You don't need to force it.

---

## No Sub-Agents

PI has no built-in sub-agent support. No orchestration, no delegation, no parallel execution.

The reasoning: sub-agents are complexity generators. Every agent boundary is a place where information gets summarized, context gets lost, and errors can cascade. A single agent with good tools outperforms a multi-agent system for most tasks because it maintains full context.

For tasks that genuinely need parallelism (large codebases, independent subtasks), the agent can spawn subprocesses through Bash.

---

## Full Context Visibility

In PI, the developer sees exactly what enters the model's context. No hidden system prompt injections, no invisible tool descriptions appended at runtime, no framework magic.

This matters for debugging and for trust. When your agent behaves unexpectedly, you can look at the exact context it received and understand why.

Many frameworks inject substantial content into the context without surfacing it — tool schemas, memory summaries, injected instructions. PI refuses to do this.

---

## Self-Extension

When an agent needs a capability it doesn't have, it writes code to add that capability.

Example: agent needs to parse a PDF. Instead of having a `parse_pdf` tool pre-registered, the agent:
1. Recognizes it doesn't have a PDF parser
2. Installs `pdfplumber` via Bash
3. Writes a Python script to extract the text
4. Runs it

This is what "self-extending" means. The agent's toolset grows to fit the task, not the other way around.

---

## Session Model

PI sessions are tree-based with branching. Each turn is a node. You can branch from any point and explore multiple paths without losing previous work.

Sessions persist as JSONL files on disk. Each event — user messages, tool calls, results, model outputs — is a line. Append-only. Crash-safe.

Context compaction runs when context gets long: the agent summarizes earlier conversation turns. This is lossy by design — a tradeoff between context length and recency.

---

## What This Means for Agent Design

If you adopt PI's philosophy (even without using PI's code):

- Keep system prompts under 1,000 tokens
- Give agents fewer, more general tools rather than many specific ones
- Trust the model's training instead of over-specifying behavior
- Don't add sub-agents unless you have evidence they help
- Make context visible, not hidden

These principles apply directly to PydanticAI agents.

---

## Sources

- https://mariozechner.at/posts/2025-11-30-pi-coding-agent/ — primary source for all philosophy above
- https://lucumr.pocoo.org/2026/1/31/pi/ — independent validation of the approach
