# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make help            # Show all available commands
make up              # Start all services
make down            # Stop all services
make restart         # Restart all services
make status          # Check service health
make logs            # View all logs (follow mode)
make logs-collector  # View OTel collector logs
make clean           # Stop and remove volumes
make validate-config # Validate OTel collector config
make setup-claude    # Show Claude Code env var setup
```

## Architecture

Docker Compose stack for Claude Code observability:

```
Claude Code → OTel Collector (4317) → Prometheus (9090) → Grafana (8000)
                                    → Loki (3100)      ↗
```

- **OTel Collector** (`collector-config.yaml`): Receives OTLP telemetry, routes metrics to Prometheus and logs to Loki
- **Prometheus**: Time-series DB for metrics (cost, tokens, LOC, commits)
- **Loki**: Log aggregation for events (prompts, tool results, API requests)
- **Grafana**: Pre-provisioned dashboard at `grafana/dashboards/claude-code-dashboard.json`

## Key Files

- `collector-config.yaml` - OTel pipelines (receivers, processors, exporters). Debug exporter enabled by default.
- `grafana/dashboards/claude-code-dashboard.json` - Pre-built dashboard (edit here, auto-reloads)
- `grafana/provisioning/datasources/datasources.yml` - Auto-configured Prometheus + Loki sources
- `.env.claude` - Environment variables template for Claude Code telemetry

## Debugging

Check if metrics are flowing:
```bash
curl -s localhost:8889/metrics | grep claude_code
```

Query Prometheus directly:
```bash
curl -s 'localhost:9090/api/v1/query?query=claude_code_cost_usage' | jq
```
