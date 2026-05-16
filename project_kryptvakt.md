---
name: Kryptvakt Project
description: Unified HSM monitoring product — Go Prometheus exporters for CipherTrust, Luna, PayShield. Phases 0–4 all merged to main; only active build project
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

## Phase status (as of 2026-05-16 — all on main)

| Phase | Description | Status |
|---|---|---|
| 0 | Repo scaffold, Go modules, fixture pattern, CI on GitHub Actions | ✅ done |
| 1 | CipherTrust Manager exporter vs VM 109 | ✅ done — live as systemd `ct-exporter.service` on VM 108, scraped by Prometheus, in Grafana |
| 2 | Luna PKCS#11 exporter vs SoftHSM2 | ✅ done — live as systemd `luna-exporter.service`, multi-vendor dashboard working, honest "DEV PROXY" label per ADR-001 |
| 3 | HTMX inventory UI, SQLite-backed | ✅ done — live at https://kryptvakt.mitchflix.co.uk, single-admin bcrypt per ADR-002 |
| 4 | docker-compose + OpenBao credential injection | ✅ done as packaging PR; not yet deployed over the systemd stopgap. Multi-binary image, KRYPTVAKT_ROLE dispatcher, profile-gated services. |

Trunk is `main`. All weekend feature branches (`ci/expand-triggers`, `phase-1-ciphertrust-exporter`, `phase-2-luna-exporter`, `phase-3-ui`, `phase-4-compose`, `phase-0-scaffold`) deleted from origin and local clones after PR #6 landed (umbrella merge).

Future branches naming convention (CI triggers match all): `feat/<thing>`, `fix/<thing>`, `phase-N-<thing>`.

ADRs shipped: ADR-001 (container as product, multi-protocol ingest, quorum scaffold, license gate), ADR-002 (single-admin UI auth).

## CI

`.github/workflows/ci.yml` on main — `go vet`, `go build`, `go test -race`, golangci-lint v1.61. Triggers on push to `main`/`phase-**`/`ci/**`/`feat/**`/`fix/**`, and on every PR.

## Lab dev environment (all live)

| VMID | Name | IP | Role |
|------|------|----|------|
| 108 | kryptvakt-dev | 192.168.50.108 | Go 1.23 + Docker + SoftHSM2 + node_exporter + Claude Code + gstack. SSH: `mitch@192.168.50.108` |
| 109 | ciphertrust-ce | 192.168.50.109 | CipherTrust Manager CE k170v-2.11.1. Fully provisioned: admin (WebUI) + ksadmin (console) creds in OpenBao, static IP, DNS resolving, JWT issuance verified |
| 105 | monitoring | 192.168.50.105 | Prometheus + Grafana. Both kryptvakt scrape jobs LIVE (`kryptvakt_ciphertrust` :9110, `kryptvakt_luna` :9111). 5 kryptvakt alert rules in `/etc/prometheus/rules/kryptvakt.yml` (also tracked at `homelab/prometheus/rules/kryptvakt.yml`). Grafana dashboard uid `kryptvakt-overview` at `monitoring.mitchflix.co.uk/d/kryptvakt-overview` |
| 107 | openbao | 192.168.50.107 | Secrets. Path `secret/kryptvakt/{ciphertrust,luna,payshield,softhsm,ui}` all seeded (ui added 2026-05-15) |
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
