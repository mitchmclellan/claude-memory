---
name: dns-and-internal-routing-architecture
description: "How internal *.mitchflix.co.uk routing works — Pi-hole authoritative + NRPT on Windows + static resolv.conf on WSL. Hosts-file mirror retired 2026-05-27."
metadata:
  node_type: memory
  type: project
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

All `*.mitchflix.co.uk` subdomains are internal-only — Mitch wants **zero internal hostnames in public DNS** (public surface lives on the IONOS VPS only). Reconfirmed 2026-05-26.

## Two layers, one source of truth (post-2026-05-27)

1. **Pi-hole (LXC 106, 192.168.50.106)** — **authoritative** for the lab hostname list AND recursive forwarder for everything else. Single canonical file at `/etc/pihole/hosts/mitchflix-local.conf` (23 lines, 25 names as of 2026-05-27 — `dns`/`pihole` and `winsrv-adcs`/`adcs` share lines). Edit-and-SIGHUP cycle: `pkill -HUP pihole-FTL` after changes.

2. **Mitch's laptop (MitchIdeaPad, Win11 + WSL2 Ubuntu 24.04)** — both halves point at Pi-hole, *without* touching the router DHCP DNS chain:
   - **Windows side (Chrome, browsers):** NRPT rule `*.mitchflix.co.uk → 192.168.50.106`. Set 2026-05-27 via `Add-DnsClientNrptRule -Namespace ".mitchflix.co.uk" -NameServers "192.168.50.106"`. Verify: `Get-DnsClientNrptRule | Where-Object Namespace -like "*mitchflix*"`. Survives reboots, network changes, Wi-Fi reconnects.
   - **WSL side (terminal, curl, dig):** static `/etc/resolv.conf` pointing at `192.168.50.106` primary + `1.1.1.1` fallback. `[network] generateResolvConf = false` in `/etc/wsl.conf` prevents WSL from auto-regenerating it. Independent of Windows DNS — goes straight to Pi-hole on the LAN.

The hosts-file mirror workflow (managed block in Windows + WSL hosts files, kept in sync via `Update-MitchflixHosts.ps1`) was **retired 2026-05-27**. Reason: every new hostname required editing three files + running an elevated PowerShell, and the Windows block kept silently getting #-commented out by something (Windows Update? security tool? hand-edits). NRPT routes the namespace transparently; one Pi-hole edit ripples everywhere.

## Why NRPT and not "just set the laptop's DNS to Pi-hole"

The naive answer is "configure Windows adapter DNS = 192.168.50.106". That works at home but **fails on every other network** — coffee shops, hotels, conference Wi-Fi. NRPT is namespace-specific: `*.mitchflix.co.uk` goes to Pi-hole, everything else uses whatever DHCP DNS the current network provides. At home you get internal + public resolution; on the road internal lookups fail fast (~2s timeout, route unreachable) and public works perfectly.

## Why not router-DHCP-DNS = Pi-hole

Tried 2026-05-16, reverted same day. **Asus router halves WAN speed when LAN-side DNS is primary** because HW NAT / CTF acceleration disables. Confirmed reproduction. NRPT-on-client avoids this — only Mitch's laptop talks to Pi-hole; the router still hands out 1.1.1.1 or ISP DNS to other devices.

## WSL belt-and-braces

The WSL `/etc/hosts` managed block (`# BEGIN mitchflix.co.uk ... # END mitchflix.co.uk`) is **kept** as a fallback. If Pi-hole dies, WSL CLI tools (`curl`, `ssh`, `dig`) still resolve internal hosts via the static hosts file. Zero cost to keep; non-zero value when Pi-hole is in maintenance. Update when adding a new hostname — same one-liner as Pi-hole.

The **Windows** hosts file managed block was stripped 2026-05-27 (backup at `C:\Windows\System32\drivers\etc\hosts.bak-nrpt-cleanup-*`). NRPT supersedes it. Chrome and Windows apps now resolve via DNS only.

## Cloudflare DNS (public zone)

- Zone: `mitchflix.co.uk` (zone ID `83365d660ef1732dda78337ef4642ce2`)
- Records: apex (GitHub Pages) + email (DMARC/DKIM/SPF) + `vps.mitchflix.co.uk` → 185.132.43.4 (DNS-only/grey) + `teleport.mitchflix.co.uk` → 185.132.43.4 (DNS-only/grey)
- **No other subdomain A/CNAME records.** Internal hostnames never go public.

## `kryptvakt.{com,io,dev}` — stays at GoDaddy

Parked, WHOIS-privacy. **Do not migrate to Cloudflare until launch** — the migration is part of the `kryptvakt/REFERENCE.md` Step 6.5 post-AB activation runbook. Pre-migrating gains nothing; stealth-build rule trumps registrar convenience. Reconfirmed 2026-05-26.

## Adding a new internal hostname (the new routine)

1. SSH to LXC 106: edit `/etc/pihole/hosts/mitchflix-local.conf`, append the line.
2. `sudo pkill -HUP pihole-FTL` on LXC 106.
3. (Optional belt-and-braces) Append the same line to WSL `/etc/hosts` managed block on MitchIdeaPad.

That's it. Chrome resolves on next page load (Windows DNS Client cache TTL'd by Pi-hole). WSL resolves on next query.

**No more editing three files. No more PowerShell scripts. No more sync drift.**

## Canonical hostname list (2026-05-27)

Traefik-fronted (→ 192.168.50.2): `ai`, `kryptvakt`, `monitoring`, `nas`, `neo4j`, `openbao`, `openclaw`, `plex`, `portainer`, `prometheus`, `traefik`, `weather`

Host-direct:
- `pve.mitchflix.co.uk` → 192.168.50.10
- `monitoring-host.mitchflix.co.uk` → 192.168.50.105 (LXC 105 direct, not Grafana — that's `monitoring`)
- `pihole.mitchflix.co.uk` / `dns.mitchflix.co.uk` → 192.168.50.106
- `arcaivm.mitchflix.co.uk` → 192.168.50.196
- `nas-host.mitchflix.co.uk` → 192.168.50.24

Kryptvakt scrape sources (direct):
- `kryptvakt-dev.mitchflix.co.uk` → 192.168.50.108
- `ciphertrust.mitchflix.co.uk` → 192.168.50.109

Tier 1 PKI lab (LXC 110):
- `pki-lab.mitchflix.co.uk` / `vault.mitchflix.co.uk` / `ejbca.mitchflix.co.uk` → 192.168.50.110

Windows Server AD CS (VM 111):
- `winsrv-adcs.mitchflix.co.uk` / `adcs.mitchflix.co.uk` → 192.168.50.90

## Diagnosing NXDOMAIN on Mitch's laptop (post-NRPT)

1. **Confirm NRPT rule still present:** `Get-DnsClientNrptRule | Where-Object Namespace -like "*mitchflix*"`. If empty, re-add with the `Add-DnsClientNrptRule ...` one-liner above.
2. **Confirm Pi-hole reachable:** `Test-NetConnection 192.168.50.106 -Port 53`. If unreachable, check Pi-hole service on LXC 106.
3. **Confirm Pi-hole has the record:** `ssh mitch@192.168.50.10 'sudo pct exec 106 -- grep <hostname> /etc/pihole/hosts/mitchflix-local.conf'`. If missing, add it + SIGHUP pihole-FTL.
4. **Chrome resolver cache:** `chrome://net-internals/#dns` → Clear host cache. (Chrome caches independently of Windows DNS Client.)
5. **Windows DNS cache:** `ipconfig /flushdns` from any PowerShell.

For WSL: check `/etc/resolv.conf` is still `192.168.50.106` primary. If WSL regenerated it after a `wsl --shutdown`, confirm `[network] generateResolvConf = false` is in `/etc/wsl.conf`.

See also: [[reference_laptop_env.md]] for WSL2/NodeSource setup; [[feedback_testing_paths.md]] for "always simulate Mitch's actual access path" rule.
