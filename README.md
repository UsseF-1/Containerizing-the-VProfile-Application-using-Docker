# VProfile Containerization (Docker + Compose)

This repository contains the **containerization setup** for the **VProfile** multi-tier application stack using **Docker** and **Docker Compose**.

The goal is to run the full stack as containers:
- **Nginx** (reverse proxy)
- **Tomcat** (Java app runtime)
- **MySQL** (database)
- **Memcached** (cache)
- **RabbitMQ** (message broker)

> The app, db, and web images are customized and built locally using Dockerfiles.
> Memcached and RabbitMQ are used directly from official Docker Hub images.

---

## Repository Structure

```text
.
├── compose.yaml
├── Docker-files
│   ├── app
│   │   └── Dockerfile
│   ├── db
│   │   ├── Dockerfile
│   │   └── db_backup.sql
│   └── web
│       ├── Dockerfile
│       └── nginxvproapp.conf
└── docs
    └── PROJECT_DOCUMENTATION.md
```

---

## Prerequisites

- Docker Engine (Linux VM or local machine)
- Docker Compose v2 (`docker compose ...`)
- Internet access to pull base images and clone the source code during the build

---

## Quick Start

1) **Clone this repo**:
```bash
git clone <YOUR_REPO_URL>
cd <YOUR_REPO_FOLDER>
```

2) **Build images**:
```bash
docker compose build
```

3) **Start the full stack**:
```bash
docker compose up -d
```

4) **Check containers**:
```bash
docker ps
```

5) **Open the app**
- If running locally: open `http://localhost`
- If running on a VM: open `http://<VM_IP>`

Default login (as used in the course):
- **Username:** `admin_vp`
- **Password:** `admin_vp`

---

## Stop & Cleanup

Stop and remove containers:
```bash
docker compose down
```

Remove volumes (database data + tomcat webapps volume):
```bash
docker volume rm vprodbdata vproappdata 2>/dev/null || true
```

Or remove unused volumes:
```bash
docker volume prune
```

Full cleanup (images + cache):
```bash
docker system prune -a
```

---

## Notes

- The **app image** uses a **multi-stage build** to compile the WAR using Maven, then copies it into a clean Tomcat image.
- The **db image** auto-initializes schema/data by placing `db_backup.sql` in `/docker-entrypoint-initdb.d/`.
- The **web image** replaces the default Nginx config and proxies traffic to `vproapp:8080`.

For detailed explanation, see: `docs/PROJECT_DOCUMENTATION.md`.
