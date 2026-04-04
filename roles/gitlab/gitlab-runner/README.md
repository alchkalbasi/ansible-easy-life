<p align="right">
  <img src="../images/gitlab-logo.svg" alt="GitLab Logo" title="GitLab" height="70" />
</p>

# Ansible Role: GitLab-Runner Deployment

## Description

This Ansible role automates the deployment and configuration of a GitLab Runner. The deployed runner handles CI/CD jobs efficiently within a stable, isolated execution environment. It supports Docker‑based builds, caching, parallel jobs, and automatic cleanup. It is optimized for reliability, fast pipelines, and scalable workloads across multiple projects.

<p align="center">
  <img src="../images/gitlab-runner.svg" alt="GitLab Runner Architecture" title="GitLab Runner" />
</p>

## Requirements

- **CPU:** At least 2 Cores.
- **RAM:** Minimum 2 GB.
- **Storage:** Sufficient disk space for Docker images, build caches, and artifacts.
- **Security:** A hardened server environment is highly recommended.
- **Docker Engine:** Installed and running on the target host.
- **Docker Compose V2:** The CLI plugin must be installed (this role utilizes the `docker_compose_v2` Ansible module).

## Runner Setup & Architecture

To use the Docker executor, the GitLab Runner requires access to the host's `docker.sock` (`/var/run/docker.sock`). This allows the runner to spawn new, isolated containers for each CI/CD job.

### Runner Configuration (`config.toml`)

The runner is primarily configured through a `config.toml` file. 

Depending on your environment, the `config.toml` file is located at:
- `/etc/gitlab-runner/` on unix systems (when executed as `root`). This is also the path for the service configuration.
- `~/.gitlab-runner/` on unix systems (when executed as a non-root user).
- `./` on other systems.

**Note on Reloading:** GitLab Runner does not require a restart for most configuration changes. It checks for modifications every 3 seconds and reloads automatically if necessary. It also reloads in response to a `SIGHUP` signal. Changes to the `[[runners]]` section and most global parameters (except `listen_address`) take effect immediately.

**Example Base Configuration:**

```toml
concurrent = 1
check_interval = 0
connection_max_age = "20m0s"
shutdown_timeout = 0

[session_server]
  session_timeout = 2800
```

### Connecting the Runner to GitLab

This role handles the connection to GitLab automatically via a `docker exec` command. Under the hood, it executes a single, non-interactive registration command similar to this:

```bash
gitlab-runner register \
  --non-interactive \
  --url "https://git.mecan.ir/" \
  --token "$RUNNER_TOKEN" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "docker-runner"
```

*If a runner is already registered, the configuration is saved, and you do not need to register it again.*
