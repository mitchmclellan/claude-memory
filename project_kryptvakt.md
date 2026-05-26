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

## Phase status (as of 2026-05-19)

| Phase | Description | Status |
|---|---|---|
| 0 | Repo scaffold, Go modules, fixture pattern, CI on GitHub Actions | ✅ done |
| 1 | CipherTrust Manager exporter vs VM 109 | ✅ done — was standalone `ct-exporter.service`, now subsumed by Phase 1d unified `kryptvakt.service` |
| 2 | Luna PKCS#11 exporter vs SoftHSM2 | ✅ done — was standalone `luna-exporter.service`, now subsumed by Phase 1d unified `kryptvakt.service` |
| 3 | HTMX inventory UI, SQLite-backed | ✅ done — was standalone `kryptvakt-ui.service`, now subsumed by Phase 1d unified `kryptvakt.service` |
| 4 | docker-compose + OpenBao credential injection | ✅ done as packaging PR; never deployed. Superseded by Phase 1d unified binary approach. |
| 1b | Canonical model + driver framework + license gate + audit chain + reconciliation + lifecycle retirement + unified `kryptvakt` cobra binary | ✅ merged to main `cab16fc` on 2026-05-19. 16 lane commits, 4 /cso passes. |
| 1c | UI read-path cutover (ListDevices+derived Status) + admin auth refactor (appliance default + change on first login) + cookie auto-detect + sign-out display fix + migration 0004 drops legacy hsms | ✅ merged to main `e692c55` on 2026-05-19. 6 commits. devices+sources is the SOLE inventory source of truth; hsms table dropped. |
| 1d step 1 | `kryptvakt run --with-ct --with-luna --with-ui` (`--all`) subsumes the three standalone binaries as concurrent goroutines in one process. License loading degrades gracefully on dev/lab builds (no embedded pubkey). | ✅ on branch `phase-1d-run-scrapers` (`c1e87e3` → `c7c1bce` → `2c1945e`), deployed to VM 108 as single systemd unit `kryptvakt.service`. Old three units stopped + disabled (binaries kept for rollback). |

Trunk is `main`. Feature branches deleted from origin + local after merge: `phase-1b-canonical-model` (2026-05-19), `phase-1c-ui-cutover` (2026-05-19). Currently open: `phase-1d-run-scrapers` (pending merge). Naming convention: `feat/<thing>`, `fix/<thing>`, `phase-N-<thing>`.

ADRs shipped: ADR-001 (container as product, multi-protocol ingest, quorum scaffold, license gate), ADR-002 (single-admin UI auth — **evolved 2026-05-19 to appliance default + change on first login; see below**), ADR-003 (canonical model + flavour taxonomy + three-layer storage + driver interface, 2026-05-18).

**Audit signer V1 posture (decided 2026-05-19):** Phase 1b ships an in-process file-backed Ed25519 SidecarSigner at `internal/audit/filesigner.go`, NOT the kryptvakt-audit co-process binary the design called for under /cso F-4. The interface (`SidecarSigner`/`SidecarReader`) is stable so the co-process swap is a drop-in; the V1 posture trades F-4 OS-level user separation for shipping speed. Future sessions: do NOT propose ripping out `filesigner.go` — propose ADDING `cmd/kryptvakt-audit/` alongside.

**ADR-002 evolution — UI admin auth (merged to main 2026-05-19 as part of `e692c55`):** Standard appliance pattern: default `admin:admin`, force change on first login via a SQLite-backed bcrypt hash (migration `0003_ui_admin`, single-row CHECK id=1, `is_default` flag). The daemon seeds the row on first startup; `requireSession` middleware force-redirects to `/change-password` while `is_default=1`. `KRYPTVAKT_UI_ADMIN_PASSWORD` env is OPTIONAL — when set, used as the seed value instead of `"admin"`; once changed via the UI, env is ignored. Future sessions: do NOT propose reverting to env-var-only or OpenBao-templated auth — see `feedback_kryptvakt_appliance_auth.md`.

**Cookie Secure auto-detect (merged with Phase 1c):** `setSessionCookie` reads request scheme from `r.TLS` / `X-Forwarded-Proto` and sets the Secure flag accordingly. `cfg.CookieSecure=true` remains as the explicit operator override. Prevents the bug class where direct-HTTP-IP deployments silently drop sessions because Secure cookies require HTTPS.

**Production deployment on VM 108 (as of 2026-05-19, after Phase 1d step 1):**
- Single systemd unit: `kryptvakt.service` runs `~/workspace/kryptvakt/bin/kryptvakt run --all` (EnvironmentFile loads `/etc/kryptvakt/{ct-exporter,luna-exporter,ui}.env`)
- Workloads in-process: CT scraper (:9110), Luna scraper (:9111), UI (:9120). All share one SQLite at `/var/lib/kryptvakt/kryptvakt.db`
- Three old units (`ct-exporter`, `luna-exporter`, `kryptvakt-ui`) stopped + disabled; binaries kept at `~/workspace/kryptvakt/bin/*.pre-1c-final` for rollback
- `ui.env`: `KRYPTVAKT_UI_ADMIN_PASSWORD` commented out (Mitch changed it via UI); `KRYPTVAKT_UI_COOKIE_SECURE=false` (lab HTTP-only deploy; auto-detect makes this a no-op but it's explicit). Backups at `ui.env.pre-1c.bak`
- `hsms` table dropped (migration 0004 applied); current tables: audit_chain, cert_key_bindings, cert_revocations, certificates, cross_source_links, devices, events, keys, licenses, partitions, policies, pqc_assessments, schema_migrations, sources, ui_admin, verifier_state

**Active branch posture:** Phase 1c merged. `phase-1d-run-scrapers` open with 3 commits (unified daemon + graceful license-degrade + docs); awaiting visual verify before merge. Next-priority queued items (per kryptvakt/NEXT-STEPS.md): license-gate wiring around scraper Poll calls; deprecate `cmd/ct-exporter` / `cmd/luna-exporter` / `cmd/kryptvakt-ui` after unified-binary uptime proves out; scraper retirement integration; `kryptvakt-audit` co-process binary (/cso gated).

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
