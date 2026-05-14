---
name: Kryptvakt Project
description: Unified HSM monitoring product — Go Prometheus exporters for CipherTrust, Luna HSM, PayShield; ONLY active project, lab fully focused on it
type: project
originSessionId: 99e8964b-901d-4108-8600-c5cf05b678fb
---
Kryptvakt is Mitch's ONLY active project. Lab has been fully refocused. All other projects (day-trading, absorb-the-borg, weather-app) killed and archived.

**Why:** Mitch is a cryptography/HSM infrastructure engineer (8yrs). This connects his professional domain with the lab.

**Architecture:**
- Go exporter binaries (one per device type), Docker-composed on kryptvakt-dev
- Prometheus on LXC 105 scrapes exporters (targets commented-in, ready to enable)
- CipherTrust CE (VM 109) is the dev target for the CipherTrust exporter
- SoftHSM2 in Docker substitutes for real Luna HSM hardware in dev
- OpenBao (LXC 107, secret/kryptvakt/) stores all HSM credentials

**Dev Workflow:**
- gstack installed at `~/.claude/skills/gstack/` (47 skills, browse binary)
- Sprint: /office-hours → /autoplan → build → /review → /cso → /ship
- OpenClaw has gstack dispatch routing + 4 native skills (office-hours, ceo-review, investigate, retro)
- Via Telegram: "run office hours for kryptvakt" triggers full pipeline via OpenClaw

**VMs:**
| VMID | Name | IP | Status |
|------|------|----|--------|
| 108 | kryptvakt-dev | .74 DHCP (target .108) | Ubuntu installed. SSH not enabled. MITCH-19: paste one-liner in console, then Claude runs g-bootstrap-vm108.sh |
| 109 | ciphertrust-ce | 192.168.50.109 | Shell ready. Mitch has 5GB OVA. MITCH-20: scp to PVE /tmp/, then Claude runs f-import-ciphertrust-ova.sh |

**OpenBao:** secret/kryptvakt/ seeded (ciphertrust, softhsm, luna, payshield — all at version 1, 2026-05-13)

**DNS:** kryptvakt-dev.mitchflix.co.uk → .108, ciphertrust.mitchflix.co.uk → .109 (live)

**Blockers for Mitch (minimal actions):**
1. VM 108: Open Proxmox console → paste the SSH bootstrap one-liner (in MITCH-19)
2. VM 109: `scp CipherTrust*.ova mitch@192.168.50.10:/tmp/ciphertrust.ova`
Then Claude handles the rest for both.

**Next step:** Run `/office-hours` in this Claude Code session on the kryptvakt project.

**How to apply:** Read workspace/kryptvakt/CONTEXT.md for full status. All design/planning starts with /office-hours.
