---
name: gstack repo URL
description: gstack lives at github.com/garrytan/gstack (not garryslist). Private SSH-only clone.
type: reference
originSessionId: a60c7881-d80b-4b36-9152-92ab906898a3
---
The gstack skill repo is at `git@github.com:garrytan/gstack.git`. Clone via SSH only — it's private and HTTPS clone will fail with "could not read Username".

The user corrected this on 2026-05-14 during the WSL laptop migration — a prior playbook had `garryslist/gstack` which doesn't exist.

Standard install path: `~/.claude/skills/gstack/`. Requires `bun` to run `./setup` (which builds the browse binary and registers skills).

Related: gstack-artifacts repo is at `git@github.com:mitchmclellan/gstack-artifacts.git`, clones into `~/.gstack/`.
