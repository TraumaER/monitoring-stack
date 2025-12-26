# monitoring-stack

A complete observability stack using Grafana, Loki, Mimir, and Alloy, configured with Docker Compose. This stack provides metrics collection, log aggregation, and visualization with S3-compatible storage backend support.

## Components

- **Grafana** (port 3102): Visualization and dashboards platform
- **Loki** (port 3100): Log aggregation and storage system
- **Mimir** (port 3101): Prometheus-compatible metrics storage
- **Alloy** (port 12345): Unified metrics and logs collector

## Prerequisites

- Docker and Docker Compose installed
- S3-compatible object storage (Garage HQ recommended)
- Access credentials for your S3 storage

## Setup

### 1. Configure S3 Storage Credentials

Copy the example environment file and configure your S3 endpoint:

```bash
cp .env.example .env
```

Edit `.env` and provide:
- `GARAGE_ENDPOINT` - Your S3 endpoint hostname (without http:// or https://)
- `GARAGE_ACCESS_KEY_ID` - S3 access key ID
- `GARAGE_SECRET_ACCESS_KEY` - S3 secret access key

**Note:** The endpoint should NOT include the scheme (http/https). The `insecure` parameter in service configs controls the connection security.

### 2. Create Required S3 Buckets

Before starting the stack, create these buckets in your S3 storage:

**Loki:**
- `loki` - Log data and index storage

**Mimir:**
- `mimir-blocks` - TSDB blocks storage
- `mimir-ruler` - Recording and alerting rules
- `mimir-alertmanager` - Alertmanager state and configuration

### 3. Start the Stack

```bash
docker-compose up -d
```

### 4. Access Grafana

- **URL:** http://localhost:3102
- **Default credentials:** `admin` / `admin`
- **Datasources:** Pre-configured for Loki and Mimir
- **Anonymous access:** Enabled with Viewer role for easy sharing

## Data Storage

### S3 Storage (Loki & Mimir)
- **Loki:** Stores logs and indexes in S3 bucket `loki`
- **Mimir:** Stores metrics blocks across multiple S3 buckets
- **Configuration:** S3 credentials loaded from `.env` file

### Local Volumes
- **Grafana:** Dashboard and configuration data (`grafana-data` volume)
- **Loki:** Index cache and active indexes (`loki-data` volume)
- **Mimir:** TSDB and compaction working data (`mimir-data` volume)

## Data Retention

- **Loki:** 744 hours (31 days) log retention
- **Mimir:** 30 days block retention
- **Compaction:** Automated compaction runs every 10 minutes (Loki) and 30 minutes (Mimir)

## Metrics & Logs Collection

Alloy automatically collects:

**Logs:**
- All Docker container logs with container metadata labels

**Metrics:**
- Loki internal metrics
- Mimir internal metrics
- Grafana metrics
- Alloy self-metrics
- cAdvisor metrics (requires separate cAdvisor container)

**Note:** The Alloy configuration includes a cAdvisor scrape target that will fail unless you deploy cAdvisor separately. This is optional for basic functionality.

## Management Commands

```bash
# View service logs
docker-compose logs -f [service-name]

# Restart a specific service
docker-compose restart [service-name]

# Stop the stack (preserves data)
docker-compose down

# Stop and remove all data (clean slate)
docker-compose down -v
```

## Architecture

- **Network:** All services communicate over the `monitoring` bridge network
- **Log Flow:** Docker containers → Alloy → Loki (S3)
- **Metrics Flow:** Services → Alloy → Mimir (S3)
- **Visualization:** Grafana queries Loki & Mimir datasources
- **Security:** S3 connections use secure SSL/TLS (configurable via `insecure` parameter)

## Configuration Files

- `docker-compose.yaml` - Service orchestration
- `loki/loki-config.yaml` - Loki configuration with S3 backend
- `mimir/mimir-config.yaml` - Mimir configuration with S3 backend
- `alloy/config.alloy` - Metrics scraping and log collection pipeline
- `grafana/provisioning/` - Datasource and dashboard provisioning

## Troubleshooting

**Service won't start:**
- Verify S3 credentials in `.env` are correct
- Check all required S3 buckets exist
- Review service logs: `docker-compose logs [service-name]`

**No metrics/logs appearing:**
- Ensure Alloy is running: `docker-compose ps alloy`
- Check Alloy is scraping targets: http://localhost:12345
- Verify Grafana datasources are healthy in Grafana UI

**Storage errors:**
- Confirm S3 endpoint is reachable from containers
- Verify bucket permissions allow read/write operations
- Check `insecure` parameter matches your S3 setup (true for HTTP, false for HTTPS)
