# PERFORMANCE_TEST.md — Load, Stress & Benchmarks

<!--
  Performance testing. When response time matters.
  AI agents use this to write k6 / Artillery scripts and verify thresholds.
-->

## Performance Strategy

- **Tool:** k6 (preferred — scriptable, CI-friendly) or Artillery
- **Scope:** Critical endpoints. Not every API.
- **When:** Before major release. After infrastructure changes. Nightly on staging.

## Performance Targets

| Endpoint | P50 | P95 | P99 | Max Error Rate |
|----------|-----|-----|-----|----------------|
| GET /api/v1/products | <50ms | <200ms | <500ms | <0.1% |
| POST /api/v1/auth/login | <100ms | <300ms | <800ms | <0.5% |
| GET /api/v1/users (paginated) | <80ms | <250ms | <600ms | <0.1% |
| POST /api/v1/orders | <200ms | <500ms | <1s | <0.1% |
| Health check | <10ms | <20ms | <50ms | 0% |

## Load Test: Normal Traffic

```javascript
// k6/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },   // Ramp up to 50 users
    { duration: '1m', target: 50 },    // Stay at 50
    { duration: '30s', target: 100 },  // Ramp to 100
    { duration: '1m', target: 100 },   // Stay at 100
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    'http_req_duration': ['p(95)<500', 'p(99)<1000'],
    'http_req_failed': ['rate<0.01'],
    'checks': ['rate>0.99'],
  },
};

export default function () {
  // Authenticate
  const loginRes = http.post('https://api.example.com/api/v1/auth/login', JSON.stringify({
    email: 'load-test@example.com',
    password: __ENV.TEST_PASSWORD,
  }), { headers: { 'Content-Type': 'application/json' } });

  check(loginRes, { 'login succeeded': (r) => r.status === 200 });
  const token = loginRes.json('data.accessToken');

  // Browse products
  const productsRes = http.get('https://api.example.com/api/v1/products?page=1&size=20', {
    headers: { Authorization: `Bearer ${token}` },
  });
  check(productsRes, { 'products loaded': (r) => r.status === 200 });

  sleep(1);
}
```

## Stress Test: Breaking Point

```javascript
// k6/stress-test.js
export const options = {
  stages: [
    { duration: '1m', target: 200 },
    { duration: '1m', target: 500 },
    { duration: '1m', target: 1000 },
    { duration: '1m', target: 0 },     // Find breaking point
  ],
};
```

**Goal:** Find where P95 > 2s or error rate > 5%. Document the limit.

## Backend Benchmark (Single-Endpoint)

```java
// JMH Benchmark
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
public class AuthBenchmark {

    @Benchmark
    public void login_validCredentials(Blackhole bh) {
        var result = authService.login("bench@example.com", "password123");
        bh.consume(result);
    }
}
```

## Database Query Performance

```sql
-- Identify slow queries
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 100  -- ms
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Check index usage
SELECT relname, seq_scan, idx_scan,
  CASE WHEN seq_scan + idx_scan = 0 THEN 0
       ELSE idx_scan::float / (seq_scan + idx_scan) * 100
  END AS idx_scan_pct
FROM pg_stat_user_tables
WHERE seq_scan + idx_scan > 0
ORDER BY idx_scan_pct;
```

## Performance Regression Checks (CI)

```yaml
# .github/workflows/perf.yml
- name: Run performance baseline
  run: |
    k6 run k6/baseline.js --summary-export=baseline.json
    jq '.metrics.http_req_duration.p(95)' baseline.json

- name: Compare with previous
  run: |
    # Fail if P95 degraded by >20%
    diff <(jq '.metrics.http_req_duration.p(95)' baseline.json) \
         <(jq '.metrics.http_req_duration.p(95)' previous.json)
```

## Rules for AI Agents

1. **K6 scripts are code.** Review, test, maintain like production code.
2. **Test against staging, never production.** Unless read-only and off-peak.
3. **Thresholds are gates.** P95 > target = failed test = fix before deploy.
4. **Database tuning before code tuning.** Missing index > slow code.
