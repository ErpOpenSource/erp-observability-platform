# erp-observability-platform

Enterprise-grade observability platform for an ERP microservices architecture.

This repository provides a reusable, production-ready observability stack:
- OpenTelemetry Collector (OTLP gRPC/HTTP ingestion)
- Grafana Tempo (distributed tracing)
- Prometheus (metrics scraping)
- Grafana (visualization and correlation)
- (Optional) Loki + Promtail (logs aggregation & collection)

## Documentation
- docs/README.md — Architecture overview & quickstart
- docs/Setup.md — How to integrate ERP services (Spring Boot) for metrics/tracing/logs
- docs/Troubleshooting.md — Common issues and fixes

## Repository structure
See `docs/README.md` for the full layout and operational notes.