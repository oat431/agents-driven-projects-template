# RUNBOOK.md — Operations & Incident Response

  The operations manual. Agents use this in production contexts — monitoring,
  incident response, and maintenance tasks. Link from AGENTS.md for ops-heavy projects.

## Service Overview
- **Service:** e.g., api-server
- **Port:** 8080
- **Health:** `GET /actuator/health`
- **Metrics:** `GET /actuator/prometheus`
- **Logs:** CloudWatch / Loki / ELK / journalctl

## Common Operations

### Deploy
```bash
# Standard deploy (no downtime — rolling update)
kubectl set image deployment/api-server api-server=myapp:${VERSION}
kubectl rollout status deployment/api-server --timeout=5m

# Rollback
kubectl rollout undo deployment/api-server
```

### Restart Service
```bash
# Graceful (drains connections)
kubectl rollout restart deployment/api-server

# Emergency (force)
kubectl delete pod -l app=api-server
```

### Check Logs
```bash
# Recent errors
kubectl logs -l app=api-server --tail=100 | grep ERROR

# Follow specific pod
kubectl logs -f api-server-abc123

# Search for user activity
kubectl logs -l app=api-server --since=1h | grep "userId=abc-123"
```

### Database
```bash
# Connection count
psql -d myapp_prod -c "SELECT count(*) FROM pg_stat_activity;"

# Running queries (>5s)
psql -d myapp_prod -c "
  SELECT pid, now() - query_start AS duration, query
  FROM pg_stat_activity
  WHERE state = 'active' AND now() - query_start > interval '5 seconds';"

# Kill a query
psql -d myapp_prod -c "SELECT pg_terminate_backend(<pid>);"
```

### Redis / Cache
```bash
# Check memory
redis-cli INFO memory | grep used_memory_human

# Flush specific key pattern
redis-cli --scan --pattern "session:*" | xargs redis-cli DEL

# Clear entire cache (careful)
redis-cli FLUSHDB
```

## Monitoring Dashboards
| Dashboard | URL | What It Shows |
|-----------|-----|---------------|
| API Overview | <!-- grafana.example.com/d/api --> | Request rate, latency, error rate |
| Database | <!-- grafana.example.com/d/db --> | Connections, query times, locks |
| Business Metrics | <!-- grafana.example.com/d/biz --> | Signups, orders, revenue |

## Alert Reference

### P0 — Wake Someone Up
| Alert | Trigger | Response |
|-------|---------|----------|
| API error rate > 5% | 2 min sustained | Rollback last deploy. Check DB. |
| API 5xx spike | >50 errors/min | Check logs for exceptions. DB connection pool? |
| Health check failing | 3 consecutive failures | Is the process alive? OOM? Disk full? |
| SSL cert expiring | <7 days | Renew cert. Run certbot. |

### P1 — Fix Within Hours
| Alert | Trigger | Response |
|-------|---------|----------|
| P95 latency > 2s | 5 min sustained | Slow queries? Cache miss storm? GC pause? |
| DB connection pool > 80% | 5 min | Check for connection leaks. Increase pool temp. |
| Disk > 85% | Any | Clean logs, old artifacts. Expand volume. |
| Redis memory > 80% | 5 min | Check key eviction. Increase or flush stale keys. |

### P2 — Fix This Week
| Alert | Trigger |
|-------|---------|
| Deprecated API version usage | Any calls to `/v1/` after deprecation |
| Failed login spike | >3x normal rate (possible credential stuffing) |
| Background job queue depth > 1000 | 10 min sustained |

## Incident Response Playbook

### 1. Triage (First 5 Minutes)
```bash
# Quick health snapshot
curl -s https://api.example.com/actuator/health | jq .

# Recent errors
kubectl logs -l app=api-server --tail=50 | grep -E "ERROR|FATAL"

# System resources
kubectl top pods -l app=api-server
```

### 2. Mitigate (Stop the Bleeding)
- **Bad deploy?** → `kubectl rollout undo deployment/api-server`
- **DB overload?** → Kill slow queries. Enable read replica.
- **Traffic spike?** → Scale up: `kubectl scale deployment/api-server --replicas=6`
- **Dependency down?** → Check circuit breaker status. Enable fallback.
- **Security incident?** → Rotate affected secrets first. Investigate after.

### 3. Investigate (Root Cause)
- Check deploy timeline: did this start after a release?
- Check dependency status: is Stripe/SendGrid/AWS down?
- Check recent config changes: `git log --since="2 hours ago"`
- Check DB for locks: `pg_stat_activity` for blocked queries

### 4. Resolve & Document
- Post-incident: create a dated incident doc in `docs/incidents/YYYY-MM-DD.md`
- Template: What happened, impact, timeline, root cause, fix, prevention

## Backup & Restore Drill
```bash
# 1. Take manual backup
pg_dump -Fc myapp_prod > pre_drill_backup.dump

# 2. Test restore to temp DB
createdb restore_test
pg_restore -d restore_test pre_drill_backup.dump

# 3. Verify
psql -d restore_test -c "SELECT count(*) FROM users;"

# 4. Clean up
dropdb restore_test
```

## Useful Queries
```sql
-- Active user sessions
SELECT count(*) FROM user_sessions WHERE expires_at > now();

-- Failed logins (last hour)
SELECT email, count(*) AS attempts
FROM login_attempts
WHERE created_at > now() - interval '1 hour'
  AND success = false
GROUP BY email
HAVING count(*) > 10
ORDER BY attempts DESC;
```
