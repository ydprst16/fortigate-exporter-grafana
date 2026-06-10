# FortiGate Exporter Deployment for Prometheus & Grafana

Deploy FortiGate monitoring using FortiGate Exporter, Prometheus, and Grafana.

## Overview

This setup collects FortiGate metrics through the FortiGate REST API and exposes them to Prometheus. Grafana is then used to visualize the collected data.

### Components

- FortiGate Exporter
- Prometheus
- Grafana

### Grafana Dashboard

Recommended dashboard:

- Dashboard ID: **12906**
- Dashboard: https://grafana.com/grafana/dashboards/12906-fortigate-exporter/

---

## Prerequisites

### Server Requirements

- Linux Server (Ubuntu/CentOS/RHEL)
- Go installed
- Prometheus installed
- Grafana installed
- Network access to FortiGate management interfaces

### FortiGate Requirements

- API access enabled
- REST API administrator account
- API token generated

---

## Installation

### Clone Repository

```bash
git clone https://github.com/bluecmd/fortigate_exporter.git
cd fortigate_exporter
```

### Build Binary

```bash
go build
```

### Install Binary

```bash
sudo mv fortigate_exporter /usr/local/bin/
```

---

## Configuration

### Create Configuration Directory

```bash
sudo mkdir -p /etc/fortigate_exporter
```

### Create Authentication File

```bash
sudo vim /etc/fortigate_exporter/fortigate-key.yaml
```

## Systemd Service

Create a systemd service file:

```bash
sudo vim /etc/systemd/system/fortigate_exporter.service
```

```ini
[Unit]
Description=FortiGate Exporter
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/fortigate_exporter \
  --insecure \
  --auth-file=/etc/fortigate_exporter/fortigate-key.yaml

[Install]
WantedBy=multi-user.target
```

### Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable fortigate_exporter
sudo systemctl start fortigate_exporter
```

### Verify Service

```bash
sudo systemctl status fortigate_exporter
```

---

## Prometheus Configuration

Edit Prometheus configuration:

```bash
sudo vim /etc/prometheus/prometheus.yml
```

Add the following scrape job:

```yaml
- job_name: 'fortigate_exporter'
  metrics_path: /probe

  static_configs:
    - targets:
        - https://10.10.10.4:8443

  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target

    - source_labels: [__param_target]
      target_label: instance
      regex: '(?:.+)(?::\/\/)([^:]*).*'

    - target_label: __address__
      replacement: localhost:9710
```

### Reload Prometheus

```bash
sudo systemctl restart prometheus
```

### Verify Targets

Open:

```text
http://<PROMETHEUS_SERVER>:9090/targets
```

Ensure the FortiGate target is shown as **UP**.

---

## Grafana Setup

### Import Dashboard

1. Open Grafana
2. Navigate to **Dashboards → Import**
3. Enter Dashboard ID:

```text
12906
```

4. Select your Prometheus datasource
5. Click **Import**

---

## Verify Metrics

Check exporter metrics:

```bash
curl http://localhost:9710/metrics
```

Check probe metrics:

```bash
curl "http://localhost:9710/probe?target=https://10.10.10.4:8443"
```

---

## Common Troubleshooting

### Exporter Not Starting

Check logs:

```bash
journalctl -u fortigate_exporter -f
```

### Prometheus Target Down

Verify:

- FortiGate API is reachable
- API token is valid
- Firewall rules allow access
- HTTPS port is correct

### SSL Certificate Errors

The deployment uses:

```bash
--insecure
```

for self-signed FortiGate certificates.

---

## Repository Structure

```text
.
├── README.md
├── fortigate-key.yaml.example
└── fortigate_exporter.service
```

---

## References

- https://github.com/bluecmd/fortigate_exporter
- https://grafana.com/grafana/dashboards/12906-fortigate-exporter/

## License

MIT License
