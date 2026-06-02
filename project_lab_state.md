---
name: homelab-current-state
description: "PVE lab inventory as of 2026-06-01 — 5 VMs + 4 LXCs running, all healthy. Memory pressure is real (20G/23G used + 3.7G swap)."
metadata: 
  node_type: memory
  type: project
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

## PVE host (192.168.50.10)
- pve-manager **8.4.19** (the "upgrading to 8.4.19" claim in lab-infra.md is closed — current and running)
- Kernel 6.8.12-23-pve
- Uptime 23 days at probe time
- Memory: **20G/23G used + 3.7G swap used — tight**. Headroom for new guests is ~3GB max
- USB external (SABRENT/Hitachi 232G) hosts `zfs1` pool; shows as `sdd` (SATA) because of ATA pass-through — see [[reference_pve_usb_external_hitachi]]

## VMs (all running 2026-06-01)
| VMID | Name | IP | RAM (balloon) | Cores | Role |
|------|------|----|---------------|-------|------|
| 102 | traefik | 192.168.50.2 | 1536 (768) | 1 | Traefik v3.0 Docker. Routes everything *.mitchflix.co.uk to LAN services. node_exporter :9100. |
| 103 | ArcAiVM | 192.168.50.196 | 4096 (3072) | 8 | **Ollama IS running** (PID 1992, `/usr/bin/ollama serve`) — llama3.1:8b + qwen2.5-coder:7b on disk, currently 0 loaded (idle). GPU passthrough OK (RTX 3050). OpenWebUI :3000, Neo4j :7474/:7687, arcai-bot v3 (Sonnet direct). No OpenClaw (retired 2026-05-26). |
| 108 | kryptvakt-dev | 192.168.50.108 | 4096 (4096) | 4 | Kryptvakt daemon. See [[project_kryptvakt]]. |
| 109 | ciphertrust-ce | 192.168.50.109 | 8192 (6144) | 4 | CipherTrust Manager CE 2.11.1. HTTP 200 on root. `onboot=0`. |
| 111 | winsrv-adcs | 192.168.50.90 (DHCP) | 4096 (2048) | 4 | Windows Server 2022 + AD CS Standalone Root CA `kryptvakt-lab-ca`. WinRM HTTPS:5986 + mTLS live since 2026-05-28. |

## LXCs (all running 2026-06-01)
| CTID | Name | IP | RAM | Role |
|------|------|----|-----|------|
| 105 | monitoring | 192.168.50.105 | 1024 | Prometheus + Grafana + Loki 3.7.2 + Alertmanager + snmp_exporter. All 12+ scrape targets up. Loki tuned 2026-05-30 (warn level, 30d retention, /var/lib/loki). |
| 106 | pihole | 192.168.50.106 | 512 | Pi-hole v6.4.2. Local DNS canonical file: `/etc/pihole/hosts/mitchflix-local.conf`. |
| 107 | openbao | 192.168.50.107 | 512 | OpenBao 2.5.3 unsealed. Auto-unseal drop-in fixed 2026-05-28 ([[project_kryptvakt_loki_observability]] is unrelated; auto-unseal lives in `/etc/systemd/system/openbao.service.d/auto-unseal.conf`). |
| 110 | pki-lab | 192.168.50.110 | 1024 | **Docker host for kryptvakt's PKI lab.** Runs `vault` (hashicorp/vault dev mode) + `ejbca` (keyfactor/ejbca-ce). Both serve canonical kryptvakt scrape sources. Not in any pre-2026-05-25 docs. |

## VPS
- 185.132.43.4 (IONOS, Ubuntu 26.04) — see [[project_kryptvakt_vps_ionos]]. NOT firewall-blocked (memory was wrong about that; SSH probe-first).

## Prometheus targets (live)
- 12+ targets all up at last check; includes kryptvakt exporters at 192.168.50.108:9110 (CT), :9111 (Luna/SoftHSM2), :9112 (OpenBao PKI), plus node_exporter on every host, snmp_exporter, etc.

## Pi-hole local DNS (canonical file `/etc/pihole/hosts/mitchflix-local.conf`)
- All Traefik-fronted services → 192.168.50.2 (kryptvakt, openbao, monitoring, prometheus, traefik, neo4j, weather, nas, plex, portainer, ai, openclaw)
- Host-direct: pve→.10, pihole→.106, arcaivm→.196, nas-host→.24, monitoring-host→.105
- kryptvakt sources: kryptvakt-dev→.108, ciphertrust→.109
- LXC 110 PKI: pki-lab/vault/ejbca→.110
- VM 111 Windows: winsrv-adcs/adcs→.90

## Quality gate
- `bash /home/mitch/workshop/lab-tests/test-lab.sh` (NOT `~/workspace/`)
