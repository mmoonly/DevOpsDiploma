# Grafana Role

This Ansible role deploys and manages the **Grafana** service, a visualization platform for displaying metrics from Prometheus. It supports configuration, service management, dashboard provisioning, and optional cleanup of the Grafana setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Prometheus service running and accessible (default port: `9090`).

## Role Variables

Defined in `defaults/main.yml`:

- `grafana_version: "latest"`: Version of the Grafana Docker image.
- `grafana_port: 3000`: Port for accessing Grafana.
- `grafana_config_dir: "/data/grafana/configs"`: Directory for configuration files.
- `grafana_provisioning_dir: "/data/grafana/configs/provisioning"`: Directory for provisioning configurations.
- `grafana_data_dir: "/data/grafana/data"`: Directory for persistent data.
- `grafana_flush: false`: If `true`, removes Grafana service, container, and directories.
- `prometheus_host: "{{ hostvars[groups['monitoring'][0]]['ansible_host'] }}"`: Prometheus server IP.
- `prometheus_port: 9090`: Prometheus server port.

## Dependencies

None.

## Role Tasks

The role performs the following tasks based on the `grafana_flush` variable:

### When `grafana_flush=true`
- Stops the Grafana service.
- Removes the Grafana Docker container.
- Deletes configuration (`/data/grafana/configs`) and data (`/data/grafana/data`) directories.
- Removes the systemd unit file (`/etc/systemd/system/grafana.service`).
- Resets failed service states and reloads systemd.

### When `grafana_flush=false`
- Installs Docker and Docker Compose.
- Creates configuration, provisioning, and data directories with appropriate permissions (owner/group: `472`, Grafana's default user).
- Deploys the Prometheus datasource configuration (`datasource.yml`) from `templates/datasource.yml.j2`.
- Deploys the dashboard provisioning configuration (`default.yml`) from `templates/dashboard-provisioning.yml.j2`.
- Copies the Node Exporter dashboard (`node-exporter-dashboard.json`) to the configuration directory.
- Creates and enables the systemd unit file (`grafana.service`) from `templates/grafana.service.j2`.
- Starts the Grafana service.

## Configuration Details

- **Datasource Configuration** (`templates/datasource.yml.j2`):
  - Configures a Prometheus datasource with the name `Prometheus`.
  - Sets the URL to `http://{{ prometheus_host }}:{{ prometheus_port }}`.
  - Uses proxy access and marks it as the default datasource.

- **Dashboard Provisioning** (`templates/dashboard-provisioning.yml.j2`):
  - Configures Grafana to load dashboards from the `/dashboards` directory.
  - Uses the `file` provider with organization ID `1`.

- **Node Exporter Dashboard** (`files/node-exporter-dashboard.json`):
  - Pre-configured dashboard for visualizing Node Exporter metrics, copied to `/data/grafana/configs`.

- **Systemd Service** (`templates/grafana.service.j2`):
  - Runs Grafana as a Docker container, mapping the specified port (`3000`) and volumes for provisioning, dashboards, and data.
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: grafana
```

## Usage

1. Ensure Prometheus is running and accessible at the specified `prometheus_host` and `prometheus_port` (default: monitoring server IP on port `9090`).
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
4. Access Grafana at `http://<monitoring_server_ip>:3000`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
