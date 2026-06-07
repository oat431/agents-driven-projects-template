## 📚 The Full Set — 11 Templates

```
workspace\templates\
├── 🎯 AGENTS.md          ← THE entry point. Always loaded.
├── 🏗️ ARCHITECTURE.md     ← Design decisions, patterns, tech debt
├── 🔌 API_PATTERNS.md     ← REST conventions, response shapes, auth
├── 🗄️ DATABASE.md         ← Schema, naming, queries, migrations, N+1 rules
├── 🧪 TESTING.md          ← Test pyramid, patterns, commands, coverage targets
├── 🔒 SECURITY.md         ← Auth flow, hard rules, sensitive data, attack defense
├── 📋 CONVENTIONS.md      ← Code style, naming, git commits, PR guidelines
├── 🚀 DEPLOYMENT.md       ← Environments, CI/CD, health checks, rollback
├── 🛠️ DEVELOPMENT.md      ← Zero-to-running setup, env vars, troubleshooting
├── 📝 TASKS.md            ← Active work, backlog, bugs, tech debt
└── 📖 GLOSSARY.md         ← Domain terms, state machines, team structure
```

---

## Which Files Matter Most

You don't need all 11 in every project. Here's the order:

| Priority | File | When to Create |
|---|---|---|
| 🔑 Must | **AGENTS.md** | Every project. No exceptions. |
| 🔑 Must | **CONVENTIONS.md** | Any project with >1 developer (or 1 dev + AI agent) |
| 🟡 High | **DATABASE.md** | Any project with a database |
| 🟡 High | **API_PATTERNS.md** | Any project with REST/GraphQL endpoints |
| 🟡 High | **SECURITY.md** | Auth, payments, PII — anything sensitive |
| 🟢 Medium | **ARCHITECTURE.md** | When design decisions pile up (>3 major choices) |
| 🟢 Medium | **TESTING.md** | When test patterns are non-obvious |
| 🟢 Medium | **DEPLOYMENT.md** | Multi-environment or >1 person can deploy |
| ⚪ Nice | **DEVELOPMENT.md** | Good for onboarding. Can live in README instead. |
| ⚪ Nice | **TASKS.md** | For AI agent workflow tracking |
| ⚪ Nice | **GLOSSARY.md** | Domain-heavy projects or cross-team work |

---

## How Agents Use These

The pattern is:

1. **AGENTS.md** loads every session — keep it tight (~1-2 KB)
2. **AGENTS.md links to others** — agent loads them on-demand when relevant
3. **Example** from AGENTS.md:
   ```markdown
   ## Further Context
   - Database patterns: [DATABASE.md](./DATABASE.md)
   - API conventions: [API_PATTERNS.md](./API_PATTERNS.md)
   - Security rules: [SECURITY.md](./SECURITY.md) (read before any auth/payment code)
   ```

The agent reads AGENTS.md → sees you're working on payments → loads SECURITY.md + DATABASE.md → writes code that follows both.

---

When you're ready to spin up that personal project, tell me the directory and I'll read your stack and fill these out for real. 🔮
