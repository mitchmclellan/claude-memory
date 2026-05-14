---
name: Lab Redesign May 2026 — Kryptvakt Focus
description: Full homelab teardown/rebuild to support Kryptvakt. Day-trading decommissioned. Two new VMs planned. RAM reallocation.
type: project
originSessionId: 99e8964b-901d-4108-8600-c5cf05b678fb
---
Lab redesign initiated 2026-05-13. Everything refocused on Kryptvakt (HSM monitoring).

**Day-trading engine:** DECOMMISSIONED. Data preserved at /home/mitch/day-trading/ on ArcAiVM. Docker image to be removed.

**RAM reallocation (pending, scripts written):**
| VM/LXC | Before | After | MB |
|--------|--------|-------|-----|
| VM 102 traefik | 2048 MB | 512 MB | -1536 |
| VM 103 ArcAiVM | 16384 MB | 4096 MB | -12288 |
| LXC 105 monitoring | 2048 MB | 1536 MB | -512 |
| LXC 106 pihole | 512 MB | 512 MB | 0 |
| LXC 107 openbao | 512 MB | 512 MB | 0 |
| VM 108 kryptvakt-dev | — | 4096 MB | +4096 |
| VM 109 ciphertrust-ce | — | 8192 MB | +8192 |
| **Total VM/LXC** | 21,008 MB | 19,456 MB | **-1,552** |

**Prometheus:** trading_engine scrape target removed (script f). TradingKillSwitch alert rule to be removed manually.

**LXC 104 status:** Already deleted (IMPROVE-3, 2026-05-09). Orphan LVM disk also gone. No action needed.

**Hardware risks flagged:**
- sda: 1 reallocated sector (SanDisk in ai-storage mirror) — plan replacement
- sdb: 22,222h WD HDD (backup storage) — aging, plan replacement

**How to apply:** Check workspace/kryptvakt/CONTEXT.md Lab Redesign Status table for which scripts have been run. Don't assume any script has run unless confirmed.
