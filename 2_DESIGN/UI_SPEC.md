# UI_SPEC.md — Design Tokens & Brand Identity

<!--
  The single source of truth for visual design. AI agents read this
  to produce on-brand frontend code without guessing.
  Share this between designers and developers.
-->

## Brand Identity

### Colors

#### Primary Palette

| Token | Hex | CSS Variable | Usage |
|-------|-----|-------------|-------|
| Primary | `#7C3AED` | `--color-primary` | Main actions, links, brand accents |
| Primary Light | `#A78BFA` | `--color-primary-light` | Hover states, subtle backgrounds |
| Primary Dark | `#5B21B6` | `--color-primary-dark` | Active/pressed states |

#### Neutral Palette

| Token | Hex | CSS Variable | Usage |
|-------|-----|-------------|-------|
| Neutral 50 | `#FAFAFA` | `--color-neutral-50` | Page background |
| Neutral 100 | `#F5F5F5` | `--color-neutral-100` | Card background |
| Neutral 200 | `#E5E5E5` | `--color-neutral-200` | Borders, dividers |
| Neutral 300 | `#D4D4D4` | `--color-neutral-300` | Disabled states |
| Neutral 600 | `#525252` | `--color-neutral-600` | Body text |
| Neutral 800 | `#262626` | `--color-neutral-800` | Headings |
| Neutral 900 | `#171717` | `--color-neutral-900` | Strong emphasis |

#### Semantic Colors

| Token | Hex | Usage |
|-------|-----|-------|
| Success | `#16A34A` | Confirmation, success toasts |
| Warning | `#F59E0B` | Warnings, pending states |
| Error | `#DC2626` | Errors, destructive actions |
| Info | `#0EA5E9` | Informational alerts |

### Typography

#### Font Family
<!-- Thai-friendly recommendations -->
- **Primary (Thai + English):** `"Sarabun", "Noto Sans Thai", sans-serif`
- **Monospace:** `"JetBrains Mono", "Fira Code", monospace`
- **Fallback strategy:** System fonts for performance

#### Scale

| Level | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| H1 | 2rem / 32px | 1.2 | 700 | Page titles |
| H2 | 1.5rem / 24px | 1.3 | 600 | Section headers |
| H3 | 1.25rem / 20px | 1.4 | 600 | Card titles |
| Body L | 1.125rem / 18px | 1.6 | 400 | Lead paragraphs |
| Body | 1rem / 16px | 1.6 | 400 | Default text |
| Body S | 0.875rem / 14px | 1.5 | 400 | Captions, metadata |
| Code | 0.875rem / 14px | 1.5 | 400 | Inline code |

### Spacing Scale

```
4px  → xs   (gap between icon + text)
8px  → sm   (inner padding, tight gaps)
12px → md   (card padding, section gaps)
16px → lg   (page padding, large gaps)
24px → xl   (section separation)
32px → 2xl  (hero spacing)
48px → 3xl  (page-level separation)
64px → 4xl  (layout sections)
```

**Rule:** Always use multiples of 4px. Never use arbitrary values.

### Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| sm | 4px | Inputs, badges |
| md | 8px | Cards, buttons, modals |
| lg | 12px | Large containers |
| full | 9999px | Pills, avatars |

### Shadows

```css
--shadow-sm:  0 1px 2px rgba(0,0,0,0.05);
--shadow-md:  0 4px 6px rgba(0,0,0,0.07);
--shadow-lg:  0 10px 25px rgba(0,0,0,0.1);
```

**Rule:** Use shadows sparingly. Only to indicate elevation (cards, modals, dropdowns). Never decorative.

## Component Library

### Framework
<!-- Pick one. This tells the agent which primitives to use. -->
- **Primary:** shadcn/ui (Radix primitives + Tailwind)
- **Install:** `npx shadcn-ui@latest add button card dialog table`
- **Never:** hand-write modal, tooltip, or select logic — use shadcn/ui

### Custom Components (Project-Specific)

| Component | shadcn Base | Modifications |
|-----------|------------|---------------|
| `DataTable` | Table | + sorting, pagination, row selection |
| `StatCard` | Card | + icon slot, trend indicator |
| `UserAvatar` | Avatar | + online indicator, fallback initials |

## Layout System

### Page Structure

```
┌──────────────────────────────────────┐
│  SIDEBAR (240px)    │   CONTENT     │
│  - Logo             │               │
│  - Navigation       │   Max-width:  │
│  - User menu        │   1200px      │
│                     │               │
│  Fixed. Hidden on   │   Padding:    │
│  mobile (<768px).   │   24px (xl)   │
└──────────────────────────────────────┘
```

### Breakpoints

| Name | Width | Target |
|------|-------|--------|
| mobile | < 640px | Phone portrait |
| tablet | 640–1024px | Tablet / small laptop |
| desktop | > 1024px | Desktop |
| wide | > 1280px | Large screens |

### Container Max-Widths

- Page content: `max-w-7xl` (1280px)
- Card grid: `max-w-6xl` (1152px)
- Form: `max-w-2xl` (672px)
- Reading text: `max-w-prose` (65ch)

## Interaction Patterns

### Buttons

```
Primary:   bg-primary text-white hover:bg-primary-dark
Secondary: bg-neutral-100 text-neutral-800 hover:bg-neutral-200
Ghost:     bg-transparent hover:bg-neutral-100
Danger:    bg-error text-white hover:bg-red-700
```

Sizes: `sm` (h-8), `md` (h-10), `lg` (h-12)

### Forms

- Labels: above input. `font-medium text-sm`.
- Inputs: `h-10 rounded-md border-neutral-200 focus:border-primary focus:ring-1`
- Error: red border + red text below input
- Submit: primary button, right-aligned or full-width on mobile

### Feedback States

- **Loading:** Skeleton screens for initial load. Spinner for button actions.
- **Empty:** Illustration + "No items yet" + CTA button.
- **Error:** Toast notification (top-right). Retry button for failed fetches.

## Accessibility Requirements

- Color contrast: WCAG AA minimum (4.5:1 for text, 3:1 for large text).
- Focus states: visible ring on all interactive elements (`focus-visible:ring-2`).
- Labels: every input has a visible or aria label.
- Alt text: all non-decorative images.
- Keyboard: Tab order is logical. Enter/Space activates. Escape closes modals.

## Icons

- **Library:** Lucide React (via `lucide-react`)
- **Size:** 16px inline, 20px standalone, 24px in buttons
- **Never:** emoji as icons, mismatched icon sets
