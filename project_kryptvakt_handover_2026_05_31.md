---
name: project-kryptvakt-handover-2026-05-31
description: "Mitch filed a Sunday handover doc 2026-05-31 with 5 candidate ADRs for the next [[project-kryptvakt-heimdall]] build session. Monday 2026-06-01 he returns with armature-pattern + playbook-seed insights from case-based method extraction."
metadata: 
  node_type: memory
  type: project
  originSessionId: 85558db8-7999-4b95-9940-0eab8caee4f0
---

Mitch wrote a weekend handover doc on 2026-05-31 (Sun, dad day, no work) parking design + architecture clarifications for the Monday 2026-06-01 [[project-kryptvakt-heimdall]] build session.

**Where it lives:** `~/workspace/kryptvakt/docs/handover-2026-05-31-design-clarifications.md`. Breadcrumb at the top of `kryptvakt/NEXT-STEPS.md` under "⚑ Monday 2026-06-01 — read FIRST".

**Five candidate ADRs (to ratify in the build session, not before):**
1. Graph as MODEL, not ENGINE — SQLite + `WITH RECURSIVE` stays; Neo4j rejected; if a graph engine is ever truly needed, embedded-only (Kùzu), never a server. Preserves the single-binary moat.
2. Determinism via enforced routing + facts-in-tools — files = source of truth; runner (code) enforces routing; tools return typed JSON; **the model never asserts a fact it didn't get from a tool**. `session.md` (model-agnostic) replaces `CLAUDE.md` + per-model adapter layer.
3. Heimdall reasons over Kryptvakt's canonical model only. Competitors (Venafi/Keyfactor/Spotlight/AD CS) are *sources, never substrate*. CBOM ingest = funnel only.
4. ADR-007 citation fix — "JC 2023 86 RTS Arts 6+7" (draft) → **CDR (EU) 2024/1774, Arts 6–7** (in-force, auditor-citable). One-line accuracy fix; doesn't change the substance. See related [[reference-dora-rts-structure]].
5. (Optional) Restraint principle as a recorded design tenet.

**Spine invariant (the lens to check every decision against):** *single binary, self-hosted, data never leaves, BYO-LLM, cold-deploy into an enterprise and it just runs.* This is the moat property vs Entrust/Keyfactor/SandboxAQ/ISARA (all SaaS). When two designs compete, the one that preserves the single binary wins by default.

**The clay/armature/eye/tools frame:** Kryptvakt canonical model = clay (raw material, verified by audit chain). Playbooks = armature (structural skeleton). Heimdall reasoning method = sculptor's eye (Mitch's framing, encoded — the irreplaceable part). BYO-LLM = tools (commodity chisel the customer owns). *The LLM is commodity; the armature + the eye are the moat.*

**Sharp formulations to use as design lenses going forward:**
- Files organise. The runner enforces. The tools decide. The model narrates.
- The model never asserts a fact it didn't get from a tool.
- Restrain the machine; don't thin the thinking.
- Graph as model, not engine; embedded-only, never a server.
- Competitors are sources, never substrate.
- Translate, don't transmit — hand them the labelled painting.

**Monday's incoming work (Mitch brings):** insights from running the case-based method-extraction (Phases 1–7 in the philosophy doc) on one real case → first armature pattern + first playbook seed.

**Companion docs Mitch references but NOT yet in the repo:** `kryptvakt-market-research-2026-05-29.md` and `heimdall-design-philosophy-and-method-extraction.md`. Ask before any work that would benefit from them — don't fabricate or reconstruct.

**How to apply:** When the Monday session starts, read the handover doc before doing anything else (it's the prep substrate for Mitch's case-based extraction). Do not start ratifying or coding ADRs before Mitch is in the room — these are *candidate* ADRs by Mitch's design (synthesis happens with him). [[feedback-codex-same-day-adr-challenge]] applies once ADRs are actually written, not to the handover itself.

Related: [[project-kryptvakt-heimdall]], [[project-kryptvakt-mvp1-principles]], [[project-kryptvakt-loki-observability]], [[reference-dora-rts-structure]], [[feedback-codex-same-day-adr-challenge]].
