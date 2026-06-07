# AGENTS.md — nuxt-storefront (Vue 3 / Nuxt 3 Frontend)

<!-- Real example. Nuxt 3 + Vue 3 + Pinia + Tailwind. -->

## Project
- **Name:** nuxt-storefront
- **Purpose:** E-commerce storefront. Product browsing, cart, checkout. Public-facing.
- **Repo:** github.com/panomete/nuxt-storefront
- **URL:** https://shop.panomete.com

## Stack
- **Language:** TypeScript 5.x (strict mode)
- **Framework:** Nuxt 3 (Vue 3 Composition API)
- **UI:** Tailwind CSS + Nuxt UI (based on Headless UI)
- **State:** Pinia (stores) + Pinia Colada (server state / caching)
- **Auth:** nuxt-auth-utils (JWT + refresh flow)
- **Forms:** vee-validate + Zod
- **Testing:** Vitest + @vue/test-utils + Playwright (E2E)
- **Package Manager:** pnpm

## Project Map
```
├── app.vue                       — Root component
├── nuxt.config.ts                — Nuxt config, modules, runtime config
├── tailwind.config.ts            — Design tokens, theme
├── pages/                        — File-based routing
│   ├── index.vue                 — Homepage
│   ├── products/
│   │   ├── index.vue             — Product listing (+ search/filter)
│   │   └── [slug].vue            — Product detail
│   ├── cart.vue                  — Shopping cart
│   ├── checkout.vue              — Checkout flow
│   └── account/
│       ├── index.vue             — Account dashboard
│       └── orders/
│           └── [id].vue          — Order detail
├── components/                   — Reusable components
│   ├── product/
│   │   ├── ProductCard.vue
│   │   └── ProductGrid.vue
│   ├── cart/
│   │   ├── CartItem.vue
│   │   └── CartSummary.vue
│   └── ui/                       — Generic UI primitives
├── composables/                  — Shared logic
│   ├── useAuth.ts                — Login/logout, user state
│   ├── useCart.ts                — Cart CRUD
│   └── useProducts.ts            — Product queries + filters
├── stores/                       — Pinia stores
│   ├── auth.ts                   — User + token state
│   └── cart.ts                   — Cart state (persisted to localStorage)
├── server/                       — Nitro server
│   ├── api/                      — BFF (Backend-for-Frontend) routes
│   │   ├── products.get.ts       — Proxy to product-service
│   │   └── cart.post.ts          — Proxy to cart-service
│   └── middleware/                — Auth middleware
├── types/                        — TypeScript types
└── public/                       — Static assets
```

## Commands
```bash
pnpm install
pnpm dev                    # → http://localhost:3000
pnpm build
pnpm preview                # Preview production build
pnpm lint                   # ESLint
pnpm typecheck              # vue-tsc --noEmit
pnpm test                   # Vitest
pnpm test:e2e               # Playwright
```

## Conventions

### Vue 3 Composition API
- `<script setup lang="ts">` always. Composition API only — no Options API.
- `defineProps<T>()` with TypeScript types.
- Composables for reusable logic. Naming: `useXxx.ts`.
- `ref()` for primitives, `reactive()` for objects (when needed).

### Components
- PascalCase files: `ProductCard.vue`, `CartItem.vue`.
- Single-file components. Template → Script → Style (in that order).
- Props: typed via `defineProps<T>()` or Zod-to-TS.
- Emits: typed via `defineEmits<{ (e: 'delete', id: string): void }>()`.

### State Management
- Pinia stores for global state (auth, cart, UI).
- `useFetch` / `$fetch` from Nuxt for server state.
- Cart: persisted to `localStorage` via Pinia plugin.
- Never mutate store state outside of store actions.

### Routing & Pages
- Nuxt file-based routing. Directory structure = URL structure.
- `useRoute()` for params, `navigateTo()` for programmatic navigation.
- Route middleware in `middleware/` for auth guards.

### Styling
- Tailwind CSS only. No inline styles, no scoped CSS (unless unavoidable).
- Use Nuxt UI components where available (Button, Modal, Table).
- Custom design tokens in `tailwind.config.ts` — don't hardcode colors.

## Constraints
- NO Options API — Composition API only.
- NO direct `fetch()` from browser to external services — go through `server/api/` (BFF).
- NO storing tokens in localStorage — use httpOnly cookies (handled by nuxt-auth-utils).
- Cart must survive page refresh (localStorage).
- Responsive: mobile-first. Test at 375px and 1280px.

## External APIs (BFF Pattern)
```
Browser → Nuxt server/api/ → Backend services
         (BFF)              (internal network)
```
- Product data: `http://product-service:8083/api/v1/products`
- Cart: `http://cart-service:8084/api/v1/cart`
- Auth: `http://auth-service:8081/api/v1/auth`
- Orders: `http://order-service:8085/api/v1/orders`

## Key Files
- `nuxt.config.ts` — Modules, runtime config, app settings
- `composables/useAuth.ts` — Auth flow, token management
- `server/api/` — BFF proxy routes (understand data flow)
- `stores/cart.ts` — Cart logic (most complex store)
