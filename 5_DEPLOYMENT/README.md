# Phase 5: Deployment

**What:** Ship to production safely. Rollback if needed.
**Who:** AI agent executes. Human approves and smoke-tests.
**When:** After tests pass. Every release.

## Templates

| File | Use When |
|------|----------|
| `DEPLOYMENT.md` | Multi-environment or multi-person deploy access. |
| `DEVELOPMENT.md` | Onboarding new devs. Local setup instructions. |

## Workflow

1. Agent reads DEPLOYMENT.md → checks CI pipeline
2. Agent verifies: all tests green, migrations backward-compatible, feature flags OFF
3. Agent deploys to staging → runs smoke tests
4. You verify staging → approve production
5. Agent deploys to prod (canary or rolling)
6. Agent monitors for 15 min: errors, latency, DB connections
7. Agent enables feature flag gradually
8. Agent updates CHANGELOG.md

## Example Prompt

```
"Deploy v2.4.0 to production per DEPLOYMENT.md.
1. Verify CI is green and migrations are backward-compatible
2. Deploy to staging, run smoke tests
3. Report staging results and wait for my approval
4. Deploy to production (canary: 1 instance first)
5. Monitor error rate, latency, DB for 15 minutes
6. Report: deploy healthy or rollback needed"
```

## Deliverables

- [ ] Staging deploy + smoke test passed
- [ ] Production deploy performed (canary or rolling)
- [ ] Post-deploy monitoring (15 min window)
- [ ] Feature flag rollout gradual
- [ ] CHANGELOG.md updated with release date
- [ ] Rollback plan confirmed (if needed)
