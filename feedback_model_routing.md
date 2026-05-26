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

**Final disposition 2026-05-26 (tidy-house audit):** The original "infra → Qwen via bot" intent was architecturally sound but unfunded on the actual substrate. Investigation showed:
- arcai-bot bypassed OpenClaw entirely and called Anthropic Sonnet directly (every Telegram query cost tokens)
- OpenClaw's `main` agent had been dormant 16 days; GPU at 0% utilisation
- ArcAiVM is **4GB RAM (3GB ballooned), not 16GB** — Llama 3.1 8B OOM-killed during GGUF load on attempted v4 rewrite
- PVE host had 595MiB free across the entire cluster — no headroom to raise ArcAiVM RAM without shrinking ciphertrust-ce or kryptvakt-dev

**Decision (Mitch, 2026-05-26):** keep bot on Sonnet permanently. OpenClaw retired from ArcAiVM (systemd user unit deleted, autostart removed). The "infra → local Qwen" convention is **deferred until ArcAiVM has the RAM** — revisit only if RAM gets reallocated or a 3B-class model proves sufficient. Until then:
- Bot infra queries → Sonnet (known token cost, accepted)
- Building apps / engineering → Claude Code (Sonnet) — unchanged
- Do NOT suggest the Telegram bot as a "cheaper" alternative — it isn't

**Reactivation gate:** would need ArcAiVM ≥8GB AND a working OpenAI-compat tool-use loop against Ollama. v4 bot.py design (Ollama-first with `/hard` Sonnet escalation) was prototyped 2026-05-26 and deleted; rewrite from scratch if revisiting, with whatever model class is then realistic for the substrate.
