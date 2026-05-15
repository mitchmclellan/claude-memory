---
name: Kryptvakt Project
description: Unified HSM monitoring product — Go Prometheus exporters for CipherTrust, Luna, PayShield. Phase 1 in flight; only active build project
type: project
originSessionId: 351b90a3-9068-4f06-9fc0-e0a7e6205ed8
---
Kryptvakt is Mitch's ONLY active build project. Lab has been fully refocused on it (day-trading, absorb-the-borg, weather-app all archived).

**Why:** Mitch is a cryptography/HSM infrastructure engineer (8 yrs). This connects his professional domain to the lab and is the candidate for going commercial.

## Repo

- **Remote:** `git@github.com:kryptvakt/kryptvakt.git` (the `kryptvakt` GitHub org, not `mitchmclellan/`)
- **Local:** `/home/mitch/workspace/kryptvakt/` on the laptop; also cloned on pve
- Phase 1 + Phase 2 cmd binaries both scaffolded: `cmd/ct-exporter`, `cmd/luna-exporter`, `cmd/kryptvakt-ui`
- Internal modules: `internal/{ciphertrust,config,db,fixtures,model}`

## Phase status (as of 2026-05-15)

| Phase | Description | Status |
|---|---|---|
| 0 | Repo scaffold, Go modules, fixture pattern | ✅ done |
| 1 | CipherTrust Manager exporter vs VM 109 | 🔧 binary built, REST client done, inventory parser done, VM 109 fixtures captured, `db.UpsertHSM` done; **needs end-to-end run + Prometheus scrape enabled** |
| 2 | Luna PKCS#11 exporter vs SoftHSM2 | 🔧 `cmd/luna-exporter` scaffolded |
| 3 | HTMX inventory UI, SQLite-backed | 🔧 `cmd/kryptvakt-ui` scaffolded |
| 4 | docker-compose, air-gap bundle, demo prep | ⏳ not started |

ADRs shipped: ADR-001 (MVP1 scope expansion — container as product, multi-protocol ingest, quorum scaffold, license gate), ADR-002 (MVP1 UI auth scoped down to single admin).

## Lab dev environment (all live)

| VMID | Name | IP | Role |
|------|------|----|------|
| 108 | kryptvakt-dev | 192.168.50.108 | Go 1.23 + Docker + SoftHSM2 + node_exporter + Claude Code + gstack. SSH: `mitch@192.168.50.108` |
| 109 | ciphertrust-ce | 192.168.50.109 | CipherTrust Manager CE k170v-2.11.1. Fully provisioned: admin (WebUI) + ksadmin (console) creds in OpenBao, static IP, DNS resolving, JWT issuance verified |
| 105 | monitoring | 192.168.50.105 | Prometheus + Grafana. Kryptvakt scrape targets pre-configured but commented-in — uncomment when ct-exporter is running |
| 107 | openbao | 192.168.50.107 | Secrets. Path `secret/kryptvakt/{ciphertrust,luna,payshield,softhsm}` all seeded |
| 106 | pihole | 192.168.50.106 | DNS — `kryptvakt-dev.mitchflix.co.uk` → .108, `ciphertrust.mitchflix.co.uk` → .109 |

## Working knowledge

- ksadmin SSH key: `~/.ssh/id_rsa_ciphertrust` (on both pve and laptop). Needed for direct ksadmin access — the CE WebUI requires SSH pubkey upload before WebUI auth completes.
- CipherTrust admin password was rotated from `admin/admin` via `PATCH /api/v1/auth/changepw` (not the WebUI page).
- See `reference_ciphertrust_api.md` for which CT Manager CE API endpoints actually work (many vendor-doc paths 404 on v2.11.1).
- See `reference_kryptvakt_session_bootstrap.md` for the load-order at session start (homelab CLAUDE.md → lab-infra → lab-improvements → kryptvakt CONTEXT + NEXT-STEPS).

## Dev workflow (gstack-driven)

Sprint pipeline per kryptvakt/CLAUDE.md:
1. `/office-hours` — product interrogation, new features
2. `/autoplan` — CEO + eng + design review
3. Build
4. `/review` — code audit; `/cso` for credential/TLS/protocol code
5. `/ship` — PR + push

OpenClaw on ArcAiVM has gstack dispatch routing — `"run office hours for kryptvakt"` via Telegram triggers the full pipeline.

## Hard rules

- No outreach to Stu/Saba/HSM buyers until Mitch says so. See `feedback_kryptvakt_email_gate.md` — this rule **overrides** anything `NEXT-STEPS.md` or design docs say about outreach.
- All secrets via OpenBao — no hardcoded creds in exporter code.
- `/cso` is mandatory before any PR that touches credential loading, TLS, or HSM protocol handling.

## How to apply

For new kryptvakt work, also read `workspace/kryptvakt/{CLAUDE.md,CONTEXT.md,NEXT-STEPS.md,docs/MVP1-design.md}` — kryptvakt is its own repo with its own roadmap doc, not derivable from the homelab framework alone.
