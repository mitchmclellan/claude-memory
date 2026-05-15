---
name: Kryptvakt session bootstrap — load homelab framework context
description: For any kryptvakt work, read /home/mitch/workspace/homelab/CLAUDE.md + lab-infra.md + lab-improvements.md at session start. Kryptvakt sits inside the homelab, not beside it.
type: reference
originSessionId: 0ae25004-a164-40d8-8e66-7916189b36aa
---
Kryptvakt is not a standalone codebase — it depends on the homelab
infrastructure for every part of the build:

- VM 108 (kryptvakt-dev) is the dev host. Go toolchain, Docker, SoftHSM2,
  bao CLI, the actual build artifacts.
- VM 109 (CipherTrust CE) is the scrape target.
- LXC 107 (OpenBao) holds every credential at `secret/kryptvakt/*`.
- LXC 105 (Prometheus + Grafana) consumes the exporter metrics.
- LXC 106 (Pi-hole) resolves the appliance DNS.

Working on kryptvakt from `/home/mitch/workspace/kryptvakt/CLAUDE.md` alone
misses all of this. The homelab framework (`/home/mitch/workspace/homelab/`)
has the rules that govern how kryptvakt operates — context loading,
testing protocol, root access posture, lab freshness, model routing.

**At the start of every kryptvakt session, read in this order:**

1. `/home/mitch/workspace/homelab/CLAUDE.md` — the framework rules
   (mandatory context loading, testing protocol, root access, conventions).
   These OVERRIDE generic defaults.
2. `/home/mitch/workspace/homelab/lab-infra.md` — current IP/VMID/service
   state. Grep here before asking "where is X". Authoritative.
3. `/home/mitch/workspace/homelab/lab-improvements.md` — open issues +
   "Needs Mitch" backlog. Existing blockers may affect what you can do.
4. `/home/mitch/workspace/kryptvakt/CLAUDE.md` — the kryptvakt-specific
   stack, dev workflow, and security gate.
5. `/home/mitch/workspace/kryptvakt/CONTEXT.md` — decisions/open questions.
6. `/home/mitch/workspace/kryptvakt/docs/MVP1-design.md` if the work
   touches build sequencing or scope.

**Lens to apply when reading the framework:** filter through "how does this
shape the kryptvakt build". Examples:

- Homelab CLAUDE.md "Testing Protocol" — translates to: test the CT
  exporter against the actual path Mitch uses to verify it (SSH from
  Windows WSL → run binary → curl `/metrics` from his browser/terminal).
  Not just `go test` passing.
- Homelab "Model Routing" — kryptvakt building is Sonnet/Opus
  (appropriate spend). Status pings about kryptvakt should go to Telegram
  Qwen (free).
- Homelab "Root Access" — Claude Code on pve has NOPASSWD sudo. From WSL
  workspace, root SSH to LXCs is NOT keyed; reach via `mitch@pve` with
  `sudo -n pct exec`. See `reference_lab_access.md`.
- Homelab "Project Init Flow" — already done for kryptvakt; the structure
  (CLAUDE.md / CONTEXT.md / REFERENCE.md) is what to expect on disk.

**At the end of any kryptvakt session that changed lab state** (installed
a tool, changed an IP, captured fixtures, ran a smoke test that proves a
new capability), update `lab-infra.md` + `lab-improvements.md` per the
homelab Docs Freshness rule. The lab and kryptvakt docs evolve together.
