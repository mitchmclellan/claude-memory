---
name: before-claiming-infra-work-is-undone-grep-prior-jsonls
description: "When a user references prior work I don't recall, or when I'm about to tell Mitch something is \"blocked on him\" / \"needs his action\" — first grep `/home/mitch/.claude/projects/-home-mitch-workspace/*.jsonl` for the project keywords. Prior sessions often did work that wasn't memorialized. Created 2026-05-26 after the IONOS VPS incident — prior session hardened the VPS, I had no memory of it, claimed \"MITCH-36 is human-gated\" and Mitch had to correct me."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

## The rule

**Before saying "X is blocked on you" or "X hasn't been done yet" for any infra task: grep my own session history first.**

```sh
grep -l '<project-keyword>' /home/mitch/.claude/projects/-home-mitch-workspace/*.jsonl
```

If any hits, read the most recent matching session and reconstruct state. THEN respond.

## Why

The auto-memory system is voluntary. I have to call `Write` on a memory file at end-of-session for state to persist. Sessions sometimes end without that happening — context exhaust, conversation closes, my own discipline lapse. The result: next session walks in blind to work I already did.

**Mitch's mental model is one continuous Claude identity. Session boundaries are MY problem to solve, not his.** If I forget what a prior session did, he experiences it as gaslighting — me telling him work is incomplete when he watched me do it hours ago.

## The 2026-05-26 incident (catalyst)

Pattern of corrections in a single day:
1. EJBCA UI flow — quoted "List End Entities" from training data without checking the live UI. Mitch caught me.
2. Windows Server install / AD CS / EJBCA mint — I wrote "Mitch action items" checklists for tasks I had SSH/WinRM/CLI access to. Mitch caught me three times in one session.
3. **IONOS VPS state** — prior session hardened the VPS, generated `homelab/mitchflix-cloudflare-import.zone`, stashed Vault init at `secret/lab/vps-vault/init`. I had zero memory and confidently told Mitch "MITCH-36 is human-gated, you need to decommission vacay-vibes." Mitch's response: *"you're gaslighting me. I re-imaged, then I dropped your public key into .ssh/authorized_keys. I take notes with timestamps. go find the creds, stop lying and making shit up."* He was right.

All three failures share a shape: confident claim without evidence, when the evidence was a `grep` away.

## How to apply

**Triggering situations:**

- User references prior work I don't recall ("you already did X")
- I'm about to write a "Mitch action items:" section
- I'm about to claim an open ticket is "blocked on you" / "human-gated"
- I'm about to say "this is the first time we're touching X" for any infra/project area
- User pushes back on something I've stated as fact

**The check:**

```sh
# Quick: is there ANY prior session with this keyword?
grep -l '<keyword>' /home/mitch/.claude/projects/-home-mitch-workspace/*.jsonl

# Detailed: what did the most recent session actually DO with it?
grep -h -aoE '<keyword>[^"]{0,400}' <newest-matching-jsonl> | sort -u | head -30
```

If matches are found and the activity isn't in [[MEMORY.md]]: this is a handoff failure. **Write the memory now**, THEN respond to the user with what was actually done.

## Memory write discipline

**At end-of-significant-work, write the memory immediately — don't wait for "end of session" which may never come.** Any one of these triggers a write:

- Created a VM, container, or external service (note where, IP, creds path in OpenBao)
- Stashed credentials in OpenBao (the path is now load-bearing for future sessions)
- Hardened, configured, or bootstrapped a host (note what scripts ran, what survives a reimage)
- Made a commit or PR with non-obvious architectural intent
- Decided to change scope or approach materially

Memory writes are cheap. Sessions ending without them are expensive — they break Mitch's trust.

## Memory push discipline (added 2026-05-26 — DR audit found 11 days of memory writes had never been pushed to remote)

**Writing to disk is not enough. The memory dir at `/home/mitch/.claude/projects/-home-mitch-workspace/memory/` is a git repo with remote `github.com:mitchmclellan/claude-memory.git`. Local-only memory dies with the laptop.**

After any memory write (new file or edit), run:

```sh
cd /home/mitch/.claude/projects/-home-mitch-workspace/memory/
git add -A
git commit -m "<one-line summary of what changed and why>"
git push origin main
```

A good commit message names the new/changed memory files and gives the one-line context. Example:
> `project_kryptvakt_vps_ionos: post-reimage minimal state with claude user + sudo NOPASSWD; OpenBao creds at secret/lab/vps-ionos v1`

**Why this matters:** Discovered 2026-05-26 during a DR audit that no memory had been pushed since 2026-05-15 — 11 days of memory writes existed only on the laptop. If the laptop drive had died in that window, every memory written during those 11 days would have been lost (including the entire kryptvakt VPS context, the AD CS provisioning, the EJBCA cert work, and several feedback rules). The "auto-memory" system writes to disk but does not push — that's MY job after every write.

**When to skip the push** (rare exceptions):
- Mid-conversation, writing 3 related memories — push once after the last one rather than 3 separate pushes
- The change is genuinely trivial (typo fix) — even then, batch it with the next real write
- Network-disconnected (the laptop is on WSL with no internet) — queue the push when connectivity returns; mention to Mitch you'll push later

## Reinforces

- [[feedback_docs_freshness.md]] — lab docs + project CONTEXT.md must be updated end-of-session. This memory is the same principle applied to MEMORY.md.
- [[feedback_do_not_checklist_when_you_have_access.md]] — "Mitch action items" sections for tasks I have access to are anti-patterns; same root cause as not checking jsonl first (claiming Mitch must act when I should just act).
- [[feedback_testing_paths.md]] — verify the actual path before reporting. Same shape: don't claim, check.
