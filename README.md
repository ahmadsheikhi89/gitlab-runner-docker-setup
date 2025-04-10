# gitlab-runner-docker-setup
Full GitLab CI/CD setup using GitLab CE and GitLab Runner with Docker Compose

## 🚀 GitLab & GitLab Runner Setup with Docker Compose | Complete Local GitLab CI/CD

This guide explains how to set up a self-hosted GitLab instance and GitLab Runner using Docker, and configure a working CI/CD pipeline — just like the one we used to test Docker execution.

---

### 📦 Prerequisites

Before getting started, make sure you have:

- ✅ Docker and Docker Compose installed
- ✅ A Linux system (or WSL2) with sudo access
- ✅ Open ports: `8080` (GitLab web), `2222` (Git SSH), `2375` (Docker socket if DinD)
- ✅ Some basic knowledge of Git, Docker, and YAML

---

### 📁 Directory Structure

```bash
project-root/
├── docker-compose.yml            # GitLab main instance
├── gitlab-runner/
│   ├── docker-compose.yml       # Runner service
│   └── config/config.toml       # Runner config
└── .gitlab-ci.yml               # Main pipeline config
```

---

### 🐙 Step 1: Run GitLab via Docker Compose

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
```

Access GitLab:
- 🌐 Web UI: [http://localhost:8080](http://localhost:8080)
- 🗝️ Default root password: found in `config/initial_root_password`

---

### 🚀 Step 2: Start GitLab Runner

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

---

### 🔑 Step 3: Register the Runner

1. Get the registration token from GitLab:
   ```bash
   docker exec -it gitlab bash
   gitlab-rails runner "puts Gitlab::CurrentSettings.current_application_settings.runners_registration_token"
   ```
2. Register the runner:
   ```bash
   docker exec -it gitlab-runner gitlab-runner register
   ```
   - GitLab URL: `http://host.docker.internal:8080` (or your host IP)
   - Token: (paste it)
   - Description: `my-runner`
   - Tags: `docker,linux`
   - Executor: `shell` or `docker`
   - Docker image (if docker): `docker:latest`

---

### ⚙️ Step 4: Configure `.gitlab-ci.yml`

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

---

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

---

### ☁️ Summary

- Self-hosted GitLab + GitLab Runner via Docker
- CI/CD tested with real Docker commands
- Safe shutdown/startup cycle
- Shell executor = easy access to Docker host

Enjoy your 🚀 DevOps setup!

