---
name: mitchideapad-laptop-dev-environment
description: MitchIdeaPad — native Ubuntu 26.04 (post-WSL migration 2026-06-01) where Claude Code runs. Working dir is ~/workshop (was ~/workspace under WSL).
metadata: 
  node_type: memory
  type: reference
  originSessionId: f3c858c2-bb03-418f-aded-42631999aaab
---

## Host
- `MitchIdeaPad` — **native Ubuntu 26.04 (oracular), Linux 7.0** (migrated from WSL2 Ubuntu 24.04 on 2026-06-01 — see [[project_laptop_migration_2026_06_01]])
- User: `mitch`, primary working dir: **`/home/mitch/workshop`** (NOT `~/workspace` — that's the old WSL path; PVE-host workspace paths unchanged)
- Display server: Wayland (`XDG_SESSION_TYPE=wayland`, `WAYLAND_DISPLAY=wayland-0`)
- Terminal emulator: **Ptyxis** (the new GNOME container-aware terminal). For copy-paste: shift-drag to select past mouse capture, `Ctrl+Shift+C/V` for clipboard. No `xclip`/`wl-copy`/`xsel` installed — `sudo apt install wl-clipboard` if CLI clipboard needed.
- Separate from the PVE homelab (`pve` at 192.168.50.10) — this is Mitch's local dev laptop where he opens Claude Code

## Sudo
- NOPASSWD configured at `/etc/sudoers.d/mitch-laptop` (mode 440)
- `sudo -n true` returns clean — Claude Code can run any `sudo` without prompting

## Node / npm
- Node 20.x from **NodeSource** repo
- Global npm packages → `/usr/local/bin`; use absolute path `sudo /usr/bin/npm install -g …` because sudo's secure_path doesn't pick up `npm`

## CLIs installed
| Tool | Path | Auth file | Notes |
|---|---|---|---|
| `codex` | `/usr/local/bin/codex` | `~/.codex/auth.json` | OpenAI CLI; default is chatgpt-account auth — flip to apikey via `codex logout && codex login --with-api-key` per [[reference_codex_auth_modes]] |
| `gemini` | `/usr/local/bin/gemini` | `~/.gemini/oauth_creds.json` | Google Gemini CLI, OAuth |
| `claude` | `~/.local/bin/claude` | — | Claude Code itself |
| `bun` | `~/.bun/bin/bun` | — | needed for gstack |
| `gh` | `/usr/bin/gh` | — | GitHub CLI |

## API keys
- `OPENAI_API_KEY` at `~/.openai_env` (mode 600), sourced from `~/.bashrc`
- `GEMINI` uses OAuth at `~/.gemini/oauth_creds.json`

## Known gaps (post-migration)
- **Internal DNS resolution wired 2026-06-02** via systemd-resolved drop-in at `/etc/systemd/resolved.conf.d/mitchflix.conf` (routing-only domain `~mitchflix.co.uk` → Pi-hole `192.168.50.106`). See [[project_dns_routing]]. If `*.mitchflix.co.uk` ever NXDOMAINs again on the laptop, check that file first.
- **Playwright/Chromium can't install on Ubuntu 26.04** — `npx playwright install chromium` errors with `Playwright does not support chromium on ubuntu26.04-x64`. Blocks every browse-driven gstack skill (`/qa`, `/qa-only`, `/design-review`, `/devex-review`, `/canary`, `/scrape`, `/automate`). Workarounds: (a) system chromium via apt + `executablePath` override; (b) wait for upstream Playwright support; (c) authed curl for non-JS HTML pages. Decision deferred per `homelab/lab-improvements.md` 2026-06-01 entry. See [[feedback_browser_test_ui_flows]] for when this matters.
- WSL-era Windows shim references are gone — no `/mnt/c/...` paths, no PowerShell, no `wsl.conf`.
- Postfix breakage from the WSL install no longer relevant on the fresh native install (verify if you ever see `dpkg returned an error` again).
