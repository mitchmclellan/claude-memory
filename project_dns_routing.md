---
name: dns-and-internal-routing-architecture
description: How internal *.mitchflix.co.uk routing works — Pi-hole authoritative for the lab. Laptop-side resolution config was retired with the WSL→native Ubuntu migration (2026-06-01); replacement on native Ubuntu is OPEN.
metadata: 
  node_type: memory
  type: project
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

All `*.mitchflix.co.uk` subdomains are internal-only — Mitch wants **zero internal hostnames in public DNS** (public surface lives on the IONOS VPS only).

## Authoritative source: Pi-hole on LXC 106

**Pi-hole (LXC 106, 192.168.50.106)** — authoritative for the lab hostname list AND recursive forwarder for everything else. Single canonical file at `/etc/pihole/hosts/mitchflix-local.conf`. Edit-and-SIGHUP cycle: `sudo pkill -HUP pihole-FTL` after changes.

PVE host itself uses Pi-hole as DNS (verified by [[feedback_testing_paths]]).

## Laptop side — wired 2026-06-02 via systemd-resolved drop-in

On native Ubuntu 26.04 the laptop uses **systemd-resolved**. Default DHCP-provided DNS is `192.168.50.1` (the Asus router), which does NOT forward `*.mitchflix.co.uk` queries to Pi-hole (because making Pi-hole the router's primary DNS halves WAN speed; tested + reverted 2026-05-16).

**Solution (installed 2026-06-02):** per-domain DNS routing rule in systemd-resolved.

```
# /etc/systemd/resolved.conf.d/mitchflix.conf
[Resolve]
DNS=192.168.50.106
Domains=~mitchflix.co.uk
```

The leading `~` makes `mitchflix.co.uk` a **routing-only domain** (not added to the search list). Only queries matching `*.mitchflix.co.uk` are sent to Pi-hole. Everything else continues using whatever DHCP/router DNS the current network provides. On a non-home network, `*.mitchflix.co.uk` lookups fail fast (~2s timeout) without breaking public name resolution.

Apply: `sudo systemctl restart systemd-resolved`. Verify: `resolvectl status` shows a `Global` block with `DNS Servers: 192.168.50.106` + `DNS Domain: ~mitchflix.co.uk`.

**Bootstrap independence:** if Pi-hole ever moves to a new LAN IP, update both the Cloudflare `dns.mitchflix.co.uk` A record AND this drop-in's `DNS=`. The Cloudflare record is the canonical bootstrap (anyone setting up a new client can resolve it externally and learn the LAN IP). systemd-resolved itself wants an IP, not a hostname, so the laptop config can't auto-track Cloudflare changes.

**What broke the post-migration setup originally (2026-06-01 → 2026-06-02 morning):** Nothing dramatic — the WSL-era setup (NRPT on Windows + static `/etc/resolv.conf` in WSL) was wiped with Windows, and no native-Ubuntu equivalent was created during the migration. Chrome carried internal hostnames in its DNS cache for a few hours after first run, then the cache expired and `DNS_PROBE_POSSIBLE_FINISHED_NXDOMAIN` started showing up. The freshness audit on 2026-06-01 evening did surface this (`getent hosts kryptvakt.mitchflix.co.uk` was already returning NOTFOUND at audit time); Chrome was just still hitting cache. Drop-in now closes the gap.

**Chrome DNS cache:** Chrome caches DNS independently of systemd-resolved. After applying this drop-in or any DNS change, clear via `chrome://net-internals/#dns` → "Clear host cache", or just close + reopen the affected tab.

## Why not router-DHCP-DNS = Pi-hole

Tried 2026-05-16, reverted same day. **Asus router halves WAN speed when LAN-side DNS is primary** because HW NAT / CTF acceleration disables. Confirmed reproduction. Solution has to live on the client, not the router.

## Cloudflare DNS (public zone)

- Zone: `mitchflix.co.uk` (zone ID `83365d660ef1732dda78337ef4642ce2`)
- Records: apex (GitHub Pages) + email (DMARC/DKIM/SPF) + `vps.mitchflix.co.uk` → 185.132.43.4 (DNS-only/grey) + `teleport.mitchflix.co.uk` → 185.132.43.4 (DNS-only/grey)
- **No other subdomain A/CNAME records.** Internal hostnames never go public.

## `kryptvakt.{com,io,dev}` — stays at GoDaddy

Parked, WHOIS-privacy. **Do not migrate to Cloudflare until launch** — the migration is part of the `kryptvakt/REFERENCE.md` Step 6.5 post-AB activation runbook.

## Adding a new internal hostname

1. SSH `mitch@192.168.50.10`, then `sudo pct exec 106 -- bash -c "echo '<ip> <fqdn>' >> /etc/pihole/hosts/mitchflix-local.conf"`.
2. `sudo pct exec 106 -- pkill -HUP pihole-FTL`.
3. PVE host + clients pointed at Pi-hole resolve immediately. Laptop still needs the replacement listed above to be wired.

## Canonical hostname list (2026-06-01, ground truth from Pi-hole)

Traefik-fronted (→ 192.168.50.2): `ai`, `kryptvakt`, `monitoring`, `nas`, `neo4j`, `openbao`, `openclaw`, `plex`, `portainer`, `prometheus`, `traefik`, `weather`

Host-direct:
- `pve.mitchflix.co.uk` → 192.168.50.10
- `monitoring-host.mitchflix.co.uk` → 192.168.50.105
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

See also: [[reference_laptop_env]] for native-Ubuntu post-migration toolchain; [[feedback_testing_paths]] for "always simulate Mitch's actual access path" rule.
