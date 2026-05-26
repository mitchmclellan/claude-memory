---
name: dora-rts-structure-domain-trap
description: "DORA Level 1 Articles ≠ JC 2023 86 RTS Articles — the cryptography obligations live in RTS Articles 6+7 of the ICT Risk Management Framework RTS, mandated under DORA L1 Articles 15+16"
metadata:
  type: reference
  originSessionId: 25aa75a3-911b-4fba-aa2c-3ced07920f2a
---

DORA (Regulation (EU) 2022/2554) is the Level 1 regulation. Its
cryptography obligations do NOT live in "DORA Article 6." That's a
common misreading.

**Actual structure:**

- **DORA Level 1** (Regulation 2022/2554) — Articles 1-64 of the
  Regulation itself. Article 8 = ICT asset management; Article 15+16
  = mandate that the European Supervisory Authorities (ESAs: EBA,
  ESMA, EIOPA via the Joint Committee) develop Regulatory Technical
  Standards (RTS) operationalising the framework.
- **JC 2023 86** — Joint Committee Final Report on draft RTS on ICT
  Risk Management Framework, published January 2024. THIS is the RTS
  developed under DORA L1 Articles 15+16. The RTS has its own
  internal article numbering.
- **JC 2023 86 RTS Article 4** — ICT asset inventory
  (operationalises DORA L1 Article 8).
- **JC 2023 86 RTS Article 6** — Encryption and cryptographic
  controls (the encryption-policy obligations: at-rest, in-transit,
  approved algorithms by classification, authentication mechanisms).
- **JC 2023 86 RTS Article 7** — Cryptographic key management (the
  key-lifecycle obligations: generation, storage, distribution,
  retirement, archival, destruction, replacement; segregation of
  duties; HSM-or-equivalent; cryptographic agility).

**The trap:** writing "DORA Article 6 RTS" (or worse "DORA Article 6")
when you mean "Article 6 of the JC 2023 86 RTS that DORA mandates" is
sloppy enough that a DORA-literate buyer will flag it. Codex caught
this on kryptvakt ADR-006 — the framework key `DORA-ART6-RTS` was
renamed to `DORA-RTS-CRYPTO` in ADR-007 because the original token
read as "Article 6 of DORA Level 1."

**How to apply:**

- When writing about DORA cryptography obligations: cite RTS articles
  with the RTS instrument name (JC 2023 86) and the article number
  within the RTS. Don't blend DORA L1 article numbers with RTS article
  numbers.
- The cryptography evidence pack scope is RTS Articles 6+7 + DORA
  Article 8 + RTS Article 4 (asset inventory operationalises Article
  8). Naming should reflect this — `DORA-RTS-CRYPTO` for the
  cryptography subset, not `DORA-ART6-RTS`.
- Framework token mapping is at `internal/heimdall/playbooks/<key>/
  manifest.json` in the kryptvakt repo — the human-readable label
  expansion lives there, NOT in a DB enum, so the canonical
  source-of-truth doesn't require migration on new playbooks.
- DORA L1 also has Articles 17-23 (incident reporting), 24-27 (DORT
  + TLPT), 28-44 (ICT third-party risk). Each gets its own RTS family
  and own future playbook. They are out-of-scope for the v1 Tier 2
  DORA-RTS-CRYPTO launch playbook per ADR-007.

**Source documents to cite when authoring (verify against published
text, not paraphrase):**

- Regulation (EU) 2022/2554 — `eur-lex.europa.eu/eli/reg/2022/2554/oj`.
- JC 2023 86 Final Report — published Jan 2024 by EBA/ESMA/EIOPA,
  available from EBA at the regulatory-activities/operational-resilience
  path on `eba.europa.eu`.

See [[project_kryptvakt_heimdall]] for the product framing, ADR-007
in the kryptvakt repo for the binding launch decision, and
`internal/heimdall/playbooks/dora-rts-crypto/manifest.json` for the
canonical regulatory_anchors structure with article-level scope_references.
