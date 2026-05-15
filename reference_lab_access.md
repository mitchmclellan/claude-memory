---
name: Lab SSH access paths (PVE, LXCs, VMs)
description: How to reach PVE host, LXCs, and VMs from this WSL workspace. Root SSH to LXCs is NOT keyed; go via mitch@pve with sudo.
type: reference
originSessionId: 0ae25004-a164-40d8-8e66-7916189b36aa
---
The AI runs from WSL (`/home/mitch/workspace`). SSH access is partial — the
lab was built by a different Claude CLI session on a different host, so not
every SSH key path is configured from this workspace.

**Direct SSH (works, no sudo needed):**
- `mitch@192.168.50.10` — PVE host (hostname: `pve`). Mitch has passwordless
  sudo here.
- `mitch@192.168.50.108` — VM 108 / kryptvakt-dev. Has Go 1.22.4
  (`/usr/local/go/bin/go`), bao CLI v2.5.3 (installed 2026-05-15).

**Indirect (via PVE host):**
- LXCs (101–109+) — no `mitch` user, root SSH not keyed. Reach via:
  `ssh mitch@192.168.50.10 'sudo -n pct exec <vmid> -- <command>'`
- e.g. `sudo -n pct exec 107 -- jq -r .root_token /root/bao-init.json`

**Not directly reachable:**
- `root@<anything>` over SSH from WSL — keys not deployed.

**OpenBao on VM 108:**
- `BAO_ADDR=http://192.168.50.107:8200` (must go in `~/.profile`, not
  `~/.bashrc` — Ubuntu's bashrc has the standard non-interactive-shell
  early-return guard, so `bash -lc` won't pick up bashrc exports).
- Token file is `~/.vault-token` (OpenBao inherits Vault's filename
  convention; `~/.bao-token` is **wrong** and silently ignored).
- Root token retrievable via the pct-exec path above.
