# COMPONENT_TREE.md — Component Hierarchy & Reusability

<!--
  The component architecture. AI agents use this to:
  - Know which components exist (don't rebuild them)
  - Understand prop flow (parent → child data direction)
  - Place new components in the right spot
-->

## Component Organization

```
components/
│
├── ui/                    ← shadcn/ui primitives (auto-generated)
│   ├── button.tsx
│   ├── card.tsx
│   ├── dialog.tsx
│   ├── dropdown-menu.tsx
│   ├── input.tsx
│   ├── table.tsx
│   └── toast.tsx
│
├── layout/                ← App shell components
│   ├── AppLayout.tsx       ← Sidebar + header + main slot
│   ├── Sidebar.tsx         ← Navigation + user menu
│   ├── Header.tsx          ← Breadcrumb + actions + search
│   └── Shell.tsx           ← Page padding + max-width wrapper
│
├── data/                  ← Data display (reusable across pages)
│   ├── DataTable.tsx       ← Generic table: sorting, pagination, selection
│   ├── StatCard.tsx        ← Metric card: icon, value, trend indicator
│   ├── StatusBadge.tsx     ← Colored badge: active, pending, cancelled...
│   └── EmptyState.tsx      ← Illustration + message + CTA
│
├── feature/               ← Feature-specific (used in 1-2 places)
│   ├── product/
│   │   ├── ProductCard.tsx
│   │   ├── ProductGrid.tsx
│   │   └── ProductGallery.tsx
│   ├── order/
│   │   ├── OrderTimeline.tsx
│   │   └── OrderSummary.tsx
│   └── user/
│       ├── UserAvatar.tsx
│       └── UserMenu.tsx
│
└── shared/                ← Cross-feature reusable
    ├── ConfirmDialog.tsx
    ├── SearchInput.tsx
    └── DateRangePicker.tsx
```

## Component Props Conventions

```typescript
// ✅ Good: explicit, narrow props
type ProductCardProps = {
  product: Product;
  onAddToCart: (productId: string) => void;
  variant?: "grid" | "list";
};

// ❌ Bad: vague, wide props
type CardProps = {
  data: any;
  onClick?: () => void;
};
```

## Rules for AI Agents

1. **Check `ui/` first.** Never rebuild a shadcn/ui primitive.
2. **Check `layout/` for shell.** Don't create a new layout wrapper.
3. **Check `data/` for displays.** StatCard, DataTable, StatusBadge — use them.
4. **New feature component?** Goes in `feature/{domain}/`.
5. **Used in 3+ features?** Promote to `shared/`.
6. **Props:** TypeScript interface, one component per file, named export.
