---
name: Don't ask Mitch to run multi-line bash commands
description: When giving Mitch shell commands to run, keep them on one line — wrapped commands get parsed as multiple commands and fail
type: feedback
originSessionId: 351b90a3-9068-4f06-9fc0-e0a7e6205ed8
---
When Mitch copies a shell command from chat into his terminal, line breaks in the rendered output **become real newlines**, and bash parses the wrapped tail as a separate command. Bit us twice in one session:

1. `echo '…' | sudo tee  \n  /etc/sudoers.d/mitch-laptop` → bash ran `echo '…' | sudo tee` (no file arg) and then tried to *execute* `/etc/sudoers.d/mitch-laptop` as a command → "No such file or directory"
2. Same trap on the verification rerun via `!` prefix

**Why:** The Claude Code chat width is narrower than a typical terminal, so any command longer than ~80 chars wraps visually. Mitch (reasonably) copies what he sees.

**How to apply:**
- Keep shell commands Mitch needs to run **under 80 chars on one line** when possible
- For longer commands, wrap the whole thing in a single `sudo sh -c "…"` or `bash -c "…"` so even if the visual line breaks, it's one shell-level command
- For long pipelines, put them in a code fence and tell him explicitly: "copy this whole block as one line"
- `!` prefix in Claude Code prompts has **no TTY** — sudo password prompts will fail there; tell him to run interactive sudo in his regular terminal instead
