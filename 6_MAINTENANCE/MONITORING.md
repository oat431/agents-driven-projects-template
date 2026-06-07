# MONITORING.md — Observability & Alerting

<!--
  What to watch, where to watch it, and what to do when it screams.
  Agents in maintenance mode use this to detect and diagnose issues.
-->

## Observability Stack
- **Metrics:** <!-- Prometheus + Grafana / Datadog / New Relic -->
- **Logs:** <!-- Loki + Grafana / CloudWatch / ELK / Datadog -->
- **Traces:** <!-- Tempo + Grafana / Jaeger / Datadog APM -->
- **Alerts:** <!-- AlertManager / PagerDuty / Opsgenie -->
- **Status Page:** <!-- status.example.com (powered by Atlassian Statuspage) -->

## Key Metrics

### Golden Signals (Every Service)

| Signal | Metric | Target | Alert At |
|--------|--------|--------|----------|
| **Latency** | P95 response time | <200ms | >500ms (5 min) |
| **Traffic** | Requests/sec | — | ±50% from baseline |
| **Errors** | 5xx rate | <1% | >5% (2 min) |
| **Saturation** | CPU / Memory / DB Connections | <70% | >85% (5 min) |

### Application Metrics

```promql
# P95 latency by endpoint
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate by endpoint
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# Active users (last 15 min)
sum(rate(logins_total[15m]))

# Failed login rate (possible attack)
sum(rate(login_attempts_total{success="false"}[5m]))
```

### Database Metrics

```promql
# Connection pool utilization
pg_stat_database_numbackends / pg_settings_max_connections

# Slow queries (>1s)
rate(pg_stat_statements_calls{query_time_seconds>1}[5m])

# Replication lag (bytes)
pg_stat_replication_replay_lag_bytes

# Cache hit ratio (target >95%)
pg_stat_database_blks_hit / (pg_stat_database_blks_hit + pg_stat_database_blks_read)
```

### Business Metrics

| Metric | Why It Matters | Query |
|--------|---------------|-------|
| Signup rate | Growth | `sum(rate(signups_total[1h]))` |
| Order value (avg) | Revenue health | `avg(order_amount) by (currency)` |
| Checkout abandonment | UX issue | `(started_checkout - completed) / started_checkout` |
| DAU / MAU | Stickiness | Dashboard: "User Engagement" |

## Dashboards (Grafana Reference)

### Dashboard: API Overview
```
Row 1: Request Rate | Error Rate | P95 Latency
Row 2: Top 5 Endpoints by Latency | Top 5 Endpoints by Errors
Row 3: 2xx / 3xx / 4xx / 5xx Breakdown
Row 4: Active Connections | Thread Pool Utilization
```

### Dashboard: Database Health
```
Row 1: Connections Used | Cache Hit Ratio | Dead Tuples
Row 2: Queries/sec | Avg Query Time | Slow Queries
Row 3: Replication Lag | WAL Generation Rate
Row 4: Table Sizes (Top 10) | Index Usage
```

## Log Conventions

### Log Levels
| Level | When |
|-------|------|
| ERROR | Something broke. Needs attention. |
| WARN | Degraded but working. Retry succeeded. Rate limited. |
| INFO | Business events: "user logged in", "order placed", "payment received" |
| DEBUG | Detailed tracing for debugging. Off in production. |

### Structured Logging Format
```json
{
  "timestamp": "2026-06-07T20:42:00Z",
  "level": "INFO",
  "service": "auth-service",
  "traceId": "abc123",
  "userId": "user-456",
  "event": "user.login.success",
  "details": {
    "method": "email",
    "ip": "203.0.113.42",
    "userAgent": "Mozilla/5.0..."
  }
}
```

### Log Queries (Loki / CloudWatch / ELK)
```logql
# Recent errors
{service="api-server"} |= "ERROR" | json | line_format "{{.message}}"

# User activity
{service="api-server"} | json | userId="user-456" | line_format "{{.event}} {{.details}}"

# Slow requests
{service="api-server"} | json | duration > 2000ms

# Rate limit hits
{service="api-server"} | json | event="rate_limit.hit"
```

## Health Check Detail

### Endpoint: GET /actuator/health
```json
{
  "status": "UP",
  "components": {
    "db": { "status": "UP", "details": { "database": "PostgreSQL", "validationQuery": "SELECT 1" } },
    "redis": { "status": "UP" },
    "diskSpace": { "status": "UP", "details": { "free": "45GB" } }
  }
}
```

### What "DOWN" Looks Like
```json
{
  "status": "DOWN",
  "components": {
    "db": { "status": "DOWN", "details": { "error": "Connection refused" } }
  }
}
```

## Synthetic Checks (External Monitoring)
```bash
# Health check
curl -s -o /dev/null -w "%{http_code}" https://api.example.com/actuator/health

# Critical flow (login + get profile)
curl -s https://api.example.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"monitor@example.com","password":"***"}' | \
  jq -r .data.accessToken | \
  xargs -I {} curl -s https://api.example.com/api/v1/users/me \
  -H "Authorization: Bearer {}"
```

Run these every 60 seconds from a separate host/region. Alert if they fail 3 times in a row.
