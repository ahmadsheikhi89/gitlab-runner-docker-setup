
![Banner](https://raw.githubusercontent.com/ahmadsheikhi89/gitlab-runner-docker-setup/main/banner.png)

[![GitHub license](https://img.shields.io/github/license/ahmadsheikhi89/gitlab-runner-docker-setup?style=for-the-badge)](https://github.com/ahmadsheikhi89/gitlab-runner-docker-setup/blob/main/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/ahmadsheikhi89/gitlab-runner-docker-setup?style=for-the-badge)](https://github.com/ahmadsheikhi89/gitlab-runner-docker-setup/stargazers)

# GitLab & GitLab Runner Setup with Docker Compose | Complete Local GitLab CI/CD

This guide explains how to set up a self-hosted GitLab instance and GitLab Runner using Docker, and configure a working CI/CD pipeline — just like the one we used to test Docker execution.

## 🚀 Prerequisites

Before getting started, make sure you have:

- ✅ Docker and Docker Compose installed
- ✅ A Linux system (or WSL2) with sudo access
- ✅ Open ports: `8080` (GitLab web), `2222` (Git SSH), `2375` (Docker socket if DinD)
- ✅ Some basic knowledge of Git, Docker, and YAML

## 📁 Directory Structure

```bash
project-root/
├── .gitignore                 # Ignore sensitive or unneeded files (e.g. logs, volumes)
├── docker-compose.yml        # GitLab main instance config
├── gitlab-runner/
│   ├── docker-compose.yml     # Runner service definition
│   └── config/config.toml     # Runner configuration file
└── .gitlab-ci.yml            # Main pipeline config
```

## 🐙 Step 1: Run GitLab via Docker Compose

Create `docker-compose.yml` in your root folder:

```yaml
version: '3'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    restart: always
    ports:
      - "8080:80"
      - "2222:22"
    volumes:
      - ./config:/etc/gitlab
      - ./logs:/var/log/gitlab
      - ./data:/var/opt/gitlab
```

Then start it:

```bash
docker compose up -d
# Optional: Verify the container is running and healthy
docker ps
```

Access GitLab:

- 🌐 Web UI: [http://localhost:8080](http://localhost:8080)
- 🗝️ Default root password: run the following command:
  ```bash
  docker exec -it gitlab cat /etc/gitlab/initial_root_password
  ```

## 🚀 Step 2: Start GitLab Runner

Inside `gitlab-runner/docker-compose.yml`:

```yaml
services:
  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    restart: always
    privileged: true
    volumes:
      - ./config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
```

Start the Runner:

```bash
cd gitlab-runner
docker compose up -d
```

## 🔑 Step 3: Register the Runner

1. Get the registration token from GitLab by running the following inside the GitLab container:

   ```bash
   docker exec -it gitlab bash
   gitlab-rails runner "puts Gitlab::CurrentSettings.current_application_settings.runners_registration_token"  # Run this inside the GitLab container with root privileges
   ```

   You should see a token like: `ABC12345xyz`. Copy this value.

2. Register the runner using this token:

   ```bash
   docker exec -it gitlab-runner gitlab-runner register
   ```

   Then follow the interactive prompts:

   - GitLab URL: `http://host.docker.internal:8080` (or your host IP)
   - Token: (paste the token you copied)
   - Description: `my-runner`
   - Tags: `docker,linux`
   - Executor: `shell` or `docker`
   - Docker image (if docker): `docker:latest`

   ✅ Once completed, you should see a confirmation that the runner was registered successfully.

## 🗂️ Backup and Restore GitLab

#### 1. **Backup GitLab Data**

To back up all your GitLab data (repositories, databases, and configurations), run the following command inside your GitLab container:

```bash
sudo docker exec -it gitlab bash
gitlab-rake gitlab:backup:create
```

This will create a backup in `/var/opt/gitlab/backups` inside the container. You can then copy these files to your local system for storage.

To transfer the backup to your local machine:

```bash
sudo docker cp gitlab:/var/opt/gitlab/backups /path/to/your/local/storage
```

#### 2. **Backup GitLab Configuration**

To back up the GitLab configuration files (e.g., SSH keys, and GitLab settings):

```bash
sudo docker cp gitlab:/etc/gitlab /path/to/your/local/storage
```

#### 3. **Restore GitLab Backup**

To restore from a backup, use the following command inside the GitLab container, replacing `<timestamp_of_backup>` with the appropriate backup timestamp:

```bash
sudo docker exec -it gitlab bash
gitlab-rake gitlab:backup:restore BACKUP=<timestamp_of_backup>
```

#### 4. **Automate Backup with Cron Jobs**

You can set up a cron job to run the backup command automatically at regular intervals. Add the following to your cron configuration to back up GitLab every day at 2 AM:

```bash
0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create
```

## ⚙️ Step 4: Configure `.gitlab-ci.yml`

In the root of the repo:

```yaml
stages:
  - deploy

test-runner:
  stage: deploy
  tags:
    - docker
  script:
    - echo "✅ CI runner works!"
```

✅ If using shell executor, `docker` will be used from the host — no need for apk installation!

### 🔌 Shutdown Guide (Safe)

Before powering off your system:

```bash
# GitLab
cd project-root
docker compose down

# Runner
cd gitlab-runner
docker compose down

# Then shut down
sudo shutdown now
```

To restart after boot:

```bash
cd project-root && docker compose up -d
cd gitlab-runner && docker compose up -d
```

## ☁️ Summary

- 📄 Licensed under the [MIT License](https://github.com/ahmadsheikhi89/gitlab-runner-docker-setup/blob/main/LICENSE). Feel free to use, modify, and share.
- Self-hosted GitLab + GitLab Runner via Docker
- CI/CD tested with real Docker commands
- Safe shutdown/startup cycle
- Shell executor = easy access to Docker host

Enjoy your 🚀 DevOps setup!
