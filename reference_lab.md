---
name: Lab Reference Pointers
description: Where to find lab documentation, PVE UI, and network device locations
type: reference
originSessionId: eef79927-f965-4861-9fad-9d94cdf98dee
---
- **Proxmox Web UI:** https://192.168.50.10:8006
- **Lab inventory:** /home/mitch/workspace/homelab/lab-infra.md — authoritative IP/VMID/service map. Always grep here first before asking "where is X".
- **Strategy:** /home/mitch/workspace/homelab/strategy.md
- **Issues / improvements backlog:** /home/mitch/workspace/homelab/lab-improvements.md
- **OpenBao:** LXC 107 at 192.168.50.107:8200 (UI + API). KV v2 at secret/. All lab credentials. `secret/kryptvakt/ciphertrust` has the CT admin + ksadmin creds.
- **Synology NAS (DS218j):** https://192.168.50.24:5001
- **Plex / Docker host:** http://192.168.50.139:32400 (Plex), :9000 (Portainer)
- **qBittorrent:** http://192.168.50.53:8080
