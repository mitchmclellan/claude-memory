---
name: CipherTrust Manager CE v2.11.1 API surface
description: Which /api/v1/* endpoints actually exist on CT Manager CE. The vendor docs imply a much wider surface; most of it 404s on the CE edition.
type: reference
originSessionId: 0ae25004-a164-40d8-8e66-7916189b36aa
---
Discovered empirically against CipherTrust Manager CE k170v-2.11.1 on VM 109
on 2026-05-15 while writing the kryptvakt CT exporter. Re-verify after any
firmware upgrade.

**Endpoints that exist and return useful data:**

| Path | Returns |
|------|---------|
| `POST /api/v1/auth/tokens` | `{jwt, duration, token_type, client_id, refresh_token, refresh_token_id}`. Duration on CE = 300s (5 min) |
| `GET /api/v1/system/info` | `{name, version, version_suffix, model, vendor, crypto_version, uptime, system_time}`. `name` is empty on a stock install |
| `GET /api/v1/nodes` | Paginated `{skip, limit, total, resources[]}` of cluster nodes. Empty on standalone CE |
| `GET /api/v1/cluster` | `{nodeID, status:{code, description}}`. nodeID empty on standalone; status.code = "none" |
| `GET /api/v1/vault/keys` | Paginated keys envelope. Use `?limit=1` if you just need `.total` |
| `GET /api/v1/vault/keys2` | Same as `vault/keys`, alternative paginated path |
| `GET /api/v1/usermgmt/users/self` | Caller user record — useful as an auth sanity check |

**Endpoints that 404 on CE (in vendor docs but not in this firmware):**

`alarms`, `alarm-configs`, `alerts`, `events`, `audit-records`, `system/services`,
`system/syslog`, `system/health`, `services`, `configs/services`, `configs/cluster`,
`cluster/nodes`, `cluster/info`, `partitions`, `vault/partitions`, `vault/cmkeys`,
`vault`, `properties`, `license`, `hsm`, `luna-hsm-servers`, `docs`, `/api/v1`.

**Quirks:**
- No hardware serial exposed via the API on CE. The kryptvakt scraper
  synthesizes `ct-<sha256(hostname):6>` as a fallback.
- Login body must include `grant_type:"password"` — older firmwares
  reportedly require it explicitly.
- TLS cert is self-signed by default; lab usage = `CT_INSECURE_SKIP_VERIFY=true`
  with the WARN log.
- All "list" endpoints follow the `{skip, limit, total, resources[]}`
  envelope.
