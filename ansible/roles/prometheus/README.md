# Prometheus Monitoring Stack

This role deploys a complete Prometheus monitoring stack for your homelab, including:

- **Prometheus**: Metrics collection and time-series database
- **Alertmanager**: Alert routing and notification management
- **Node Exporter**: System/hardware metrics
- **cAdvisor**: Container resource usage metrics

## Quick Start

Deploy the entire monitoring stack:

```bash
cd ansible && ansible-playbook playbooks/site.yml --tags monitoring --ask-vault-pass
```

## Services

### Prometheus (Port 9091)
- Metrics collection and querying
- Access: http://192.168.1.36:9091
- Configuration: `/opt/docker-data/prometheus/config/prometheus.yml`

### Alertmanager (Port 9093)
- Alert notification management
- Access: http://192.168.1.36:9093
- Configuration: `/opt/docker-data/prometheus/config/alertmanager.yml`

### cAdvisor (Port 8081)
- Container performance metrics
- Access: http://192.168.1.36:8081

### Node Exporter
- System metrics (CPU, memory, disk, network)
- Internal only (accessed by Prometheus via Docker network)

## Configuration

### Port Configuration

Default ports can be changed in `defaults/main.yml`:

```yaml
prometheus_port: 9091  # Note: 9090 conflicts with Cockpit
alertmanager_port: 9093
cadvisor_port: 8081
```

### Email Alerts with Gmail

To configure Gmail notifications, add these to your `vault.yml`:

```yaml
# 1. Enable 2FA on your Google account
# 2. Generate App Password: https://myaccount.google.com/apppasswords
# 3. Add to vault.yml:

alertmanager_email_from: "your-email@gmail.com"
alertmanager_email_to: "recipient@example.com"
alertmanager_smtp_smarthost: "smtp.gmail.com:587"
alertmanager_smtp_auth_username: "your-email@gmail.com"
alertmanager_smtp_auth_password: "your-app-password"
alertmanager_smtp_require_tls: true
```

Deploy with updated config:

```bash
cd ansible && ansible-playbook playbooks/site.yml --tags monitoring --ask-vault-pass
```

### Slack Alerts (Optional)

Add to `group_vars/all.yml` or `vault.yml`:

```yaml
alertmanager_slack_webhook: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
alertmanager_slack_channel: "#homelab-alerts"
```

## Metrics Collection

Prometheus automatically scrapes:

1. **Itself** - Prometheus server metrics
2. **Node Exporter** - System metrics (CPU, memory, disk, network)
3. **cAdvisor** - Docker container metrics
4. **Docker Daemon** - Docker engine metrics (if enabled)

## Pre-configured Alerts

The stack includes alerts for common issues:

- **InstanceDown**: Service unavailable for 5+ minutes
- **HighCPUUsage**: CPU > 80% for 10+ minutes
- **HighMemoryUsage**: Memory > 85% for 10+ minutes
- **DiskSpaceLow**: Disk space < 15%
- **ContainerDown**: Container stopped for 5+ minutes
- **ContainerHighCPU**: Container using > 80% CPU
- **ContainerHighMemory**: Container using > 90% of memory limit

Edit alert rules in: `/opt/docker-data/prometheus/config/alerts.yml`

## Useful Commands

```bash
# Check Prometheus status
ansible homelab-1 -m shell -a "docker ps | grep prometheus"

# View Prometheus logs
ansible homelab-1 -m shell -a "docker logs prometheus"

# View Alertmanager logs
ansible homelab-1 -m shell -a "docker logs alertmanager"

# Restart monitoring stack
ansible homelab-1 -m shell -a "docker restart prometheus alertmanager node-exporter cadvisor"

# Check current alerts
curl http://192.168.1.36:9091/api/v1/alerts

# Reload Prometheus config (without restart)
curl -X POST http://192.168.1.36:9091/-/reload
```

## Data Retention

Prometheus keeps metrics for **15 days** by default. To change:

Edit `ansible/roles/prometheus/tasks/main.yml`:

```yaml
command:
  - '--storage.tsdb.retention.time=30d'  # Change to desired retention
```

## Troubleshooting

### Prometheus not scraping targets

Check target status in Prometheus UI: http://192.168.1.36:9091/targets

Common issues:
- Docker network not connected
- Container not running
- Firewall blocking access

### Alerts not sending

1. Check Alertmanager logs: `docker logs alertmanager`
2. Verify SMTP settings in `/opt/docker-data/alertmanager/config/alertmanager.yml`
3. Test alert: Go to Prometheus UI > Alerts and trigger a test alert

### High memory usage
xxw
Prometheus stores data in memory. If memory usage is high:
- Reduce retention time
- Decrease scrape frequency in `prometheus.yml`
- Reduce number of metrics being collected

## Integration with Grafana

While Grafana is not included in this stack, you can easily add it:

1. Add Prometheus as a data source: http://prometheus:9090
2. Import popular dashboards:
   - Node Exporter Full: Dashboard ID 1860
   - Docker Container & Host Metrics: Dashboard ID 179
   - Prometheus Stats: Dashboard ID 2