# DEPLOYMENT.md — Environments, CI/CD & Infrastructure

  How this project gets to production. Agents use this to understand deployment
  constraints, environment differences, and infrastructure requirements.

## Environments

| Env | URL | Branch | Data | Access |
|-----|-----|--------|------|--------|
| **local** | localhost:8080 | any | Docker containers | Developer |
| **dev** | dev.example.com | `develop` | Seeded test data | Team |
| **staging** | staging.example.com | `main` | Anonymized prod snapshot | Team + QA |
| **prod** | api.example.com | `main` (tagged) | Live | Restricted |

## Infrastructure

### Stack
- **Compute:** AWS ECS Fargate / Kubernetes / VPS (Hetzner)
- **Database:** RDS PostgreSQL / Cloud SQL / self-managed
- **Cache:** ElastiCache Redis / Memorystore / local
- **Storage:** S3 / GCS / MinIO
- **CDN:** CloudFront / Cloudflare
- **DNS:** Route53 / Cloudflare

### Resource Sizing
| Service | CPU | Memory | Instances |
|---------|-----|--------|-----------|
| api-server | 2 vCPU | 4 GB | 3 min |
| worker | 1 vCPU | 2 GB | 2 min |
| db | db.t3.large | 8 GB | 1 (multi-AZ) |

## CI/CD Pipeline

### Pipeline Flow
```
Push → Build → Test → Lint → (PR) → Review → Merge → Deploy Staging → Smoke Test → Deploy Prod
```

### Build Steps
```bash
# 1. Build
./mvnw clean package -DskipTests

# 2. Unit tests
./mvnw test

# 3. Integration tests
docker compose -f docker-compose.test.yml up -d
./mvnw verify -P integration

# 4. Static analysis
./mvnw checkstyle:check pmd:check spotbugs:check

# 5. Build image
docker build -t myapp:${VERSION} .

# 6. Push
docker push registry.example.com/myapp:${VERSION}
```

### Deployment Command
```bash
# Kubernetes
kubectl set image deployment/api-server api-server=myapp:${VERSION}
kubectl rollout status deployment/api-server

# Docker Compose (simpler setups)
docker compose -f docker-compose.prod.yml up -d
```

## Health Checks
- **Liveness:** `GET /actuator/health/liveness` (is the process alive?)
- **Readiness:** `GET /actuator/health/readiness` (can it serve traffic?)
- **Custom:** DB connectivity, Redis ping, disk space check

## Rollback Procedure
```bash
# 1. Identify last good version
kubectl rollout history deployment/api-server

# 2. Roll back
kubectl rollout undo deployment/api-server --to-revision=3

# 3. Verify
curl -s https://api.example.com/actuator/health | jq .status
```

## Secrets Management
- **Dev:** `.env` file (gitignored, never committed)
- **Prod:** AWS Secrets Manager / HashiCorp Vault / GitHub Secrets
- **Rotation:** DB passwords rotated quarterly
- **Never:** Hardcoded keys, config files with secrets in repo

## Backup & Restore
```bash
# Backup (daily cron, 03:00 UTC)
pg_dump -Fc myapp_prod > backups/myapp_$(date +%Y%m%d).dump

# Restore (emergency)
pg_restore -d myapp_prod --clean backups/myapp_20260607.dump

# Retention: 7 daily, 4 weekly, 12 monthly
```
