# Prometheus Role

This Ansible role deploys and manages the **Prometheus** service, a monitoring system for collecting and querying metrics from Node Exporter and triggering alerts via Alertmanager. It supports configuration, service management, and optional cleanup of the Prometheus setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Node Exporter service running and accessible (default port: `9100`).
- Alertmanager service running and accessible (default port: `9093`).

## Role Variables

Defined in `defaults/main.yml`:

- `prometheus_version: "latest"`: Version of the Prometheus Docker image.
- `prometheus_port: 9090`: Port for accessing Prometheus.
- `prometheus_config_dir: "/data/prometheus/configs"`: Directory for configuration files.
- `prometheus_data_dir: "/data/prometheus/data"`: Directory for persistent data.
- `prometheus_flush: false`: If `true`, removes Prometheus service, container, and directories.
- `alertmanager_port: 9093`: Port for Alertmanager integration.
- `node_exporter_port: 9100`: Port for Node Exporter metrics.

## Dependencies

None.

## Role Tasks

The role performs the following tasks based on the `prometheus_flush` variable:

### When `prometheus_flush=true`
- Stops the Prometheus service.
- Removes the Prometheus Docker container.
- Deletes configuration (`/data/prometheus/configs`) and data (`/data/prometheus/data`) directories.
- Removes the systemd unit file (`/etc/systemd/system/prometheus.service`).
- Resets failed service states and reloads systemd.

### When `prometheus_flush=false`
- Installs Docker and Docker Compose.
- Creates configuration and data directories with appropriate permissions.
- Deploys the Prometheus configuration file (`prometheus.yml`) from `templates/prometheus.yml.j2`.
- Copies the alert rules (`alert.rules.yml`) from `files/alert.rules.yml`.
- Creates and enables the systemd unit file (`prometheus.service`) from `templates/prometheus.service.j2`.
- Starts the Prometheus service.

## Configuration Details

- **Prometheus Configuration** (`templates/prometheus.yml.j2`):
  - Sets global scrape and evaluation intervals to `15s`.
  - Configures Alertmanager integration with targets from the `monitoring` group on `alertmanager_port`.
  - Specifies alert rules from `alert.rules.yml`.
  - Defines scrape jobs for:
    - Prometheus itself (`prometheus` job, targeting its own port).
    - Node Exporter (`node-exporter` job, targeting all hosts in the `all` group on `node_exporter_port`).

- **Alert Rules** (`files/alert.rules.yml`):
  - Defines rules for:
    - `InstanceDown`: Triggers if a target is down for over 1 minute (critical).
    - `HighCPUUsage`: Triggers if CPU usage exceeds 80% for 5 minutes (warning).
    - `LowMemory`: Triggers if available memory is below 10% for 5 minutes (critical).
    - `LowDiskSpace`: Triggers if available disk space is below 10% for 5 minutes (critical).

- **Systemd Service** (`templates/prometheus.service.j2`):
  - Runs Prometheus as a Docker container, mapping the specified port (`9090`) and volumes for configuration and data.
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: prometheus
```

## Usage

1. Ensure Node Exporter and Alertmanager are running and accessible on their respective ports (`9100` and `9093`).
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
4. Access Prometheus at `http://<monitoring_server_ip>:9090`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
