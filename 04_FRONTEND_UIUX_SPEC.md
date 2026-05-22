# TESKEL — Frontend Specification
## Next.js 15 — All Pages, Components, Flows, UI/UX Details

---

## 1. APP ROUTER STRUCTURE

```
app/
├── (marketing)/          # Public marketing — NO auth required
├── (auth)/                # Auth pages — NO auth required
├── (app)/                 # Dashboard — AUTH REQUIRED (member+)
├── (store)/               # Storefront — PUBLIC
├── (marketplace)/         # Marketplace — PUBLIC
├── (checkout)/            # Checkout — PUBLIC (buyer-facing)
├── (buyer)/               # Buyer portal — AUTH REQUIRED (buyer)
├── api/                   # Next.js API routes (webhooks only)
└── layout.tsx             # Root layout
```

---

## 2. ROOT LAYOUT

```tsx
// app/layout.tsx
// - ThemeProvider (light/dark/system)
// - Toaster (sonner)
// - Global CSS (Tailwind + shadcn)
// - Font: Inter (Google Fonts)
// - Metadata: default OG, favicon
```

---

## 3. MIDDLEWARE

```typescript
// middleware.ts
// Runs on EVERY request before page loads

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Routes that REQUIRE auth
  if (pathname.startsWith('/dashboard') || 
      pathname.startsWith('/purchases') ||
      pathname.startsWith('/subscriptions') ||
      pathname.startsWith('/licenses')) {
    const session = getSessionCookie(request);
    if (!session) {
      return NextResponse.redirect(new URL('/login', request.url));
    }
  }

  // Routes that REDIRECT if already logged in
  if (pathname === '/login' || pathname === '/signup') {
    const session = getSessionCookie(request);
    if (session) {
      return NextResponse.redirect(new URL('/dashboard', request.url));
    }
  }

  return NextResponse.next();
}
```

---

## 4. EVERY PAGE — Detailed Spec

### 4.1 HOME (`/`)

**Purpose:** Convert visitors → signups
**Layout:** Marketing layout (navbar + footer, no sidebar)

```
SECTIONS (scroll order):

1. NAVBAR
   - Logo (left)
   - Links: Features, Pricing, Docs, Blog (left-center)
   - Buttons: Login, Get Started (right)
   - Mobile: hamburger menu
   - Sticky on scroll, blur background

2. HERO
   - Badge: "✨ The OS for digital commerce"
   - H1: "Sell digital products with infrastructure that scales"
   - Subtitle: "From $2 prompt packs to $500 service packages. One system."
   - Primary CTA: [Start Selling — Free] (large, primary color)
   - Secondary CTA: [Read the Docs →] (outline)
   - Social proof: "2,400+ creators · $1.2M+ in sales"
   - Trusted by logos row (grayscale)

3. PRODUCT TYPES
   - Section title: "Sell anything digital"
   - 13 type cards in responsive grid (3 cols desktop, 2 tablet, 1 mobile)
   - Each card: icon + name + short description
   - Types: Download, License, Prompt Pack, Micro SaaS, Service, Community, Course, Bundle, PWYW, Tiered, Link, Collection, Agent

4. HOW IT WORKS
   - 3 steps with illustrations
   - Step 1: "Create your product" — Upload files, set type, add description
   - Step 2: "Set your pricing" — One-time, subscription, tiered, PWYW
   - Step 3: "Start selling" — Share link, embed, or list on marketplace

5. FEATURES GRID
   - 6 features in 2x3 grid
   - Each: icon + title + description
   - Features: AI Pricing, License API, Funnel Builder, Creator Collab, 1-Click Checkout, Embed Anywhere

6. PRICING TABLE
   - 4 columns: Free, Pro ($19/mo), Scale ($49/mo), Enterprise
   - Rows: Platform fee, Products, Custom domain, Analytics, AI features, etc.
   - Highlighted column: Pro (most popular)
   - Each column: [Get Started] button

7. TESTIMONIALS
   - 3 cards with avatar, name, role, quote
   - Auto-carousel on mobile

8. FAQ
   - 8 accordion items
   - Questions: "What can I sell?", "Fees?", "Payouts?", "Custom domain?", "License keys?", "API access?", "Cancel anytime?", "Compare to Gumroad?"

9. CTA BANNER
   - Full-width gradient background
   - "Start selling in 2 minutes"
   - [Get Started — Free] button

10. FOOTER
    - 4 columns: Product, Developers, Company, Legal
    - Social links: Twitter, GitHub, Discord
    - Copyright: © 2025 TESKEL
```

---

### 4.2 SIGNUP (`/signup`)

**Layout:** Auth layout (centered card, no sidebar)

```
- Title: "Create your TESKEL account"
- OAuth buttons:
  [Continue with Google] (outline, Google icon)
  [Continue with GitHub] (outline, GitHub icon)
- Divider: "or continue with email"
- Form fields:
  Email:    [input type=email, required]
  Password: [input type=password, required, min 8 chars, show/hide toggle]
- Checkbox: "I agree to the Terms of Service and Privacy Policy"
- Submit: [Create Account] (full width, primary)
- Footer link: "Already have an account? Log in"

VALIDATIONS:
- Email: required, valid format, unique
- Password: required, min 8 chars, show strength meter
- On error: show inline error below field
- On success: redirect to /onboarding

LOADING: Button shows spinner, disabled state
```

---

### 4.3 LOGIN (`/login`)

```
- Title: "Welcome back"
- OAuth buttons (same as signup)
- Email + Password fields
- [Log In] button
- Links: "Forgot password?" → /forgot-password
         "Don't have an account? Sign up" → /signup
```

---

### 4.4 ONBOARDING (`/onboarding/*`)

**Layout:** Centered card with step indicator at top

```
4-step wizard:

STEP 1: /onboarding — Profile
  Step indicator: ●●○○○
  Fields: Full Name, Display Name, Avatar (optional upload)
  [Continue] button

STEP 2: /onboarding/store — Store Setup
  Step indicator: ●●●○○
  Fields:
    Store Name (auto-generates slug below: teskel.com/[slug])
    Category dropdown: [Design, Development, Marketing, Productivity, AI, Business, Education, Other]
    Logo upload (optional)
  [Continue] button

STEP 3: /onboarding/product — First Product
  Step indicator: ●●●●○
  Type selector (13 types, clickable cards)
  Product Name
  Price ($ input)
  File upload (drag & drop)
  [Skip for now] [Create & Continue]

STEP 4: /onboarding/stripe — Stripe Connect
  Step indicator: ●●●●●
  Explanation text + benefits
  [Connect with Stripe] → Opens Stripe Connect onboarding in new tab
  [Skip for now] [Go to Dashboard]
```

---

### 4.5 DASHBOARD OVERVIEW (`/dashboard`)

**Layout:** Dashboard layout (sidebar + topbar + main content)

```
SIDEBAR (left, collapsible):
  Logo
  ──
  📊 Overview
  📦 Products
  🛒 Orders
  👥 Customers
  📈 Analytics
  🔀 Funnels
  🏷️ Discounts
  🤝 Affiliates
  📧 Emails
  🏪 Marketplace
  ──
  ⚙️ Settings
  ──
  [Cmd+K palette trigger]

TOPBAR (top):
  Breadcrumb: Dashboard
  Search (Cmd+K trigger)
  🔔 Notifications (dropdown)
  👤 Avatar (dropdown: Profile, Switch Org, Logout)

MAIN CONTENT:

Row 1: 4 Metric Cards
  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
  │ Revenue  │ │ Orders   │ │Customers │ │   MRR    │
  │ $4,287   │ │   127    │ │    89    │ │  $1,240  │
  │ ↑ +12%   │ │ ↑ +8%    │ │ ↑ +15%   │ │ ↑ +5%    │
  └──────────┘ └──────────┘ └──────────┘ └──────────┘
  Each card: icon, label, value, change% with color (green=up, red=down)

Row 2: Revenue Chart (full width)
  - Area chart (Recharts)
  - Time range selector: 7d | 30d | 90d | 1y
  - Tooltip: date + revenue + order count

Row 3: Two columns
  Left: Recent Orders table (5 rows)
    Columns: Order ID, Customer, Product, Amount, Status, Date
    Status badges: paid(green), pending(yellow), refunded(red)
    [View All Orders →]

  Right: Top Products list (5 items)
    Each: rank, name, revenue, sales count
    [View All Products →]

Row 4: AI Suggestions (collapsible)
  - 💡 "Raise Template Kit to $29 → +$340/mo predicted"
  - 💡 "3 customers showing churn risk → email them?"
  - 💡 "Add order bump to Prompt Pack → +$180/mo"
  Each: [Apply] [Dismiss]
```

---

### 4.6 PRODUCT LIST (`/dashboard/products`)

```
TOPBAR: "Products" + [New Product] button (primary)

FILTERS ROW:
  Search input (debounced)
  Status filter: [All] [Active] [Draft] [Archived]
  Type filter: [All] [Download] [License] [Prompt Pack] ...
  View toggle: Grid | Table

GRID VIEW:
  Product cards (3 cols desktop, 2 tablet, 1 mobile)
  Each card:
    - Featured image (16:9 ratio)
    - Name
    - Type badge
    - Price
    - Status badge
    - Sales count
    - [Edit] [··· menu: Duplicate, Archive, Delete]

TABLE VIEW:
  Columns: Image, Name, Type, Price, Status, Sales, Revenue, Created
  Sortable columns
  Row click → edit
  Checkbox for bulk actions (Archive, Delete)

EMPTY STATE:
  Illustration + "Create your first product"
  [New Product] button
```

---

### 4.7 PRODUCT EDITOR (`/dashboard/products/[id]`)

```
TOPBAR: Product name + [Save] [Publish] buttons

TABS:
  Details | Pricing | Files | Access Rules | Delivery | Analytics | Settings

DETAILS TAB:
  - Name (text input)
  - Description (rich text editor — TipTap)
  - Type (readonly, set at creation)
  - Category (dropdown)
  - Tags (multi-select with create)
  - Featured Image (upload with preview + crop)
  - Cover Image (upload)
  - Preview Images (multiple upload, reorderable)
  - Video Preview URL
  - Visibility: Public | Unlisted | Private | Embed Only
  - SEO: Meta Title, Meta Description (auto-generated from name/description)

PRICING TAB:
  - Pricing model selector: One-time | Subscription | PWYW | Tiered | Free | Usage-based
  - Price cards (add/remove):
    Each price card:
      Name (e.g., "Standard", "Pro")
      Amount ($ input)
      Currency (dropdown, default USD)
      Interval (if subscription: Monthly | Yearly)
      Trial period (if subscription: X days)
      Is default (radio)
      Features list (add/remove items)
      Highlighted badge (optional: "Most Popular")
      CTA text (optional)
  - Preview: live pricing table preview on right side

FILES TAB:
  - Upload area (drag & drop, multi-file)
  - File list:
    Each file:
      Icon (by content type)
      File name + size
      Version input
      Associated price (dropdown: All / Specific price)
      [Delete] button
  - Upload progress bar per file
  - Max file size: 500MB per file

ACCESS RULES TAB:
  - Rule list (add/remove)
  - Each rule:
    Type: Require Purchase | Require Subscription | Require Tier | Require License | Download Limit | Time Limit
    Condition inputs (dynamic based on type)
    Priority (number)
  - Preview: what buyer sees for each access scenario

DELIVERY TAB:
  - Delivery strategy (auto-selected by product type, can override):
    File Download | License Key | Account Access | Invite Link | Booking | Stream | Webhook | Email | Redirect
  - Configuration per strategy:
    File: download limit, link expiry hours
    License: max activations, key format, features
    Invite: platform (Discord/Telegram/Slack), bot token, role ID
    Webhook: URL, payload template
    Booking: duration, calendar link, timezone
  - Test delivery button (sends test to yourself)

ANALYTICS TAB:
  - Revenue chart (30d)
  - Sales chart (30d)
  - Conversion rate
  - Top referrers
  - Buyer geography

SETTINGS TAB:
  - Slug (editable)
  - Published date (scheduler)
  - Marketplace listing (opt-in/out)
  - Delete product (danger zone, confirmation modal)
```

---

### 4.8 NEW PRODUCT WIZARD (`/dashboard/products/new`)

```
Multi-step modal or page:

Step 1: Choose Type
  13 type cards (same as landing page, but clickable)
  Select one → [Next]

Step 2: Details
  Name, Description, Category, Tags, Images
  [Next]

Step 3: Pricing
  Pricing model + price cards
  [Next]

Step 4: Files & Delivery
  Upload files, configure delivery
  [Create Product]

On success: redirect to /dashboard/products/[id]
```

---

### 4.9 ORDER LIST (`/dashboard/orders`)

```
TOPBAR: "Orders" + [Export CSV] button

FILTERS:
  Search (order ID, customer email)
  Status: [All] [Paid] [Pending] [Refunded] [Disputed]
  Date range picker
  Product filter (dropdown)

TABLE:
  Columns: Order ID, Customer, Email, Products, Total, Status, Date
  Status badges with colors
  Row click → detail
  Sortable + paginated

EXPORT:
  [Export CSV] → downloads filtered orders as CSV
```

---

### 4.10 ORDER DETAIL (`/dashboard/orders/[id]`)

```
Header: Order #ORD-xxxx + Status badge

Two columns:

Left: Order Info
  Customer: name, email
  Date: created at
  UTM: source, medium, campaign (if any)
  Affiliate: code (if any)

  Line Items table:
    Product | Price | Amount | License Key (if any)

  Pricing:
    Subtotal: $xx
    Discount: -$xx (show code)
    Tax: $xx
    Total: $xx

Right: Actions
  [Refund] button → opens refund modal:
    - Full refund or partial
    - Partial: select items + amounts
    - Reason (required)
    - [Confirm Refund]

  Delivery Status:
    For each item: delivery type + status
    [Resend Delivery] if failed

  Timeline:
    - Order created
    - Payment confirmed
    - Delivery sent
    - (If refunded) Refund processed
```

---

### 4.11 STOREFRONT (`/[storeSlug]`)

**Layout:** Store layout (creator-themed, no TESKEL branding visible)

```
HEADER:
  Logo (left)
  Store name
  Bio text
  Social links (Twitter, GitHub, Website)
  Navigation: Products | About

ANNOUNCEMENT BAR (if active):
  Colored bar with text + optional link
  Dismissible (×)

PRODUCT GRID:
  Responsive grid (3 cols desktop, 2 tablet, 1 mobile)
  Each product card:
    Featured image (16:9, hover zoom effect)
    Product name
    Price (from $X if multiple prices)
    Rating stars + count
    Type badge (small, subtle)
    [Buy Now] button on hover

EMAIL CAPTURE:
  Popup after 10s on page (if not already subscribed)
  Or inline section at bottom:
    "Get notified when I launch new products"
    [email input] [Subscribe]

FOOTER:
  © Creator Name | Powered by TESKEL (small, subtle)
```

---

### 4.12 PRODUCT DETAIL (`/[storeSlug]/products/[productSlug]`)

```
Two columns (desktop), stacked (mobile):

LEFT: Media
  Main image (large)
  Thumbnail gallery below
  Video preview (if exists)

RIGHT: Purchase Info
  Product name (h1)
  Creator name + avatar + link
  Rating: ★★★★★ (47 reviews)
  Sales count: "847 sold"

  Price selector:
    If one price: "$19.00" + [Buy Now]
    If multiple: radio buttons with name + price
      (●) Standard — $19
      ( ) Extended — $39

  [Buy Now — $19] (large, primary, sticky on mobile)

  Trust badges:
    🔒 Secure checkout | 📧 Instant delivery | 🔄 30-day guarantee

  Description (rendered markdown)
  What's included (checklist)
  Reviews section
  Related products

SEO:
  SSR with meta tags
  OG image (generated via @vercel/og)
  JSON-LD structured data (Product schema)
```

---

### 4.13 CHECKOUT (`/checkout/[sessionId]`)

**Layout:** Minimal (no navbar/sidebar, focused checkout)

```
Two columns (desktop), stacked (mobile):

LEFT: Order Summary
  Product image + name
  Price selected
  ─────────
  Subtotal: $19.00
  Discount: [Have a code? Enter here →]
  Tax: $0.00
  ─────────
  Total: $19.00

  Order Bump (if funnel active):
    ┌────────────────────────────────────┐
    │ 💡 Add "Prompt Pack" for just $7? │
    │ [☑ Yes, add to my order!]          │
    └────────────────────────────────────┘

RIGHT: Payment
  Email: [input]
  Card: [Stripe Elements card input]
  [Apple Pay] [Google Pay] (if available)
  [Complete Purchase — $19] (large, primary)

  🔒 Powered by Stripe
  🔒 Encrypted & Secure
```

---

### 4.14 SUCCESS (`/success/[orderId]`)

```
✅ Purchase Complete!

Order #ORD-xxxx
Template Kit — Standard — $19.00

DOWNLOADS:
  📄 template-v2.zip (24 MB) [⬇ Download]
  📄 bonus-guide.pdf (2 MB) [⬇ Download]
  ⏰ Links valid for 24 hours

LICENSE KEYS (if any):
  🔑 TSKL-XXXX-XXXX-XXXX [📋 Copy]

EMAIL SENT:
  📧 Receipt sent to your@email.com

UPSELL (if funnel active):
  ┌────────────────────────────────────────┐
  │ 🔥 Special Offer!                     │
  │ Upgrade to Extended — $20 more        │
  │ [Upgrade Now] [No thanks]             │
  └────────────────────────────────────────┘

[View My Purchases] [Back to Store]
```

---

### 4.15 MARKETPLACE (`/explore`)

```
SEARCH BAR (top, prominent):
  🔍 [Search templates, prompts, tools... ] [Search]

CATEGORY TABS:
  [All] [Templates] [Prompts] [UI Kits] [Plugins] [Courses] [Services] [Communities]

TRENDING SECTION:
  Horizontal scroll of trending product cards

PRODUCT GRID:
  Same card format as storefront
  Additional: creator avatar + name below

SIDEBAR FILTERS (desktop):
  Price range slider
  Rating: ★4+ ★3+
  Type: checkboxes
  Sort: Trending | Newest | Price ↑ | Price ↓ | Rating

PAGINATION: Load more button
```

---

### 4.16 BUYER PORTAL

```
/purchases — List of all purchases with download links
/purchases/[id] — Purchase detail + downloads + license keys
/subscriptions — Active subscriptions with [Cancel] button
/licenses — All license keys with status + [Validate] button
/settings — Name, email update
```

---

## 5. KEY COMPONENTS

### 5.1 Dashboard Sidebar

```tsx
// Collapsible sidebar with icons + labels
// Collapsed: icons only (48px wide)
// Expanded: icons + labels (240px wide)
// Toggle button at bottom
// Active item: highlighted with primary color
// Sections separated by dividers
// Mobile: overlay drawer
```

### 5.2 Command Palette (Cmd+K)

```tsx
// Searchable command palette
// Groups: Pages, Actions, Products, Settings
// Keyboard navigation (↑↓ Enter)
// Recent items shown first
// Powered by cmdk library
```

### 5.3 File Uploader

```tsx
// Drag & drop zone
// Multi-file support
// Progress bar per file
// File type validation
// Size limit display
// Preview for images
// Upload to R2 via presigned URL
```

### 5.4 Funnel Builder

```tsx
// React Flow based visual editor
// Node types: product_page, checkout, order_bump, upsell, downsell, thank_you
// Edges: flow connections with conditional logic
// Sidebar: step configuration panel
// Drag nodes from palette
// Click node → edit config in sidebar
// Auto-layout
```

### 5.5 Email Editor

```tsx
// Rich text editor for email body (TipTap)
// Variable insertion: {{customer_name}}, {{product_name}}, etc.
// Preview panel (desktop + mobile email view)
// Subject line input
// Send test email button
```

### 5.6 Pricing Editor

```tsx
// Dynamic price card builder
// Add/remove price tiers
// Drag to reorder
// Each tier: name, amount, interval, features list, badge, CTA
// Live preview of pricing table
// Stripe sync indicator
```

### 5.7 Theme Customizer

```tsx
// Live preview sidebar
// Controls:
  - Primary color (color picker)
  - Font family (dropdown: Inter, Plus Jakarta, Space Grotesk, etc.)
  - Border radius (slider)
  - Mode (light/dark/system toggle)
  - Cover image (upload)
  - Custom CSS (textarea, advanced)
// Preview updates in real-time
// [Save] [Reset to default]
```

---

## 6. UI/UX PRINCIPLES

```
1. SPEED: Every page <2s load, optimistic updates, skeleton loaders
2. FEEDBACK: Toast on every action (success/error), loading states on buttons
3. CONSISTENCY: shadcn/ui components everywhere, no custom styling unless necessary
4. ACCESSIBILITY: WCAG 2.1 AA, keyboard navigation, focus rings, aria labels
5. RESPONSIVE: Mobile-first, breakpoint at 640/768/1024/1280px
6. DARK MODE: System preference default, manual toggle
7. ERROR STATES: Friendly error messages, retry buttons, empty states with CTAs
8. ONBOARDING: Tooltips for first-time users, progressive disclosure
9. TRUST: Security badges on checkout, Stripe branding, SSL indicators
10. ANIMATIONS: Subtle (150ms), no gratuitous motion, respect prefers-reduced-motion
```

---

## 7. SHADCN/UI COMPONENTS NEEDED

```
Core (~50):
  button, card, dialog, dropdown-menu, form, input, select,
  table, tabs, toast, badge, avatar, command, sidebar, chart,
  skeleton, separator, sheet, popover, tooltip, switch, checkbox,
  radio-group, slider, textarea, label, alert, accordion,
  breadcrumb, calendar, date-picker, pagination, progress,
  scroll-area, toggle, toggle-group, aspect-ratio, collapsible,
  hover-card, navigation-menu, menubar, resizable,
  context-menu, alert-dialog, drawer

Custom (~15):
  file-uploader, pricing-editor, theme-customizer,
  funnel-builder, email-editor, access-rule-builder,
  delivery-config, stripe-connect, metric-card,
  live-activity-feed, ai-suggestion-card, announcement-bar,
  email-capture-popup, product-type-selector, order-bump
```

---

*Total: ~55 pages, ~65 components, complete UI/UX specs with layout, content, interactions, and validations.*
