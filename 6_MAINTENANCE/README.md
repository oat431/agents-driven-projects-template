# Phase 6: Maintenance

**What:** Keep it running. Fix bugs. Improve. The cycle never ends.  
**Who:** AI agent monitors + triages. Human makes decisions.  
**When:** Ongoing. Daily checks, weekly tech debt review, as-needed incidents.

---

## Template Index

```
6_MAINTENANCE/
│
├── 📄 README.md           ← This file
├── 📄 RUNBOOK.md          ← Ops commands, incident response playbook
├── 📄 MONITORING.md       ← Dashboards, metrics, alerts, health checks
├── 📄 CHANGELOG.md        ← Release history, deprecation tracking
└── 📄 TASKS.md            ← Active work, backlog, bugs, tech debt
```

---

## Daily Rhythm

```
09:00 — Agent checks dashboards (MONITORING.md)
        Reports: "P95 latency normal. 0 errors. 2FA adoption 34%."

10:00 — Agent scans error logs for new patterns
        "New: intermittent timeout on /api/v1/orders?status=pending"

14:00 — Agent checks dependency vulnerabilities
        "Spring Boot 3.3.4 available. 1 CVE fixed. Safe to upgrade?"

17:00 — Agent summarizes day: deploys, incidents, anomalies
```

---

## Weekly Rhythm

```
Monday   — Review TASKS.md. Top 3 tech debt items to tackle.
Tuesday  — Dependency update PR (if safe).
Wednesday— Mid-week health: any slow queries? Disk usage?
Thursday — Prep release notes for CHANGELOG.md.
Friday   — NO DEPLOYS. Review on-call handoff.
```

---

## Incident Lifecycle

```
ALERT FIRES
    │
    ▼
┌──────────┐
│  TRIAGE  │  ← Agent reads RUNBOOK.md. Diagnoses.
│  5 min   │    "Symptom: 502 on /api/v1/orders. DB connection pool exhausted."
└────┬─────┘
     │
     ▼
┌──────────┐
│ MITIGATE │  ← Agent proposes. Human approves. Agent executes.
│  15 min  │    "Kill slow query PID 4821. Increase pool 20→30."
└────┬─────┘
     │
     ▼
┌──────────┐
│ RESOLVE  │  ← Root cause fix. Deploy. Verify.
│  1-4 hr  │    "Added connection timeout. PR #467 merged."
└────┬─────┘
     │
     ▼
┌──────────┐
│ DOCUMENT │  ← Agent writes incident doc in docs/incidents/YYYY-MM-DD.md
│  30 min  │    What happened. Timeline. Root cause. Prevention.
└──────────┘
```

---

## Which File When?

| Situation | Read |
|-----------|------|
| Something broke at 3 AM | `RUNBOOK.md` → Incident Response Playbook |
| Is the system healthy? | `MONITORING.md` → Health Check section |
| What changed in v2.4? | `CHANGELOG.md` → Release entry |
| What are we working on? | `TASKS.md` → This Week |
| How do I restart the service? | `RUNBOOK.md` → Common Operations |
| What does this alert mean? | `RUNBOOK.md` → Alert Reference |
| Is this dependency still supported? | `CHANGELOG.md` → Deprecation Schedule |

---

## Example Prompts

### Daily Health Check
```
"Check MONITORING.md dashboards. Report:
- API: request rate, error rate, P95 latency (vs baseline)
- DB: connection pool %, slow queries, replication lag
- Business: signups, orders (last 24h vs yesterday)
- Alerts: any triggered in last 24h?

Flag anything outside normal range."
```

### Incident Response
```
"Alert: API error rate 12%. Follow RUNBOOK.md incident playbook.
1. Triage: check dashboards, recent deploys, DB status
2. Identify: what's failing, when did it start
3. Propose mitigation. Wait for approval before executing.
4. After resolved: write incident doc."
```

### Weekly Tech Debt
```
"Review TASKS.md tech debt section. Propose 3 items for this sprint.
Rank by: production risk, developer friction, effort.
For each: what's the fix, estimated time, risk of NOT doing it."
```

---

## Deliverables (Ongoing)
- [ ] Daily health checks (automated or agent-driven)
- [ ] Incidents documented with root cause + prevention
- [ ] Tech debt reduced (tracked in TASKS.md)
- [ ] Dependencies updated (security patches applied within 48h)
- [ ] Deprecation timelines honored (from CHANGELOG.md)
- [ ] Weekly on-call handoff (if multi-person team)
