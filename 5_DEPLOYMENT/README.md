# Phase 5: Deployment

**What:** Ship to production safely. Multi-environment pipeline. Rollback capable.  
**Who:** AI agent executes deployments. Human approves and smoke-tests.  
**When:** After tests pass. Every release.

---

## Template Index

```
5_DEPLOYMENT/
│
├── 📄 README.md                  ← This file
├── 📄 DEPLOYMENT_WORKFLOW.md     ← Multi-environment pipeline + promotion gates
├── 📄 CI_CD.md                   ← GitHub Actions / GitLab CI pipeline design
├── 📄 DEPLOYMENT.md              ← Infrastructure + deployment commands (existing)
├── 📄 DEVELOPMENT.md             ← Local setup + troubleshooting (existing)
└── 📄 INFRASTRUCTURE.md          ← IaC: Docker, K8s, Terraform (optional)
```

---

## The Environment Chain

```
LOCAL  ──▶  DEV  ──▶  QA  ──▶  UAT  ──▶  STAGING  ──▶  PROD
  │          │        │        │           │            │
  │     auto on     manual    manual     manual     restricted
  │     push to    promote    promote    promote     deploy
  │     develop    from dev   from qa    from uat    from main
  │          │        │        │           │            │
  ▼          ▼        ▼        ▼           ▼            ▼
no gates   smoke    full     UAT sign-   perf test   canary →
           tests    suite    off         green       10% → 100%
```

---

## Which File When?

| I need to... | Read This |
|-------------|-----------|
| Understand environments | `DEPLOYMENT_WORKFLOW.md` → Environment Map |
| Set up CI/CD | `CI_CD.md` → Pipeline design |
| Deploy to specific env | `DEPLOYMENT_WORKFLOW.md` → Promotion Commands |
| Debug a pipeline failure | `CI_CD.md` → Pipeline overview |
| Roll back a bad deploy | `DEPLOYMENT_WORKFLOW.md` → Rollback Procedure |
| Set up local dev | `DEVELOPMENT.md` → Quick Start |
| Write Docker/K8s config | `INFRASTRUCTURE.md` → IaC patterns |

---

## Workflow

### Standard Release
1. All tests green on `develop`
2. Manual promote: DEV → QA → UAT (per DEPLOYMENT_WORKFLOW.md)
3. Product owner signs off UAT
4. Merge `develop` → `main`
5. Deploy to STAGING, run perf test
6. Deploy to PROD (canary 10% → monitor → 100%)
7. Post-deploy monitoring (30 min)

### Hotfix
1. Branch from `main`: `hotfix/xxx`
2. Fix + test + PR → merge to `main`
3. Deploy directly to STAGING (skip DEV/QA)
4. Verify → PROD (canary)
5. Cherry-pick fix back to `develop`

## Example Prompt (Full Deploy)
```
"Deploy v2.4.0 to production per DEPLOYMENT_WORKFLOW.md.

Pre-deploy:
- Verify CI green on main branch
- Check migration is backward-compatible
- Confirm staging deploy is healthy

Deploy:
- Canary: 10% traffic, monitor 10 min
- Error rate stable? P95 latency normal?
- If yes: 100% rollout. If no: rollback.

Post-deploy:
- Smoke test critical endpoints
- Monitor 30 min (error rate, latency, DB connections)
- Update CHANGELOG.md with release date
- Report deployment summary"
```

---

## Deliverables
- [ ] CI/CD pipeline green
- [ ] Security scan clean (no HIGH/CRITICAL)
- [ ] Database migration backward-compatible
- [ ] DEV → QA → UAT promotions successful
- [ ] UAT sign-off complete
- [ ] Staging deploy + performance test green
- [ ] Production canary deploy successful
- [ ] Post-deploy monitoring clear (30 min)
- [ ] CHANGELOG.md updated
