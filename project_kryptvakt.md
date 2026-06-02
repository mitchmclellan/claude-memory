---
name: kryptvakt-project
description: Unified HSM/PKI/SSH monitoring product. Phase 2m runtime-truth hardening closed; Heimdall Slice 2 live (PR
metadata: 
  node_type: memory
  type: project
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

Kryptvakt is Mitch's ONLY active build project. The lab has been refocused on it since 2026-05-13 (day-trading, absorb-the-borg, weather-app archived).

**Why:** Mitch is a cryptography/HSM infrastructure engineer (8 yrs). Connects his professional domain to the lab; commercial candidate post-relocation + Swedish AB (see [[feedback_kryptvakt_email_gate]]).

## Repo
- **Remote:** `git@github.com:kryptvakt/kryptvakt.git` (the `kryptvakt` GitHub org)
- **Local:** `/home/mitch/workshop/kryptvakt/` on the laptop (NOT `~/workspace/`); also at `~/workspace/kryptvakt/` on VM 108 (VM-side path is unchanged)
- Trunk: `main`; long-lived feature branch `phase-2m-runtime-truth-hardening` is the active integration branch
- Single `cmd/kryptvakt` cobra binary + `internal/{heimdall,db,license,audit,driver,scrapers/*,ui,...}`

## Live state on VM 108 (2026-06-01)
- `kryptvakt.service` active. Binary at `/home/mitch/workspace/kryptvakt/bin/kryptvakt` (May 28 build = Slice 2). Service is single-process with all workloads as goroutines.
- 9 sources in the canonical model (`sources` table):
  | id | flavour | vendor | endpoint | tier | rotation | last_poll |
  |----|---------|--------|----------|------|----------|-----------|
  | 1 | monitoring | thales-ct | https://192.168.50.109 | tier-1 | 365 | complete |
  | 2 | monitoring | thales-luna | softhsm:?token=kryptvakt-dev | tier-0 | 365 | **partial** |
  | 3 | pki | openbao-pki | http://192.168.50.107:8200/pki | tier-1 | 90 | complete |
  | 4 | ssh | openssh | (4 LAN hosts) | tier-0 | 365 | complete |
  | 5 | pki | step-ca | https://localhost:9000 | tier-2 | 365 | complete |
  | 6 | pki | hashicorp-vault | http://127.0.0.1:18200/pki (via autossh tunnel to VPS) | tier-1 | 90 | complete |
  | 7 | pki | adcs | https://winsrv-adcs.mitchflix.co.uk:5986/wsman | tier-0 | 90 | complete |
  | 8 | ssh | openssh | (duplicate of 4 with reordered targets) | tier-0 | 365 | complete |
  | 9 | pki | ejbca | https://192.168.50.110:8443/ejbca | tier-1 | 90 | complete |
- Heimdall runs (`heimdall_runs` table): 5 total, 4 succeeded, 1 failed (smoke run #1 from Slice 2 night). LLM mode = gpt-4o (~10s/run, ~2.7k tokens), deterministic mode ~300ms.
- Env files under `/etc/kryptvakt/`: ct-exporter.env, luna-exporter.env, ui.env, audit.env, openai.env, pki.env, vault-ce.env, stepca.env, ssh.env, adcs.env, ejbca.env. ADCS + EJBCA wired with grounding metadata via `KRYPTVAKT_<SOURCE>_GROUNDING` JSON envs.

## Phase status (current as of 2026-06-01)
- ✅ Phase 0–1d: legacy CT/Luna/UI subsumed into unified `kryptvakt run --all` binary
- ✅ Phase 1b/1c: canonical model, driver framework, license gate, audit chain, lifecycle retirement, appliance-style UI auth (admin:admin + force-change)
- ✅ Phase 2: PKI flavours (OpenBao, step-ca, EJBCA), SSH flavour, AD CS (Windows)
- ✅ Phase 2m runtime-truth hardening (closed 2026-05-27): persist-failure completeness propagation, lifecycle/audit error sentinels, MaxSources enforcement, errors.Join in runOnce, source-identity stability
- ✅ Heimdall Slice 0 (paper prototype) + Slice 1 (env-var grounding wiring across 8 source initializers) + Slice 2 (`internal/heimdall/` package + 4 tools + playbook runner + `/heimdall` UI panel + DORA-RTS-CRYPTO playbook + LLM/deterministic modes + evidence pack)
- 🟡 Slice 2.5 polish pending: `goldmark.WithRendererOptions(html.WithUnsafe())` to restore inline `<details>` collapsibility in UI render (download `.md` is unaffected)
- 🟡 ADR-007 citation fix pending (draft "JC 2023 86 RTS" → in-force CDR (EU) 2024/1774, Arts 6–7)
- 🔄 Phase 3 Heimdall reasoning expansion — see [[project_kryptvakt_handover_2026_05_31]] for 5 candidate ADRs awaiting Mitch's Monday method-extraction work

## CI
`.github/workflows/ci.yml` — `go vet`, `go build`, `go test -race`, golangci-lint v2.12.2 (bumped 2026-05-26). Targets Go 1.25.10 via go.mod toolchain.

## Lab dev environment (live)
See [[project_lab_state]] for the full inventory. Kryptvakt-relevant subset:
- VM 108 kryptvakt-dev → daemon, SoftHSM2, step-ca, autossh tunnel to VPS Vault
- VM 109 ciphertrust-ce → CT Manager scrape target
- VM 111 winsrv-adcs → AD CS scrape target (HTTPS:5986 + mTLS)
- LXC 105 monitoring → Prometheus + Grafana + Loki (kryptvakt observability spine)
- LXC 107 openbao → secrets at `secret/kryptvakt/{ciphertrust,luna,payshield,softhsm,ui,adcs,ejbca,vault-ce,teleport}`
- LXC 110 pki-lab → Vault dev + EJBCA CE Docker containers (scrape targets)
- VPS 185.132.43.4 → upstream Vault CE + Teleport CE (Vault scraped via autossh tunnel)

## Working knowledge
- ksadmin SSH key: `~/.ssh/id_rsa_ciphertrust` (on both pve and laptop)
- CT API surface: see [[reference_ciphertrust_api]] (vendor docs lie about which paths exist)
- Session bootstrap order: see [[reference_kryptvakt_session_bootstrap]]
- MVP1 principles (ADR-001): see [[project_kryptvakt_mvp1_principles]]
- Loki on LXC 105 is load-bearing for kryptvakt: see [[project_kryptvakt_loki_observability]]

## Hard rules
- No outreach to Stu/Saba/HSM buyers until post-move + Swedish AB exists ([[feedback_kryptvakt_email_gate]]) — overrides any NEXT-STEPS or design-doc text suggesting outreach
- All secrets via OpenBao — no hardcoded creds
- `/cso` + `scripts/multi-audit.sh` two-pass gate mandatory before PRs touching credential loading, TLS, HSM/PKI protocol handling, audit-chain sidecar permissions, scope_config validation (per `kryptvakt/CLAUDE.md`)
- Same-day codex challenge of new strategic ADRs ([[feedback_codex_same_day_adr_challenge]])

## How to apply
For new kryptvakt work, also read `workshop/kryptvakt/{CLAUDE.md,CONTEXT.md,NEXT-STEPS.md,docs/MVP1-design.md}` — kryptvakt has its own roadmap doc, not derivable from the homelab framework.
