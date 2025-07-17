# Elasticsearch Role

This Ansible role deploys and manages the **Elasticsearch** service, a core component of the ELK Stack for storing and indexing logs. It supports configuration, service management, and optional cleanup of the Elasticsearch setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.

## Role Variables

Defined in `defaults/main.yml`:

- `elasticsearch_version: "9.0.3"`: Version of the Elasticsearch Docker image.
- `elasticsearch_port: 9200`: Port for accessing Elasticsearch.
- `elk_data_dir: "/data/elk"`: Base directory for Elasticsearch data.
- `elk_config_dir: "/data/elk/configs"`: Base directory for configuration files.
- `elasticsearch_flush: false`: If `true`, removes Elasticsearch service, container, and directories.

## Dependencies

None.

## Role Tasks

The role performs the following tasks based on the `elasticsearch_flush` variable:

### When `elasticsearch_flush=true`
- Stops the Elasticsearch service.
- Removes the Elasticsearch Docker container.
- Deletes data (`/data/elk/elasticsearch`) and configuration (`/data/elk/configs/elasticsearch`) directories.
- Removes the systemd unit file (`/etc/systemd/system/elasticsearch.service`).
- Resets failed service states and reloads systemd.

### When `elasticsearch_flush=false`
- Installs Docker and Docker Compose.
- Creates data and configuration directories with appropriate permissions.
- Deploys the Elasticsearch configuration file (`elasticsearch.yml`) from `templates/elasticsearch.yml.j2`.
- Creates and enables the systemd unit file (`elasticsearch.service`) from `templates/elasticsearch.service.j2`.
- Starts the Elasticsearch service.

## Configuration Details

- **Elasticsearch Configuration** (`templates/elasticsearch.yml.j2`):
  - Sets the cluster name to `elasticsearch`.
  - Configures the service to listen on all network interfaces (`network.host: 0.0.0.0`).
  - Specifies the HTTP port (`http.port: {{ elasticsearch_port }}`).

- **Systemd Service** (`templates/elasticsearch.service.j2`):
  - Runs Elasticsearch as a Docker container, mapping the specified port (`9200`) and volumes for data and configuration.
  - Sets environment variables:
    - `discovery.type=single-node`: Runs Elasticsearch in single-node mode.
    - `xpack.security.enabled=false`: Disables security features for simplicity.
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: elasticsearch
```

## Usage

1. Include the role in your playbook.
2. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
3. Access Elasticsearch at `http://<monitoring_server_ip>:9200`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
