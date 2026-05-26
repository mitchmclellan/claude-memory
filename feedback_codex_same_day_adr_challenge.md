---
name: codex-same-day-adr-challenge-pattern
description: After landing a strategic ADR, run /codex against it same-day before docs propagate; if 2+ premises break, write a superseding ADR rather than editing in place
metadata:
  type: feedback
  originSessionId: 25aa75a3-911b-4fba-aa2c-3ced07920f2a
---

When a strategic ADR (product structure, scope, wedge, sequencing)
lands in a project that has an ADR convention forbidding in-place
edits, **run `/codex` against it the same day** before downstream
docs and code propagate against it. The /codex skill returns
structured verdicts (`BROKEN` / `NEEDS-EVIDENCE` / `DEFENSIBLE` /
`MIXED`) per pressure-test item that make the keep-or-supersede
decision mechanical.

**Why:** Demonstrated 2026-05-26 on kryptvakt ADR-006 → ADR-007.
ADR-006 was the office-hours #2 output (dual-track DORA + PCI Tier 2
launch). /codex challenge ~45 minutes after landing returned 3 of 4
pressure-tested premises BROKEN — "zero opportunity cost" calendar
claim, 4-8 week authoring estimate (real estimate 12-24 weeks),
"DORA Article 6 RTS" naming (conflates DORA Level 1 with JC 2023 86
RTS internal articles), plus 1 NEEDS-EVIDENCE on the budget-bucket
claim. The corrections meant the dual-track decision itself was
justified by claims that didn't hold up.

Because the ADR convention is "never edit historical ADRs in place;
supersede with a new one that references the old," and because the
findings were strong enough to flip the decision, the right move was
ADR-007 same-day rather than editing ADR-006. The cost of the extra
supersession hop in the ADR chain is bounded; the cost of preserving
a wrong decision through future readers is unbounded.

**How to apply:**

- For any session that lands a strategic ADR (or equivalent — design
  doc, scope decision), schedule `/codex` against it before the
  session ends OR within 48 hours, before downstream docs reference
  the decision concretely. The closer to landing, the smaller the
  rollback cost if a finding breaks a premise.
- Pressure-test the ADR's load-bearing premises explicitly — the
  /codex prompt should list 3-6 specific claims to challenge, not a
  generic "review this." Vague prompts get vague verdicts.
- If 2+ premises return BROKEN AND a fourth alternative path is
  available, write a superseding ADR same-day rather than editing.
  Update the prior ADR's `Status` field to point at the new one
  (Status field edits don't count as in-place content edits — they're
  meta-references to the supersession chain).
- If 0-1 premises return BROKEN, fold the corrections into the prior
  ADR via a "Findings" or "Amendments" section appended to the same
  file (or just update the Status to "Accepted post-codex review").
- For kryptvakt specifically, the ADR review-plan SHOULD include
  "Re-run `/codex` once it's lived for ~48 hours" as a standing
  same-week stability gate (per ADR-007's own review plan). The
  same-day correction is fast; the same-week stability is the real
  durability test.

This is a discipline pattern, not just a kryptvakt one. Applies any
time a strategic decision is encoded in a write-once-supersede-by-new
log (ADRs, decision logs, design docs in a Markdown chain).
