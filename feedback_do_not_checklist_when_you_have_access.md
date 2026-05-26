---
name: when-you-have-access-execute-the-task-yourself-don-t-hand-back-a-checklist
description: "Default to executing tasks myself when access exists. Only ask Mitch for human-gated steps (portal signups, family/legal decisions, physical access). Established 2026-05-19, reinforced 2026-05-26 after I repeated the violation TWICE in one session (EJBCA UI clickthrough + Windows AD CS role install)."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

## The rule (sharpened 2026-05-26)

**Test before handing Mitch anything: "Could I do this myself with the access I have?" If yes, DO IT. The only valid asks are:**

1. **Portal signups requiring human verification** — Utimaco, Atalla, Microsoft EvalCenter, IONOS account creation. Company-vetted forms I can't bypass.
2. **Family/legal/business decisions** — naming the AB, deciding what to disclose to Stu, picking a launch date.
3. **Physical access** — touching the PVE box, swapping a drive, plugging in a USB.
4. **License clicks I literally cannot make** — terms-of-service buttons where my agreement has no legal force.
5. **Credit card / payment entry** — Mitch's money, Mitch's hands.
6. **A judgment call Mitch hasn't delegated** — choosing a brand name, a password he wants to remember himself, a design direction. Default: if he hasn't told me to decide, ask.

**Everything else: do it.** Installing software on lab VMs, configuring services, creating service accounts, setting passwords (when delegated), minting certs, configuring CAs, opening firewall ports, joining domains, installing roles, running PowerShell over WinRM, driving JSF apps via their underlying CLIs — all in scope.

## Anti-patterns I've now done multiple times

- Handing a multi-step shell checklist for a lab deploy when I have SSH (2026-05-19, kryptvakt-ui)
- Telling Mitch to "click through Windows installer" when I have QEMU sendkey + console screenshot loop (2026-05-26 morning, almost — caught myself)
- Telling Mitch to navigate EJBCA admin UI to add an end entity when `ejbca.sh ra addendentity` inside the container works in one command (2026-05-26 afternoon — needed Mitch's prompt to fix)
- Documenting "Mitch action items" steps for installing AD CS role / creating LDAP service account when I have WinRM admin access to the box (2026-05-26 — same session, same hour)

## Why I keep falling back to checklisting

- **Reflex from generic safety priors** that treat "create a Windows service account" or "install a server role" as weighty. In Mitch's lab they aren't — they're routine ops on systems I admin.
- **MITCH-N items in lab-improvements.md** are written as "Mitch action items:" because the file was scaffolded that way. **Stop using that header.** Default to "Claude action items:" and only spin out "Mitch action items:" for the genuine human-gated subset.
- **Treating UI flows as the canonical path.** Almost every product has a CLI or REST surface that's easier to drive than the UI. Look for it first; web-UI is a fallback, not a default.

## How to apply

- When writing a new MITCH-N section: **list the human-gated steps only**, mark the rest as "(Claude executes)" or just do them inline before writing the section.
- When the user gives me a problem statement: think "what blocks me from doing this right now" before drafting any user-facing instruction. If the answer is "nothing" — proceed. If "X portal access I don't have" — ask only for that.
- **Read this memory at the start of every session that touches the lab.** Pin it harder than other memories. The violation pattern is sticky.

## Edge case: long-running installs

When a step takes 5+ minutes (Windows install, container build, ZFS resilver), do the work in the background and tell Mitch what's happening — don't hand off the wait either.

## Original incident (2026-05-19)

[Original kryptvakt-ui checklist incident retained here so the pattern stays visible.]
When the user gives me a clear action and I have the access needed
(SSH to a lab host, file system write, command execution), **default
to executing the task myself**. Do NOT hand back a step-by-step
checklist of shell commands for the user to copy-paste.

**Why:** On 2026-05-19, during Phase 1c lab verify, the user asked
me to deploy and verify the new kryptvakt-ui build on VM 108. My
initial response was a multi-step checklist:

```
ssh mitch@192.168.50.108
cd ~/kryptvakt
git fetch origin && git checkout phase-1c-ui-cutover && git pull
go build -o /tmp/kryptvakt-ui-1c ./cmd/kryptvakt-ui
sudo systemctl stop kryptvakt-ui
sudo cp /tmp/kryptvakt-ui-1c /usr/local/bin/kryptvakt-ui
sudo systemctl start kryptvakt-ui
```

Mitch pushed back: *"you were giving me instructions to perform tasks
you could clearly do your self, why? find out why and address that
too."*

He was right. I had documented SSH access to VM 108 in
`project_kryptvakt.md` (`SSH: mitch@192.168.50.108`), I had used it
in earlier turns of the same session, and the deploy was a routine
sequence of commands. Handing back a checklist made him do work I
could've done in ~30 seconds.

**Why I defaulted to checklisting:**
1. Reflex from a generic "don't take destructive actions without
   confirmation" pattern that's appropriate for production systems
   I've never touched, but not for a lab host I have keyed access to.
2. Treating "deploy to the lab" as more weighty than it is — per
   `feedback_lab_not_prod.md`, the lab is not prod and the cautious
   posture is wrong.
3. Not internalizing that handing back a checklist IS a tax on the
   user, not a courtesy.

**How to apply:**

- **When the user asks for an action I can perform**, do it. Don't
  ask for permission for routine actions on systems where I already
  have access. Confirm only for genuinely destructive or
  hard-to-reverse operations (`rm -rf`, force-push to main, deleting
  a vault, etc.).
- **When I have SSH access to a lab host**, treat it as my own
  environment for the duration of the task. Build, restart, edit
  config, inspect logs — all in-scope.
- **A "you should…" checklist is a tell.** If I find myself writing
  one for a task I have access to, rewrite as "I'll do X, Y, Z; here
  are the results." Skip the imperative-second-person voice.
- **The exception is when the action requires Mitch's judgment** —
  e.g., picking a new admin password, deciding which of two design
  directions to commit to, naming a new branch. For those, I should
  ask, not pick on his behalf.
- **For verify steps that need a human's eyes** (visual UI review,
  "does this look right"), capture screenshots / text via tools
  (`gstack browse`, `screenshot`) and surface them — don't hand off
  the verification itself.

**Reinforces:**
- `feedback_workstyle.md`: "act autonomously, document blockers in
  lab-improvements.md, don't wait"
- `feedback_lab_not_prod.md`: "stop performative caution; just do
  the work"
- `feedback_browser_test_ui_flows.md`: when verifying UI, drive the
  browser myself (gstack browse), don't ask the user to do it

Same shape across all three: the user wants me to be a colleague
with hands, not a tour guide narrating what they should do next.
