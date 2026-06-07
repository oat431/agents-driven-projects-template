# UI_SPEC.md — project-web (Next.js Admin Dashboard)

## Brand Identity

### Colors

```css
:root {
  --color-primary: #7C3AED;
  --color-primary-light: #A78BFA;
  --color-primary-dark: #5B21B6;

  --color-bg: #FAFAFA;
  --color-surface: #FFFFFF;
  --color-border: #E5E5E5;

  --color-text: #262626;
  --color-text-muted: #737373;
  --color-text-inverse: #FFFFFF;

  --color-success: #16A34A;
  --color-warning: #F59E0B;
  --color-error: #DC2626;
}
```

### Typography

- **Font:** Inter (headings), Sarabun (body — Thai-friendly)
- **Scale:** 14/16/18/20/24/32px
- **Weights:** 400 (body), 500 (label), 600 (heading), 700 (title)
- **Line height:** 1.5 (body), 1.3 (heading)

### Spacing (Tailwind)

- `p-2` (8px) — icon padding
- `p-4` (16px) — card padding
- `p-6` (24px) — page padding
- `gap-4` — card grid gaps
- `gap-6` — section gaps

### Component Patterns

**DataTable (reusable):**

```tsx
<DataTable
  columns={columns}
  data={users}
  pagination={{ page, pageSize, totalItems }}
  onPaginationChange={setPagination}
  onSortChange={setSort}
  onRowSelect={handleBulkAction}
  searchable
  exportable
/>
```

**StatCard:**

```tsx
<StatCard
  title="Active Users"
  value="1,234"
  trend="+12%"
  trendDirection="up"
  icon={<UsersIcon />}
/>
```

### Layout

```
┌──────────────────────────────────────────────┐
│ SIDEBAR (w-64)       │ HEADER (h-16)         │
│                       │                       │
│ ┌─────┐               │ Breadcrumb + Actions  │
│ │Logo │               ├───────────────────────┤
│ └─────┘               │                       │
│ ─────────────────     │ CONTENT               │
│ Nav items             │ max-w-7xl             │
│  • Dashboard          │ p-6                   │
│  • Users     ●active  │                       │
│  • Orders             │                       │
│  • Analytics          │                       │
│ ─────────────────     │                       │
│ User menu (bottom)    │                       │
└──────────────────────────────────────────────┘
```

### Responsive

| Breakpoint | Layout |
|-----------|--------|
| < 768px | Sidebar → hamburger drawer. Table → cards. |
| 768–1024px | Sidebar collapsed (icons only). |
| > 1024px | Full sidebar. Full table. |
