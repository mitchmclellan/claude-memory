---
name: Kryptvakt MVP1 product principles (post-ADR-001)
description: Four principles that govern Phase 1+ scope. Established 2026-05-15. Apply these as the lens when planning any kryptvakt feature, not just when reading the design doc.
type: project
originSessionId: 0ae25004-a164-40d8-8e66-7916189b36aa
---
Per ADR-001 (in `/home/mitch/workspace/kryptvakt/ADR.md`), MVP1 scope
expanded on 2026-05-15. The original 2026-05-14 design doc had a narrower
Phase 1; Mitch's product-taste pass added the four principles below.
They apply from Phase 1 onward, not "eventually" or "in Phase 4".

**Why:** Mitch identified that the original Phase 1 output would be
structurally undeployable in the buyer environment (banks running HSMs
disallow external library fetches from corp intranets), would miss the
highest-ROI demo surface (the vendor tools Kryptvakt competes with
paywall exactly SNMP/syslog/audit), would lock the v2 quorum feature
into a rewrite (node identity and peer discovery are foundational), and
would position the product as a free dev tool with monetization TBD
(banks procure paid products).

**How to apply:**

1. **"Code isn't real until containerized."** When evaluating any
   feature or PR, the question is not "does it compile and pass tests"
   but "does it ship as part of a self-contained `docker-compose up`
   that works on a clean corp-intranet host with no external pulls".
   Default state of new code is "behind the container line".

2. **Multi-protocol ingest is in-scope from Phase 1.** SNMP (UDP 162
   v2c+v3), syslog (UDP/TCP 514 + RFC 5425 TLS), audit log
   (file + REST poll + SHA-256 hash chain). The listening sockets and
   routing plumbing are wired from Phase 1 even when individual parsers
   are incomplete. Retrofitting these later invalidates the
   out-of-the-box claim. Phrase that has come up: "SNMP/syslog is free
   money".

3. **kryptvakt-quorum scaffolding ships in MVP1.** Node identity (boot
   UUID), static peer list config, 30s heartbeat + RTT, NTP drift log.
   Not consensus, not replication — those remain v2. But the hooks
   ship. v2 builds on v1 plumbing, never rewrites it.

4. **License gate from day one.** Offline JWT validation (public key
   embedded in binary). Trial license bundled with the image (30-day
   standalone). Full license unlocks multi-exporter / ingest /
   cluster. No licensing server, no self-service in MVP1. Customer
   pastes a key on first run.

**Three-page UI design taste:** Phase 3 ships three pages —
**Physical** (HSM inventory), **Syslog/Events**, **Audit** (with the
hash chain visible inline) — **with per-page authentication**. Mitch
calls this the jaws-drop moment. SOC team sees syslog, audit team sees
audit, physical-ops sees inventory. Separation of duties matches buyer
expectation.

**Cut-line discipline:** if Phase 1 slips past wk 7, drop to "container
+ CT exporter only" and re-scope SNMP/syslog/audit/quorum/license as
Phase 1.5 before Phase 2. Only invoke this if absolutely necessary —
the demo loses its highest-ROI surface.

**Review gates:** ADR-001 awaits ratification via `/plan-ceo-review`
(does this read as ambition or feature creep?), `/plan-eng-review`
(sub-phase sequencing + container build approach + listening-surface
security posture), `/cso` post-implementation (UDP hardening, SNMP v3
defaults, syslog TLS, JWT key rotation, audit hash chain), and
`/codex` adversarial (can the license gate be bypassed by binary
patching — acceptable for MVP1 if yes, but document the threat model).

Mitch's direction is firm: he is prepared to defend this scope against
`/plan-ceo-review` if challenged. Treat the principles as load-bearing
when planning, not as "soft preference".
