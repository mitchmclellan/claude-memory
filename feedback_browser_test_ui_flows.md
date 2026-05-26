---
name: Use gstack browse for any UI flow involving auth, cookies, or redirects — curl is insufficient
description: When verifying a UI deploy that touches login, sessions, cookies, or redirect chains, use the gstack browse binary (~/.claude/skills/gstack/browse/dist/browse) to drive a real browser. curl silently ignores the cookie Secure flag over HTTP and follows redirects by default, which masks browser-only failure modes. Established 2026-05-19 when a curl "passing" verify hid a Secure-cookie-on-HTTP bug that broke Mitch's actual browser login.
type: feedback
originSessionId: a2039218-eab6-4636-adc9-9cdb4937419b
---
When verifying a UI deploy that touches **login, sessions, cookies, or
redirect chains**, use `~/.claude/skills/gstack/browse/dist/browse` to
drive a real headless Chromium session. Do NOT rely on curl as the
sole verification.

**Why:** On 2026-05-19, mid Phase 1c lab verify, I tested
`kryptvakt-ui` login with curl from the VM itself:

```
curl -sk -c /tmp/jar -b /tmp/jar -X POST http://localhost:9120/login \
  -d 'password=admin' -i
```

Curl returned `303 See Other` + `Set-Cookie: kv_session=...; Secure`
and I marked the verify as "passing" because the cookie was issued
and the redirect target was correct. But Mitch's browser silently
dropped that cookie on the next request because the `Secure` flag
disallows transmission over plain HTTP (the lab deploy was
HTTP-only at the IP path). To Mitch, login looked like "form did
nothing" — no `invalid credentials` flash (the POST succeeded),
just a re-bounce to `/login` because `/hsm` saw no session cookie.

**Curl bypasses browser-equivalent cookie semantics in at least these
ways:**
- Ignores the cookie `Secure` flag when storing/sending over HTTP
- Doesn't enforce SameSite cookie policies the way browsers do
- Follows 3xx redirects automatically (with `-L`) without surfacing
  the per-hop cookie/redirect dynamics
- Lacks the full request stack a browser sends (Referer, Origin,
  Sec-Fetch-* headers) that some auth/middleware logic depends on
- Doesn't render JS or honor `<meta http-equiv>` redirects

**How to apply:**

- For any UI verify involving login, logout, change-password,
  session expiry, cookie-based auth, or multi-hop redirects: use
  `gstack browse` from the lab host or a host with network access.
- The browse CLI lives at `~/.claude/skills/gstack/browse/dist/browse`
  (NOT `which browse` — that resolves to xdg-open). Common pattern:
  ```sh
  browse=~/.claude/skills/gstack/browse/dist/browse
  $browse goto <url>
  $browse text                       # read page content
  $browse fill <selector> <value>
  $browse click <selector>
  $browse url                        # confirm landing page
  $browse cookies                    # inspect session state
  $browse screenshot                 # capture for the user to review
  $browse restart                    # clear cookies (or use it to recover from crashes)
  ```
- Curl is still fine for non-UI surfaces — health endpoints, JSON
  APIs without browser-specific cookie behavior, smoke-checking that
  a service is bound. The rule is specifically about **auth +
  cookie + redirect** flows.

**Reinforces `feedback_testing_paths.md`:** that memory says "always
simulate the user's actual path before reporting fixed". The user's
actual path for a web UI is a browser, not curl. Same principle,
specific surface.
