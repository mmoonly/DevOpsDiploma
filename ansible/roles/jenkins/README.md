# Jenkins Role

This Ansible role deploys and manages the **Jenkins** service, a CI/CD automation server for building, testing, and deploying the Java application. It supports configuration, service management, Maven installation, and optional cleanup of the Jenkins setup.

## Requirements

- Ubuntu 24.04 (or a compatible Linux distribution).
- Docker and Docker Compose installed.
- Ansible for role execution.
- OpenJDK 21 JRE for running Jenkins and Maven.
- A prepared Jenkins home archive (`jenkins_home_prepared.tar.gz`) if `use_prepared_home` is enabled.

## Role Variables

Defined in `defaults/main.yml`:

- `jenkins_flush: false`: If `true`, removes Jenkins service, container, home directory, and Maven installation.
- `jenkins_home: "/opt/jenkins_home"`: Directory for Jenkins configuration and data.
- `jenkins_port: 8080`: Port for accessing Jenkins.
- `jenkins_image: "jenkins/jenkins:lts-jdk21"`: Jenkins Docker image and tag.
- `jenkins_memory: "2g"`: Memory limit for the Jenkins container.
- `jenkins_cpus: "2"`: CPU limit for the Jenkins container.
- `use_prepared_home: true`: If `true`, uses a pre-configured Jenkins home archive.
- `prepared_home_archive: "/opt/jenkins_home_prepared.tar.gz"`: Path to the archive on the remote host.
- `archive_on_host: "/home/ubuntu/DevOpsDiploma/ci/jenkins_home_prepared.tar.gz"`: Path to the archive on the Ansible control node.

## Role Tasks

The role performs the following tasks based on the `jenkins_flush` variable:

### When `jenkins_flush=true`
- Stops the Jenkins service.
- Removes the Jenkins Docker container.
- Deletes the Jenkins home directory (`/opt/jenkins_home`).
- Removes the prepared home archive (`/opt/jenkins_home_prepared.tar.gz`).
- Removes the systemd unit file (`/etc/systemd/system/jenkins.service`).
- Removes Maven installation (archive, directory, and symlink).
- Resets failed service states and reloads systemd.

### When `jenkins_flush=false`
- Installs OpenJDK 21 JRE.
- Installs Maven 3.9.9 by downloading and extracting the archive, creating a symlink for `mvn`.
- Creates the Jenkins home directory with appropriate permissions (owner/group: `1000`).
- If `use_prepared_home=true`, copies and extracts the prepared Jenkins home archive to `jenkins_home`.
- Creates and enables the systemd unit file (`jenkins.service`) from `templates/jenkins.service.j2`.
- Starts the Jenkins service and waits for it to be accessible on the specified port.
- Verifies the Jenkins container is running.

## Configuration Details

- **Systemd Service** (`templates/jenkins.service.j2`):
  - Runs Jenkins as a Docker container in host network mode, mapping ports `8080` (HTTP) and `50000` (agent communication).
  - Sets environment variable `TZ=Europe/Minsk` for timezone configuration.
  - Configures memory (`jenkins_memory`) and CPU (`jenkins_cpus`) limits.
  - Mounts volumes for:
    - Jenkins home directory (`/var/jenkins_home`).
    - Docker socket (`/var/run/docker.sock`) for Docker commands.
    - Maven installation (`/usr/share/maven`).
    - Docker binary (`/usr/bin/docker`).
  - Runs as user/group `1000:110` for compatibility with Jenkins Docker image.
  - Uses `oneshot` service type with `RemainAfterExit=yes` for Docker container management.

## Usage

1. If using `use_prepared_home=true` (default), ensure the `jenkins_home_prepared.tar.gz` archive is available at `/home/ubuntu/DevOpsDiploma/ci/` on the control node. See the main `README.md` in the project root for instructions on creating this archive.
2. Include the role in your playbook.
3. Run the playbook:
   ```bash
   ansible-playbook -i inventories/hosts.yml playbooks/setup.yml
   ```
4. Access Jenkins at `http://<jenkins_server_ip>:8080`.

## Author Information

This role is part of the **DevOpsDiploma** project. For feedback or contributions, open an issue or pull request on [GitHub](https://github.com/mmoonly/DevOpsDiploma).
