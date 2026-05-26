---
name: vendor-portal-signups-pending-verification-2026-05-26
description: Thales DPoD and Utimaco simulator portal accounts are submitted but pending vendor-side verification. Do NOT re-ask Mitch about these — vendor turnaround can take days/weeks.
metadata: 
  node_type: memory
  type: project
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

As of 2026-05-26, Mitch has submitted signup requests for both vendor portals; both are waiting on the vendor's human verification step.

**Pending portals:**

1. **Utimaco simulator portal** (https://hsm.utimaco.com/products-services/hsm-simulators-and-trial/) — blocks `MITCH-37` (GP simulator + Atalla AT1000 Payment HSM sim) → blocks NEXT-STEPS #2 (Utimaco GP scraper, second HSM vendor). Mitch used "Platform Smart" as company name, personal email `mitch.c.mclellan@gmail.com`. Stealth-build language used — no mention of Kryptvakt.

2. **Thales DPoD trial** — blocks `MITCH-38` (30-day cloud-HSM sprint window). Personal email, same stealth-build rules.

**Why:** Both vendors gate simulator/trial access behind manual company verification. Turnaround is typically a few business days; sometimes longer if the vendor sends qualification follow-ups.

**How to apply:**

- **Do NOT poll** Mitch on these (don't ask "any update from Utimaco?" — he'll volunteer when access lands).
- **Do NOT** suggest re-submitting or chasing the vendor unless 2+ weeks have passed and Mitch hasn't mentioned status.
- When Mitch says "Utimaco/Thales access arrived", THEN ask for download URLs + license keys and proceed with [[project_kryptvakt.md]] NEXT-STEPS #2 / #6.
- For weekly retros / status questions: just say "pending vendor verification" without further detail unless Mitch asks.

**Atalla AT1000 specifically:** Mitch noted 2026-05-26 that the Atalla sim is on the same Utimaco portal flow (Utimaco owns the Atalla brand). Same signup, same pending state. He used "Platform Smart" as company name — note for consistency if any follow-up vendor contact happens.

See also: [[feedback_kryptvakt_email_gate.md]] (no UK enterprise outreach pre-AB), [[feedback_do_not_checklist_when_you_have_access.md]] (legitimate human-gated items vs. ones I should just do).
