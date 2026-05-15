---
name: MitchIdeaPad Laptop Dev Environment
description: WSL2 Ubuntu 24.04 laptop where Claude Code runs locally — sudo, node, CLI tools, key locations
type: reference
originSessionId: 351b90a3-9068-4f06-9fc0-e0a7e6205ed8
---
## Host
- `MitchIdeaPad` — WSL2 Ubuntu 24.04 (noble), kernel 5.15 microsoft-standard-WSL2
- User: `mitch`, primary working dir: `/home/mitch/workspace`
- Separate from the PVE homelab (`pve` at 192.168.50.10) — this is Mitch's local dev laptop where he opens Claude Code

## Sudo
- NOPASSWD configured at `/etc/sudoers.d/mitch-laptop` (mode 440)
- `sudo -n true` returns clean — Claude Code can run any `sudo` without prompting

## Node / npm
- Node 20.x from **NodeSource** repo at `/etc/apt/sources.list.d/nodesource.list` (keyring at `/usr/share/keyrings/nodesource.gpg`)
- `/usr/bin/node` (v20.20.2), `/usr/bin/npm` (v10.8.2)
- Global npm packages install to `/usr/local/bin` — need `sudo /usr/bin/npm install -g …` (sudo's secure_path doesn't pick up `npm` from regular PATH on this box, use absolute path)

## CLIs installed (Linux-side)
| Tool | Path | Auth file | Notes |
|---|---|---|---|
| `codex` | `/usr/local/bin/codex` | `~/.codex/auth.json` | OpenAI CLI, used by gstack `/codex` skill |
| `gemini` | `/usr/local/bin/gemini` | `~/.gemini/oauth_creds.json` | Google Gemini CLI, OAuth-based |
| `claude` | `~/.local/bin/claude` | — | Claude Code itself |
| `bun` | `~/.bun/bin/bun` | — | v1.3.x |
| `gh` | `/usr/bin/gh` | — | GitHub CLI |

Windows-side `npm` shims under `/mnt/c/Users/mitch/AppData/Roaming/npm/` are **broken** in WSL (they try to `exec node` and fail — no Linux node in their shim). PATH puts `/usr/local/bin` first so Linux versions win — leave the Windows shims alone, don't try to use them.

## API keys
- `OPENAI_API_KEY` persisted at `~/.openai_env` (mode 600), sourced from `~/.bashrc` via `[ -f ~/.openai_env ] && . ~/.openai_env`
- New shells get the key automatically; codex CLI itself reads `~/.codex/auth.json` and doesn't need the env var
- Gemini CLI uses OAuth (`~/.gemini/oauth_creds.json`), no API key needed

## Known dpkg breakage (cosmetic, doesn't block)
- `postfix` and `bsd-mailx` are stuck in "not configured" state
- Every `apt install` ends with `Errors were encountered while processing: postfix bsd-mailx / Sub-process /usr/bin/dpkg returned an error code (1)`
- The package you actually asked for still installs successfully — ignore the trailing errors unless they reference your target package
- Fix path (not yet done): `sudo dpkg --remove --force-remove-reinstreq postfix bsd-mailx` or properly configure postfix
