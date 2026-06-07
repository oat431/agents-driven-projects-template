# AGENTS.md — homelab (Docker Compose Infrastructure)

<!-- Real example. Multi-service Docker Compose for self-hosted homelab. -->

## Project

- **Name:** homelab
- **Purpose:** Self-hosted infrastructure on panomete.com. API gateway, auth, databases, monitoring.
- **Host:** Hetzner VPS (4 vCPU, 8 GB RAM, 80 GB SSD)
- **OS:** Ubuntu 24.04 LTS
- **Config Location:** `/opt/homelab/`

## Stack

- **Orchestration:** Docker Compose v2
- **Reverse Proxy:** nginx (custom image, Let's Encrypt via certbot)
- **Services:** Spring Boot 3.3 (Java 21), FastAPI (Python 3.12)
- **Databases:** PostgreSQL 16, Redis 7, MinIO (S3-compatible)
- **Monitoring:** Prometheus + Grafana + Loki + cAdvisor
- **CI/CD:** GitHub Actions → self-hosted runner

## Service Map

```
panomete.com
├── nginx                         — Reverse proxy, SSL termination (:80, :443)
├── auth-service       (:8081)    — JWT auth, user management
├── short-link         (:8000)    — URL shortener (FastAPI)
├── snippets           (:8082)    — Code snippet sharing
├── postgres           (:5432)    — Primary database
├── redis              (:6379)    — Cache + rate limiting
├── minio              (:9000)    — Object storage (S3)
├── prometheus         (:9090)    — Metrics collection
├── grafana            (:3000)    — Dashboards
├── loki               (:3100)    — Log aggregation
├── cadvisor           (:8088)    — Container metrics
└── watchtower                     — Auto-update containers
```

## Commands

```bash
# All commands run from /opt/homelab/

# Start everything
docker compose up -d

# Start specific service
docker compose up -d auth-service

# Restart after config change
docker compose restart nginx

# View logs
docker compose logs -f --tail=100 auth-service
docker compose logs -f --tail=100 nginx | grep ERROR

# Check status
docker compose ps

# Update images + restart changed
docker compose pull && docker compose up -d --remove-orphans

# Full rebuild (after code change)
docker compose build auth-service && docker compose up -d auth-service

# Database backup
docker exec -t postgres pg_dumpall -U postgres > backups/$(date +%Y%m%d).sql

# Restore
docker exec -i postgres psql -U postgres < backups/20260607.sql
```

## File Map

```
/opt/homelab/
├── docker-compose.yml            — Main compose file (all services)
├── .env                          — Secrets + config (NEVER commit)
├── .env.example                  — Template (safe to commit)
├── nginx/
│   ├── nginx.conf                — Main nginx config
│   ├── conf.d/                   — Per-service proxy configs
│   └── ssl/                      — Certbot certificates
├── prometheus/
│   └── prometheus.yml            — Scrape config
├── grafana/
│   └── dashboards/               — Exported dashboard JSON
├── backups/                      — DB dumps (gitignored)
└── scripts/
    ├── backup-db.sh              — Nightly backup cron
    └── update.sh                 — Pull + restart all
```

## Conventions

- One service per directory. Config + Dockerfile + README in each.
- Environment variables: never hardcode in `docker-compose.yml`. Use `.env`.
- Container names: kebab-case, match directory (`auth-service`, `short-link`).
- Networks: `homelab_net` (internal), `homelab_public` (nginx-facing only).
- Volumes: named volumes for persistent data. Bind mounts only for config files.
- Health checks: every service has `healthcheck` in compose.
- Logging: JSON to stdout → Loki picks up. No log files inside containers.
- Restart policy: `unless-stopped` for all services.

## Security Rules

- nginx: ONLY ports 80 and 443 exposed. Everything else on internal network.
- All HTTP → HTTPS redirect. HSTS enabled.
- PostgreSQL: no port exposed to host. Access via internal network only.
- Redis: password-protected (`requirepass` in config).
- `.env` contains secrets. Never committed. `.env.example` is the template.
- SSH: key-only auth. No root login.
- UFW firewall: only 22, 80, 443 open.
- Automatic security updates: `unattended-upgrades` enabled.

## Constraints

- Do NOT expose database ports to host unless debugging (and close immediately after).
- Do NOT modify `nginx.conf` without testing `nginx -t` first.
- Do NOT delete named volumes without confirmed backup.
- Docker images: use specific tags (`auth-service:v2.4.0`), never `:latest` in prod.

## Cron Jobs (on host)

```bash
# Nightly backup (3:00 AM)
0 3 * * * /opt/homelab/scripts/backup-db.sh

# Renew SSL certs (weekly)
0 0 * * 0 certbot renew --quiet && docker compose restart nginx

# Update containers (weekly, Sunday 4 AM)
0 4 * * 0 /opt/homelab/scripts/update.sh
```

## Troubleshooting

| Symptom | Check |
|---------|-------|
| 502 Bad Gateway | `docker compose ps` — is backend running? Check ports. |
| SSL expired | `certbot renew --dry-run` |
| DB connection refused | `docker compose logs postgres` — is it still starting? |
| Disk full | `df -h` — check Docker volumes: `docker system df` |
| High memory | `docker stats` — which container? Check Grafana: `:3000` |
