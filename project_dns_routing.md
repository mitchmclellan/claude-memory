---
name: dns-and-internal-routing-architecture
description: "How internal *.mitchflix.co.uk routing works — Pi-hole authoritative single-file, laptop hosts-file mirror, Cloudflare public-only"
metadata: 
  node_type: memory
  type: project
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

All `*.mitchflix.co.uk` subdomains are internal-only — Mitch wants **zero internal hostnames in public DNS** (public surface lives on the IONOS VPS only). Reconfirmed 2026-05-26.

## Three layers, one source of truth

1. **Pi-hole (LXC 106, 192.168.50.106)** — **authoritative** for the lab hostname list. Single canonical file at `/etc/pihole/hosts/mitchflix-local.conf` (24 records as of 2026-05-26). `custom.list` no longer carries mitchflix entries — consolidated 2026-05-26 to eliminate drift. Edit-and-SIGHUP cycle: `pkill -HUP pihole-FTL` after changes.

2. **WSL `/etc/hosts`** (MitchIdeaPad, Ubuntu 24.04) — managed block between `# BEGIN mitchflix.co.uk` / `# END mitchflix.co.uk`. **`generateHosts = false` is set in `/etc/wsl.conf`** as of 2026-05-26 so WSL does not regenerate `/etc/hosts` on every boot (this had been silently nuking the block).

3. **Windows hosts file** (`C:\Windows\System32\drivers\etc\hosts`) — same managed block as WSL. Updated via self-elevating PowerShell script at `C:\Users\mitch\Desktop\Update-MitchflixHosts.ps1` (reads block source from `C:\Users\mitch\Desktop\hosts-mitchflix-block.txt`, replaces the managed block, flushes DNS, prints resolution test). Right-click → Run with PowerShell, accept UAC.

## Why this layout

- **Router DHCP DNS is NOT pointing at Pi-hole** (reverted 2026-05-16 — Asus router halves WAN speed when LAN-side DNS is primary; HW NAT/CTF acceleration disables). Other home devices use ISP DNS / 1.1.1.1.
- **Mitch's laptop is the only device that needs internal hostnames**, hence hosts-file mirror on both sides (WSL terminal + Windows Chrome). Phone, Fire TV, etc. accept the breakage.
- **Pi-hole stays running** because (a) it's the human-editable source-of-truth list, (b) PVE host + lab VMs still query it as their DNS resolver, (c) cheap insurance if router DHCP ever points back.

## Cloudflare DNS (public zone)

- Zone: `mitchflix.co.uk` (zone ID `83365d660ef1732dda78337ef4642ce2`)
- Records: apex (GitHub Pages) + email (DMARC/DKIM/SPF). **No subdomain A/CNAME records.**
- Future addition: `kryptvakt-edge.mitchflix.co.uk` → IONOS VPS public IP, A record, **proxy OFF (DNS-only/grey)** — proxy=on would break SSH/Teleport/EJBCA REST. Added once MITCH-36 completes and IP is known.

## `kryptvakt.{com,io,dev}` — stays at GoDaddy

Parked, WHOIS-privacy. **Do not migrate to Cloudflare until launch** — the migration is part of the `kryptvakt/REFERENCE.md` Step 6.5 post-AB activation runbook. Pre-migrating gains nothing; stealth-build rule trumps registrar convenience. Reconfirmed 2026-05-26.

## Adding a new internal hostname (the routine)

1. Edit `/etc/pihole/hosts/mitchflix-local.conf` on LXC 106. Add line. `pkill -HUP pihole-FTL`.
2. Add the same line to the managed block in WSL `/etc/hosts`.
3. Update `C:\Users\mitch\Desktop\hosts-mitchflix-block.txt` (add line). Mitch runs `Update-MitchflixHosts.ps1` to sync Windows hosts + flush DNS.
4. Chrome restart (or `chrome://net-internals/#dns` → Clear host cache) to dump Chrome's own resolver cache.

## Canonical hostname list (2026-05-26)

Traefik-fronted (→ 192.168.50.2): `ai`, `openclaw`, `monitoring`, `prometheus`, `traefik`, `weather`, `nas`, `plex`, `portainer`, `neo4j`, `openbao`, `kryptvakt`

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

## Diagnosing NXDOMAIN on Mitch's laptop

The single highest-likelihood cause is the Windows hosts file managed block being **either missing or commented-out** (every line prefixed with `#`). Pi-hole isn't in the laptop's resolver chain. Check `C:\Windows\System32\drivers\etc\hosts` for live (uncommented) entries between the BEGIN/END markers; if any line starts with `#`, that's the bug. Fix with the staged PS1 script. After updating, also `ipconfig /flushdns` and restart Chrome (or clear `chrome://net-internals/#dns`).

For WSL CLI tools (curl, dig, ssh): check `/etc/hosts` for the same managed block. If missing, append. Also verify `[network] generateHosts = false` is in `/etc/wsl.conf` so the block survives `wsl --shutdown`.

See also: [[reference_laptop_env.md]] for WSL2/NodeSource setup; [[feedback_testing_paths.md]] for "always simulate Mitch's actual access path" rule.
