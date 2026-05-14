---
name: Keep strategy.md and lab docs always current
description: Mitch requires strategy.md, lab-infra.md, and lab-improvements.md to never be stale — update them as part of every session that changes the lab
type: feedback
originSessionId: d58be864-9c0f-49e1-b61a-647c24fb708f
---
Always update strategy.md, lab-infra.md, and lab-improvements.md as part of any session that changes infrastructure, services, or architectural decisions. Never let these go stale.

**Why:** Mitch explicitly instructed this as a standing rule. These files are the memory layer between sessions and between models — stale docs mean the next session starts blind.

**How to apply:** Before closing out any session that touched the lab, update all three docs to reflect current true state. Tick off completed items in lab-improvements.md, add new blockers, and sync strategy.md to reflect what's actually built vs what was planned. This is not optional cleanup — it is part of the task.
