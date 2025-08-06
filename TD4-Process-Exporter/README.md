# TD4 - Process Exporter on Debian VM

This guide will walk you through setting up a Process Exporter on a Debian VM and integrating it with Prometheus and Grafana. Process Exporter allows you to monitor system processes and visualize their resource usage through pre-configured Grafana dashboards.

## Introduction to Process Exporter

Process Exporter is a Prometheus exporter that collects metrics about running processes. It provides detailed information about:

- CPU and memory usage per process
- Process runtime and state
- File descriptors and thread count
- Disk I/O rates
- And more

This makes it perfect for monitoring system processes and applications in a production environment.

## 1 - Installing Process Exporter

Download the latest release of Process Exporter and install it:

```bash
sudo apt install -y process-exporter

# Verify installation
process-exporter --version
```

## 2 - Configuring Process Exporter

Create a configuration file to specify which processes to monitor:

```bash
# Create configuration directory
sudo mkdir -p /etc/process-exporter

# Create configuration file
sudo nano /etc/process-exporter/config.yml
```

Add the following content to the configuration file:

```yaml
process_names:
  - name: "{{.Comm}}"
    cmdline:
    - '.+'
```

This configuration will monitor all processes.

## 3 - Creating a Systemd Service

Create a systemd service file to manage the Process Exporter:

```bash
# Create service file
sudo nano /etc/systemd/system/process-exporter.service
```

Add the following content:

```
[Unit]
Description=Process Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/bin/process-exporter --config.path=/etc/process-exporter/config.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

## 4 - Starting the Service

Enable and start the Process Exporter service:

```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable and start the service
sudo systemctl enable process-exporter
sudo systemctl start process-exporter

# Check service status
sudo systemctl status process-exporter
```

Verify that the exporter is working by querying its metrics endpoint:

```bash
curl http://localhost:9256/metrics | grep process
```

## 5 - Configuring Prometheus

Add the Process Exporter as a scrape target in your Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'process-exporter'
    static_configs:
      - targets: ['localhost:9256']
```

Reload Prometheus configuration:

```bash
curl -X POST http://localhost:9090/-/reload
```

## 6 - Importing the Grafana Dashboard

Process Exporter has ready-made dashboards available on Grafana.com. To import one:

1. Log into your Grafana instance
2. Navigate to "Dashboards" > "Import"
3. Enter the dashboard ID: 249 (Process Exporter Full)
4. Select your Prometheus data source
5. Click "Import"

## 7 - Dashboard Features

The imported dashboard provides various visualizations:

- System overview (CPU and memory load)
- Per-process metrics
- Resource usage history
- I/O statistics

You can:

- Filter by process name
- Select time ranges
- Configure alerts
- Customize visualizations

## 8 - Setting Up Alerts

To set up alerts:

1. Select a panel
2. Click "Edit" > "Alert"
3. Define conditions:
   - e.g., WHEN min() OF query(A, 5m, now) IS ABOVE 90
4. Configure notifications:
   - Alert message
   - Contact point (email, Slack, etc.)
   - Delay and repeat intervals

## Troubleshooting

If you encounter issues:

- Service not starting: `sudo systemctl restart process-exporter`
- Prometheus not scraping: check firewall and configuration
- Missing metrics: verify config.yml and filters
- No data in Grafana: check data source configuration
