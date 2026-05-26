---
name: Kryptvakt auth defaults to appliance-style (admin:admin + change on first login)
description: For kryptvakt and similar customer-shipped appliance-style products, default to standard appliance auth (admin:admin + force change on first login). Don't over-engineer with env-var/secrets-manager-templated passwords during stealth-build. Reason established 2026-05-19 when OpenBao-templated password setup made the operator unable to lab-verify after losing the saved password.
type: feedback
originSessionId: a2039218-eab6-4636-adc9-9cdb4937419b
---
For kryptvakt and similar customer-shipped appliance-style products,
default to **standard appliance auth: admin:admin + force change on
first login**. Do NOT over-engineer with env-var-templated or
secrets-manager-injected passwords during the stealth-build phase.

**Why:** On 2026-05-19, mid lab-verify of Phase 1c, Mitch could not log
in to kryptvakt-ui because the saved `KRYPTVAKT_UI_ADMIN_PASSWORD`
(24-char base64-ish string templated from OpenBao) didn't match what
the running process expected, and DNS was down so he couldn't reach
OpenBao to re-fetch. Quote: *"this admin pw stuff is overkill ... why
is it not just admin:admin , like every other application ever made?
even when we give it to the customer, that's what it should be with a
requirement to change it after first loging or something. ... who
cares? just make it admin:admin and let's get back to work."*

This was a real, generalizable preference — not a one-off frustration.
HSM vendors that kryptvakt competes with (Thales CipherTrust, Entrust,
Utimaco) all ship appliance-style defaults with forced change on first
login. Matching that pattern reduces deployment friction, removes the
"where's my password stored / which version is current" question, and
fits the operational profile of the eventual buyer (security/ops teams
who know the pattern from every other appliance they own).

**How to apply:**

- **For any operator-facing auth surface in kryptvakt** (UI, future
  TLS cert rotation flows, MCP credential setup, etc.): default to a
  well-known seed value, force change on first login, store the
  changed value in the local SQLite (or the canonical
  per-tenant store), and ignore the env-var/secrets-manager-templated
  override after first change. The reference implementation is
  `internal/db/uiadmin.go` + `internal/ui/auth.go` +
  `migrations/0003_ui_admin.up.sql` on the kryptvakt repo.

- **Do NOT propose reverting** to env-var-only or
  OpenBao-templated-only auth in future sessions, even with security-
  hardening intent. The SQLite-backed pattern IS the secure path
  because (a) bcrypt at rest, (b) the seed is never deployed (daemon
  generates the hash at runtime so no known-plaintext shipping in the
  binary), (c) operator change is auditable via `updated_at` +
  `is_default` flag.

- **Env var stays optional, not required.** When set, it changes the
  seed value from `admin` to whatever the operator specified — useful
  for organizations with policy against `admin:admin` even
  transiently. Once changed via the UI, the env var is ignored.

- **The same pattern applies to the future customer-facing product.**
  Buyers expect appliance-style defaults. Build it the same way the
  lab runs it; don't introduce a separate "enterprise auth" path for
  the buyer SKU unless a specific design partner asks for it.

- **Reinforces `feedback_lab_not_prod.md`:** the OpenBao-templated
  setup was security-theatre for a single-user lab — it added
  operational friction without adding meaningful protection (Mitch
  is the only operator; the kryptvakt user already has read on the
  full secrets path; the bcrypt hash on disk via the daemon is
  exactly as secure as the bcrypt hash on disk via OpenBao templating).
  Apply the same skepticism to other infrastructure auth surfaces
  before suggesting OpenBao integration.
