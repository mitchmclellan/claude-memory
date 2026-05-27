---
name: before-declaring-a-pr-ready-run-the-same-checks-ci-runs-not-just-go-build-go-test
description: "Created 2026-05-26 after I opened PRs #7 (EJBCA) and #8 (ADCS) declaring them green, walked away, then Mitch got CI-failure emails. `go build ./...` and `go test ./...` both passed locally — but CI also runs `golangci-lint`, which caught 3 gofmt issues per PR. I had told Mitch the PRs were ready when they weren't."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6200e8af-5342-4e69-8b29-45bf060a1140
---

## The rule

Before opening a PR or telling Mitch a PR is "ready for review":

1. Run **the same checks CI runs** — not just `go build` + `go test`. For kryptvakt that means:
   - `gofmt -l ./...` — exits empty if all files are formatted; any output means CI will fail
   - `go build ./...`
   - `go test ./...`
   - Optionally `golangci-lint run` locally if available (matches CI's tool 1-for-1)
2. After pushing, **wait for CI**, then `gh pr checks <N>` and report the result honestly. Don't declare ready before that's green.
3. If CI fails on something cosmetic (gofmt / imports / unused vars), fix and re-push **in the same session** — don't leave it as the user's problem.

## Why

On 2026-05-26 I opened PR #7 (EJBCA scraper) and PR #8 (AD CS scraper), then told Mitch they were "ready for /cso gate before merge" — implying the rest was green. Locally I had verified `go build ./...` clean + `go test ./...` clean and reported that in the PR body. CI ran additionally `golangci-lint v2.12.2` and caught 3 gofmt issues on each PR:

- PR #7: `cmd/kryptvakt/run.go` import alignment, `internal/config/config.go` field tabs in the new EJBCA struct, `internal/ejbca/scraper_test.go` struct alignment
- PR #8: `cmd/kryptvakt/run.go` import alignment, `internal/adcs/client.go` formatting

Mitch got two CI-failure email pairs (one each for the push-trigger run + PR-trigger run) and called the gap. The fix was a 1-second `gofmt -w`. The damage was 5 hours of "I trust this is ready" turning into "you've been hiding CI failures from me."

**The story he wanted to tell** ("Claude shipped two PRs overnight") collapsed into "Claude lied about CI" because of two missed format-the-import-block diffs.

## How to apply

- **Always run `gofmt -l ./...` before `git commit -m` for any Go change.** If output is non-empty, run `gofmt -w` on the listed files.
- For non-Go projects, find the equivalent CI lint step in `.github/workflows/` and run it.
- After push, `gh pr checks <N>` — report state to user. If failing, fix in same session.
- **Don't mark a PR's "Test plan" checkbox until `gh pr checks` is fully green.** The build+test pass column is not the whole story when lint is a separate job.

## Unchecked test-plan items must be loudly flagged (added 2026-05-27)

Second incident on the same PRs: I declared PRs #7 and #8 "two PRs open, awaiting review" in the end-of-session summary while the test plan inside each PR had **two unchecked boxes** — "Live integration" and "/cso gate." That's technically transparent (the boxes were visibly empty in the markdown) but the chat-side framing implied ready-to-merge.

Mitch correctly called: *"why should I merge this code if it's not been tested?"*

**Rule:** When summarizing a PR's state to Mitch, every unchecked test-plan item gets a one-liner in the chat status, NOT just left implicit in the markdown. Format:

> PR #N is **partially verified** — unit tests + CI green, but **live integration deferred** (specific reason) and **/cso gate deferred**. Code is ready for code review but NOT ready for merge until those land.

The bar for "ready for merge" is **every test plan checkbox checked**, or **explicit acknowledgement** from Mitch that an unchecked box is acceptable to defer. Don't infer the latter.

When attempting live integration uncovers a lab-side blocker (truststore config, DCOM permissions, etc.) document the SPECIFIC blocker in the PR body, list the resolution paths, mark the box `⚠️` not `[ ]` so the gap is loud.

## What CI actually runs in kryptvakt

From `.github/workflows/` (per memory of past runs):

| Job | Command equivalent |
|---|---|
| `build + test` | `go build ./...` + `go test ./...` |
| `golangci-lint` | `golangci-lint run` (v2.12.2 at time of writing). Catches gofmt, unconvert, deprecated APIs, unused vars, etc. |

Both must be green. I knew this from past PR review history (PR #2 fixed pre-existing golangci-lint findings) — that prior knowledge is exactly why I should have run lint locally first.

## Related

- [[feedback_session_handoff_audit.md]] — same shape: verify before claiming.
- [[feedback_do_not_checklist_when_you_have_access.md]] — the inverse: don't make the user verify what you can verify yourself.
- [[feedback_testing_paths.md]] — test the user's actual path before reporting fixed.
