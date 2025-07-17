# Filebeat Role

This Ansible role deploys and manages the **Filebeat** service, a lightweight shipper for collecting and forwarding log data to Logstash as part of the ELK Stack. It supports configuration, service management, and optional cleanup of the Filebeat setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Logstash service running and accessible (default port: `5044`).

## Role Variables

Defined in `defaults/main.yml`:

- `filebeat_version: "9.0.3"`: Version of the Filebeat Docker image.
- `elk_data_dir: "/data/elk"`: Base directory for ELK data.
- `elk_config_dir: "/data/elk/configs"`: Base directory for configuration files.
- `logstash_host: "{{ hostvars[groups['monitoring'][0]]['ansible_host'] }}:5044"`: Logstash host and port for log forwarding.
- `filebeat_flush: false`: If `true`, removes Filebeat service, container, and configuration directory.
- `filebeat_user: "root"`: User for running the Filebeat container.
- `log_volume: "/var/log:/var/log:ro"`: Mounts `/var/log` as read-only for log collection.

## Role Tasks

The role performs the following tasks based on the `filebeat_flush` variable:

### When `filebeat_flush=true`
- Stops the Filebeat service.
- Removes the Filebeat Docker container.
- Deletes the configuration directory (`/data/elk/configs/filebeat`).
- Removes the systemd unit file (`/etc/systemd/system/filebeat.service`).
- Resets failed service states and reloads systemd.

### When `filebeat_flush=false`
- Installs Docker and Docker Compose.
- Creates the configuration directory with appropriate permissions.
- Deploys the Filebeat configuration file (`filebeat.yml`) from `templates/filebeat.yml.j2`.
- Sets permissions on `/var/log` to ensure log access.
- Creates and enables the systemd unit file (`filebeat.service`) from `templates/filebeat.service.j2`.
- Starts the Filebeat service.

## Configuration Details

- **Filebeat Configuration** (`templates/filebeat.yml.j2`):
  - Configures Filebeat to collect logs from:
    - `/var/log/syslog`
    - `/var/log/auth.log`
    - `/var/log/kern.log`
    - `/var/log/messages`
  - Excludes files like `/var/log/dpkg.log`, compressed logs (`.gz`), and rotated logs (`.1`).
  - Forwards logs to the specified Logstash host (`logstash_host`).

- **Systemd Service** (`templates/filebeat.service.j2`):
  - Runs Filebeat as a Docker container in host network mode.
  - Mounts `/var/log` as read-only and the configuration file (`filebeat.yml`).
  - Runs as the specified `filebeat_user` (default: `root`).
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: filebeat
```

## Usage

1. Ensure Logstash is running and accessible at the specified `logstash_host` (default: monitoring server IP on port `5044`).
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
