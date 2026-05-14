---
name: Model Routing Convention
description: Mitch wants infra/maintenance tasks to use Qwen (local, free) not Sonnet API credits
type: feedback
originSessionId: bd505ebc-a29e-4d94-9285-08795c849e18
---
Use Qwen (via Telegram bot / OpenClaw) for routine infra work. Reserve Claude Code (Sonnet) for building apps and complex engineering.

**Why:** Mitch has a Sonnet API quota and doesn't want to burn it on health checks, monitoring queries, or routine maintenance — that's what the local Qwen model is for.

**How to apply:**
- Infra queries, health checks, lab monitoring → direct Mitch to Telegram bot (hits Qwen tier 1, escalates to Claude only if needed)
- Building apps, writing code, architecture decisions → Claude Code (Sonnet is appropriate here)
- Never suggest using Claude Code for something the Telegram bot / Qwen can handle
- When doing infra maintenance tasks inside a Claude Code session (because Mitch already opened one), keep it brief and batch work to minimise turns
