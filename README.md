# DevOpsDiploma 🌟

Welcome to the **DevOpsDiploma** project! This is a diploma-focused initiative for deploying a Java-based microservices application. It leverages **Infrastructure as Code (IaC)** with Ansible, an automated **CI/CD pipeline** with Jenkins, and robust **monitoring/logging** using Prometheus, Grafana, Alertmanager, and the ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat). The goal? Automate deployment, monitor health, and log events seamlessly! 🎉

## Project Structure 📂

- **`ansible/`**: Houses Ansible playbooks, roles, and inventory for infrastructure automation.
  - **`ansible.cfg`**: Config file (uses `hosts.yml` inventory, disables host key checking, sets `roles_path` and `remote_user` to `ubuntu`).
  - **`inventories/`**:
    - **`group_vars/all.yml`**: Central hub for global variables and role configurations.
    - **`hosts.yml`**: Inventory with server details.
  - **`playbooks/setup.yml`**: Main playbook to deploy monitoring, logging, and Jenkins.
  - **`roles/`**: Individual roles for each service (e.g., `alertmanager`, `prometheus`, `grafana`).
    - Each role includes `defaults/`, `tasks/`, `templates/`, `vars/`, and `tests/`.
- **`ci/`**: Contains `jenkins_home_prepared.tar.gz` (excluded via `.gitignore`).
- **`docs/`**: Placeholder for additional documentation.
- **`java-app/`**: Java app source code and Dockerfile.
  - `Dockerfile`: Builds the app container.
  - `pom.xml`: Maven build file.
  - `src/`: Java source and test files.
  - `target/`: Build artifacts.
- **`Jenkinsfile`**: Defines the CI/CD pipeline.
- **`README.md`**: This file—your project overview and guide! 📖
- **`vault_pass.txt`**: Ansible Vault password file (excluded via `.gitignore`).
- **`.gitignore`**: Excludes sensitive files (`roles/alertmanager/vars/secrets.yml`, `vault_pass.txt`) and local data (`ci/`).

## Setup Instructions 🛠️

### Prerequisites ✅
- Ubuntu 24.04 (or compatible Linux distro).
- Docker and Docker Compose installed.
- Java 21 and Maven installed.
- Git for version control.
- Ansible installed on the control node.

### Installation Steps 🚀
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/mmoonly/DevOpsDiploma.git
   cd DevOpsDiploma
   ```
2. **Set Up Ansible Vault**:
   - Ensure `vault_pass.txt` has the password for `ansible/roles/*/vars/secrets.yml`.
   - Run Ansible:
     ```bash
     ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml
     ```
3. **Configure Inventory**:
   - Edit `ansible/inventories/hosts.yml` with your server IPs and SSH keys.
   - Example:
     ```yaml
     all:
       children:
         jenkins:
           hosts:
             jenkins_server:
               ansible_host: 192.168.110.67
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
         app:
           hosts:
             app_server:
               ansible_host: 192.168.110.74
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
         monitoring:
           hosts:
             monitoring_server:
               ansible_host: 192.168.110.71
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
     ```
4. **Deploy Infrastructure**:
   ```bash
   ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml
   ```
5. **Set Up Jenkins CI/CD**:
   - Access Jenkins at `http://192.168.110.67:8080`.
   - Configure credentials (`docker-creds`, `jenkins-to-app-ssh`, `telegram-bot-token`, `telegram-chat-id`) in Jenkins.
   - `Jenkinsfile` triggers builds every minute via `pollSCM`.

## Role Configurations and Tasks 🛡️

| Role            | Version       | Port  | Config Dir             | Data Dir               | Tasks                                      |
|-----------------|---------------|-------|------------------------|------------------------|--------------------------------------------|
| **alertmanager** | `latest`      | 9093  | `/data/alertmanager/configs` | `/data/alertmanager/data` | Remove if `alertmanager_flush=true`; install Docker, copy configs, start service if `false`. |
| **elasticsearch** | `9.0.3`   | 9200  | `/data/elk/configs/elasticsearch` | `/data/elk/elasticsearch` | Remove if `elasticsearch_flush=true`; install Docker, copy configs, start service if `false`. |
| **filebeat**    | `9.0.3`       | -     | `/data/elk/configs/filebeat` | -                | Remove if `filebeat_flush=true`; install Docker, copy configs, start service if `false`. |
| **grafana**     | `latest`      | 3000  | `/data/grafana/configs` | `/data/grafana/data` | Remove if `grafana_flush=true`; install Docker, copy dashboards, start service if `false`. |
| **jenkins**     | `lts-jdk21`   | 8080  | `/opt/jenkins_home`    | -                | Remove if `jenkins_flush=true`; install Docker/Java/Maven, start service if `false`. |
| **kibana**      | `9.0.3`       | 5601  | `/data/elk/configs/kibana` | `/data/elk/kibana` | Remove if `kibana_flush=true`; install Docker, copy configs, start service if `false`. |
| **logstash**    | `9.0.3`       | 5044  | `/data/elk/configs/logstash` | `/data/elk/logstash` | Remove if `logstash_flush=true`; install Docker, copy configs, start service if `false`. |
| **node-exporter** | `latest` | 9100  | -                    | -                | Remove if `node_exporter_flush=true`; install Docker, start service if `false`. |
| **prometheus**  | `latest`      | 9090  | `/data/prometheus/configs` | `/data/prometheus/data` | Remove if `prometheus_flush=true`; install Docker, copy configs, start service if `false`. |

## CI/CD Pipeline ⚙️
- **Stages**:
  - `Checkout`: Clones repo and updates submodules.
  - `Lint`: Runs Checkstyle on the Java app.
  - `Build`: Builds with Maven.
  - `Test`: Runs unit tests.
  - `Archive Artifacts`: Stores `.jar` file.
  - `Docker Build & Push`: Pushes image to Docker Hub.
  - `Deploy to App Server`: Deploys to `192.168.110.74:8080`.
  - `Notify`: Sends Telegram updates.
- **Trigger**: `pollSCM('H/1 * * * *')` checks every minute.

## Monitoring and Logging 📊
- **Prometheus**: Scrapes `node-exporter` metrics.
- **Alertmanager**: Sends Telegram alerts.
- **Grafana**: Visualizes at `http://192.168.110.71:3000`.
- **ELK Stack**:
  - `Elasticsearch`: Logs at `http://192.168.110.71:9200`.
  - `Logstash`: Processes at port `5044`.
  - `Kibana`: Visualizes at `http://192.168.110.71:5601`.
  - `Filebeat`: Collects from `/var/log`.
- **Access Points**:
  - Prometheus: `http://192.168.110.71:9090`
  - Grafana: `http://192.168.110.71:3000`
  - Kibana: `http://192.168.110.71:5601`

## Tools 🛠️
- **IaC**: Ansible
- **CI/CD**: Jenkins
- **Monitoring**: Prometheus, Grafana, Alertmanager, Node Exporter
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat)
- **Application**: Java (Maven-based)

## Status 🚧
- Infrastructure deployed.
- CI/CD pipeline active.
- Monitoring/logging configured (testing needed).

## Troubleshooting 🛑
- **pollSCM not triggering**: Clear Jenkins history or push new commits.
- **Ansible Vault issues**: Check `vault_pass.txt` permissions.
- **Docker errors**: Verify `/var/run/docker.sock` access.
- **Service failures**: Ensure ports (e.g., 9090, 9200) are free.

## Contributing 🤝
Work in progress! Suggestions for roles, pipelines, or docs are welcome. Open an issue or PR on GitHub!
