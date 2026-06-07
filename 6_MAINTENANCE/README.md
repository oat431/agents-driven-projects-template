# Phase 6: Maintenance

**What:** Keep it running. Fix bugs. Improve. The cycle never ends.
**Who:** AI agent monitors and triages. Human makes decisions.
**When:** Ongoing. Daily checks, weekly tech debt review, as-needed incidents.

## Templates

| File | Use When |
|------|----------|
| `RUNBOOK.md` | Production service. Ops commands, incident response playbook. |
| `MONITORING.md` | Production service. Dashboards, metrics, alert definitions. |
| `CHANGELOG.md` | Any project with releases. History, deprecation tracking. |
| `TASKS.md` | Active development. Backlog, bugs, tech debt tracking. |

## Daily Workflow

1. Agent checks MONITORING.md dashboards
2. Agent scans error logs for new patterns
3. Agent reports: anomalies, adoption metrics, alert status

## Incident Workflow

1. Alert fires → agent reads RUNBOOK.md
2. Agent triages: "Symptom X, likely cause Y. Proposed mitigation Z."
3. You approve → agent executes mitigation
4. Agent writes incident doc in `docs/incidents/YYYY-MM-DD.md`

## Tech Debt Workflow

1. Agent scans TASKS.md weekly
2. Agent proposes top 3 tech debt items by impact
3. You pick → agent implements → PR → merge

## Example Prompts

**Daily Check:**
```
"Check MONITORING.md dashboards. Report anomalies.
Scan error logs for new patterns. Report adoption metrics.
Flag any triggered alerts."
```

**Incident Response:**
```
"PagerDuty: API error rate 12%. Follow RUNBOOK.md incident playbook.
Triage the issue. Propose mitigation. Do not execute without approval."
```

**Tech Debt Review:**
```
"Review TASKS.md tech debt section. Identify top 3 items by:
- Production risk
- Developer friction
- Implementation effort
Propose which to tackle this sprint."
```

## Deliverables
- [ ] Daily health checks automated
- [ ] Incidents documented with root cause + prevention
- [ ] Tech debt reduced (tracked in TASKS.md)
- [ ] Dependencies updated (security patches)
- [ ] Deprecation timelines honored (from CHANGELOG.md)
