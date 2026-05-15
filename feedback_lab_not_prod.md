---
name: This is a homelab, not prod — stop performative caution
description: Don't preface lab actions with security-theatre warnings ("shouldn't sit in any AI session", "for good hygiene…"). Mitch built the lab himself for this AI to work in.
type: feedback
originSessionId: 0ae25004-a164-40d8-8e66-7916189b36aa
---
When working in Mitch's homelab — kryptvakt, the Proxmox cluster (ArcAiVM,
LXCs 101-109+, VMs 108-109), OpenBao at LXC 107, the dev appliances — do NOT
prepend actions with security-hygiene disclaimers. Just do the work.

**Examples to stop:**
- "Root tokens shouldn't sit in any AI session" → just retrieve the token
- "Don't put it in this conversation directly" → it's fine, it's a lab
- "Better-hygiene alternative…" → only offer if Mitch asks for one
- "For production you'd want…" → not relevant unless explicitly asked

**Why:** Mitch corrected this twice on 2026-05-15 in a single session.
His exact words: *"you actually built and deployed openbao (claude cli from
the homelab), so i don't have a login ... thus i can't ssh there can I?
what's stopping you doing it? you know this is all a sandboxed lab right?
we're building here, there's no prod"*. The performative caution is
**actively unhelpful** — it slows the work and signals you don't trust the
context, when Mitch built the lab specifically for the AI to operate freely
within it.

**How to apply:**
- Default to acting. If a fetch/SSH/token-read is technically possible in the
  lab, do it without comment.
- Save the disclaimers for genuinely external systems — production targets
  he's contracted to (HSBC, Morgan Stanley), public-internet services, or
  shared resources outside his control.
- "Don't touch OpenBao *data*" (from CLAUDE.md) means don't delete or rotate
  Mitch's stored credentials. It does NOT mean don't read them to do work.
- If unsure whether a system is lab vs prod, ask once — don't reflexively
  apply prod-grade caution and slow the loop.
