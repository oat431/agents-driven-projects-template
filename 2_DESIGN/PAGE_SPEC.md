# PAGE_SPEC.md — Routes, Pages & Layout Patterns

<!--
  Every route in the application. AI agents use this to understand
  the full navigation tree, what each page does, and what data it needs.
-->

## Route Map

```
/                               → Landing page (public)
│
├── /login                       → Login page (public)
├── /register                    → Registration (public)
│
└── /app/                        → Authenticated layout
    │
    ├── /dashboard               → Overview: stats, recent activity, quick actions
    │
    ├── /users/                  → User management (admin only)
    │   ├── /                    → User list + search + bulk actions
    │   └── /[id]                → User detail + edit
    │
    ├── /products/               → Product catalog
    │   ├── /                    → Product grid + filters + pagination
    │   └── /[slug]              → Product detail + variants + add to cart
    │
    ├── /orders/                 → Order management
    │   ├── /                    → Order list (filterable by status)
    │   └── /[id]                → Order detail + timeline + actions
    │
    ├── /analytics/              → Charts + reports
    │
    └── /settings/               → User settings
        ├── /profile             → Edit profile
        ├── /security            → Change password, 2FA
        └── /api-keys            → Manage API keys
```

## Page Specification Template

Each page should be documented with:

### Page: <!-- e.g., /products/[slug] -->

**Route:** `/products/[slug]`  
**Layout:** Authenticated layout (sidebar + header)  
**Access:** Any authenticated user  

**Purpose:**  
<!-- One sentence. What does the user accomplish here? -->

**Data Requirements:**
| Data | Source | Loading Strategy |
|------|--------|-----------------|
| Product detail | `GET /api/v1/products/{slug}` | SSR (good for SEO) |
| Related products | `GET /api/v1/products?related={id}` | Client-side (not blocking) |
| User's cart | Pinia store | Already loaded |

**States to Handle:**
- [ ] Loading: Product skeleton (image placeholder + text lines)
- [ ] Not Found: 404 illustration + "Product not found" + back to products
- [ ] Error: Toast + retry button
- [ ] Empty (no related products): Just hide the section
- [ ] Success: Full product page

**Key Interactions:**
- Add to cart button → optimistic update + toast
- Quantity selector → debounced, max = stock count
- Image gallery → click to zoom, swipe on mobile
- Share button → copy link to clipboard

**Edge Cases:**
- Out of stock: Show "Notify me" instead of "Add to cart"
- Sale price: Show original price strikethrough + sale badge
- Long product name: Truncate at 2 lines in cards, full on detail page
