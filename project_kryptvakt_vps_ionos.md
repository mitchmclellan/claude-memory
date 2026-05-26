---
name: kryptvakt-ionos-vps-185-132-43-4-current-post-reimage-state-2026-05-26-18-53z
description: "IONOS VPS 6-8-240 (Ubuntu 26.04) at 185.132.43.4. Reimaged 2026-05-26 evening after a prior session bricked it via aggressive ufw + key-only hardening. THIS session set up minimal access — claude user with sudo NOPASSWD as the default SSH target; root + spearclock password preserved as Mitch's fallback. No bootstrap, no Vault, no Teleport, no ufw run yet. Full state in OpenBao `secret/lab/vps-ionos` v1."
metadata: 
  node_type: memory
  type: project
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

## Access (current, verified working from PVE)

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

Both paths verified working 2026-05-26 18:53Z (this session). Root password and key both stored in `secret/lab/vps-ionos` v1.

## OpenBao secret structure (`secret/lab/vps-ionos` v1)

Source of truth for VPS access. If anything here drifts from reality, the OpenBao record wins until proven otherwise. 22 fields including: `host_ip`, `root_password`, `admin_user=claude`, `admin_user_pubkey_label=claude@pve`, `default_ssh_target=claude@185.132.43.4`, `fallback_ssh_target=root@185.132.43.4`, `sshd_config_modified=false`, `ufw_state=inactive`, `cloud_init_disabled=false`, `bootstrap_run=false`, `set_up_at=2026-05-26T18:53Z`, `set_up_session_jsonl=<path>`.

## What is and isn't done

| State | Status |
|---|---|
| sshd active on :22 | ✅ |
| claude user with sudo NOPASSWD | ✅ |
| Claude's pubkey in `/home/claude/.ssh/authorized_keys` | ✅ |
| Claude's pubkey in `/root/.ssh/authorized_keys` | ✅ |
| Root password `spearclock` working | ✅ |
| ufw | ❌ inactive (deliberate — see "Lockout protection" below) |
| cloud-init disabled | ❌ not done (Mitch deferred 2026-05-26) — note that `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf` has `PasswordAuthentication no`. Effective sshd config after a reboot may stop allowing root-with-password login; Claude's key auth will still work. Tell Mitch BEFORE the next reboot. |
| `vps-bootstrap.sh` | ❌ not run (last run bricked us) |
| `vps-vault.sh` / Vault CE | ❌ not installed |
| `vps-teleport.sh` / Teleport CE | ❌ not installed |
| Cloudflare A records (`vps`, `teleport`) | ❌ Mitch's task; needed before Teleport runs |

## Lockout protection — what actually works

**ufw is NOT what protects against lockout.** ufw rules can themselves cause lockout if mis-set (this is what bricked the box on 2026-05-25). The real layers:

1. **IONOS web KVM console** (Mitch's account) — out-of-band, works even if sshd is dead. The actual safety net.
2. **Dual auth paths** — key login as claude AND password login as root. If either path dies, the other still works. **Do not collapse these to one path.**
3. **No automatic hardening scripts that lock the door on exit.** If a script disables `PasswordAuthentication`, it MUST first verify (in a parallel shell, before exiting the deploying shell) that key login works.
4. **Cloud-init is unstable in default Ubuntu cloud images.** Will re-run on some conditions; can rewrite sshd_config.d. When the next intentional reboot happens, write `/etc/cloud/cloud-init.disabled` first or accept that root-password login may stop working (Claude's key path remains).

## When running `vps-bootstrap.sh` later

Use a modified version that:
- Does NOT enable ufw without first `ufw allow 22/tcp` AND running a verification SSH from a parallel shell that confirms 22 still answers
- Does NOT set `PasswordAuthentication no` until Mitch's key is verified in /root/.ssh/authorized_keys AND he's confirmed in-session he wants the password path removed
- Does NOT remove `/root/.ssh/authorized_keys` entries (Mitch may add his own laptop key there too)

The current bootstrap script at `homelab/scripts/vps-bootstrap.sh` is the version that bricked us. Don't run it as-is.

## How to apply (for future sessions)

- **Default to `ssh claude@185.132.43.4` for any VPS work**, not `ssh root@`. Root path is Mitch's break-glass, not the daily driver.
- **Before any infra change on this VPS:** read this memory + `bao kv get secret/lab/vps-ionos` for current state. If they disagree, OpenBao wins; update this memory.
- **Before claiming the VPS is unreachable / broken / undone:** test it with `ssh claude@185.132.43.4 hostname`. If that works, the VPS is fine. Don't tell Mitch otherwise without trying first.
- **If a deploy script DOES brick the box:** apologise plainly. Mitch reimages. We restart from this memory's "current state" baseline.

See also:
- [[feedback_session_handoff_audit.md]] — grep prior jsonls before claiming infra work undone
- [[project_dns_routing.md]] — Cloudflare records for `vps` + `teleport` go here, with proxy=OFF; NEVER import the `homelab/mitchflix-cloudflare-import.zone` as-is (LAN records contaminate public DNS)
- [[feedback_do_not_checklist_when_you_have_access.md]] — VPS work is mine to execute via SSH, not yours to checklist
