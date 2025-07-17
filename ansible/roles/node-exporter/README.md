# Node Exporter Role

This Ansible role deploys and manages the **Prometheus Node Exporter** service, which collects system metrics (e.g., CPU, memory, disk usage) for monitoring by Prometheus. It supports service management and optional cleanup of the Node Exporter setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Prometheus service configured to scrape metrics from Node Exporter (default port: `9100`).

## Role Variables

Defined in `defaults/main.yml`:

- `node_exporter_version: "latest"`: Version of the Node Exporter Docker image.
- `node_exporter_port: 9100`: Port for accessing Node Exporter metrics.
- `node_exporter_flush: false`: If `true`, removes Node Exporter service and container.

## Dependencies

None.

## Role Tasks

The role performs the following tasks based on the `node_exporter_flush` variable:

### When `node_exporter_flush=true`
- Stops the Node Exporter service.
- Removes the Node Exporter Docker container.
- Removes the systemd unit file (`/etc/systemd/system/node-exporter.service`).
- Resets failed service states and reloads systemd.

### When `node_exporter_flush=false`
- Installs Docker and Docker Compose.
- Creates and enables the systemd unit file (`node-exporter.service`) from `templates/node-exporter.service.j2`.
- Starts the Node Exporter service.

## Configuration Details

- **Systemd Service** (`templates/node-exporter.service.j2`):
  - Runs Node Exporter as a Docker container, mapping the specified port (`9100`).
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: node-exporter
```

## Usage

1. Ensure Prometheus is configured to scrape metrics from `node_exporter_port` (default: `9100`).
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
4. Access Node Exporter metrics at `http://<monitoring_server_ip>:9100/metrics`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
