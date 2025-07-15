# DevOpsDiploma 🌟 — Welcome to My DevOps Playground!

Hey there, fellow code wrangler! This is **DevOpsDiploma**—my diploma project where I’ve tamed a Java microservices app with some serious automation mojo. I’ve thrown in **IaC** with Ansible, rigged up a slick **CI/CD pipeline** with Jenkins, and stuffed it with monitoring/logging goodness using Prometheus, Grafana, Alertmanager, and the ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat).

## What’s Inside? 📦

- **`ansible/`**: My command center with playbooks, roles, and inventory for taming the infra beast.
  - **`ansible.cfg`**: Config file where I ditched key checks and set `ubuntu` as my trusty sidekick.
  - **`inventories/`**:
    - **`group_vars/all.yml`**: Where all my variables chill like one big happy family.
    - **`hosts.yml`**: The roster of servers I’m trying to keep in line.
  - **`playbooks/setup.yml`**: The main playbook that kicks off this wild ride.
  - **`roles/`**: Roles for each service (from `alertmanager` to `prometheus`)—each packed with `tasks/`, `templates/`, and more goodies.
- **`ci/`**: Home to `jenkins_home_prepared.tar.gz`—hidden from Git thanks to `.gitignore`!
- **`java-app/`**: The Java app code and Dockerfile for container magic.
  - `Dockerfile`: The recipe to brew this chaos into a container.
  - `pom.xml`: Maven file to keep the compiler happy.
  - `src/`: Where I code and test like a boss.
  - `target/`: A pile of build goodies.
- **`Jenkinsfile`**: The heartbeat of my CI/CD pipeline.
- **`README.md`**: You’re reading it now—my guide to this madness! 📖
- **`vault_pass.txt`**: Ansible Vault password, locked away from prying eyes.
- **`.gitignore`**: My shield against leaks—hiding secrets and `ci/`.

## How to Kick It Off? 🛠️

### What You Need ✅
- Ubuntu 24.04 (or something close enough to work).
- Docker and Docker Compose—non-negotiable!
- Java 21 and Maven—for that Java wizardry.
- Git—to snag my code.
- Ansible—the maestro of this orchestra.

### Installation Steps 🚀
1. **Grab the Repo**:
   ```bash
   git clone https://github.com/mmoonly/DevOpsDiploma.git
   cd DevOpsDiploma
   ```
2. **Set Up Ansible Vault**:
   - Make sure `vault_pass.txt` has the password for `ansible/roles/*/vars/secrets.yml`.
   - Fire it up:
     ```bash
     ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml
     ```
3. **Tweak the Inventory**:
   - Edit `ansible/inventories/hosts.yml` with your server IPs and SSH keys.
   - Example:
     ```yaml
     all:
       children:
         jenkins:
           hosts:
             jenkins_server:
               ansible_host: 192.168.100.15
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
         app:
           hosts:
             app_server:
               ansible_host: 192.168.100.13
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
         monitoring:
           hosts:
             monitoring_server:
               ansible_host: 192.168.100.14
               ansible_user: ubuntu
               ansible_ssh_private_key_file: ~/.ssh/id_rsa
     ```
4. **Deploy the Beast**:
   ```bash
   ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml
   ```
5. **Boot Up Jenkins CI/CD**:
   - Hit `http://192.168.100.15:8080` to check it out.
   - Set up creds (`docker-creds`, `jenkins-to-app-ssh`, `telegram-bot-token`, `telegram-chat-id`) in Jenkins.
   - `Jenkinsfile` will ping the code every minute with `pollSCM`.

## Roles and Their Moves 🛡️

| Role            | Version       | Port  | Config Folder          | Data Folder            | What It Does                              |
|-----------------|---------------|-------|------------------------|------------------------|-------------------------------------------|
| **alertmanager** | `latest`      | 9093  | `/data/alertmanager/configs` | `/data/alertmanager/data` | Nukes it if `alertmanager_flush=true`; sets up Docker and configs if `false`. |
| **elasticsearch** | `9.0.3`   | 9200  | `/data/elk/configs/elasticsearch` | `/data/elk/elasticsearch` | Wipes out if `elasticsearch_flush=true`; deploys and runs if `false`. |
| **filebeat**    | `9.0.3`       | -     | `/data/elk/configs/filebeat` | -                | Clears logs if `filebeat_flush=true`; sets up and collects if `false`. |
| **grafana**     | `latest`      | 3000  | `/data/grafana/configs` | `/data/grafana/data` | Tears down if `grafana_flush=true`; adds dashboards and rolls if `false`. |
| **jenkins**     | `lts-jdk21`   | 8080  | `/opt/jenkins_home`    | -                | Blows away if `jenkins_flush=true`; sets up Java/Maven and fires up if `false`. |
| **kibana**      | `9.0.3`       | 5601  | `/data/elk/configs/kibana` | `/data/elk/kibana` | Cleans up if `kibana_flush=true`; deploys and visualizes if `false`. |
| **logstash**    | `9.0.3`       | 5044  | `/data/elk/configs/logstash` | `/data/elk/logstash` | Deletes if `logstash_flush=true`; sets up and filters if `false`. |
| **node-exporter** | `latest` | 9100  | -                    | -                | Dumps if `node_exporter_flush=true`; spins up metrics if `false`. |
| **prometheus**  | `latest`      | 9090  | `/data/prometheus/configs` | `/data/prometheus/data` | Erases if `prometheus_flush=true`; deploys and monitors if `false`. |

## CI/CD Pipeline ⚙️
- **Stages**:
  - `Checkout`: Grabs the code and updates submodules.
  - `Lint`: Runs Checkstyle to keep the code pretty.
  - `Build`: Compiles with Maven like a pro.
  - `Test`: Hammers the tests.
  - `Archive Artifacts`: Packs up the `.jar`.
  - `Docker Build & Push`: Ships the image to Docker Hub.
  - `Deploy to App Server`: Drops it on `192.168.100.13:8080`.
  - `Notify`: Pings Telegram with updates.
- **Trigger**: `pollSCM('H/1 * * * *')`—checks for changes every minute.

## Monitoring and Logging 📊
- **Prometheus**: Sucks up metrics from `node-exporter`.
- **Alertmanager**: Blasts alerts to Telegram (don’t sleep through them!).
- **Grafana**: Sweet visuals at `http://192.168.100.14:3000`.
- **ELK Stack**:
  - `Elasticsearch`: Stores logs at `http://192.168.100.14:9200`.
  - `Logstash`: Filters on port `5044`.
  - `Kibana`: Paints logs at `http://192.168.100.14:5601`.
  - `Filebeat`: Scoops logs from `/var/log`.
- **Where to Peek**:
  - Prometheus: `http://192.168.100.14:9090`
  - Grafana: `http://192.168.100.14:3000`
  - Kibana: `http://192.168.100.14:5601`

## Tools 🛠️
- **IaC**: Ansible—my trusty sidekick.
- **CI/CD**: Jenkins—automation king.
- **Monitoring**: Prometheus, Grafana, Alertmanager, Node Exporter.
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat).
- **App**: Java—crafted with love on Maven.

## Status 🚧
- Infra’s up, but not perfect yet.
- CI/CD is live and kicking.
- Monitoring/logging’s in place—time to test it out!

## Let’s Collaborate! 🤝
This is a work in progress! Got ideas for roles, pipelines, or docs? Drop an issue or PR on GitHub—I’m all ears! 😄
