---
name: Test the exact user path, not an equivalent one
description: Never report a fix as working without testing from the same network/DNS path the user will actually use
type: feedback
originSessionId: db930de3-eba3-4c1b-bb9d-4430c8a99cc0
---
Do not test from the PVE host and report it as confirming user-facing access. PVE host uses Pi-hole for DNS; Mitch's LAN devices used 1.1.1.1. These are completely different paths:
- PVE host: Pi-hole DNS → resolves *.mitchflix.co.uk → Traefik (192.168.50.2) → works
- LAN device with 1.1.1.1 DNS: Cloudflare edge → tunnel → cloudflared → broken remote config → 503

Twice reported "fixed" when only the Pi-hole/Traefik path was tested, not the Cloudflare tunnel path Mitch's devices were actually using.

**Why:** Testing the wrong path gives false confidence and wastes Mitch's time verifying.

**How to apply:** Before saying "fixed":
1. Identify which DNS path the user's device uses (check their router DNS setting)
2. Simulate that exact path — e.g. `curl --dns-servers 1.1.1.1 https://sub.mitchflix.co.uk` or `curl --connect-to sub.mitchflix.co.uk:443:[cloudflare-ip]:443 https://sub.mitchflix.co.uk`
3. Only report working if that specific path returns the right response
