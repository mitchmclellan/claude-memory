---
name: lab-reference-pointers
description: "Where to find lab documentation, PVE UI, and network device locations"
metadata: 
  node_type: memory
  type: reference
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

- **Proxmox Web UI:** https://192.168.50.10:8006
- **Lab inventory:** `/home/mitch/workshop/homelab/lab-infra.md` — authoritative IP/VMID/service map. Always grep here first before asking "where is X". (Path changed from `~/workspace/` after the laptop migration — see [[project_laptop_migration_2026_06_01]]; PVE-host paths under `~/workspace/` are unchanged.)
- **Strategy:** `/home/mitch/workshop/homelab/strategy.md`
- **Issues / improvements backlog:** `/home/mitch/workshop/homelab/lab-improvements.md`
- **OpenBao:** LXC 107 at 192.168.50.107:8200 (UI + API). KV v2 at `secret/`. All lab credentials. `secret/kryptvakt/ciphertrust` has the CT admin + ksadmin creds.
- **Synology NAS (DS218j):** https://192.168.50.24:5001
- **Plex / Docker host:** http://192.168.50.139:32400 (Plex), :9000 (Portainer)
- **qBittorrent:** http://192.168.50.53:8080
- **Traefik:** routes via `*.mitchflix.co.uk → 192.168.50.2` for monitoring, prometheus, openbao, kryptvakt, ai, neo4j, plex, portainer, traefik dashboard, nas, weather
