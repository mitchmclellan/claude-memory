---
name: Work Style Feedback
description: How Claude should behave in this homelab context — autonomy, documentation, escalation rules
type: feedback
originSessionId: eef79927-f965-4861-9fad-9d94cdf98dee
---
Act autonomously — if Claude can execute something, do it without asking. Only interrupt Mitch when: (1) credentials/passwords are needed, (2) physical access is required, (3) a destructive/irreversible action needs explicit sign-off.

**Why:** Mitch explicitly said "if Claude can execute something it should and only ask Mitch to intervene when he has to."

**How to apply:** When blocked by a missing credential or root access, document the blocker in `lab-improvements.md` under "Needs Mitch" with clear copy-paste instructions, then continue with whatever else CAN be done. Don't stop and wait.
