# Logstash Role

This Ansible role deploys and manages the **Logstash** service, a data processing pipeline for ingesting logs from Filebeat and forwarding them to Elasticsearch as part of the ELK Stack. It supports configuration, service management, and optional cleanup of the Logstash setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Elasticsearch service running and accessible (default port: `9200`).
- Filebeat configured to send logs to Logstash (default port: `5044`).

## Role Variables

Defined in `defaults/main.yml`:

- `logstash_version: "9.0.3"`: Version of the Logstash Docker image.
- `logstash_port: 5044`: Port for receiving logs from Filebeat.
- `elasticsearch_host: "{{ hostvars[groups['monitoring'][0]]['ansible_host'] }}"`: Elasticsearch server IP.
- `elasticsearch_port: 9200`: Elasticsearch server port.
- `elk_data_dir: "/data/elk"`: Base directory for Logstash data.
- `elk_config_dir: "/data/elk/configs"`: Base directory for configuration files.
- `logstash_flush: false`: If `true`, removes Logstash service, container, and directories.

## Role Tasks

The role performs the following tasks based on the `logstash_flush` variable:

### When `logstash_flush=true`
- Stops the Logstash service.
- Removes the Logstash Docker container.
- Deletes data (`/data/elk/logstash`) and configuration (`/data/elk/configs/logstash`) directories.
- Removes the systemd unit file (`/etc/systemd/system/logstash.service`).
- Resets failed service states and reloads systemd.

### When `logstash_flush=false`
- Installs Docker and Docker Compose.
- Creates data, configuration, and pipeline directories with appropriate permissions.
- Deploys the Logstash pipeline configuration (`logstash.conf`) from `templates/logstash.conf.j2`.
- Creates and enables the systemd unit file (`logstash.service`) from `templates/logstash.service.j2`.
- Starts the Logstash service.

## Configuration Details

- **Logstash Pipeline Configuration** (`templates/logstash.conf.j2`):
  - Configures an input plugin to receive logs from Filebeat on the specified port (`logstash_port`).
  - Configures an output plugin to send logs to Elasticsearch at `http://{{ elasticsearch_host }}:{{ elasticsearch_port }}`.
  - Indexes logs with a dynamic index name (`logs-%{+YYYY.MM.dd}`) based on the date.

- **Systemd Service** (`templates/logstash.service.j2`):
  - Runs Logstash as a Docker container in host network mode.
  - Mounts volumes for data (`/usr/share/logstash/data`) and pipeline configuration (`/usr/share/logstash/pipeline`).
  - Uses the default entrypoint (`/usr/local/bin/docker-entrypoint`) for Logstash.
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: logstash
```

## Usage

1. Ensure Elasticsearch is running and accessible at the specified `elasticsearch_host` and `elasticsearch_port` (default: monitoring server IP on port `9200`).
2. Ensure Filebeat is configured to send logs to `logstash_port` (default: `5044`).
3. Include the role in your playbook.
4. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
