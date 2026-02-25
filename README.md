# erp-observability-platform

> Part of the [ErpOpenSource](https://github.com/ErpOpenSource) organization.

Shared observability infrastructure for the ERP microservices platform.
Spins up a complete metrics + traces + logs stack with a single `docker compose up`.

---

## What's included

| Service | Image | Purpose |
|---|---|---|
| **OpenTelemetry Collector** | `otel/opentelemetry-collector-contrib:0.97.0` | Ingests OTLP traces and metrics from all microservices |
| **Grafana Tempo** | `grafana/tempo:2.4.1` | Stores and queries distributed traces |
| **Prometheus** | `prom/prometheus:v2.52.0` | Scrapes and stores metrics |
| **Grafana** | `grafana/grafana:10.4.2` | Correlated visualization: metrics ↔ traces ↔ logs |
| **Loki** *(optional)* | `grafana/loki:2.9.4` | Log aggregation backend |
| **Promtail** *(optional)* | `grafana/promtail:2.9.4` | Collects container stdout and ships to Loki |

---

## Quick start

```bash
# 1. Create the shared Docker network (once per machine)
docker network create erp-platform

# 2. Copy and configure environment variables
cd docker
cp .env.example .env

# 3. Start the core stack (OTel, Tempo, Prometheus, Grafana)
docker compose --env-file .env up -d

# 4. (Optional) Enable log aggregation
docker compose --profile logs up -d
```

| UI | URL | Default credentials |
|---|---|---|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |
| Tempo | http://localhost:3200 | — |

> Change `GRAFANA_ADMIN_PASSWORD` in `docker/.env` before exposing to a network.

---

## Ports

| Service | Port | Protocol |
|---|---|---|
| OTel Collector — OTLP gRPC | 4317 | gRPC |
| OTel Collector — OTLP HTTP | 4318 | HTTP |
| OTel Collector — Prometheus exporter | 8889 | HTTP |
| Tempo | 3200 | HTTP |
| Prometheus | 9090 | HTTP |
| Grafana | 3000 | HTTP |
| Loki *(optional)* | 3100 | HTTP |

All ports are configurable in `docker/.env`.

---

## Repository structure

```
erp-observability-platform/
├── config/
│   ├── otel-collector-config.yml   # OTel Collector pipeline (receivers, processors, exporters)
│   ├── prometheus.yml              # Scrape targets — add new services here
│   ├── tempo.yml                   # Tempo storage and ingestion config
│   └── promtail.yml                # Log collection from Docker containers
├── docker/
│   ├── docker-compose.yml          # Service definitions
│   ├── .env.example                # Environment variables template
│   └── .env                        # Your local config (git-ignored)
└── docs/
    ├── README.md                   # Architecture overview
    ├── Setup.md                    # How to integrate a Spring Boot service
    └── Troubleshooting.md          # Common issues and fixes
```

---

## Integrating a new microservice

Three things are needed for a service to appear in the observability stack:

**1. Join the shared Docker network**

```yaml
# In the service's docker-compose.yml
networks:
  erp-platform:
    external: true
    name: erp-platform
```

**2. Export traces via OTLP**

```yaml
# Environment variables for the service container
- OTEL_SERVICE_NAME=erp-sales-service
- OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
- OTEL_EXPORTER_OTLP_PROTOCOL=grpc
```

**3. Register the scrape target in Prometheus**

Add to `config/prometheus.yml`:

```yaml
- job_name: "erp-sales-service"
  metrics_path: "/actuator/prometheus"
  static_configs:
    - targets: ["erp-sales-service:8082"]
```

Then restart Prometheus:

```bash
docker compose restart prometheus
```

For the full integration checklist see [docs/Setup.md](docs/Setup.md).
For a complete new module walkthrough see the [platform docs](https://github.com/ErpOpenSource/erp-platform/blob/main/docs/NEW_MODULE.md).

---

## Telemetry pipeline

```
Microservices
    │
    │  OTLP gRPC :4317 / HTTP :4318
    ▼
OpenTelemetry Collector
    ├── traces  ──► Grafana Tempo  ──► Grafana
    └── metrics ──► Prometheus     ──► Grafana

Docker stdout (JSON)
    └── Promtail ──► Loki ──► Grafana   (optional profile)
```

---

## Stopping the stack

```bash
# Stop containers (preserves data volumes)
docker compose down

# Full reset — removes all stored metrics, traces, and dashboards
docker compose down -v
```

---

## License

MIT
