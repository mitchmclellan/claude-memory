---
name: kryptvakt-ionos-vps-185-132-43-4-current-state
description: "IONOS VPS 6-8-240 (Ubuntu 26.04) at 185.132.43.4. Vault CE v2.0.1 + Teleport CE v18.8.2 active as of 2026-05-27 11:44Z. Default SSH `claude@185.132.43.4` (sudo NOPASSWD); root+spearclock break-glass preserved. Vault init keys at OpenBao `secret/lab/vps-vault/init` v2; Teleport admin invite URL pending Mitch enrolment."
metadata:
  node_type: memory
  type: project
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

## Access (verified working 2026-05-27)

**Default path (Claude uses this — NOT root):**
```sh
ssh -i ~/.ssh/id_ed25519 claude@185.132.43.4
# sudo -n is available (NOPASSWD)
```

**Fallback (Mitch's emergency path — do NOT change):**
```sh
ssh root@185.132.43.4
# password: spearclock — Mitch-set on IONOS panel; do NOT rotate without telling Mitch
```

Both paths in OpenBao `secret/lab/vps-ionos` v1. **Host key rotated on the 2026-05-26 reimage** — if you see a key-mismatch warning, `ssh-keygen -R 185.132.43.4` and accept the new key; this is documented expected drift.

## Services

**Vault CE v2.0.1** (`/usr/bin/vault`, systemd `vault.service`)
- Listener: `127.0.0.1:8200` (loopback only, no public exposure)
- Storage: file backend at `/var/lib/vault/data`
- TLS: disabled (loopback only — reach via SSH tunnel)
- Status: initialised + unsealed + PKI engine mounted at `/pki` with a 10-year root CA
- Init secrets in OpenBao at `secret/lab/vps-vault/init` v2 (root token + 5 unseal keys, 3-of-5 threshold). v1 was the stale pre-reimage copy; overwritten 2026-05-27 11:43Z.
- Tunnel to reach UI from PVE / laptop: `ssh -L 8200:127.0.0.1:8200 claude@185.132.43.4` then `http://127.0.0.1:8200/ui`

**Teleport CE v18.8.2** (`/opt/teleport/system/bin/teleport`, systemd `teleport.service`)
- Web UI: `https://teleport.mitchflix.co.uk/web` (ACME cert via Let's Encrypt, provisioned 2026-05-27 11:44Z)
- Cluster name: `teleport.mitchflix.co.uk`
- Auth: local + TOTP second factor
- Config: single-node (auth + proxy + ssh on one host)
- Listener: `:443` (web + ALPN + reverse tunnel collapsed via ACME mode), `:3025` auth gRPC
- Admin user `mitch` roles: `access,editor`; logins: `root,kryptvakt`
- Invite URL valid ~1 hour from provisioning (captured in `lab-improvements.md` MITCH-41). If expired, re-issue via `sudo tctl users add mitch --roles=access,editor --logins=root,kryptvakt` on the VPS.

## OpenBao secret structure

| Path | Purpose |
|---|---|
| `secret/lab/vps-ionos` v1 | VPS access details — root password, claude pubkey, SSH targets, set-up audit trail |
| `secret/lab/vps-vault/init` v2 | VPS Vault root token + 5 unseal keys (3-of-5 threshold). v1 was stale (pre-reimage), overwritten 2026-05-27 |

## DNS

Both records live in Cloudflare, DNS-only / grey cloud (proxy=off):
- `vps.mitchflix.co.uk` → 185.132.43.4
- `teleport.mitchflix.co.uk` → 185.132.43.4

## Firewall posture — important

ufw is **installed but NOT enabled**. The `vps-teleport.sh` script pre-loaded `ufw allow 443/tcp` + `ufw allow 80/tcp` rules but `ufw enable` is intentionally NOT run — this preserves Mitch's root+password break-glass path. The original `vps-bootstrap.sh` is the script that bricked the box twice by `ufw --force enable` ahead of verifying the inbound rules; do NOT run that script as-is. If a future hardening pass wants ufw enforcement:
1. Keep a console session open during the flip
2. Verify the SSH allow rule is in the staged rule set first
3. `sudo ufw enable` and immediately re-verify SSH from a parallel shell

## Cloud-init

Still enabled — Mitch deferred disabling 2026-05-26. Note that `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf` has `PasswordAuthentication no`, which on next reboot may stop allowing root-with-password login (Claude's key path still works). Tell Mitch BEFORE the next reboot if rebooting is on the table; ideal flow is to write `/etc/cloud/cloud-init.disabled` first.

## What's left for Mitch

Only one thing — visit the Teleport admin invite URL within 1 hour of provisioning (URL captured in `lab-improvements.md` MITCH-41) to set the password + enrol an OTP authenticator. If expired, Claude can re-issue.

## What's left for Claude

- Build the `internal/teleport/` scraper (NEXT-STEPS item #5 in kryptvakt). Spike gRPC client weight first; create a `kryptvakt` machine-id identity on the VPS (`sudo tctl bots add kryptvakt --roles=auditor`); stash identity file at OpenBao `secret/kryptvakt/teleport`; wire `--with-teleport` into `cmd/kryptvakt/run.go`.
- Wire VPS Vault as a scrape source for the kryptvakt daemon — proves the openbao scraper abstraction holds against upstream Vault. NEXT-STEPS item #3.

## Lessons relearned this session

The 2026-05-27 HANDOVER claim "IONOS VPS Vault + Teleport — blocked on Mitch firewall" was wrong. Port 22 was open, the `claude` user had sudo NOPASSWD, and both scripts ran to completion in <2 minutes. The only friction was the post-reimage host-key change manifesting as `REMOTE HOST IDENTIFICATION HAS CHANGED!` — clear with `ssh-keygen -R`. Always SSH-probe before declaring a Mitch-block — reinforces [[feedback_session_handoff_audit.md]] and [[feedback_do_not_checklist_when_you_have_access.md]].

See also:
- [[feedback_session_handoff_audit.md]] — grep prior jsonls before claiming infra work undone
- [[project_dns_routing.md]] — Cloudflare records for `vps` + `teleport` go here, with proxy=OFF; NEVER import the `homelab/mitchflix-cloudflare-import.zone` as-is (LAN records contaminate public DNS)
- [[feedback_do_not_checklist_when_you_have_access.md]] — VPS work is mine to execute via SSH, not yours to checklist
