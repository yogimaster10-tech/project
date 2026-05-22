# TESKEL — Modern Pro Clean UI/UX Specification
## Every Screen Must Look Like It Was Designed by a Top-Tier Design Agency

---

## 0. DESIGN PHILOSOPHY

```
TESKEL's UI must feel like:
  - Linear (precision, clarity, speed)
  - Vercel (developer-grade aesthetics)
  - Stripe Dashboard (data density without clutter)
  - Notion (simple surface, powerful depth)
  - Arc Browser (modern, confident, delightful)

NOT like:
  - WordPress admin (cluttered, dated, inconsistent)
  - Old SaaS dashboards (too many borders, gradients, icons)
  - Templates (generic, no personality)
  - Figma plugins (too colorful, too playful for commerce)
```

### Core Principles

1. **Whitespace is a feature.** More space = more clarity. Never fear empty space.
2. **Typography does the work.** Size, weight, and color hierarchy — not boxes and borders.
3. **Color is intentional.** Primary brand color appears < 5% of the screen. Most of the UI is neutral.
4. **Motion is subtle.** 150ms transitions. Never bouncy. Never flashy. Never slow.
5. **Information density scales.** Dashboard = dense. Landing = spacious. Checkout = focused.
6. **Dark mode is not an afterthought.** Design for both from day 1.
7. **Mobile is not a shrunk desktop.** Rethink layout per breakpoint.

---

## 1. LAYOUT SYSTEM

### 1.1 Page Anatomy

```
┌─────────────────────────────────────────────────────────────┐
│ TOPBAR (h-14, border-b, sticky top-0, z-50)                │
│ Logo | Breadcrumb                    Search | Notif | Avatar│
├──────────┬──────────────────────────────────────────────────┤
│ SIDEBAR  │ MAIN CONTENT                                    │
│ (w-64)   │                                                  │
│          │ ┌─────────────────────────────────────────────┐  │
│ Nav      │ │ PAGE HEADER                                 │  │
│ items    │ │ Title + Description + Actions               │  │
│          │ ├─────────────────────────────────────────────┤  │
│          │ │                                             │  │
│          │ │ PAGE BODY                                   │  │
│          │ │ (max-w-6xl mx-auto px-6 py-8)              │  │
│          │ │                                             │  │
│          │ │                                             │  │
│          │ └─────────────────────────────────────────────┘  │
├──────────┴──────────────────────────────────────────────────┤
│ (no footer in dashboard)                                    │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Spacing Rules

```
Page padding:     px-6 (24px) on desktop, px-4 (16px) on mobile
Section gap:      space-y-8 (32px) between major sections
Card padding:     p-6 (24px)
Card gap:         gap-4 (16px) between cards in grid
Form field gap:   space-y-4 (16px) between fields
Table row height: h-12 (48px) minimum
Button height:    h-9 (36px) default, h-10 (40px) large, h-8 (32px) small
Input height:     h-10 (40px)
Icon size:        16px (inline), 20px (button), 24px (nav)
Avatar size:      32px (small), 40px (medium), 64px (large)
```

### 1.3 Grid System

```
Dashboard pages:
  - Stats:    grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4
  - Content:  grid-cols-1 lg:grid-cols-3 gap-6 (2/3 main + 1/3 sidebar)
  - Products: grid-cols-1 sm:grid-cols-2 xl:grid-cols-3 gap-4
  - Settings: max-w-2xl (centered, narrow)

Storefront:
  - Product grid: grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6
  - Product detail: grid-cols-1 lg:grid-cols-5 gap-8 (3/5 info + 2/5 buy box)

Landing:
  - Hero: full-width, max-w-4xl text center
  - Features: grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8
  - Pricing: grid-cols-1 md:grid-cols-3 gap-6
```

---

## 2. COMPONENT VISUAL SPEC

### 2.1 Buttons

```
Variants:
  default     — bg-primary text-primary-foreground
  secondary   — bg-secondary text-secondary-foreground
  outline     — border border-input bg-background hover:bg-accent
  ghost       — hover:bg-accent hover:text-accent-foreground
  destructive — bg-destructive text-destructive-foreground
  link        — text-primary underline-offset-4 hover:underline

Sizes:
  sm  — h-8 px-3 text-xs rounded-md
  md  — h-9 px-4 text-sm rounded-md (default)
  lg  — h-10 px-6 text-sm rounded-md
  xl  — h-12 px-8 text-base rounded-lg (landing CTA only)
  icon — h-9 w-9 rounded-md

States:
  hover:    brightness +5%, subtle scale(1.01)
  active:   scale(0.98), 100ms
  disabled: opacity-50, cursor-not-allowed
  loading:  spinner icon replacing text, same width (no layout shift)

Rules:
  - Primary button: MAX 1 per visible section
  - Destructive: Only for irreversible actions (delete, cancel subscription)
  - Icon + text: icon-left for actions, icon-right for navigation
  - Full-width buttons: Only on mobile or in modals
```

### 2.2 Cards

```
Base card:
  rounded-lg border bg-card text-card-foreground shadow-sm

Variants:
  default   — border-border (subtle gray)
  elevated  — shadow-md (for highlighted/featured items)
  outlined  — border-2 border-primary (selected state)
  ghost     — no border, no shadow (used in grids where container provides visual grouping)

Anatomy:
  ┌─────────────────────────────────────┐
  │ [Optional: Image / Cover]           │
  ├─────────────────────────────────────┤
  │ p-6                                 │
  │                                     │
  │ Title (text-base font-semibold)     │
  │ Description (text-sm text-muted)    │
  │                                     │
  │ [Optional: Metrics / Tags]          │
  │                                     │
  ├─────────────────────────────────────┤
  │ [Optional: Footer with actions]     │
  │ px-6 py-4 border-t                  │
  └─────────────────────────────────────┘

Rules:
  - Never nest cards inside cards
  - Max 3 lines of text in card body (truncate with line-clamp)
  - Hover: translateY(-1px) + shadow-md transition 150ms
  - Click: entire card clickable (cursor-pointer, no separate button unless needed)
```

### 2.3 Tables

```
Structure:
  ┌──────────────────────────────────────────────────────────────────┐
  │ TABLE HEADER                                                     │
  │ [Search input]  [Filter dropdowns]  [Bulk actions]  [+ Create]  │
  ├──────────────────────────────────────────────────────────────────┤
  │ ☐ │ Product     │ Status  │ Revenue │ Sales │ Created  │ •••   │
  ├───┼─────────────┼─────────┼─────────┼───────┼──────────┼───────┤
  │ ☐ │ Template Kit│ ● Active│ $1,240  │ 47    │ May 12   │ •••   │
  │ ☐ │ Prompt Pack │ ○ Draft │ $0      │ 0     │ May 18   │ •••   │
  │ ☐ │ Plugin Pro  │ ● Active│ $890    │ 23    │ Apr 3    │ •••   │
  ├──────────────────────────────────────────────────────────────────┤
  │ Showing 1-10 of 24                          < 1 2 3 >           │
  └──────────────────────────────────────────────────────────────────┘

Styling:
  - Header row: text-xs font-medium text-muted-foreground uppercase tracking-wide
  - Data rows: text-sm, h-12, border-b last:border-b-0
  - Hover row: bg-muted/50
  - Selected row: bg-primary/5
  - Status dots: ● green (active), ○ gray (draft), ● red (error)
  - Numbers: tabular-nums (monospace for alignment)
  - Actions: ••• dropdown (Edit, Duplicate, Archive, Delete)
  - Empty state: centered illustration + "No products yet" + CTA button

Rules:
  - Always provide search/filter for lists > 10 items
  - Pagination: 10/20/50 per page
  - Column sort: click header to sort (↑↓ indicator)
  - Responsive: On mobile, switch to card layout (not horizontal scroll)
  - Loading: skeleton rows (3-5 rows), same structure as data
```

### 2.4 Forms

```
Field anatomy:
  ┌────────────────────────────────────────┐
  │ Label (text-sm font-medium)            │
  │ [Optional: Helper text below label]    │
  │                                        │
  │ ┌────────────────────────────────────┐ │
  │ │ Input (h-10 px-3 rounded-md)       │ │
  │ └────────────────────────────────────┘ │
  │                                        │
  │ [Optional: Error message / char count] │
  └────────────────────────────────────────┘

Input states:
  default:  border-input
  focus:    ring-2 ring-ring ring-offset-2
  error:    border-destructive ring-destructive
  disabled: opacity-50 bg-muted

Form patterns:
  - Inline validation (on blur, not on every keystroke)
  - Error messages appear below field, text-sm text-destructive
  - Required: red asterisk after label (*)
  - Optional: (optional) text after label in muted
  - Character count: show when input has maxLength (right-aligned, text-xs)
  - Multi-step forms: progress bar at top, step indicators

Submit button:
  - Right-aligned for forms
  - Loading state on submit (spinner + "Saving...")
  - Disable while loading
  - Success: green check + "Saved!" (auto-dismiss 2s)
  - Error: toast notification with error message

Rules:
  - Max form width: max-w-lg (for settings forms)
  - Never more than 7 fields visible at once (use tabs/steps for complex forms)
  - Labels always above inputs (not inline, not floating)
  - Placeholder: example value, not instruction (placeholder="john@email.com" not "Enter email")
```

### 2.5 Modals & Dialogs

```
Sizes:
  sm: max-w-sm (confirmation, simple input)
  md: max-w-lg (forms, detail views) — DEFAULT
  lg: max-w-2xl (complex editors)
  xl: max-w-4xl (full editors, previews)
  full: inset-4 (almost fullscreen)

Anatomy:
  ┌─────────────────────────────────────────┐
  │ Title              [X close]            │
  │ Description (text-sm text-muted)        │
  ├─────────────────────────────────────────┤
  │                                         │
  │ BODY (max-h-[70vh] overflow-y-auto)     │
  │                                         │
  ├─────────────────────────────────────────┤
  │              [Cancel]  [Confirm]        │
  └─────────────────────────────────────────┘

Animation:
  - Backdrop: fade-in 150ms (bg-black/50)
  - Content: fade-in + scale from 0.95 → 1.0, 150ms ease-out
  - Close: fade-out 100ms

Rules:
  - Close on Escape key
  - Close on backdrop click (except for forms with unsaved changes)
  - Focus trap (keyboard can't leave modal)
  - Scroll lock on body when modal open
  - Never stack modals (modal within modal = bad UX, use drawer or navigation instead)
  - Destructive confirm: require typing resource name ("Type 'delete' to confirm")
```

### 2.6 Navigation

```
SIDEBAR (Desktop):
  ┌────────────────────┐
  │ 🟣 TESKEL          │ ← Logo + wordmark (h-14)
  ├────────────────────┤
  │                    │
  │ ▸ Dashboard        │ ← Active: bg-accent text-accent-foreground font-medium
  │   Products         │ ← Default: text-muted-foreground hover:text-foreground
  │   Orders           │
  │   Customers        │
  │   ──────────       │ ← Separator (my-2)
  │   Funnels          │
  │   Emails           │
  │   Affiliates       │
  │   ──────────       │
  │   Analytics        │
  │   Marketplace      │
  │   ──────────       │
  │   Settings         │
  │                    │
  ├────────────────────┤
  │ 📊 Pro Plan        │ ← Plan badge (bottom)
  │ [Org Switcher ▼]   │ ← Multi-org dropdown
  └────────────────────┘

  Width: w-64 (expanded), w-16 (collapsed, icon-only)
  Toggle: user preference, persisted in localStorage
  Mobile: hidden by default, opened via hamburger as sheet (slide from left)

TOPBAR:
  ┌────────────────────────────────────────────────────────────────┐
  │ [≡] Breadcrumb: Dashboard / Products / Template Kit            │
  │                                                                │
  │                    [⌘K Search] [🔔 3] [Avatar ▼]              │
  └────────────────────────────────────────────────────────────────┘

  - Breadcrumb: clickable segments, current page not linked
  - Search: Cmd+K shortcut, opens command palette
  - Notifications: bell icon + red badge (unread count)
  - Avatar: dropdown → Profile, Appearance, Logout

BOTTOM NAV (Mobile only, < 768px):
  ┌────────┬────────┬────────┬────────┬────────┐
  │  Home  │Products│ Orders │Analytics│  More  │
  │  🏠   │  📦   │  🛒   │  📊   │  •••  │
  └────────┴────────┴────────┴────────┴────────┘
```

### 2.7 Toast Notifications

```
Library: sonner (built for Next.js)

Position: bottom-right (desktop), bottom-center (mobile)

Types:
  success — ✓ green icon, "Product created successfully"
  error   — ✗ red icon, "Failed to save. Please try again."
  warning — ⚠ yellow icon, "Your trial expires in 3 days"
  info    — ℹ blue icon, "New version available"
  loading — spinner, "Saving changes..."

Anatomy:
  ┌───────────────────────────────────────────┐
  │ ✓  Product created successfully     [×]  │
  │    View product →                         │
  └───────────────────────────────────────────┘

Rules:
  - Auto-dismiss: 4s (success/info), 6s (warning), never auto (error)
  - Max visible: 3 toasts stacked
  - Action link: optional, right side or second line
  - Never use toast for form validation errors (show inline)
  - Always use toast for: save success, delete success, copy success, async operation complete
```

---

## 3. PAGE-BY-PAGE VISUAL SPEC

### 3.1 Landing Page

```
Structure:
  ─────────────────────────────────────────────────
  NAV (sticky, transparent → white on scroll)
  Logo | Features | Pricing | Docs | [Login] [Get Started ➜]
  ─────────────────────────────────────────────────

  HERO (min-h-[80vh], centered)
  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │          The Digital Commerce                           │
  │          Operating System                               │
  │                                                         │
  │   Sell downloads, licenses, subscriptions, services,    │
  │   and more — with one platform.                         │
  │                                                         │
  │       [Get Started Free]  [See Demo →]                  │
  │                                                         │
  │   Trusted by 2,400+ creators · $3.2M+ processed        │
  │                                                         │
  │   ┌───────────────────────────────────────────────┐     │
  │   │           Dashboard Screenshot                │     │
  │   │         (with subtle shadow + tilt)           │     │
  │   └───────────────────────────────────────────────┘     │
  │                                                         │
  └─────────────────────────────────────────────────────────┘

  SOCIAL PROOF
  ┌─────────────────────────────────────────────────────────┐
  │  "Used by creators at"                                  │
  │  [Google] [Meta] [Stripe] [Vercel] [Notion] [Linear]    │
  └─────────────────────────────────────────────────────────┘

  FEATURES (grid-cols-3)
  ┌───────────────┬───────────────┬───────────────┐
  │ 📦 13 Types   │ 🔑 License API│ 🛍️ Storefront │
  │ Downloads,    │ Validate,     │ Beautiful     │
  │ licenses,     │ activate,     │ custom store  │
  │ SaaS, more    │ manage keys   │ in seconds    │
  ├───────────────┼───────────────┼───────────────┤
  │ 🔄 Funnels    │ 🌍 Global     │ 🧩 Embed      │
  │ Order bumps,  │ Multi-currency│ Sell on any   │
  │ upsells       │ PPP pricing   │ website       │
  ├───────────────┼───────────────┼───────────────┤
  │ 📊 Analytics  │ 📧 Emails     │ 🤝 Affiliates │
  │ Real-time     │ Sequences,    │ Track, pay    │
  │ revenue data  │ automation    │ commissions   │
  └───────────────┴───────────────┴───────────────┘

  PRODUCT SHOWCASE (horizontal scroll or tabs)
  [Downloads] [Licenses] [Subscriptions] [Services] [Bundles]
  Each tab: screenshot of that product type in dashboard + store

  PRICING (grid-cols-3)
  ┌──────────────┬──────────────────┬──────────────┐
  │ Free         │ Pro ✦ (popular)  │ Scale        │
  │ $0/mo        │ $29/mo           │ $79/mo       │
  │              │ ────────────     │              │
  │ 5% fee      │ 3% fee           │ 1.5% fee     │
  │ 3 products  │ Unlimited        │ Unlimited    │
  │ Basic store │ Custom domain    │ White-label  │
  │ Email only  │ + Funnels        │ + API/SDK    │
  │             │ + Affiliates     │ + Priority   │
  │             │ + Email sequences│ + SLA        │
  │             │                  │              │
  │ [Start Free]│ [Start Pro ➜]    │ [Contact]    │
  └──────────────┴──────────────────┴──────────────┘
  Note: Popular card has border-2 border-primary + "Most Popular" badge

  TESTIMONIALS (carousel)
  ┌─────────────────────────────────────────────────────────┐
  │ ⭐⭐⭐⭐⭐                                               │
  │ "TESKEL replaced Gumroad + ConvertKit + ThriveCart      │
  │  for me. One platform, lower fees, better UX."          │
  │                                                         │
  │ 🧑 Sarah Chen — @sarahcodes · $24K/mo revenue          │
  └─────────────────────────────────────────────────────────┘

  CTA (full-width gradient bg)
  ┌─────────────────────────────────────────────────────────┐
  │                                                         │
  │   Ready to sell smarter?                                │
  │   Start free. No credit card required.                  │
  │                                                         │
  │   [Get Started Free ➜]                                  │
  │                                                         │
  └─────────────────────────────────────────────────────────┘

  FOOTER
  ┌─────────────────────────────────────────────────────────┐
  │ TESKEL           Product      Resources     Legal       │
  │ Digital Commerce Features     Docs          Terms       │
  │ Operating System Pricing      Blog          Privacy     │
  │                  Changelog    Help Center   DPA         │
  │ [Twitter][GitHub]API Docs     Status                    │
  └─────────────────────────────────────────────────────────┘

Visual style:
  - Background: pure white (light) or #09090b (dark)
  - Hero gradient: subtle radial gradient (brand-100 → transparent) behind text
  - Screenshot: with border + rounded-xl + shadow-2xl + slight perspective rotate
  - Feature icons: Lucide icons, 24px, inside rounded-lg bg-primary/10 p-3 container
  - Animations: fade-in-up on scroll (framer-motion, once, 0.5s)
```

### 3.2 Dashboard Overview

```
┌─────────────────────────────────────────────────────────────────┐
│ Dashboard                                       [This month ▼]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│ │ Revenue  │ │ Orders   │ │ Customers│ │ Views    │           │
│ │ $4,230   │ │ 124      │ │ 89       │ │ 3,420   │           │
│ │ +12.5% ↑ │ │ +8.3% ↑  │ │ +15.2% ↑ │ │ -2.1% ↓ │           │
│ │ ▁▂▃▄▅▆▇█│ │ ▁▂▃▅▃▆▇▅│ │ ▁▁▂▃▃▄▅▆│ │ ▇▆▅▅▄▃▃▂│           │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
│                                                                 │
│ ┌───────────────────────────────┐ ┌───────────────────────────┐ │
│ │ Revenue Chart (area)          │ │ Live Activity             │ │
│ │                               │ │                           │ │
│ │ $                             │ │ 🟢 LIVE                   │ │
│ │ ████                          │ │                           │ │
│ │ ████████                      │ │ 🛒 $19 — Template Kit     │ │
│ │ ████████████                  │ │ ⭐ 5★ review received     │ │
│ │ ████████████████              │ │ 🔑 License activated      │ │
│ │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │ │ 💳 $47 — Service Pkg     │ │
│ │ Jan  Feb  Mar  Apr  May  Jun  │ │                           │ │
│ └───────────────────────────────┘ └───────────────────────────┘ │
│                                                                 │
│ ┌───────────────────────────────┐ ┌───────────────────────────┐ │
│ │ Top Products                  │ │ Recent Orders             │ │
│ │                               │ │                           │ │
│ │ 1. Template Kit    $1,240     │ │ #1024 · $19 · john@..    │ │
│ │ 2. Prompt Pack     $890       │ │ #1023 · $47 · sarah@..   │ │
│ │ 3. Plugin Pro      $650       │ │ #1022 · $9  · mike@..    │ │
│ │ 4. Course Access   $420       │ │ #1021 · $29 · anna@..    │ │
│ │ 5. Design System   $310       │ │ #1020 · $19 · li@..      │ │
│ │                               │ │                           │ │
│ │ [View all →]                  │ │ [View all →]              │ │
│ └───────────────────────────────┘ └───────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Visual rules:
  - Stat cards: border, rounded-lg, p-6, sparkline chart at bottom (h-8, no axes)
  - Percentage change: green ↑ (positive), red ↓ (negative), gray = (neutral)
  - Revenue chart: area chart with gradient fill (brand-500 → transparent)
  - Live activity: real-time WebSocket feed, max 10 items, scrollable
  - Top products: horizontal bar chart or simple list with revenue number
  - Recent orders: compact table, click to navigate to order detail
  - Date range picker: "Today | 7d | 30d | 90d | Custom" pills
```

### 3.3 Product List

```
┌─────────────────────────────────────────────────────────────────┐
│ Products                                    [+ New Product]     │
│                                                                 │
│ [🔍 Search products...]  [Status ▼]  [Type ▼]  [Sort ▼]       │
│                                                                 │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │ ☐ │ 📄│ Template Kit      │ ● Active │ $19   │ 47 sales │⋯│ │
│ │ ☐ │ 📄│ Prompt Pack       │ ○ Draft  │ $9    │ 0 sales  │⋯│ │
│ │ ☐ │ 🔑│ Plugin License    │ ● Active │ $49   │ 23 sales │⋯│ │
│ │ ☐ │ 🔄│ SaaS Subscription │ ● Active │ $29/mo│ 12 active│⋯│ │
│ │ ☐ │ 📦│ Mega Bundle       │ ● Active │ $99   │ 8 sales  │⋯│ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ Showing 1-5 of 5 products                                       │
└─────────────────────────────────────────────────────────────────┘

Product type icons:
  📄 download
  🔑 license
  💬 prompt_pack
  👥 community
  🔄 micro_saas (subscription)
  📅 service
  📦 bundle
  💸 pwyw (pay what you want)
  📊 tiered
  🔗 link
  📚 collection
  🤖 agent
  🎓 course

Empty state (no products):
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │            [Illustration: empty box with sparkles]          │
  │                                                             │
  │            No products yet                                  │
  │            Create your first digital product                │
  │            and start selling in minutes.                    │
  │                                                             │
  │            [+ Create Product]                               │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

### 3.4 Product Create/Edit

```
┌─────────────────────────────────────────────────────────────────┐
│ ← Back to Products                                              │
│                                                                 │
│ New Product                             [Save Draft] [Publish]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ ┌──────────────────────────────────┐ ┌────────────────────────┐ │
│ │ MAIN (2/3)                       │ │ SIDEBAR (1/3)          │ │
│ │                                  │ │                        │ │
│ │ Product Type                     │ │ Status                 │ │
│ │ [Download ▼]                     │ │ [● Draft ▼]            │ │
│ │                                  │ │                        │ │
│ │ Name *                           │ │ Visibility             │ │
│ │ [Notion Finance Template    ]    │ │ [🌐 Public ▼]          │ │
│ │                                  │ │                        │ │
│ │ Description                      │ │ ─────────────          │ │
│ │ ┌──────────────────────────────┐ │ │                        │ │
│ │ │ Rich text editor (TipTap)   │ │ │ Cover Image            │ │
│ │ │ B I U • Link • List • Code  │ │ │ ┌──────────────────┐   │ │
│ │ │                              │ │ │ │  [Drop image or  │   │ │
│ │ │                              │ │ │ │   click to upload]│   │ │
│ │ └──────────────────────────────┘ │ │ └──────────────────┘   │ │
│ │                                  │ │                        │ │
│ │ ─── Pricing ───────────────────  │ │ Category               │ │
│ │                                  │ │ [Templates ▼]          │ │
│ │ Pricing Model                    │ │                        │ │
│ │ [One-time ▼]                     │ │ Tags                   │ │
│ │                                  │ │ [notion] [finance] [+] │ │
│ │ Price *                          │ │                        │ │
│ │ [$] [19.00] [USD ▼]             │ │ ─────────────          │ │
│ │                                  │ │                        │ │
│ │ Compare at (strikethrough)       │ │ Marketplace            │ │
│ │ [$] [29.00]                      │ │ [✓] List on explore    │ │
│ │                                  │ │                        │ │
│ │ ─── Files ────────────────────   │ │ PPP Pricing            │ │
│ │                                  │ │ [✓] Enable regional    │ │
│ │ ┌──────────────────────────────┐ │ │                        │ │
│ │ │ 📎 template-v2.zip  (4.2MB) │ │ │                        │ │
│ │ │    [Replace] [Remove]        │ │ │                        │ │
│ │ └──────────────────────────────┘ │ │                        │ │
│ │ [+ Upload file]                  │ │                        │ │
│ │                                  │ │                        │ │
│ └──────────────────────────────────┘ └────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Visual rules:
  - 2-column layout on desktop (main + sidebar)
  - Single column on mobile (sidebar collapses below)
  - Section dividers: text-xs uppercase text-muted-foreground tracking-wide + hr
  - File upload: drag-and-drop zone, progress bar during upload
  - Price input: currency symbol prefix (inside input, not separate)
  - Autosave: every 30s for draft, show "Saved" timestamp in header
```

### 3.5 Checkout Page

```
┌─────────────────────────────────────────────────────────────────┐
│                       TESKEL CHECKOUT                            │
│                                                                 │
│ ┌──────────────────────────────┐ ┌──────────────────────────┐   │
│ │ PAYMENT (left)               │ │ ORDER SUMMARY (right)    │   │
│ │                              │ │                          │   │
│ │ Email *                      │ │ ┌──────────────────────┐ │   │
│ │ [your@email.com         ]    │ │ │ 🖼 Product Image     │ │   │
│ │                              │ │ │                      │ │   │
│ │ ┌──────────────────────────┐ │ │ │ Template Kit         │ │   │
│ │ │ 🍎 Apple Pay             │ │ │ │ by @creator          │ │   │
│ │ └──────────────────────────┘ │ │ └──────────────────────┘ │   │
│ │ ┌──────────────────────────┐ │ │                          │   │
│ │ │ G  Google Pay            │ │ │ Subtotal      $19.00    │   │
│ │ └──────────────────────────┘ │ │ Discount      -$5.00    │   │
│ │                              │ │ ──────────────────────── │   │
│ │ ─── or pay with card ─────  │ │ Total         $14.00    │   │
│ │                              │ │                          │   │
│ │ Card number                  │ │ ┌──────────────────────┐ │   │
│ │ [4242 4242 4242 4242    ]    │ │ │ 🎁 ORDER BUMP        │ │   │
│ │                              │ │ │ Add Prompt Pack +$9  │ │   │
│ │ Expiry        CVC            │ │ │ [☐ Add to order]     │ │   │
│ │ [12/28]       [123]          │ │ └──────────────────────┘ │   │
│ │                              │ │                          │   │
│ │ Discount code                │ │ 🔒 Secured by Stripe    │   │
│ │ [SAVE20    ] [Apply]         │ │ 💯 30-day guarantee     │   │
│ │ ✓ Applied! -$5.00           │ │                          │   │
│ │                              │ │ 🌍 Regional pricing     │   │
│ │ ┌──────────────────────────┐ │ │    applied (-55%)       │   │
│ │ │                          │ │ │                          │   │
│ │ │    [Pay $14.00 ➜]        │ │ │                          │   │
│ │ │                          │ │ │                          │   │
│ │ └──────────────────────────┘ │ │                          │   │
│ │                              │ │                          │   │
│ └──────────────────────────────┘ └──────────────────────────┘   │
│                                                                 │
│              847 people have purchased this product              │
└─────────────────────────────────────────────────────────────────┘

Visual rules:
  - Clean white background (no distractions)
  - Max-w-4xl centered
  - No navigation links (prevent exit)
  - Trust badges below pay button
  - Social proof at bottom
  - Order bump: highlighted card with dashed border + subtle bg color
  - Mobile: single column, summary at top (collapsed, expandable)
  - Pay button: full-width, h-12, bg-primary, font-semibold, rounded-lg
  - Loading during payment: button shows spinner, all inputs disabled
```

### 3.6 Storefront

```
┌─────────────────────────────────────────────────────────────────┐
│ NAV: [Logo/Store Name]                         [Search] [Cart]  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ HERO (creator profile)                                          │
│ ┌─────────────────────────────────────────────────────────────┐ │
│ │                                                             │ │
│ │   [Avatar 64px]                                             │ │
│ │   Creator Name                                              │ │
│ │   Building tools for designers · 12 products · ⭐ 4.8       │ │
│ │                                                             │ │
│ │   [Twitter] [Website] [Email]                               │ │
│ │                                                             │ │
│ └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
│ PRODUCTS (grid)                                                 │
│ ┌───────────────┐ ┌───────────────┐ ┌───────────────┐          │
│ │ [Cover Image] │ │ [Cover Image] │ │ [Cover Image] │          │
│ │               │ │               │ │               │          │
│ │ Template Kit  │ │ Prompt Pack   │ │ Plugin Pro    │          │
│ │ ⭐ 4.9 (47)   │ │ ⭐ 4.7 (12)   │ │ ⭐ 5.0 (8)    │          │
│ │               │ │               │ │               │          │
│ │ $19.00        │ │ $9.00         │ │ $49.00        │          │
│ └───────────────┘ └───────────────┘ └───────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Product card:
  - Aspect ratio: 16:9 for cover image
  - Rounded-lg overflow-hidden
  - Hover: shadow-md + scale(1.02) transition 150ms
  - Price: font-semibold text-lg
  - Rating: stars + count in parentheses
  - Compare price: line-through text-muted (if discounted)
  - Badge: "New" (< 7 days), "Popular" (top 10%)

Store theme (creator-customizable):
  - Primary color (applies to buttons, links, accents)
  - Font choice: Inter (default), Plus Jakarta Sans, DM Sans, Space Grotesk
  - Layout: grid (default) or list
  - Header style: minimal (default) or full-width cover image
  - Dark/light mode toggle available for buyers
```

### 3.7 Success Page

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                   🎉 (confetti animation)                       │
│                                                                 │
│               Thank you for your purchase!                      │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                                                         │   │
│   │  📦 Template Kit                                        │   │
│   │  Order #TSKL-1024                                       │   │
│   │                                                         │   │
│   │  ┌─────────────────────────────────────────────────┐    │   │
│   │  │                                                 │    │   │
│   │  │   [⬇ Download template-v2.zip (4.2MB)]         │    │   │
│   │  │                                                 │    │   │
│   │  └─────────────────────────────────────────────────┘    │   │
│   │                                                         │   │
│   │  A receipt has been sent to john@email.com              │   │
│   │                                                         │   │
│   │  ─────────────────────────────────────────────────      │   │
│   │                                                         │   │
│   │  🎁 SPECIAL OFFER                                       │   │
│   │  Get Prompt Pack for 50% off (only for you!)            │   │
│   │  $9 → $4.50                                             │   │
│   │  [Add to order — $4.50 ➜]     [No thanks]              │   │
│   │                                                         │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Share your purchase:                                          │
│   [🐦 Tweet] [📋 Copy link] [💬 WhatsApp]                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

Visual rules:
  - Confetti: canvas-confetti library, 2s burst on mount
  - Download button: large, prominent, bg-primary
  - Upsell: card with dashed border + bg-amber-50 (attention, not annoying)
  - Share buttons: ghost style, row
  - Receipt info: text-sm text-muted
```

---

## 4. EMPTY STATES, LOADING, ERROR

### 4.1 Empty States

```
Every list/table page must have a designed empty state:

Pattern:
  [Illustration (64px, muted)]
  [Heading: "No [items] yet"]
  [Description: "Create your first [item] to get started."]
  [CTA Button: "+ Create [Item]"]

Examples:
  Products: Package illustration + "No products yet" + "Create your first digital product and start selling in minutes."
  Orders: Receipt illustration + "No orders yet" + "Orders will appear here when customers buy your products."
  Customers: People illustration + "No customers yet" + "Your customer list will grow as you make sales."
  Analytics: Chart illustration + "Not enough data" + "Analytics will populate after your first sale."

Rules:
  - Illustration: simple line art, brand-muted color
  - Text: centered, max-w-sm
  - Never show empty table headers with no rows
  - Always provide actionable CTA
```

### 4.2 Loading States

```
Skeleton pattern:
  - Match exact layout of loaded content
  - Use animate-pulse (opacity 0.5 → 1, 1.5s infinite)
  - Rounded shapes matching text/image dimensions
  - Show 3-5 skeleton rows for tables
  - Show skeleton for each stat card

Example (Product list loading):
  ┌─────────────────────────────────────────────────────────────┐
  │ ░░░░░░░░░░░░░░ │ ░░░░░░░│ ░░░░│ ░░░░░░░│                  │
  │ ░░░░░░░░░░░░   │ ░░░░░  │ ░░░░│ ░░░░░  │                  │
  │ ░░░░░░░░░░░░░░░│ ░░░░░░░│ ░░░░│ ░░░░░░░│                  │
  └─────────────────────────────────────────────────────────────┘

Rules:
  - NEVER use full-page spinners
  - Skeleton must appear within 100ms (no blank screen)
  - Use React Suspense + loading.tsx files per route
  - Charts: show axis labels + empty chart area with skeleton grid lines
  - Images: solid bg-muted rounded placeholder → fade-in image on load
```

### 4.3 Error States

```
API Error:
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  ⚠️ Something went wrong                                   │
  │  We couldn't load your products. Please try again.          │
  │                                                             │
  │  [Retry]  [Go to dashboard]                                 │
  │                                                             │
  │  Error: FETCH_FAILED (for devs: expand for details)         │
  └─────────────────────────────────────────────────────────────┘

404 Page:
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  404                                                        │
  │  This page doesn't exist.                                   │
  │                                                             │
  │  [Go home]  [Back]                                          │
  └─────────────────────────────────────────────────────────────┘

Form validation error:
  - Inline below field: text-sm text-destructive
  - Input border: border-destructive
  - Focus: ring-destructive
  - Never use alert() or modal for validation errors

Rules:
  - Error boundaries at route level (catch rendering errors)
  - Show actionable recovery (retry button, navigation link)
  - Log error to Sentry (not visible to user)
  - Never show raw error messages / stack traces to users
```

---

## 5. INTERACTION PATTERNS

### 5.1 Data Mutation Feedback

```
Pattern: Optimistic Update + Toast

User action → Immediate UI update → Background API call
  ├─ Success → Toast "Saved ✓" (2s, auto-dismiss)
  └─ Failure → Revert UI + Toast "Failed. Please try again." (persistent until dismissed)

Examples:
  Toggle product status:
    Click toggle → toggle switches instantly → API PATCH → toast on success/failure

  Delete product:
    Click delete → confirm dialog → fade out row → API DELETE → toast "Product archived"
    (Note: don't remove from DOM until API confirms, just add opacity-50)

  Reorder items:
    Drag and drop → visual reorder instantly → API PATCH order → toast on success
```

### 5.2 Keyboard Shortcuts

```
Global:
  Cmd+K         — Command palette (search everything)
  Cmd+/         — Toggle sidebar
  Cmd+N         — New product (context-aware)
  Cmd+S         — Save current form
  Escape        — Close modal/drawer/palette

Table:
  ↑/↓           — Navigate rows
  Enter         — Open selected row
  Delete/Backspace — Archive selected (with confirm)
  Cmd+A         — Select all

Editor:
  Cmd+B         — Bold
  Cmd+I         — Italic
  Cmd+K         — Insert link
  Cmd+Shift+C   — Code block

Show shortcuts: ? key opens shortcut reference sheet (like GitHub)
```

### 5.3 Drag and Drop

```
Used for:
  - Product file ordering
  - Funnel step ordering
  - Bundle item ordering
  - Navigation menu ordering (storefront)
  - Dashboard widget rearranging (Phase 2)

Library: @dnd-kit/core (accessible, performant)

Visual feedback:
  - Drag handle: ⠿ icon (6 dots), visible on hover
  - Dragging: item lifts with shadow-lg + scale(1.02) + opacity-90
  - Drop zone: dashed border + bg-primary/5 highlight
  - Drop: item settles into place with 150ms ease
```

---

## 6. COLOR USAGE RULES

```
Primary (brand-500, indigo):
  USE FOR: Primary buttons, active nav items, links, focus rings, chart accents
  MAX: 5% of screen real estate

Success (green-500):
  USE FOR: Active status dots, positive metrics, success toasts, checkmarks
  NEVER FOR: Buttons (use primary instead)

Warning (amber-500):
  USE FOR: Warning badges, expiring items, caution messages
  NEVER FOR: Decorative elements

Error (red-500):
  USE FOR: Error messages, destructive buttons, negative metrics, validation errors
  NEVER FOR: Non-error states

Neutral (gray scale):
  USE FOR: Everything else — text, borders, backgrounds, icons
  This is 90% of the UI

Text hierarchy:
  text-foreground      — Primary text (headings, important content)
  text-muted-foreground — Secondary text (descriptions, labels, timestamps)
  text-muted-foreground/60 — Tertiary (placeholders, disabled)

Background hierarchy:
  bg-background  — Page background (white / #09090b)
  bg-card        — Card surfaces (same as bg in light, slightly lighter in dark)
  bg-muted       — Subtle highlights, hover states, code blocks
  bg-accent      — Active states, selected items
```

---

## 7. ICON SYSTEM

```
Library: Lucide React (consistent, MIT, 1000+ icons)

Size rules:
  12px — Inline text indicators (sort arrows, status dots)
  16px — Inside buttons with text, table row actions
  20px — Sidebar nav items, card decorations
  24px — Page section headers, feature cards
  32px — Empty state illustrations (outlined style)
  48px — Onboarding step icons

Stroke width: 2px (default Lucide)
Color: currentColor (inherits from text color)

Consistent icon choices:
  Dashboard    → LayoutDashboard
  Products     → Package
  Orders       → ShoppingCart
  Customers    → Users
  Analytics    → BarChart3
  Funnels      → GitBranch
  Emails       → Mail
  Affiliates   → Link2
  Marketplace  → Globe
  Settings     → Settings
  Search       → Search
  Notifications→ Bell
  Help         → HelpCircle
  Logout       → LogOut
  Create       → Plus
  Edit         → Pencil
  Delete       → Trash2
  Archive      → Archive
  Download     → Download
  Upload       → Upload
  External     → ExternalLink
  Copy         → Copy
  Check        → Check
  Close        → X
  ChevronDown  → ChevronDown
  More         → MoreHorizontal
  Sort         → ArrowUpDown
  Filter       → Filter
  Calendar     → Calendar
  Clock        → Clock
  Star         → Star
  Heart        → Heart
  Share        → Share2
  Lock         → Lock
  Unlock       → Unlock
  Eye          → Eye
  EyeOff       → EyeOff
  Refresh      → RefreshCw
  Warning      → AlertTriangle
  Info         → Info
  Success      → CheckCircle2
  Error        → XCircle
```

---

## 8. CHART & DATA VISUALIZATION

```
Library: Recharts (React-native, responsive, customizable)

Chart types used:
  Area chart    — Revenue over time (gradient fill)
  Bar chart     — Product comparison, top items
  Line chart    — Trend lines, growth metrics
  Donut chart   — Distribution (product types, traffic sources)
  Sparkline     — Mini inline charts in stat cards

Design rules:
  - Grid lines: subtle dashed (stroke-dasharray="3 3", opacity-20)
  - Axis labels: text-xs text-muted-foreground
  - Tooltip: rounded-md shadow-lg p-3, white bg, compact info
  - Colors: use 5-color palette max (brand-500, brand-400, brand-300, gray-400, gray-300)
  - Legend: below chart, horizontal, text-sm
  - Responsive: chart container aspect-ratio (16/9 desktop, 4/3 mobile)
  - Animation: draw-in on mount (500ms ease-out)
  - No chart if < 3 data points (show "Not enough data" instead)
  - Number format: compact notation for large numbers ($12.4K, not $12,400)

Stat cards (KPI):
  ┌──────────────────────────┐
  │ Revenue          ℹ       │  ← title + info tooltip
  │ $4,230                   │  ← value (text-2xl font-bold tabular-nums)
  │ +12.5% vs last period    │  ← comparison (text-sm, green/red)
  │ ▁▂▃▄▅▆▇█▇▆▅▄▃           │  ← sparkline (h-8, no axes)
  └──────────────────────────┘
```

---

## 9. RESPONSIVE BEHAVIOR

### 9.1 Breakpoint Adaptations

```
SIDEBAR:
  < 768px:   Hidden (sheet/drawer from left, triggered by hamburger)
  768-1024:  Collapsed (w-16, icons only, expand on hover)
  > 1024:    Expanded (w-64, full labels)

TABLES:
  < 640px:   Convert to card layout (stack columns vertically per row)
  640-1024:  Hide low-priority columns (created date, category)
  > 1024:    Show all columns

FORMS:
  < 768px:   Single column, full-width inputs
  > 768px:   2-column layout for short fields (first name + last name side by side)

PRODUCT GRID:
  < 640px:   1 column
  640-768:   2 columns
  768-1280:  3 columns
  > 1280:    4 columns (marketplace only)

STAT CARDS:
  < 640px:   1 column (stack)
  640-768:   2 columns
  > 768:     4 columns

CHARTS:
  < 640px:   Full width, aspect-ratio 4/3
  > 640px:   Full width, aspect-ratio 16/9

MODALS:
  < 640px:   Full-screen (inset-0, rounded-none)
  > 640px:   Centered with max-width
```

### 9.2 Touch Interactions (Mobile)

```
- All tap targets: min 44×44px
- Swipe left on table row: show actions (archive, delete)
- Swipe down on list: pull-to-refresh
- Long press on item: select for bulk actions
- Bottom sheet instead of dropdown on mobile
- Sticky bottom bar for primary action on form pages:
  ┌──────────────────────────────────────┐
  │ [Cancel]              [Save Product] │
  └──────────────────────────────────────┘
```

---

## 10. DARK MODE IMPLEMENTATION

```
Strategy: class-based (controlled by user preference + system fallback)

Implementation:
  1. Check localStorage('theme') first
  2. If not set, use prefers-color-scheme media query
  3. Apply class="dark" to <html> element
  4. All colors via CSS variables (already defined in design tokens)

Specific dark mode considerations:
  - Cards: slightly lighter than page bg (bg-zinc-900 on bg-zinc-950)
  - Borders: border-zinc-800 (subtle, not harsh)
  - Shadows: softer, lower opacity (dark shadows on dark bg look wrong → use border instead)
  - Charts: brighter colors in dark mode (brand-400 instead of brand-500)
  - Images: no filter (show original colors)
  - Code blocks: dark bg even in light mode (consistent with developer expectations)
  - Skeletons: bg-zinc-800 animate-pulse
  - Status indicators: slightly more saturated in dark mode for visibility
  
Toggle UX:
  - Location: user avatar dropdown → "Appearance" → [Light | Dark | System]
  - Transition: 150ms ease on background-color (prevent jarring flash)
  - Persist: localStorage
  - SSR: inject class on server based on cookie (prevent flash of wrong theme)
```

---

## 11. REFERENCE UI EXAMPLES

```
When building TESKEL, use these real products as visual reference:

Dashboard feel:
  → Linear (linear.app) — Sidebar, command palette, minimal chrome
  → Vercel Dashboard — Stats cards, deployment list, clean tables
  → Stripe Dashboard — Revenue chart, recent payments, data density

Landing page:
  → Linear.app/features — Elegant hero, feature grid, dark mode
  → Vercel.com — Bold headline, screenshot hero, trust badges
  → Resend.com — Clean typography, API code snippets, developer-focused

Storefront:
  → Gumroad creator pages — Product grid, creator profile header
  → Lemonsqueezy.com/store — Product cards with pricing, clean layout
  → Notion templates gallery — Category filters, search, grid

Checkout:
  → Stripe Checkout — Single-page, card form, express payment
  → Gumroad checkout — Overlay modal, minimal fields
  → Paddle checkout — Clean summary, multi-currency

Settings:
  → GitHub Settings — Left nav tabs, form sections, clear hierarchy
  → Linear Settings — Grouped sections, toggle switches, inline save

Emails:
  → Linear emails — Minimal, single-purpose, clear CTA
  → Stripe receipts — Clean layout, itemized, brand-neutral
  → Notion emails — Simple text, one action button
```

---

*This UI/UX spec ensures every screen of TESKEL looks and feels like a $10M-funded startup product from day 1. Follow these patterns exactly — deviations make the product feel inconsistent and amateur.*
