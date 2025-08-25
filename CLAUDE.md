# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the Claude Code Observability Stack - a comprehensive monitoring solution for analyzing Claude Code usage, performance, costs, and productivity. It provides Grafana dashboards, OpenTelemetry collection, and production-ready Docker infrastructure for tracking AI-assisted development workflows.

## Architecture

The stack follows this data flow:
```
Claude Code → OpenTelemetry Collector → Prometheus (metrics) + Loki (events/logs)
                                     ↓
                              Grafana (visualization & analysis)
```

### Core Components

- **OpenTelemetry Collector**: Metrics and logs ingestion (ports 4317 gRPC, 4318 HTTP)
- **Prometheus**: Time-series metrics storage and querying (port 9090)
- **Loki**: Log aggregation and storage (port 3100)
- **Grafana**: Dashboard visualization and analysis (port 3000, admin/admin)

## Development Commands

### Stack Management
```bash
# Start the full observability stack
make up

# Stop the stack
make down

# Restart services
make restart

# Clean up containers and volumes completely
make clean

# Check status of all services
make status
```

### Monitoring & Debugging
```bash
# View logs from all services
make logs

# View specific service logs
make logs-collector      # OpenTelemetry collector only
make logs-prometheus     # Prometheus only
make logs-grafana       # Grafana only

# Validate configurations
make validate-config
```

### Claude Code Telemetry Setup
```bash
# Show setup instructions for enabling telemetry
make setup-claude

# For testing/debugging (faster export intervals)
export OTEL_METRIC_EXPORT_INTERVAL=10000
export OTEL_LOGS_EXPORT_INTERVAL=5000
```

## Key Configuration Files

### Core Infrastructure
- `docker-compose.yml`: Main stack orchestration with service definitions
- `Makefile`: Management commands and operational shortcuts

### OpenTelemetry & Monitoring
- `collector-config.yaml`: OpenTelemetry collector routing and processing configuration
- `prometheus.yml`: Prometheus scraping and storage configuration
- `grafana-datasources.yml`: Grafana data source connections (Prometheus, Loki)
- `grafana-dashboards.yml`: Dashboard provisioning configuration

### Dashboard & Visualization
- `claude-code-dashboard.json`: Complete Grafana dashboard with all monitoring panels

## Monitored Metrics

The stack tracks Claude Code usage through these key metrics:

### Core Usage Metrics
- `claude_code.session.count`: CLI sessions started
- `claude_code.lines_of_code.count`: Lines of code modified (added/removed)
- `claude_code.pull_request.count`: Pull requests created
- `claude_code.commit.count`: Git commits created
- `claude_code.cost.usage`: Cost breakdown by model
- `claude_code.token.usage`: Token usage (input/output/cache/creation)

### Performance & Events
- `claude_code.api_request`: API requests with duration and token metrics
- `claude_code.api_error`: API errors with status codes
- `claude_code.tool_result`: Tool execution results and timings
- `claude_code.user_prompt`: User prompt submissions (when enabled)

## Dashboard Structure

The Grafana dashboard provides comprehensive analysis through organized sections:

1. **📊 Overview**: High-level KPIs (sessions, costs, tokens, code changes) + Claude Code Pro usage tracking
2. **💰 Cost & Usage Analysis**: Model cost tracking and API request monitoring
3. **🔧 Tool Usage & Performance**: Tool frequency and success rates
4. **⚡ Performance & Errors**: API latency and error rate tracking
5. **📝 User Activity & Productivity**: Code changes, commits, and productivity metrics
6. **🔍 Event Logs**: Real-time tool execution events and error analysis

### Claude Code Pro Usage Monitoring

The dashboard now includes dedicated panels for tracking Claude Code Pro subscription limits:

- **🕐 5시간 사용량 (예상 $10 한도)**: Gauge showing usage against estimated 5-hour limit
- **📅 1주 사용량 (예상 $50 한도)**: Gauge showing usage against estimated weekly limit
- **⏰ 5시간 내 비용**: Actual cost used in current 5-hour period
- **💵 1주 사용 비용**: Actual cost used in current week

**Note**: These limits are estimated based on cost metrics as proxy indicators, since Claude Code Pro limits are actually based on prompt count and usage time rather than direct cost.

## Service URLs

When the stack is running:
- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Loki**: http://localhost:3100
- **OpenTelemetry Collector**: http://localhost:4317 (gRPC), http://localhost:4318 (HTTP)

## Development Notes

### Telemetry Configuration
This stack requires Claude Code to be configured with telemetry enabled. The essential environment variables are:
```bash
CLAUDE_CODE_ENABLE_TELEMETRY=1
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

### Privacy Controls
User prompt content logging is disabled by default. Enable with `OTEL_LOG_USER_PROMPTS=1` if needed for analysis.

### Dashboard Development & Troubleshooting

**Adding New Panels**:
1. Edit `claude-code-dashboard.json` directly
2. Ensure proper file permissions: `chmod 644 claude-code-dashboard.json`
3. Restart Grafana: `docker restart grafana`
4. Check Grafana logs for errors: `docker logs grafana --tail 20`

**Common Issues**:
- **Permission Denied**: Grafana container cannot read dashboard file
  - Solution: `chmod 644 claude-code-dashboard.json`
- **Prometheus Query Errors**: Invalid query syntax (e.g., wrong function usage)
  - Solution: Use `clamp_max(value, limit)` instead of `min(limit, value)`
- **Layout Issues**: Panel positioning conflicts
  - Solution: Adjust `gridPos` coordinates systematically

### Extending the Stack
- Dashboard modifications: Edit `claude-code-dashboard.json` and restart Grafana
- Metric collection: Modify `collector-config.yaml` for new data sources
- Retention policies: Adjust Prometheus and Loki configurations for data lifecycle management
- New services: Add to `docker-compose.yml` with proper network connectivity

## Reference Documentation

The implementation follows the official [Claude Code Observability Documentation](CLAUDE_OBSERVABILITY.md) patterns and recommendations for metric naming, data structure, and analysis approaches.