---
name: DNS and Internal Routing Architecture
description: How internal *.mitchflix.co.uk routing works — Pi-hole, Traefik, Android Private DNS blocker
type: project
originSessionId: 03413022-6e33-429c-9ca6-c74a8d3a913e
---
All *.mitchflix.co.uk subdomains are internal-only (no public DNS records). Stack:

- **Router (192.168.50.1, ASUS)**: DHCP DNS1=192.168.50.106 (Pi-hole), DNS2=1.1.1.1 — set by Mitch
- **Pi-hole (192.168.50.106)**: Local DNS overrides for all mitchflix.co.uk subdomains → 192.168.50.2 (Traefik). Records in `/etc/pihole/custom.list` (also referenced as `/etc/pihole/hosts/mitchflix-local.conf`).
- **Traefik (192.168.50.2)**: Routes by Host header → backend services. Config at `/home/mitch/traefik/config/ai.yml` on VM 102. TLS via Cloudflare DNS challenge (CF_DNS_API_TOKEN in `/home/mitch/traefik/.env`).

**Cloudflare DNS**: Only has records for mitchflix.co.uk root → GitHub Pages and email (DMARC/DKIM/SPF). No public A/CNAME records for any subdomain. Zone ID: `83365d660ef1732dda78337ef4642ce2`.

**Android Private DNS**: Android 10+ ignores DHCP DNS if Private DNS is set. **DoT proxy is live on LXC 106** — stunnel4 on :853, valid Let's Encrypt cert for `dns.mitchflix.co.uk`. Android setting: Settings → Network & internet → Private DNS → Hostname → `dns.mitchflix.co.uk`. Cert auto-renews via certbot timer + post-hook restarts stunnel4.

**Why:** Cloudflare tunnel was removed 2026-05-10. Pi-hole + Traefik replaced it for internal access.

**How to apply:** When any device fails to resolve mitchflix.co.uk subdomains despite being on LAN — first check if Pi-hole logs show queries from that device's IP. If not, the device has Private DNS override. Do NOT assume it's a Pi-hole or Traefik problem.
