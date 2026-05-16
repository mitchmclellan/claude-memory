---
name: DNS and Internal Routing Architecture
description: How internal *.mitchflix.co.uk routing works — current state (Pi-hole still up, but NOT router-wide); hosts-file pattern on Mitch's devices
type: project
originSessionId: 351b90a3-9068-4f06-9fc0-e0a7e6205ed8
---
All *.mitchflix.co.uk subdomains are internal-only — Mitch wants **zero internal hostnames in public DNS** (cloud VPS handles anything public-facing). Reconfirmed 2026-05-16.

## Current state (2026-05-16, after Asus speed-issue rollback)

- **Router (192.168.50.1, ASUS)**: DHCP DNS reverted to ISP/1.1.1.1 — NOT pointing at Pi-hole anymore. Setting router-wide DNS to LAN-side IP was halving home internet speed (almost certainly Asus router disabling HW NAT acceleration when DNS goes to a LAN address — well-documented quirk). Wife escalation.
- **Pi-hole (192.168.50.106, LXC 106)**: Still running, healthy, sub-millisecond responses, but no longer in the router DNS chain. Local DNS records preserved (12 entries: mitchflix.co.uk + 10 Traefik subdomains + kryptvakt). Available as a manual DNS target if needed.
- **Traefik (192.168.50.2, VM 102)**: Unchanged. Routes by Host header → backend services. Config at `/home/mitch/traefik/config/ai.yml`. TLS via Cloudflare DNS challenge.
- **Mitch's laptop**: Windows hosts file + WSL `/etc/hosts` both have the mitchflix.co.uk block (managed between `# BEGIN mitchflix.co.uk` / `# END mitchflix.co.uk` markers). Chrome + WSL processes resolve correctly.
- **Mitch's phone**: No internal resolution — accepts breakage until needed; not worth fighting Android Private DNS for the few times a year he hits internal services from mobile.

## Pi-hole was diagnosed innocent on 2026-05-16

- LXC 106: 2 cores / 512MB / 7.8GB, using ~15% RAM, load 0.34 — not a resource problem
- pihole-FTL running 6+ days, 25MB RSS
- Upstream: 1.1.1.1 + 1.0.0.1 (Cloudflare)
- Query latency: **0ms cached, faster than direct 1.1.1.1 (18ms)** — DNS itself is fast
- Gravity: 4.6MB — normal
- Conclusion: speed regression was the **Asus router**, not Pi-hole. Routing all home traffic through a LAN-side DNS server disables HW NAT/CTF acceleration on many Asus firmwares. Fix would be router-side (enable NAT Acceleration, possibly toggle AiProtection), not Pi-hole-side. **Not pursuing** the router fix because the hosts-file approach is good enough for Mitch's needs.

## Mitch's `*.mitchflix.co.uk` subdomain list (canonical, 2026-05-16)

All → `192.168.50.2` (Traefik) unless noted:

- `mitchflix.co.uk` (apex)
- `ai.mitchflix.co.uk` (OpenWebUI)
- `openclaw.mitchflix.co.uk`
- `monitoring.mitchflix.co.uk` (Grafana)
- `prometheus.mitchflix.co.uk`
- `weather.mitchflix.co.uk`
- `pve.mitchflix.co.uk`
- `traefik.mitchflix.co.uk`
- `neo4j.mitchflix.co.uk`
- `openbao.mitchflix.co.uk`
- `kryptvakt.mitchflix.co.uk`
- `ciphertrust.mitchflix.co.uk` → `192.168.50.109` (direct, ksadmin UI)
- `kryptvakt-dev.mitchflix.co.uk` → `192.168.50.108` (direct, dev VM)

## When new subdomains land

Updating Pi-hole *and* both hosts files (WSL + Windows). Pi-hole records are still authoritative for the lab list; the hosts files are Mitch-laptop-only mirrors. The block on each side is delimited by `# BEGIN mitchflix.co.uk` / `# END mitchflix.co.uk` for clean idempotent updates.

## Cloudflare DNS

Only has records for `mitchflix.co.uk` apex → GitHub Pages + email (DMARC/DKIM/SPF). No subdomain A/CNAME records. Zone ID: `83365d660ef1732dda78337ef4642ce2`. **Hard rule per Mitch (2026-05-16):** internal hostnames stay out of public DNS until/unless there's a specific resource we want to show off publicly.

## Android Private DNS (legacy)

DoT proxy stunnel4 on Pi-hole LXC 106 :853 — still configured (cert auto-renews via certbot). Useful if Mitch ever wants his phone back on the lab's DNS without router involvement. Not in active use.

## How to apply

When something on Mitch's laptop can't resolve a mitchflix.co.uk hostname: check the block in `/etc/hosts` (WSL) and `C:\Windows\System32\drivers\etc\hosts` (Windows) — both should mirror Pi-hole's record set. If the subdomain isn't in either, add it to both AND to Pi-hole (`PUT http://192.168.50.106/api/config/dns/hosts/{url-encoded entry}` with Pi-hole API auth).

For Mitch's phone or other home devices: accept the breakage. Don't suggest re-enabling router-wide DNS — it'll re-trigger the Asus speed issue and family will be cross.
