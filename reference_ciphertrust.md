---
name: CipherTrust Manager CE Operational Reference
description: Account model (ksadmin vs admin), known WebUI hang on password change, REST API workarounds, SSH key requirement
type: reference
originSessionId: 33bc6205-5a78-46e0-be22-04d2cb443bbd
---
CipherTrust Manager CE (k170v-2.11.1+9742) operational notes from VM 109 provisioning 2026-05-14.

**Two-tier account model — DO NOT CONFUSE:**
- `ksadmin` = OS-level console root-equivalent (think de-toothed `su`). Set at first-boot console prompt right after GRUB. Restricted "jailshell". SSH **only** accepts public-key auth — never password. Used for `ks_*` scripts in `/opt/keysecure/` (factory reset, security lockdown, upgrade).
- `admin` = WebUI/API application account. Defaults to `admin/admin`, forces change on first login. Has full management rights inside the appliance (keys, policies, users).

**WebUI password-change hangs indefinitely on this build.** Workaround: use REST API directly. The endpoint accepts the forced-change without an authenticated session:
```
PATCH https://<host>/api/v1/auth/changepw
{"username":"admin","password":"<old>","new_password":"<new>"}
```
Password complexity rules: min 8, max 30, ≥1 upper, ≥1 lower, ≥1 digit, ≥1 special.

**Static IP via API (no console needed once admin works):**
```
PATCH /api/v1/system/network/interfaces/ens18
{"inet":{"method":"static","ip":"...","gateway":"...","netmask":"...","dns":[...]}}
```
The PATCH causes the connection to drop mid-request — that's expected; verify by hitting the new IP after ~5s.

**SSH pubkey for ksadmin must be added through WebUI System → SSH Keys** *before* WebUI auth completes — Mitch confirmed this is the only way. Public key on PVE: `~/.ssh/id_rsa_ciphertrust.pub`; private: `~/.ssh/id_rsa_ciphertrust`.

**First-boot ksadmin prompt is right after GRUB on the Proxmox console.** Missing it is fatal — first-boot completes with no recoverable password. The only fix is destroy VMID + re-import OVA. There is no GRUB recovery entry that resets the wizard.

**Where things live:**
- All credentials: OpenBao `secret/kryptvakt/ciphertrust` (LXC 107, root token at `/root/bao-init.json`)
- OVA was at `/tmp/ciphertrust.ova` on PVE during import (cleaned after import — Mitch keeps the source OVA on his own machine)
- Disk: `ai-storage/vm-109-disk-0` (raw, 50GB)
- Import script: `~/workspace/kryptvakt/scripts/f-import-ciphertrust-ova.sh` — note: doesn't accept `.ova` directly on this Proxmox version, must extract with `tar xf` first and pass the `.ovf`

**How to apply:** When working on CipherTrust integration (exporter, key management, API), reach for OpenBao creds + the API workarounds before assuming the WebUI is the only path. The WebUI hang is reproducible on this build — don't waste time on it.
