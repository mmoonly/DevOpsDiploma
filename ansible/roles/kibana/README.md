# Kibana Role

This Ansible role deploys and manages the **Kibana** service, a visualization tool for exploring and analyzing logs stored in Elasticsearch as part of the ELK Stack. It supports configuration, service management, and optional cleanup of the Kibana setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- Elasticsearch service running and accessible (default port: `9200`).

## Role Variables

Defined in `defaults/main.yml`:

- `kibana_version: "9.0.3"`: Version of the Kibana Docker image.
- `kibana_port: 5601`: Port for accessing Kibana.
- `elasticsearch_host: "{{ hostvars[groups['monitoring'][0]]['ansible_host'] }}"`: Elasticsearch server IP.
- `elasticsearch_port: 9200`: Elasticsearch server port.
- `elk_data_dir: "/data/elk"`: Base directory for Kibana data.
- `elk_config_dir: "/data/elk/configs"`: Base directory for configuration files.
- `kibana_flush: false`: If `true`, removes Kibana service, container, and directories.

## Dependencies

None.

## Role Tasks

The role performs the following tasks based on the `kibana_flush` variable:

### When `kibana_flush=true`
- Stops the Kibana service.
- Removes the Kibana Docker container.
- Deletes data (`/data/elk/kibana`) and configuration (`/data/elk/configs/kibana`) directories.
- Removes the systemd unit file (`/etc/systemd/system/kibana.service`).
- Resets failed service states and reloads systemd.

### When `kibana_flush=false`
- Installs Docker and Docker Compose.
- Creates data and configuration directories with appropriate permissions.
- Deploys the Kibana configuration file (`kibana.yml`) from `templates/kibana.yml.j2`.
- Creates and enables the systemd unit file (`kibana.service`) from `templates/kibana.service.j2`.
- Starts the Kibana service.

## Configuration Details

- **Kibana Configuration** (`templates/kibana.yml.j2`):
  - Sets the server port (`server.port: {{ kibana_port }}`).
  - Configures Kibana to listen on all network interfaces (`server.host: "0.0.0.0"`).
  - Specifies the Elasticsearch host (`elasticsearch.hosts: ["http://{{ elasticsearch_host }}:{{ elasticsearch_port }}"]`).

- **Systemd Service** (`templates/kibana.service.j2`):
  - Runs Kibana as a Docker container, mapping the specified port (`5601`) and volumes for data and configuration.
  - Ensures the service restarts automatically and depends on Docker.

## Example Playbook

```yaml
- hosts: monitoring
  roles:
    - role: kibana
```

## Usage

1. Ensure Elasticsearch is running and accessible at the specified `elasticsearch_host` and `elasticsearch_port` (default: monitoring server IP on port `9200`).
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
4. Access Kibana at `http://<monitoring_server_ip>:5601`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
