---
name: Homelab Current State
description: Lab state 2026-05-13 — fully focused on Kryptvakt, dead projects cleared, gstack live, VMs 108+109 shell-created
type: project
originSessionId: 99e8964b-901d-4108-8600-c5cf05b678fb
---
**VM/CT inventory (as of 2026-05-13):**
- VM 102 `traefik` (192.168.50.2, 512MB RAM) — Traefik v3.0. Hardened. node_exporter :9100.
- VM 103 `ArcAiVM` (192.168.50.196, 4GB RAM, 3GB ballooned) — ACTIVE. Ollama+OpenWebUI+Neo4j+arcai-bot. OpenClaw RETIRED 2026-05-26 (user systemd unit deleted; npm pkg + ~/.openclaw config dir retained for archive only). 4GB is too small for 7-8B Ollama models (OOM on load); GPU has 7.8GB free VRAM but system RAM is the bottleneck. arcai-bot remains on Anthropic Sonnet — Ollama routing is documented-as-unfunded until RAM expansion.
- VM 108 `kryptvakt-dev` (192.168.50.108, 4GB RAM) — Shell created. Ubuntu install needed via PVE console.
- VM 109 `ciphertrust-ce` (192.168.50.109, 8GB RAM) — Shell created (80GB ai-storage). Thales download needed.
- LXC 105 `monitoring` (192.168.50.105, 1.5GB RAM) — Prometheus 5/5 targets. Kryptvakt targets commented-in.
- LXC 106 `pihole` (192.168.50.106) — Pi-hole v6.4.2. DNS: kryptvakt-dev+ciphertrust added.
- LXC 107 `openbao` (192.168.50.107) — OpenBao 2.5.3. KV v2. Central secret store.

**gstack (PVE host):**
- Installed at `~/.claude/skills/gstack/` — 47 skills, browse binary (Chromium)
- Bun 1.3.14 at `~/.bun/bin/bun`
- Skills available: /office-hours, /autoplan, /review, /cso, /ship, /investigate, etc.

**OpenClaw on ArcAiVM:** RETIRED 2026-05-26 (per audit: bot bypassed it entirely, main agent dormant 16 days, GPU stranded). Service unit deleted, autostart removed. Plugin-skills + gateway no longer reachable from ArcAiVM. The npm package + `~/.openclaw/` config dir kept on disk as archive; reactivation would require fresh systemd unit + Ollama model that fits in 4GB.

**ArcAiVM services (post-retirement):**
- Ollama (GPU, localhost:11434) — installed, llama3.1:8b + qwen2.5-coder:7b downloaded, currently UNUSED. Bot does not dispatch through it (RAM-bound; see ArcAiVM line above).
- OpenWebUI (:3000 LAN), Neo4j (:7474/:7687 LAN) — running but not actively driven.
- arcai-bot Telegram relay — v3, direct Anthropic Sonnet (claude-sonnet-4-6).
- Dead: day-trading, weather-app, absorb-the-borg (all archived ~/\*.archived)

**Prometheus state:** 5 targets (pve-host, arcaivm, traefik, monitoring, nvidia-dcgm). No trading targets. TradingKillSwitch rule removed.

**Pi-hole custom DNS:**
All mitchflix.co.uk → 192.168.50.2 (Traefik) PLUS:
- kryptvakt-dev.mitchflix.co.uk → 192.168.50.108
- ciphertrust.mitchflix.co.uk → 192.168.50.109
Custom file: /etc/pihole/hosts/custom.list on LXC 106

**Cloudflare Tunnel:** Removed 2026-05-10 (MITCH-18+14). Internal-only routing.

**Quality gate:** `bash /home/mitch/workspace/lab-tests/test-lab.sh`
