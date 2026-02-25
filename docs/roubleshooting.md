# Troubleshooting

This guide covers common problems:
- Traces not arriving
- Prometheus scraping failing
- Grafana cannot query Tempo
- Port conflicts
- Docker networking issues (shared external network)

> This file will be completed after the stack configuration files are added.


## OTel Collector fails to start (resource processor / env templating)

Symptoms:
- Collector restarts in a loop
- Error: `failed to create "resource" processor ... Either field "value", "from_attribute" or "from_context" must be specified`

Root cause:
- YAML/env templating can be parsed incorrectly when using default syntax with colon (`${env:VAR:default}`) in certain configurations.

Resolution (baseline):
- Keep the collector config minimal and stable (no resource processor).
- Reintroduce resource attributes later using safe env substitution (prefer `${env:VAR}`) and ensure variables are defined in `docker/.env`.