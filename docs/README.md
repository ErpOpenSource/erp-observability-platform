# Observability Platform — Overview

This repository (`erp-observability-platform`) hosts the shared observability layer for the ERP
microservices ecosystem, decoupled from the API Gateway and business services.

## High-level architecture

ERP Services (Spring Boot, etc.)
  -> OTLP traces/metrics -> OpenTelemetry Collector
  -> Traces -> Tempo -> Grafana
  -> Metrics -> Prometheus -> Grafana
  -> Logs (stdout) -> Promtail -> Loki -> Grafana (optional)

## Goals
- Standardize telemetry ingestion for all ERP microservices
- Provide correlated troubleshooting (metrics ↔ traces ↔ logs)
- Keep everything reproducible with Docker Compose (local) and scalable to production

## Quickstart (placeholder)
The actual commands will live in `docs/Setup.md` once the compose and configs are added.

## Networking strategy
This stack uses a dedicated Docker network (planned): `erp-platform`, allowing:
- Gateway and services in the same compose
- Gateway and services in separate compose files attached to the external shared network