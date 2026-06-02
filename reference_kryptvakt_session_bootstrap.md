---
name: kryptvakt-session-bootstrap-load-homelab-framework-context
description: "For any kryptvakt work, read ~/workshop/homelab/CLAUDE.md + lab-infra.md + lab-improvements.md at session start. Kryptvakt sits inside the homelab, not beside it."
metadata: 
  node_type: memory
  type: reference
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

Kryptvakt is not a standalone codebase — it depends on the homelab
infrastructure for every part of the build:

- VM 108 (kryptvakt-dev) is the dev host. Go toolchain, Docker, SoftHSM2, bao CLI, the actual build artifacts.
- VM 109 (CipherTrust CE) is the scrape target.
- VM 111 (winsrv-adcs) is the AD CS scrape target.
- LXC 107 (OpenBao) holds every credential at `secret/kryptvakt/*`.
- LXC 105 (Prometheus + Grafana + Loki) consumes the exporter metrics + log stream.
- LXC 106 (Pi-hole) resolves the appliance DNS.
- LXC 110 (pki-lab) hosts the Vault + EJBCA Docker containers (scrape sources).

Working on kryptvakt from `~/workshop/kryptvakt/CLAUDE.md` alone misses all of this. The homelab framework (`~/workshop/homelab/`) has the rules that govern how kryptvakt operates — context loading, testing protocol, root access posture, lab freshness, model routing.

**At the start of every kryptvakt session, read in this order:**

1. `~/workshop/homelab/CLAUDE.md` — the framework rules (mandatory context loading, testing protocol, root access, conventions). These OVERRIDE generic defaults.
2. `~/workshop/homelab/lab-infra.md` — current IP/VMID/service state. Grep here before asking "where is X". Authoritative.
3. `~/workshop/homelab/lab-improvements.md` — open issues + "Needs Mitch" backlog. Existing blockers may affect what you can do.
4. `~/workshop/kryptvakt/CLAUDE.md` — the kryptvakt-specific stack, dev workflow, and security gate.
5. `~/workshop/kryptvakt/CONTEXT.md` — decisions/open questions.
6. `~/workshop/kryptvakt/NEXT-STEPS.md` — top of queue + Mitch-blocked.
7. `~/workshop/kryptvakt/docs/MVP1-design.md` if the work touches build sequencing or scope.

(Path note: the laptop working dir migrated from `~/workspace` to `~/workshop` on 2026-06-01 — see [[project_laptop_migration_2026_06_01]]. PVE-host paths under `~/workspace/` are unchanged and are referenced as such on VM 108.)

**Lens to apply when reading the framework:** filter through "how does this shape the kryptvakt build". Examples:

- Homelab CLAUDE.md "Testing Protocol" — translates to: test the kryptvakt scrapers against the actual path Mitch uses to verify it (SSH from laptop terminal → run binary → curl `/metrics` from his browser/terminal). Not just `go test` passing. For UI flows requiring auth/cookies, use the gstack browse tool per [[feedback_browser_test_ui_flows]] (subject to the Playwright/Chromium-on-26.04 gap noted in [[reference_laptop_env]]).
- Homelab "Model Routing" — kryptvakt building is Claude Code (Sonnet/Opus). Status pings about kryptvakt go via Telegram bot which is also Sonnet direct (Ollama routing is unfunded until ArcAiVM RAM expansion — see [[feedback_model_routing]]).
- Homelab "Root Access" — Claude Code on pve has NOPASSWD sudo. From the laptop, root SSH to LXCs is NOT keyed; reach via `mitch@192.168.50.10` with `sudo -n pct exec`. See [[reference_lab_access]].
- Homelab "Project Init Flow" — already done for kryptvakt; the structure (CLAUDE.md / CONTEXT.md / REFERENCE.md / NEXT-STEPS.md / HISTORY.md) is what to expect on disk.

**At the end of any kryptvakt session that changed lab state** (installed a tool, changed an IP, captured fixtures, ran a smoke test that proves a new capability), update `lab-infra.md` + `lab-improvements.md` per the homelab Docs Freshness rule ([[feedback_docs_freshness]]). The lab and kryptvakt docs evolve together.
