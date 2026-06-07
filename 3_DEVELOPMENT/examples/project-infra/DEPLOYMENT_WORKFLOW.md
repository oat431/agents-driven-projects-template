# DEPLOYMENT_WORKFLOW.md — project-infra (Homelab Infrastructure)

## Environment Chain

```
LOCAL (laptop) ──▶ DEV (VPS test) ──▶ PROD (panomete.com)
                         │                    │
                    docker compose       docker compose
                    test profile         prod profile
```

## Environments

| Env | Host | Access | Data |
|-----|------|--------|------|
| local | Laptop (Docker Desktop) | Developer only | Docker volumes |
| dev | VPS test subdomain | SSH key | Test database |
| prod | panomete.com (Hetzner VPS) | SSH key (restricted) | Live data |

## Deployment Commands

```bash
# Local dev
docker compose up -d
docker compose logs -f auth-service

# Deploy to dev
ssh dev "cd /opt/homelab && docker compose pull && docker compose up -d"
ssh dev "docker compose ps"

# Deploy to prod (manual, never automated)
ssh prod "cd /opt/homelab && docker compose pull auth-service"
ssh prod "cd /opt/homelab && docker compose up -d auth-service"
ssh prod "docker compose ps && curl -s localhost:8081/actuator/health"
```

## Promotion Gates

### LOCAL → DEV

- [ ] Tests pass locally
- [ ] `docker compose build` succeeds
- [ ] PR reviewed

### DEV → PROD

- [ ] Integration tests pass on dev
- [ ] Smoke test: login, create, list, delete
- [ ] Database migration tested on dev
- [ ] Backup taken before deploy
- [ ] Deploy window: evening (low traffic)
- [ ] Rollback plan: `docker compose up -d` (uses previous image tag)

## Rollback

```bash
# 1. Edit .env: set IMAGE_TAG back one version
# 2. Redeploy
ssh prod "cd /opt/homelab && docker compose up -d"
# 3. Verify
ssh prod "curl -s localhost:8081/actuator/health"
```

## Cron Jobs (on Host)

```bash
# Nightly database backup
0 3 * * * /opt/homelab/scripts/backup-db.sh

# SSL renewal (weekly)
0 0 * * 0 certbot renew --quiet && docker compose restart nginx

# Update containers (weekly, Sunday 4 AM)
0 4 * * 0 /opt/homelab/scripts/update.sh

# Clean old Docker images (monthly)
0 2 1 * * docker system prune -af
```

## Health Checks

```bash
# Quick health
curl -s https://panomete.com/api/v1/health | jq .status

# All services
docker compose ps --format "table {{.Name}}\t{{.Status}}"

# Disk usage
df -h / && docker system df
```
