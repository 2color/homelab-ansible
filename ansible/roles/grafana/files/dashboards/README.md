# Grafana Dashboards

This directory contains pre-configured Grafana dashboards for monitoring the homelab infrastructure.

## Dashboard Sources

### 1. Node Exporter Full (node-exporter-full.json)
- **Source**: https://grafana.com/grafana/dashboards/1860
- **Author**: starsliao
- **Description**: Comprehensive system metrics dashboard showing CPU, memory, disk, network, and system statistics from Node Exporter
- **Modifications**: Datasource template variables (`${ds_prometheus}`) replaced with direct references to the "Prometheus" datasource

### 2. Prometheus Internal Stats (prometheus-stats.json)
- **Source**: https://grafana.com/grafana/dashboards/11449
- **Author**: Grafana Labs
- **Description**: Comprehensive Prometheus internal metrics dashboard showing server performance, TSDB stats, scrape durations, query performance, and resource usage
- **Modifications**: Datasource references updated to use "Prometheus", __inputs section removed

### 3. cAdvisor Exporter (cadvisor-exporter.json)
- **Source**: https://grafana.com/grafana/dashboards/14282
- **Author**: cAdvisor community
- **Description**: Docker container metrics from cAdvisor including CPU, memory, network, and filesystem usage per container
- **Modifications**: Datasource references updated to use the "Prometheus" datasource name

## Updating Dashboards

To update these dashboards to newer versions:

```bash
# Download latest versions
curl -o node-exporter-full.json "https://grafana.com/api/dashboards/1860/revisions/latest/download"
curl -o prometheus-stats.json "https://grafana.com/api/dashboards/11449/revisions/latest/download"
curl -o cadvisor-exporter.json "https://grafana.com/api/dashboards/14282/revisions/latest/download"
```

After downloading, you may need to update datasource references. Grafana will automatically match datasources by name ("Prometheus") when the dashboards are provisioned.

## License

These dashboards are sourced from Grafana.com and are subject to their respective licenses. Please refer to the original dashboard pages for licensing information.