# Phase 06 — Prometheus Monitoring

## Objective

Configure Prometheus to scrape metrics from all four Silo nodes in the local Docker Compose cluster and verify monitoring through the Prometheus web interface.

## Architecture

- **Metrics collector:** Prometheus
- **Storage cluster:** Four Silo nodes (`silo1` through `silo4`)
- **Metrics endpoint:** `/minio/metrics/v3`
- **Scrape interval:** 15 seconds
- **Prometheus UI:** `http://127.0.0.1:9090/graph`
- **Container network:** `silo_network`

Prometheus and the Silo nodes communicate over the Docker Compose network.

## Configuration

Prometheus configuration is maintained in:

`prometheus/prometheus.yml`

The scrape job uses the following settings:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "silo"
    metrics_path: /minio/metrics/v3
    static_configs:
      - targets:
          - "silo1:9000"
          - "silo2:9000"
          - "silo3:9000"
          - "silo4:9000"
