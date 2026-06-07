# DEPLOYMENT_WORKFLOW.md — Multi-Environment Pipeline

<!--
  The complete deployment flow. AI agents use this to understand:
  - Which environment does what
  - How code promotes from dev → production
  - What gates exist between environments
-->

## Environment Map

```
┌─────────┐    ┌──────────┐    ┌──────┐    ┌──────┐    ┌──────────┐
│  LOCAL  │───▶│   DEV    │───▶│  QA  │───▶│ UAT  │───▶│   PROD   │
│         │    │          │    │      │    │      │    │          │
│ Laptop  │    │ Shared   │    │ Test │    │ Sign-│    │  Live    │
│ Docker  │    │ Cluster  │    │ Team │    │ off  │    │  Traffic │
└─────────┘    └──────────┘    └──────┘    └──────┘    └──────────┘
     │              │              │            │             │
     ▼              ▼              ▼            ▼             ▼
  No gates     Auto-deploy    Manual       Product       Restricted
  (dev does   on push to     promote      owner         deploy
   whatever)  `develop`      from dev     approves      from main
```

---

## Environment Details

| Env | Purpose | Data | Who Deploys | Access | URL |
|-----|---------|------|-------------|--------|-----|
| **local** | Development on laptop | Docker containers | Developer | Self only | `localhost:*` |
| **dev** | Integration testing | Seeded test data | Auto (push to `develop`) | Dev team | `dev.api.example.com` |
| **qa** | Automated testing | Anonymized prod sample | Manual promote from dev | QA + Dev | `qa.api.example.com` |
| **uat** | User acceptance | Anonymized prod snapshot | Manual promote from QA | Product + QA | `uat.api.example.com` |
| **staging** | Pre-prod mirror | Prod replica (anonymized) | Manual promote from UAT | DevOps only | `staging.api.example.com` |
| **prod** | Live traffic | Real data | Restricted (senior only) | On-call | `api.example.com` |

---

## Promotion Flow

```
feature/xxx  →  PR  →  merge to `develop`
                            │
                            ▼
                     [DEV] auto-deploy
                            │
                     Run integration smoke tests
                            │
                     ┌──────┴──────┐
                     │   PASS?     │
                     ├─────────────┤
                     │ YES         │ NO
                     ▼             ▼
              Promote to QA    Fix + re-deploy
                     │
                     ▼
              [QA] manual deploy
                     │
              Run full test suite
                     │
              ┌──────┴──────┐
              │   PASS?     │
              ├─────────────┤
              │ YES         │ NO
              ▼             ▼
        Promote to UAT   Bug fix → back to dev
              │
              ▼
        [UAT] manual deploy
              │
        Product owner tests
              │
        ┌─────┴─────┐
        │ SIGN OFF? │
        ├───────────┤
        │ YES       │ NO
        ▼           ▼
   Merge to main  Fix per UAT feedback
        │
        ▼
  [STAGING] deploy
        │
  Full regression + perf test
        │
  ┌─────┴─────┐
  │  GREEN?   │
  ├───────────┤
  │ YES       │ NO
  ▼           ▼
[PROD]      Rollback staging
canary      → fix → re-deploy
  │
  ▼
Monitor 30 min
  │
  ▼
Full rollout
```

---

## Promotion Commands

```bash
# Promote from DEV to QA
./scripts/promote.sh dev qa v2.4.0

# Promote from QA to UAT
./scripts/promote.sh qa uat v2.4.0

# Deploy to staging (from main)
./scripts/deploy.sh staging v2.4.0

# Deploy to production (canary)
./scripts/deploy.sh prod v2.4.0 --strategy=canary --canary-percent=10
```

---

## Per-Environment Config Differences

| Setting | DEV | QA | UAT | STAGING | PROD |
|---------|-----|----|-----|---------|------|
| Log level | DEBUG | DEBUG | INFO | INFO | WARN |
| Rate limiting | OFF | OFF | ON (relaxed) | ON | ON (strict) |
| Email sending | Mailhog | Mailhog | Real (test list) | Real (test list) | Real (all) |
| Payment processing | Sandbox | Sandbox | Sandbox | Sandbox | Live |
| Feature flags | All ON | Most ON | Prod config | Prod config | Prod config |
| DB replicas | 0 | 0 | 0 | 1 (read) | 2 (read) |
| Min instances | 1 | 1 | 1 | 2 | 3 |

---

## Deployment Strategies

| Strategy | When | How | Rollback |
|----------|------|-----|----------|
| **Rolling** | Standard deploy | Replace instances one by one | `kubectl rollout undo` |
| **Canary** | High-risk changes | 10% traffic → monitor → 100% | Shift traffic back |
| **Blue-Green** | Database migrations | Deploy new stack, switch DNS | Switch DNS back |
| **Recreate** | Dev/QA only | Tear down, bring up | Re-deploy previous version |

---

## Promotion Gates

### DEV → QA

- [ ] All unit tests pass
- [ ] Integration tests pass
- [ ] No lint errors
- [ ] PR reviewed + approved

### QA → UAT

- [ ] Full regression suite green
- [ ] Performance baselines met (no >20% degradation)
- [ ] Security scan clean (no HIGH/CRITICAL)
- [ ] QA sign-off

### UAT → STAGING

- [ ] All UAT scenarios pass (UAT.md)
- [ ] Product owner sign-off
- [ ] CHANGELOG.md updated

### STAGING → PROD

- [ ] Full regression green
- [ ] Performance test green
- [ ] Database migration tested (no locks, no data loss)
- [ ] Rollback plan documented + tested
- [ ] On-call engineer notified
- [ ] Deployment window: Tue–Thu, 10:00–16:00 (never Friday)

---

## Rollback Procedure

```bash
# 1. Detect issue (alert fires or engineer notices)
# 2. Decide: rollback or fix-forward?
#    - Data corruption / customer impact → ROLLBACK
#    - Minor bug, quick fix → FIX-FORWARD

# Rollback (Kubernetes)
kubectl rollout undo deployment/api-server
kubectl rollout status deployment/api-server --timeout=5m

# Rollback (Docker Compose)
docker compose down
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
# (uses previous image tag from .env)

# 3. Verify health
curl -s https://api.example.com/actuator/health | jq .status
# Expected: "UP"

# 4. Notify team
# "Production rollback to v2.3.1 complete. Investigating v2.4.0. ETA for fix: TBD."
```

## Post-Deploy Verification (Every Environment)

```bash
# 1. Health check
curl -s https://{env}.api.example.com/actuator/health | jq .status

# 2. Critical endpoint smoke test
curl -s https://{env}.api.example.com/api/v1/products?size=1 | jq .success

# 3. Check error rate (last 5 min)
# Grafana or: kubectl logs -l app=api-server --since=5m | grep -c ERROR

# 4. Check DB connectivity
kubectl exec -it deployment/api-server -- curl -s localhost:8080/actuator/health/db
```
