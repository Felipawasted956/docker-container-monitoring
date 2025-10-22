# Docker - Container Monitoring

This repository provides a self-contained monitoring stack for Docker containers using Prometheus, Grafana and cAdvisor, with a small helper exporter to surface per-container network metrics on macOS (Docker Desktop).

Overview

- Prometheus: scrapes metrics from cAdvisor, node-exporter, and a small docker-stats exporter that polls the Docker Engine for per-container network stats.
- Grafana: ships a pre-provisioned dashboard `docker-monitoring` (see `monitoring/grafana/provisioning/dashboards/docker-monitoring.json`).
- cAdvisor: collects container metrics (CPU, memory, filesystem, block IO) and is scraped by Prometheus.
- docker-stats-exporter: a lightweight Python exporter that reads the Docker socket and exposes per-container network RX/TX counters (useful on macOS where cAdvisor often reports host-level interfaces only).

What this dashboard shows
# Docker - Container Monitoring


This repository provides a self-contained monitoring stack for Docker containers using Prometheus, Grafana and cAdvisor. It also includes a small helper exporter to surface per-container network metrics on macOS (Docker Desktop).

Overview

- Prometheus: scrapes metrics from cAdvisor, node-exporter, and a small docker-stats exporter that polls the Docker Engine for per-container network stats.

- Grafana: ships a pre-provisioned dashboard `docker-monitoring` (see `monitoring/grafana/provisioning/dashboards/docker-monitoring.json`).

- cAdvisor: collects container metrics (CPU, memory, filesystem, block IO) and is scraped by Prometheus.

- docker-stats-exporter: a lightweight Python exporter that reads the Docker socket and exposes per-container network RX/TX counters (useful on macOS where cAdvisor often reports host-level interfaces only).

What this dashboard shows

- CPU usage (per-container)

- Memory usage (bytes and %)

- Network I/O (per-container RX/TX via `docker-stats-exporter`, with a cAdvisor host-level fallback)

- Disk I/O (reads/writes)

- Filesystem usage (used, limit, %)

Files of interest

- `docker-compose.yml` — brings up Prometheus, Grafana, cAdvisor, node-exporter, and the docker-stats-exporter.

- `monitoring/prometheus/prometheus.yml` — Prometheus scrape config.

- `monitoring/grafana/provisioning/dashboards/docker-monitoring.json` — the Grafana dashboard (provisioned at container start).

- `monitoring/docker-stats-exporter/` — small Python exporter (Dockerfile + app.py) that polls Docker stats and exposes per-container network counters.


Quick start

1. Build and start everything with Docker Compose:

```bash
docker compose up -d --build
```

1. Open Grafana in your browser:

- Grafana: <http://localhost:3000>

1. Open Prometheus (optional):

- Prometheus: <http://localhost:9090>

1. Verify targets:

- Prometheus should show `cadvisor`, `node-exporter`, and `docker-stats-exporter` as UP on the Targets page (`/targets`).

Notes about Network I/O on macOS

- cAdvisor on macOS/Docker Desktop often reports host-level interfaces (id="/") instead of per-container veth interfaces. To provide per-container network metrics on macOS, this repo includes `docker-stats-exporter`, which polls the Docker Engine API (via `/var/run/docker.sock`) and exposes per-container cumulative RX/TX bytes to Prometheus. Prometheus then scrapes that exporter and the Grafana dashboard uses those metrics by default.

PromQL examples used in the dashboard

- Container RX rate (1m):

```promql
rate(docker_container_network_receive_bytes_total[1m])
```

- Container TX rate (1m):

```promql
rate(docker_container_network_transmit_bytes_total[1m])
```

Where to put the dashboard screenshot

- If you'd like the README to display the live dashboard screenshot, add your screenshot as `monitoring/grafana/provisioning/dashboards/dashboard.png` and this README will reference it. The repo doesn't include large binary assets by default, so you can place your preferred screenshot there.

Example (once you add the file):

![Dashboard](monitoring/grafana/provisioning/dashboards/dashboard.png)

Troubleshooting

- If the Network panel shows no per-container data on macOS, check that `docker-stats-exporter` is running and that Prometheus shows it as UP:

```bash
docker ps --filter name=docker-stats-exporter
curl -sS <http://localhost:9090/api/v1/targets> | jq '.data.activeTargets[] | {job:.labels.job, health:.health}'
```

- If you change the dashboard JSON, restart Grafana to pick up provisioning changes:

```bash
docker compose restart grafana
```

Security note

- `docker-stats-exporter` mounts the Docker socket read-only which is common for local monitoring. Be cautious about exposing this stack in untrusted networks.

License

- MIT
