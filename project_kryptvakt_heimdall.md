---
name: kryptvakt-heimdall-phase-3-agent-norse-naming-theme
description: "Kryptvakt Phase 3 ships Heimdall — read-only conversational agent on customer's BYO LLM. Norse-god naming theme for product components."
metadata: 
  node_type: memory
  type: project
  originSessionId: 25aa75a3-911b-4fba-aa2c-3ced07920f2a
---

**Decision date:** 2026-05-25 (1:30am middle-of-the-night clarity from Mitch).

**Heimdall = Phase 3 product.** Vertical-AI conversational interface to
the canonical kryptvakt model + curated playbook library. Designed to
run on the **customer's own LLM endpoint** (Azure OpenAI / AWS Bedrock /
GCP Vertex / OpenAI direct / on-prem Ollama). Mitch never sees a
customer query; customer's existing AI vendor governance covers it.

**Positioning:** Kryptvakt is not just monitoring — it is *embedded
expertise as a product*. The scrapers commoditize within 12 months of
shipping; an 8-year HSM/cryptography contractor's pattern library does
not. Heimdall is how that expertise becomes recurring revenue instead
of £700/day of Mitch's calendar. Estimated 5-10x ARR multiplier over
monitoring alone (compares against contractor day-rates and audit-firm
fees, not against Datadog).

**Non-negotiable architectural stances** — push back hard if a future
session proposes alternatives:

- **BYO LLM only.** Heimdall does NOT call Anthropic / OpenAI / etc on
  Mitch's behalf. Customer brings their endpoint. Removes Mitch from
  inference-cost + AI-vendor-relationship risk; lets customers reuse
  their existing AI governance.
- **Read-only by structure, not by advisory.** Heimdall has zero
  mutation tools. Same trust invariant that makes kryptvakt itself
  shippable into HSM environments.
- **Tool returns are typed JSON, never freetext.** OWASP LLM01 defence
  against prompt injection via attacker-controlled cert subjects, key
  labels, vendor error messages.
- **Heimdall queries audit-chain on the customer side.** Operator can
  review what was asked + answered.

**Norse-god naming theme.** Kryptvakt is Swedish-rooted ("crypt
watch"); product components draw from the adjacent Norse mythology
well. Heimdall = all-seeing watcher of Bifröst, hears the grass grow.
Other names are NOT pre-claimed — name new components as they emerge
(Mimir for a knowledge-base / search? Huginn-Muninn for alerting /
pattern detection? Bifröst for the secure data flow?). Do not
brainstorm unused names speculatively; let each component earn its name.

**Build-order discipline.** Heimdall is NOT on the active queue. Build
the source-substrate first (Tier 1+2 inventory expansion — Utimaco,
Vault, EJBCA, Teleport, AD CS, DPoD, Lemur). Heimdall has nothing
interesting to say at 3 sources; becomes magical at 8+. BUT every
Tier 1+2 scraper MUST capture grounding-metadata (compliance /
criticality / owner / rotation / notes) at config time — see
[[project_kryptvakt]] CONTEXT.md "Phase 3 intent — Heimdall" for the
schema. Retrofitting 8 scrapers later = pain; adding 5 env vars now =
trivial.

**Stealth-build posture.** Same as the rest of kryptvakt — design now,
ship post-relocation + Swedish AB. No external Heimdall mention until
the post-move activation sequence fires. See
[[feedback_kryptvakt_email_gate]].

**Full design is in `workspace/kryptvakt/CONTEXT.md`** under "Phase 3
intent — Heimdall" — read CONTEXT before reasoning about Heimdall
architecture in any future session.
