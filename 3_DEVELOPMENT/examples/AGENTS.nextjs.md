# AGENTS.md — web-dashboard (Next.js 14 Frontend)

<!-- Real example. Next.js 14 App Router + TypeScript + TanStack Query. -->

## Project

- **Name:** web-dashboard
- **Purpose:** Admin dashboard for managing users, orders, and analytics. Internal tool — auth required for all routes.
- **Repo:** github.com/panomete/web-dashboard
- **URL:** <https://admin.panomete.com>

## Stack

- **Language:** TypeScript 5.x (strict mode)
- **Framework:** Next.js 14 (App Router)
- **UI:** Tailwind CSS + shadcn/ui (Radix primitives)
- **State:** TanStack Query v5 (server state), Zustand (client state)
- **Forms:** React Hook Form + Zod validation
- **Table:** TanStack Table v8
- **Auth:** next-auth v5 (Auth.js) + JWT
- **Testing:** Vitest + Testing Library + Playwright (E2E)
- **Package Manager:** pnpm

## Project Map

```
src/
├── app/                           — App Router pages
│   ├── layout.tsx                 — Root layout (auth check)
│   ├── page.tsx                   — Dashboard home
│   ├── (auth)/                    — Auth group (login, reset-password)
│   └── (dashboard)/               — Protected routes group
│       ├── users/                 — /users, /users/[id]
│       ├── orders/                — /orders, /orders/[id]
│       └── analytics/             — /analytics
├── components/                    — Shared components
│   ├── ui/                        — shadcn/ui primitives (Button, Table, Dialog...)
│   ├── data-table/                — Reusable TanStack Table wrapper
│   └── charts/                    — Recharts wrappers
├── lib/                           — Utilities
│   ├── api.ts                     — Axios instance with interceptors
│   ├── auth.ts                    — next-auth config
│   └── utils.ts                   — cn(), formatCurrency(), etc.
├── hooks/                         — Custom hooks (useUsers, useOrders...)
├── types/                         — TypeScript types + Zod schemas
└── server/                        — Server actions + API route handlers

tests/
├── unit/                          — Vitest
└── e2e/                           — Playwright
```

## Commands

```bash
# Install
pnpm install

# Dev server
pnpm dev                     # → http://localhost:3000

# Build
pnpm build

# Lint + format
pnpm lint                    # ESLint
pnpm format                  # Prettier
pnpm typecheck               # tsc --noEmit

# Test
pnpm test                    # Vitest (unit)
pnpm test:e2e                # Playwright (needs dev server running)

# Storybook
pnpm storybook               # → http://localhost:6006
```

## Conventions

### TypeScript

- Strict mode. No `any` — use `unknown` and narrow.
- Prefer `type` over `interface` for data shapes.
- Zod schema → inferred type (single source of truth):

  ```ts
  const UserSchema = z.object({ id: z.string().uuid(), email: z.string().email() });
  type User = z.infer<typeof UserSchema>;
  ```

### Components

- One component per file. Named export (no default exports).
- Server Components by default. `"use client"` only when needed (state, effects, browser APIs).
- Props type: `type UserCardProps = { user: User; onDelete: (id: string) => void; }`.
- Use `cn()` from `@/lib/utils` for conditional classes.

### Data Fetching

- TanStack Query for all client-side data.
- Query keys: `["users", userId]`, `["orders", { status, page }]`.
- Server Components can fetch directly in async component body.
- Never fetch in `useEffect` — use TanStack Query.

### Naming

- Files: kebab-case (`user-card.tsx`, `use-orders.ts`).
- Components: PascalCase (`UserCard`, `OrderTable`).
- Hooks: `use` prefix (`useUsers`, `useDebounce`).
- API routes: RESTful under `src/app/api/`.

### Git

- Branch: `feature/JIRA-XXX-description`
- Commit: `feat(users): add bulk delete` (conventional commits)
- PR: Under 400 lines. Screenshot for UI changes.

## Constraints

- NO `any` types. Ever.
- NO direct `fetch()` in client components — use TanStack Query hooks.
- NO inline styles — use Tailwind classes.
- DO NOT modify `tailwind.config.ts` without discussing.
- shadcn/ui components: use `npx shadcn-ui@latest add <component>` — never hand-copy.

## External Dependencies

- **auth-service** — `https://api.panomete.com/auth` (login, token refresh)
- **user-service** — `https://api.panomete.com/users` (CRUD)
- **order-service** — `https://api.panomete.com/orders` (list, details)
- **analytics-service** — `https://api.panomete.com/analytics` (dashboards)

## Key Files to Read First

- `src/lib/api.ts` — Axios setup, base URL, interceptors, error handling
- `src/lib/auth.ts` — next-auth config, callbacks, JWT decoding
- `src/types/index.ts` — Shared types
- `src/app/layout.tsx` — Root layout, providers, auth check
