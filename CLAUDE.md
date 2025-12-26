# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a monitoring stack configured with docker-compose, featuring:
- **Grafana**: Visualization and dashboards
- **Loki**: Log aggregation system
- **Mimir**: Metrics storage (Prometheus-compatible)

## Architecture

The stack is designed to run as containerized services orchestrated by docker-compose, providing a complete observability platform for collecting, storing, and visualizing logs and metrics.

### Expected Components

- **docker-compose.yaml**: Main orchestration file defining all services and their configurations
- **Grafana**: Dashboard and visualization service with datasource configurations for Loki and Mimir
- **Loki**: Log aggregation backend with configuration for storage and retention
- **Mimir**: Metrics storage backend compatible with Prometheus remote write
- **Configuration files**: Service-specific configs (grafana.ini, loki-config.yaml, mimir-config.yaml, etc.)
- **Provisioning**: Grafana datasource and dashboard provisioning files

## Development Commands

When implementing the stack, typical commands will include:

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f [service-name]

# Stop all services
docker-compose down

# Remove volumes (clean state)
docker-compose down -v

# Rebuild services after config changes
docker-compose up -d --build
```

## Configuration Considerations

- **Persistence**: Configure volume mounts for data persistence across container restarts
- **Networking**: Services should communicate over a dedicated docker network
- **Port mappings**: Grafana typically on 3000, Loki on 3100, Mimir on 9009
- **Security**: Use secrets management for sensitive credentials, avoid hardcoding passwords
- **Resource limits**: Set appropriate memory and CPU limits for each service
- **Retention policies**: Configure log and metric retention based on storage capacity
