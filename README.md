DevOpsDiploma
DevOps diploma project for deploying a Java-based microservices application. This project implements Infrastructure as Code (IaC) with Ansible, an automated CI/CD pipeline with Jenkins, and monitoring/logging using Prometheus, Grafana, Alertmanager, and the ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat). The goal is to automate deployment, monitor application health, and log system events for a diploma submission.
Project Structure

ansible/: Contains Ansible playbooks, roles, and inventory for infrastructure automation.
ansible.cfg: Configuration file for Ansible (uses hosts.yml inventory, disables host key checking, sets roles_path and remote_user to ubuntu).
inventories/: Host and group variable definitions.
group_vars/all.yml: Global variables, including flush flags and centralized role configurations.
hosts.yml: Inventory file with server details.


playbooks/setup.yml: Main playbook to deploy monitoring, logging, and Jenkins infrastructure.
roles/: Individual roles for each service.
alertmanager/: Configures Alertmanager for alerting.
elasticsearch/: Sets up Elasticsearch for log storage.
filebeat/: Collects logs from servers.
grafana/: Visualizes metrics with pre-configured dashboards.
jenkins/: Deploys Jenkins with prepared home.
kibana/: Visualizes logs from Elasticsearch.
logstash/: Processes and forwards logs.
node-exporter/: Exports node metrics.
prometheus/: Scrapes and stores metrics.
Each role includes defaults/, tasks/, templates/, vars/, and tests/ for configuration and testing.




ci/: Contains prepared Jenkins home configuration (jenkins_home_prepared.tar.gz), excluded via .gitignore.
docs/: Placeholder for additional documentation (to be expanded).
java-app/: Source code and Dockerfile for the Java application.
Dockerfile: Builds the Java app container.
pom.xml: Maven build file.
src/: Java source and test files.
target/: Build artifacts.


Jenkinsfile: Defines the CI/CD pipeline for building, testing, and deploying the Java app.
README.md: This file, providing project overview and setup instructions.
vault_pass.txt: Ansible Vault password file, excluded via .gitignore.
.gitignore: Excludes sensitive files (roles/alertmanager/vars/secrets.yml, vault_pass.txt) and local data (ci/).

Setup Instructions
Prerequisites

Ubuntu 24.04 (or compatible Linux distribution).
Docker and Docker Compose installed.
Java 21 and Maven installed.
Git for version control.
Ansible installed on the control node.

Installation Steps

Clone the Repository:
git clone https://github.com/mmoonly/DevOpsDiploma.git
cd DevOpsDiploma


Set Up Ansible Vault:

Ensure vault_pass.txt contains the password for ansible/roles/*/vars/secrets.yml.
Run Ansible with the password file:ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml




Configure Inventory:

Update ansible/inventories/hosts.yml with your server IPs and SSH keys.
Example:all:
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




Deploy Infrastructure:

Run the playbook to set up monitoring, logging, and Jenkins:ANSIBLE_VAULT_PASSWORD_FILE=vault_pass.txt ansible-playbook -i ansible/inventories/hosts.yml ansible/playbooks/setup.yml




Set Up Jenkins CI/CD:

Access Jenkins at http://192.168.110.67:8080 (default port).
Configure credentials (docker-creds, jenkins-to-app-ssh, telegram-bot-token, telegram-chat-id) in Jenkins.
The Jenkinsfile will trigger builds automatically via pollSCM every minute.



Role Configurations and Tasks

alertmanager:
Version: latest
Port: 9093
Config dir: /data/alertmanager/configs
Data dir: /data/alertmanager/data
Tasks:
Removes service and data if alertmanager_flush=true.
Installs Docker, creates directories, copies alertmanager.yml.j2 and telegram.tmpl, sets up alertmanager.service.j2, and starts the service if alertmanager_flush=false.




elasticsearch:
Version: 9.0.3
Port: 9200
Data dir: /data/elk/elasticsearch
Tasks:
Removes service and data if elasticsearch_flush=true.
Installs Docker, creates directories, copies elasticsearch.yml.j2, sets up elasticsearch.service.j2, and starts the service if elasticsearch_flush=false.




filebeat:
Version: 9.0.3
Log volume: /var/log:/var/log:ro
Output: Logstash at monitoring_server:5044
Tasks:
Removes service and data if filebeat_flush=true.
Installs Docker, creates directory, copies filebeat.yml.j2, sets up filebeat.service.j2, and starts the service if filebeat_flush=false.




grafana:
Version: latest
Port: 3000
Config dirs: /data/grafana/configs, /data/grafana/data
Datasource: Prometheus at monitoring_server:9090
Tasks:
Removes service and data if grafana_flush=true.
Installs Docker, creates directories, copies datasource.yml.j2, dashboard-provisioning.yml.j2, and node-exporter-dashboard.json, sets up grafana.service.j2, and starts the service if grafana_flush=false.




jenkins:
Port: 8080
Image: jenkins/jenkins:lts-jdk21
Home: /opt/jenkins_home
Tasks:
Removes service, data, and Maven if jenkins_flush=true.
Installs Docker, Java, Maven, creates home directory, extracts jenkins_home_prepared.tar.gz (if use_prepared_home=true), sets up jenkins.service.j2, and starts the service if jenkins_flush=false.




kibana:
Version: 9.0.3
Port: 5601
Connects to Elasticsearch at monitoring_server:9200
Tasks:
Removes service and data if kibana_flush=true.
Installs Docker, creates directories, copies kibana.yml.j2, sets up kibana.service.j2, and starts the service if kibana_flush=false.




logstash:
Version: 9.0.3
Port: 5044
Outputs to Elasticsearch at monitoring_server:9200
Tasks:
Removes service and data if logstash_flush=true.
Installs Docker, creates directories, copies logstash.conf.j2, sets up logstash.service.j2, and starts the service if logstash_flush=false.




node-exporter:
Version: latest
Port: 9100
Tasks:
Removes service if node_exporter_flush=true.
Installs Docker, sets up node-exporter.service.j2, and starts the service if node_exporter_flush=false.




prometheus:
Version: latest
Port: 9090
Config dir: /data/prometheus/configs
Data dir: /data/prometheus/data
Tasks:
Removes service and data if prometheus_flush=true.
Installs Docker, creates directories, copies prometheus.yml.j2 and alert.rules.yml, sets up prometheus.service.j2, and starts the service if prometheus_flush=false.





CI/CD Pipeline

Stages:
Checkout: Clones the repo and updates submodules.
Lint: Runs Checkstyle on the Java app.
Build: Builds the Java app with Maven.
Test: Runs unit tests.
Archive Artifacts: Stores the .jar file.
Docker Build & Push: Builds and pushes the Docker image to Docker Hub.
Deploy to App Server: Deploys the app to 192.168.110.74:8080.
Notify: Sends Telegram notifications on build status.


Trigger: pollSCM('H/1 * * * *') checks for changes every minute.

Monitoring and Logging

Prometheus: Scrapes metrics from node-exporter on all hosts.
Alertmanager: Handles alerts and sends Telegram notifications.
Grafana: Visualizes metrics at http://192.168.110.71:3000.
ELK Stack:
Elasticsearch: Stores logs at http://192.168.110.71:9200.
Logstash: Processes logs at port 5044.
Kibana: Visualizes logs at http://192.168.110.71:5601.
Filebeat: Collects logs from /var/log on all servers.


Access points:
Prometheus: http://192.168.110.71:9090
Grafana: http://192.168.110.71:3000
Kibana: http://192.168.110.71:5601



Tools

IaC: Ansible
CI/CD: Jenkins
Monitoring: Prometheus, Grafana, Alertmanager, Node Exporter
Logging: ELK Stack (Elasticsearch, Logstash, Kibana, Filebeat)
Application: Java (Maven-based)

Status

Initial infrastructure deployed.
CI/CD pipeline functional with manual and automatic triggers.
Monitoring and logging partially configured (to be tested and optimized).

Troubleshooting

pollSCM not triggering: Clear Jenkins build history or ensure new commits are pushed.
Ansible Vault issues: Verify vault_pass.txt and permissions.
Docker errors: Check /var/run/docker.sock accessibility.
Service failures: Ensure ports (e.g., 9090, 9200) are not in use.

Contributing
Work in progress. Suggestions for improving roles, pipelines, or documentation are welcome.
