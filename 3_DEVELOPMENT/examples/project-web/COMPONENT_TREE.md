# COMPONENT_TREE.md — project-web (Next.js Admin Dashboard)

```
components/
├── ui/                          ← shadcn/ui (auto-generated)
│   ├── button.tsx
│   ├── card.tsx
│   ├── dialog.tsx
│   ├── dropdown-menu.tsx
│   ├── input.tsx
│   ├── select.tsx
│   ├── table.tsx
│   ├── toast.tsx
│   ├── tabs.tsx
│   └── avatar.tsx
│
├── layout/
│   ├── AppLayout.tsx             ← Sidebar + Header + main slot
│   ├── Sidebar.tsx               ← Nav items, active state, collapse
│   └── Header.tsx                ← Breadcrumb, search, notifications
│
├── data/
│   ├── DataTable.tsx             ← Generic: sort, paginate, select, export
│   ├── StatCard.tsx              ← Icon, value, trend indicator
│   ├── StatusBadge.tsx           ← active/grey, pending/yellow, error/red
│   ├── EmptyState.tsx            ← Illustration + "No results" + CTA
│   └── ConfirmDialog.tsx         ← "Are you sure?" with loading state
│
├── users/
│   ├── UserTable.tsx             ← Uses DataTable with user columns
│   ├── UserForm.tsx              ← Create/Edit form (React Hook Form + Zod)
│   └── UserDetail.tsx            ← Profile view with tabs
│
├── orders/
│   ├── OrderTable.tsx
│   ├── OrderDetail.tsx
│   └── OrderTimeline.tsx         ← Status history with timestamps
│
└── analytics/
    ├── RevenueChart.tsx           ← Recharts AreaChart
    ├── UserGrowthChart.tsx        ← Recharts LineChart
    └── TopProductsTable.tsx
```

## Rules for Agents

1. `ui/` are shadcn/ui primitives — never rebuild them
2. `layout/` is the app shell — all pages wrapped in `<AppLayout>`
3. `data/` are shared across all feature pages — check here first
4. Feature folders (`users/`, `orders/`) contain page-specific components
5. New component used in 3+ features → promote to `data/`
6. Use `data-test` attributes for E2E selectors, not CSS classes
