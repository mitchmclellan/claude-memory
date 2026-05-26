---
name: Keep lab docs AND project docs always current
description: Mitch requires strategy.md, lab-infra.md, lab-improvements.md, AND every active project's CONTEXT.md + NEXT-STEPS.md to never be stale — update them as part of every session that touches the relevant area
type: feedback
originSessionId: d58be864-9c0f-49e1-b61a-647c24fb708f
---
Always update the relevant set of docs at the end of every session that
changes lab state OR project state. The rule applies in TWO scopes:

**Lab-level docs** (when the session changes infra/services/lab decisions):
- `homelab/strategy.md`
- `homelab/lab-infra.md`
- `homelab/lab-improvements.md`

**Project-level docs** (when the session changes a specific project's state):
- `workspace/<project>/CONTEXT.md` — decisions, open questions, current state
- `workspace/<project>/NEXT-STEPS.md` — what's next, gates, blockers

**Why:** Mitch explicitly instructed this as a standing rule. These files are the memory layer between sessions and between models — stale docs mean the next session starts blind. **Recurring failure mode (observed 2026-05-18):** gstack skills (`/office-hours`, `/plan-eng-review`) write their own artifacts to `~/.gstack/projects/...` and treat that as "done", leaving the project's CONTEXT.md and NEXT-STEPS.md stale. Mitch caught this and corrected: "find out why it went stale in teh first place and fix that".

**How to apply:**
- Lab session touched infra/services? Update all three lab docs.
- Skill session touched a specific project (kryptvakt, day-trading, absorb-the-borg, etc.)? Update that project's CONTEXT.md and NEXT-STEPS.md alongside whatever artifact the skill itself produced. The skill's gstack artifact is NOT a substitute for the project doc.
- Tick off completed items, add new blockers, sync to reflect what's actually built vs what was planned.
- This is part of the task, not optional cleanup. Doing the work and leaving the docs stale = task incomplete.

**Project-level reinforcement:** each project's own CLAUDE.md should also carry an explicit docs-freshness clause so the rule is loaded at session start without depending on this memory being recalled. The homelab CLAUDE.md already requires this loading; project CLAUDE.md files should restate the freshness obligation locally.
