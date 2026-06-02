---
name: project-kryptvakt-loki-observability
description: "Loki on LXC 105 IS part of kryptvakt's observability stack — VM 108 promtail ships kryptvakt.service journal here, feeds per-vendor Grafana dashboards' logs panels. Do not uninstall."
metadata: 
  node_type: memory
  type: project
  originSessionId: 85558db8-7999-4b95-9940-0eab8caee4f0
---

LXC 105 (monitoring, 192.168.50.105) runs Loki 3.7.2 on `:3100`. It is not vestigial — it is part of [[project-kryptvakt]] observability:

- **VM 108 (kryptvakt-dev) promtail** scrapes the `kryptvakt.service` systemd journal, parses slog JSON for `vendor`/`serial`/`workload` labels, and pushes to `http://192.168.50.105:3100/loki/api/v1/push`. Config at `/etc/promtail/config.yml` on VM 108.
- **Grafana datasource** at `/etc/grafana/provisioning/datasources/loki.yaml` on LXC 105 — UID `loki`, URL `http://127.0.0.1:3100`. The kryptvakt per-vendor dashboards' logs panels reference a `loki` template variable that defaults to this datasource.
- **Storage:** `/var/lib/loki/` on LXC 105 rootfs (moved from `/tmp/loki` on 2026-05-30). Retention 30d via `limits_config.retention_period: 720h` + `compactor` block. Analytics disabled.
- **Log level:** `warn` (was `debug` until 2026-05-30 — see incident in lab-improvements.md). Future appliance-side promtail configs (CT/Luna syslog forwarders, AD CS event logs) plug into the same Loki.

**Why:** Customer demos and DORA-relevant audit scenarios for [[project-kryptvakt]] need "show me the metric AND the corresponding log line" — that's the Loki+Prometheus+Grafana canonical pattern. Logs panels in the per-vendor dashboards are part of the product story, not optional dev infra.

**How to apply:** If a future session sees Loki on LXC 105 looking idle/half-configured, do NOT propose uninstall. Check VM 108 promtail's `ss -tn ... :3100` connection and the `loki.yaml` datasource first. The local LXC 105 promtail (scraping only `/var/log/messages`) IS vestigial — that one can be removed or repointed — but the Loki server itself is load-bearing.

Related: [[project-kryptvakt]], [[project-kryptvakt-mvp1-principles]], [[reference-lab]].
