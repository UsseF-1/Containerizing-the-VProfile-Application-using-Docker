# Project Documentation — VProfile Containerization

## 1) Overview

This project containerizes the **VProfile** application stack using Docker.
Instead of running each service on separate VMs, we run them as **containers** and orchestrate them using **Docker Compose**.

**Services in the stack:**
- **vproweb**: Nginx reverse proxy (port 80)
- **vproapp**: Tomcat hosting the VProfile WAR (port 8080)
- **vprodb**: MySQL database (port 3306)
- **vprocache01**: Memcached (port 11211)
- **vpromq01**: RabbitMQ (port 5672)

---

## 2) Architecture

**Request flow**
1. User hits Nginx on port 80 (`vproweb`)
2. Nginx forwards requests to Tomcat (`vproapp:8080`)
3. The application connects to:
   - MySQL (`vprodb:3306`)
   - Memcached (`vprocache01:11211`)
   - RabbitMQ (`vpromq01:5672`)

**Key point:** Containers communicate over the internal Docker network created by Compose.

---

## 3) Why Containerization?

Containerization solves common problems in VM-based deployments:
- **Less resource waste** compared to full OS per service
- **Portability**: “works on my machine” becomes “works everywhere”
- **Repeatability**: same image runs in Dev/QA/Prod
- **Microservices-friendly**: easier to scale and isolate services

---

## 4) Images & Tags Used

Base images used in this project:
- MySQL: `mysql:8.0.33`
- Tomcat: `tomcat:10-jdk21`
- Nginx: `nginx:latest`
- Memcached: `memcached:latest`
- RabbitMQ: `rabbitmq:latest`
- Maven build stage: `maven:3.9.9-eclipse-temurin-21-jammy`

> In real-world projects you should align tags with the exact versions used by developers.

---

## 5) Dockerfiles Explained

### 5.1 App Image (Tomcat)

**File:** `Docker-files/app/Dockerfile`

This is a **multi-stage build**:

- **Stage 1 (build_image):**  
  Uses Maven + JDK to clone the upstream VProfile repo (containers branch) and build the WAR.

- **Stage 2 (runtime):**  
  Uses the official Tomcat image, removes default apps, and copies the built WAR as `ROOT.war`.

Benefits:
- Final runtime image is smaller (no Maven cache/deps in final layer)
- Faster startup and fewer attack surface components

---

### 5.2 DB Image (MySQL)

**File:** `Docker-files/db/Dockerfile`

Uses official MySQL image and:
- Sets MySQL root password and database name via `ENV`
- Adds `db_backup.sql` into `/docker-entrypoint-initdb.d/`

MySQL entrypoint runs this SQL automatically on the first container start.

---

### 5.3 Web Image (Nginx)

**File:** `Docker-files/web/Dockerfile`

- Removes the default Nginx config
- Copies a custom config that proxies to `vproapp:8080`

**Config file:** `Docker-files/web/nginxvproapp.conf`

---

## 6) Docker Compose Explained

**File:** `compose.yaml`

Compose:
- Builds 3 custom images (`app`, `db`, `web`)
- Pulls 2 official images (`memcached`, `rabbitmq`)
- Creates a dedicated network automatically
- Creates 2 named volumes:
  - `vprodbdata` -> persists MySQL data
  - `vproappdata` -> optional Tomcat webapps mount

**Important:**
- Container names must match the application config expectations:
  - `vprodb`
  - `vprocache01`
  - `vpromq01`
  - `vproapp`

---

## 7) Runbook

### Build
```bash
docker compose build
```

### Start
```bash
docker compose up -d
```

### Verify
```bash
docker ps
docker compose logs -f --tail=100
```

### Stop
```bash
docker compose down
```

### Cleanup
```bash
docker volume prune
docker system prune -a
```

---

## 8) Publishing Images to Docker Hub

1) Login:
```bash
docker login
```

2) Push (example names):
```bash
docker push vprocontainers/vprofileapp:latest
docker push vprocontainers/vprofiledb:latest
docker push vprocontainers/vprofileweb:latest
```

Ensure your `image:` names in `compose.yaml` are aligned with your Docker Hub username/org.

---

## 9) Common Troubleshooting

- **App can't connect to DB**:
  - Confirm `vprodb` container is healthy and ports are correct
  - Check logs: `docker compose logs vprodb`

- **Nginx shows 502 Bad Gateway**:
  - Ensure `vproapp` is running and listening on 8080
  - Verify Nginx config points to `vproapp:8080`

- **RabbitMQ auth issues**:
  - Ensure env vars are set:
    - `RABBITMQ_DEFAULT_USER=guest`
    - `RABBITMQ_DEFAULT_PASS=guest`

---

## 10) What To Customize Next

- Pin exact tags instead of `latest`
- Add healthchecks in Compose
- Add a CI pipeline to build & push images automatically
- Migrate to Kubernetes (next step in many DevOps learning paths)
