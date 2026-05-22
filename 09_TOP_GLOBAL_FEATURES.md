# TESKEL — Top Global Pro Modern Features Spec
## What Makes TESKEL Compete at World-Class Level

---

## 0. REASONING: WHY TESKEL CAN BE TOP GLOBAL

```
Masalah dunia saat ini:
  Gumroad → stale, no innovation since 2020, fees tinggi
  Whop    → US-only mindset, community-heavy, bukan infra
  Lemon   → Checkout hanya, bukan OS
  Paddle  → SaaS billing saja, bukan digital products
  Stripe  → Payment layer, bukan commerce layer

Gap yang TESKEL isi:
  1. TIDAK ADA satu platform yang menyatukan:
     Digital product infra + AI-powered + Global-first + Developer-friendly + Creator-friendly
  
  2. Existing platforms semuanya:
     - US-centric (pricing, payment methods, UI)
     - Feature-heavy but not composable
     - No real AI (hanya label marketing)
     - No developer API yang serius
     - No embed/white-label

  3. TESKEL punya moat yang impossible to copy:
     a. Network effect (more creators → more data → better AI → more creators)
     b. Composable architecture (setiap fitur bisa dipakai sendiri via API)
     c. Global-first (bukan "add international later")
     d. Developer-first (API/SDK/Embed dari day 1)
```

---

## 1. DESIGN SYSTEM — World-Class UI Foundation

### 1.1 Design Tokens

```css
/* Color System — Semantic, not just pretty */
:root {
  /* Brand */
  --brand-50: #eef2ff;
  --brand-100: #e0e7ff;
  --brand-200: #c7d2fe;
  --brand-300: #a5b4fc;
  --brand-400: #818cf8;
  --brand-500: #6366f1;  /* Primary */
  --brand-600: #4f46e5;
  --brand-700: #4338ca;
  --brand-800: #3730a3;
  --brand-900: #312e81;
  --brand-950: #1e1b4b;

  /* Semantic */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;

  /* Surface (auto-switches in dark mode) */
  --surface-0: #ffffff;
  --surface-1: #f9fafb;
  --surface-2: #f3f4f6;
  --surface-3: #e5e7eb;
  --surface-4: #d1d5db;

  /* Typography */
  --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --font-display: 'Plus Jakarta Sans', var(--font-sans);

  /* Type Scale (clamp for fluid typography) */
  --text-xs: clamp(0.7rem, 0.65rem + 0.25vw, 0.75rem);
  --text-sm: clamp(0.8rem, 0.75rem + 0.25vw, 0.875rem);
  --text-base: clamp(0.9rem, 0.85rem + 0.25vw, 1rem);
  --text-lg: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --text-xl: clamp(1.1rem, 1rem + 0.5vw, 1.25rem);
  --text-2xl: clamp(1.3rem, 1.1rem + 1vw, 1.5rem);
  --text-3xl: clamp(1.6rem, 1.3rem + 1.5vw, 1.875rem);
  --text-4xl: clamp(2rem, 1.5rem + 2.5vw, 2.25rem);
  --text-5xl: clamp(2.5rem, 1.8rem + 3.5vw, 3rem);

  /* Spacing Scale (4px base) */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-5: 1.25rem;   /* 20px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-10: 2.5rem;   /* 40px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-20: 5rem;     /* 80px */
  --space-24: 6rem;     /* 96px */

  /* Border Radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-2xl: 1.5rem;
  --radius-full: 9999px;

  /* Shadow (elevation system) */
  --shadow-xs: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.1);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
  --shadow-xl: 0 20px 25px rgba(0,0,0,0.1);
  --shadow-inner: inset 0 2px 4px rgba(0,0,0,0.06);

  /* Animation */
  --duration-fast: 100ms;
  --duration-base: 150ms;
  --duration-slow: 300ms;
  --duration-slower: 500ms;
  --ease-default: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);

  /* Z-Index Scale */
  --z-dropdown: 50;
  --z-sticky: 100;
  --z-overlay: 200;
  --z-modal: 300;
  --z-popover: 400;
  --z-toast: 500;
  --z-command: 600;
}
```

### 1.2 Dark Mode

```
Strategy: CSS variables + class-based toggle

[data-theme="dark"] {
  --surface-0: #09090b;
  --surface-1: #18181b;
  --surface-2: #27272a;
  --surface-3: #3f3f46;
  --surface-4: #52525b;
  
  --text-primary: #fafafa;
  --text-secondary: #a1a1aa;
  --text-muted: #71717a;
}

Rules:
  - Default: system preference
  - Toggle: localStorage persistence
  - Transitions: 150ms on theme switch
  - Images: no filter (original colors)
  - Charts: different color palette for dark
  - Shadows: lighter opacity in dark mode
```

### 1.3 Responsive Breakpoints

```
Mobile-first: design for 320px, then scale up.

Breakpoints:
  sm:  640px   — Large phones, small tablets
  md:  768px   — Tablets
  lg:  1024px  — Laptops, small desktops
  xl:  1280px  — Desktops
  2xl: 1536px  — Large desktops

Key layout shifts:
  < 640px:  Single column, bottom nav, full-width cards
  640-768:  2-col grid, sidebar hidden (drawer)
  768-1024: 2-col grid, sidebar collapsed (icons only)
  1024+:    3-col grid, sidebar expanded
  1280+:    Full layout, all panels visible
```

### 1.4 Animation & Micro-Interactions

```
Principles:
  1. Purpose: Every animation communicates state change.
  2. Speed: Default 150ms. Never > 300ms for UI interactions.
  3. Preference: Respect prefers-reduced-motion.
  4. Consistency: Same easing curve everywhere.

Animations used:
  - Page transitions: fade-in 150ms
  - Modal open: scale from 0.95 + fade 150ms
  - Toast enter: slide from right + fade 200ms
  - Button press: scale(0.98) 100ms
  - Card hover: translateY(-2px) + shadow-md 150ms
  - Skeleton: pulse opacity 0.5→1 1.5s infinite
  - Chart draw: line draw-in 500ms ease-out
  - Counter: number roll-up 300ms
  - Menu expand: height auto 200ms ease-out (via framer-motion)
  - Tab switch: content fade-in 100ms
  - Error shake: translateX(±4px) 3 cycles 300ms

Library: framer-motion (for complex), CSS transitions (for simple)
```

---

## 2. INTERNATIONALIZATION (i18n) — Global-First

### 2.1 Multi-Language Architecture

```
Stack: next-intl (for Next.js App Router)

Languages (MVP):
  - en (English) — default
  - id (Indonesian)
  - es (Spanish)
  - pt-BR (Portuguese)
  - zh (Chinese Simplified)
  - ja (Japanese)
  - ko (Korean)
  - ar (Arabic — RTL)

File structure:
  messages/
    en.json
    id.json
    es.json
    ...

Route structure:
  /en/dashboard
  /id/dashboard
  or: detect from browser Accept-Language + store in cookie

Translatable content:
  - UI strings (buttons, labels, messages)
  - Error messages
  - Email templates
  - Legal pages
  - Marketing pages
  - Marketplace categories

NOT translatable (creator-owned):
  - Product names
  - Product descriptions
  - Store names
  - Creator profiles
  (These stay in creator's language; AI auto-translate for buyers is Phase 2)
```

### 2.2 Multi-Currency

```
Supported currencies (Stripe):
  USD, EUR, GBP, JPY, AUD, CAD, CHF, HKD, SGD, SEK,
  NOK, DKK, NZD, MXN, BRL, INR, IDR, MYR, PHP, THB,
  TWD, VND, PLN, CZK, HUF, RON, ZAR, KRW, AED, SAR

Storage: Always in cents (integer) + currency code
Display: Intl.NumberFormat(locale, { style: 'currency', currency })

Example:
  Stored: { priceCents: 1900, currency: 'USD' }
  Display (en-US): $19.00
  Display (id-ID): Rp 300.000 (if IDR)
  Display (ja-JP): ¥2,800 (if JPY)

Creator chooses base currency.
Buyer sees in their local currency (converted via Stripe).
```

### 2.3 Purchasing Power Parity (PPP)

```
Implementation:
  1. Detect buyer country from IP (Cloudflare CF-IPCountry header)
  2. Lookup PPP discount for country
  3. Auto-apply discount at checkout
  4. Show badge: "🌍 Adjusted for your region"

PPP table:
  ┌────────────┬──────────┬───────────────────────────┐
  │ Country    │ Discount │ Example ($19 product)     │
  ├────────────┼──────────┼───────────────────────────┤
  │ US/EU/AU   │ 0%       │ $19.00                    │
  │ Brazil     │ 40%      │ $11.40                    │
  │ India      │ 60%      │ $7.60                     │
  │ Indonesia  │ 55%      │ $8.55                     │
  │ Turkey     │ 70%      │ $5.70                     │
  │ Argentina  │ 75%      │ $4.75                     │
  │ Nigeria    │ 65%      │ $6.65                     │
  └────────────┴──────────┴───────────────────────────┘

Creator control:
  - Enable/disable PPP per product
  - Set custom discount cap per region
  - Whitelist/blacklist countries
```

### 2.4 Local Payment Methods

```
Phase 1 (Stripe built-in):
  - Card (worldwide)
  - Apple Pay / Google Pay (mobile)
  - Link (Stripe's 1-click checkout)

Phase 2 (Stripe additional):
  - iDEAL (Netherlands)
  - Bancontact (Belgium)
  - SEPA Direct Debit (EU)
  - Alipay (China)
  - WeChat Pay (China)
  - FPX (Malaysia)
  - GrabPay (SEA)
  - BLIK (Poland)

Phase 3 (via payment orchestrator):
  - GoPay, OVO, DANA, ShopeePay (Indonesia)
  - PIX (Brazil)
  - UPI (India)
  - M-Pesa (Africa)
  - Paynow (Singapore)
  - PromptPay (Thailand)
```

---

## 3. REAL-TIME FEATURES — Live Dashboard

### 3.1 WebSocket Architecture

```
Stack: Soketi (self-hosted Pusher-compatible) or Ably (managed)

Channels:
  private-org.{orgId}        — Org-level updates
  private-org.{orgId}.orders — New orders
  private-org.{orgId}.stats  — Real-time stats

Events:
  order.created       → Dashboard metric update, notification
  customer.created    → Customer count update
  review.created      → Review notification
  payout.completed    → Payout notification
  license.activated   → License activity feed
  checkout.started    → Live visitor count
  checkout.completed  → Sale notification with confetti

Client (React):
  const { data } = useChannel('private-org.{orgId}', 'order.created', (event) => {
    // Optimistic update: increment order count, add to recent orders
    queryClient.invalidateQueries(['orders']);
    queryClient.invalidateQueries(['analytics-summary']);
    toast.success(`New sale! ${event.productName} — ${formatCurrency(event.totalCents)}`);
  });
```

### 3.2 Live Activity Feed

```
Dashboard widget: "Live Activity"

┌──────────────────────────────────────────────────────┐
│ 🟢 LIVE                                             │
│                                                      │
│ 🛒 $19.00 sale — Template Kit          just now    │
│ 👁 47 people viewing your store         30s ago    │
│ ⭐ New 5★ review on Prompt Pack         2min ago   │
│ 🔑 License activated (MacBook Pro)      5min ago   │
│ 💳 $47.00 sale — Service Package        12min ago  │
│ 📧 New subscriber from popup            15min ago  │
│                                                      │
│ Today: $234 revenue · 12 orders · 3 new customers   │
└──────────────────────────────────────────────────────┘

Implementation:
  - Server pushes events via WebSocket
  - Client renders activity list (max 20 items, FIFO)
  - Persisted in Redis sorted set (TTL 24h)
  - Also stored in analytics_events table for history
```

### 3.3 Real-Time Notifications

```
Types:
  - In-app (bell icon, badge count, dropdown)
  - Email (configurable per event type)
  - Browser push (Service Worker, optional opt-in)

In-app notification schema:
  notifications: pgTable({
    id: uuid,
    orgId: uuid,
    userId: uuid,
    type: varchar,        // 'order.paid', 'review.created', 'payout.completed'
    title: varchar,       // "New sale! Template Kit"
    body: text,           // "$19.00 from john@email.com"
    link: text,           // "/dashboard/orders/xxx"
    read: boolean,
    createdAt: timestamp,
  })

Creator notification preferences:
  ┌────────────────────┬───────┬───────┬──────────┐
  │ Event              │ In-App│ Email │ Push     │
  ├────────────────────┼───────┼───────┼──────────┤
  │ New sale           │ ✅    │ ✅    │ ✅       │
  │ New review         │ ✅    │ ✅    │ ❌       │
  │ New subscriber     │ ✅    │ ❌    │ ❌       │
  │ Payout completed   │ ✅    │ ✅    │ ❌       │
  │ License activated  │ ✅    │ ❌    │ ❌       │
  │ Chargeback         │ ✅    │ ✅    │ ✅       │
  │ Refund requested   │ ✅    │ ✅    │ ✅       │
  └────────────────────┴───────┴───────┴──────────┘
```

---

## 4. ADVANCED ANALYTICS — Not Just Charts

### 4.1 Event Taxonomy

```
All trackable events:

PAGE VIEWS:
  page.viewed                → { path, referrer, utm }
  product.viewed             → { productId, source }
  store.viewed               → { storeSlug, referrer }

COMMERCE:
  checkout.started           → { productId, priceId, source }
  checkout.abandoned         → { productId, durationMs }
  checkout.completed         → { orderId, totalCents, paymentMethod }
  order.bump.accepted        → { bumpProductId }
  order.bump.rejected        → { bumpProductId }
  upsell.shown               → { upsellProductId }
  upsell.accepted            → { upsellProductId }
  upsell.rejected            → { upsellProductId }

ENGAGEMENT:
  product.downloaded         → { productId, fileId }
  license.validated          → { key, valid }
  license.activated          → { key, fingerprint }
  review.created             → { productId, rating }
  email.subscribed           → { source }
  email.unsubscribed         → { reason }

CREATOR:
  product.created            → { type, pricingModel }
  product.published          → { productId }
  stripe.connected           → { }
  funnel.created             → { steps }
  discount.created           → { type, value }
```

### 4.2 Funnel Analytics

```
Built-in funnels (auto-calculated):

Store Funnel:
  Store Visit → Product View → Checkout Started → Payment → Download
  
  Metrics per step:
    - Count
    - Conversion rate (%)
    - Drop-off rate (%)
    - Average time in step

Product Funnel:
  Product Page View → Add to Cart → Checkout → Payment → Review

Revenue Metrics:
  - GMV (Gross Merchandise Value)
  - Net Revenue (after platform fee)
  - MRR (Monthly Recurring Revenue)
  - ARR (Annual Recurring Revenue)
  - ARPU (Average Revenue Per User/Creator)
  - LTV (Lifetime Value per customer)
  - Churn Rate (subscription)
  - Expansion Revenue (upsells, cross-sells)

Cohort Analysis:
  - Group customers by signup month
  - Track retention over 12 months
  - Track revenue per cohort
  - Identify best-performing acquisition channels
```

### 4.3 Creator Insights (AI-Powered, Phase 2)

```
Weekly email digest to creator:

"📊 Your Week in Review

Revenue: $1,240 (+12% vs last week)
Orders: 47 (best day: Tuesday, $340)
New customers: 23

🧠 AI Insights:
• Your Template Kit converts 2x better from Twitter traffic than from Google.
  → Consider running more Twitter content.
• 14 people viewed Prompt Pack but didn't buy. Most drop off at price.
  → Try a $5 "lite" tier? Predicted +$180/mo.
• 3 customers are at risk of churning (no login in 30 days).
  → Send them a personal email? [Draft email →]

🎯 This Week's Goals:
• Publish 1 new product (you haven't in 3 weeks)
• Reply to 2 pending reviews
• Test a new price for Prompt Pack
"
```

---

## 5. SEO & PERFORMANCE — World-Class Web Vitals

### 5.1 Core Web Vitals Targets

```
Target (for ALL pages):
  LCP (Largest Contentful Paint): < 2.5s
  FID (First Input Delay): < 100ms
  CLS (Cumulative Layout Shift): < 0.1
  INP (Interaction to Next Paint): < 200ms
  FCP (First Contentful Paint): < 1.8s
  TTFB (Time to First Byte): < 800ms

Per page type:
  Landing page:   LCP < 1.5s (hero image optimized)
  Storefront:     LCP < 2.0s (product images lazy-loaded)
  Product page:   LCP < 2.0s (featured image priority)
  Dashboard:      LCP < 2.5s (charts loaded async)
  Checkout:       LCP < 1.5s (minimal, fast)
```

### 5.2 Performance Budget

```
Bundle size limits:
  - First load JS: < 100KB (gzipped)
  - Page JS: < 50KB per page (code-split)
  - CSS: < 30KB (Tailwind purge)
  - Font: < 40KB (subset Inter, woff2 only)
  - Total page weight: < 500KB (including images)

Image strategy:
  - Format: WebP (with AVIF for modern browsers)
  - Sizes: srcset with 640/768/1024/1280/1920 widths
  - Loading: lazy (below fold), eager (above fold, priority)
  - Placeholder: blur placeholder (10px base64)
  - CDN: Cloudflare Polish + auto-resize

Font strategy:
  - Inter: subset Latin + Latin-ext only
  - Display: swap (FOUT over FOIT)
  - Preload: <link rel="preload" href="inter.woff2" as="font">
  - Variable font: single file, all weights
```

### 5.3 Technical SEO

```
For every public page:
  - <title> unique per page (< 60 chars)
  - <meta name="description"> unique (< 160 chars)
  - <link rel="canonical"> (prevent duplicate content)
  - Open Graph: og:title, og:description, og:image, og:url
  - Twitter Card: twitter:card, twitter:title, twitter:image
  - JSON-LD structured data

Product pages:
  - Product schema (name, description, price, availability, rating)
  - BreadcrumbList schema
  - Review schema (aggregateRating)

Creator stores:
  - Organization schema (name, logo, url)
  - Sitelinks searchbox

Marketplace:
  - ItemList schema (product listing)
  - SearchAction schema

Technical:
  - robots.txt (allow all public, disallow dashboard/checkout)
  - sitemap.xml (dynamic, updated daily)
  - Alternate hreflang tags (when i18n active)
  - 301 redirects for slug changes
  - og:image via @vercel/og (dynamic, per product)
```

---

## 6. NOTIFICATION & COMMUNICATION SYSTEM

### 6.1 Email Templates (React Email)

```
Templates needed:

Transactional (auto-sent):
  1. welcome                   — After signup
  2. email-verification        — Verify email
  3. password-reset            — Reset password
  4. purchase-receipt          — After purchase (buyer)
  5. license-delivery          — License key delivery
  6. subscription-confirmed    — Subscription started
  7. subscription-expiring     — 3 days before renewal
  8. subscription-canceled     — Subscription canceled
  9. community-invite          — Discord/Telegram invite
  10. booking-confirmed        — Service booking confirmation
  11. refund-processed         — Refund complete (buyer)
  12. team-invite              — Invited to org
  13. payout-completed         — Payout arrived (creator)
  14. chargeback-alert         — Dispute opened (creator)

Marketing (creator-configured):
  15. abandoned-checkout       — 1h after abandon
  16. post-purchase-sequence   — Drip sequence
  17. product-launch           — New product announcement
  18. weekly-digest            — Creator's weekly stats
  19. review-request           — Ask buyer for review (7 days after purchase)

Design:
  - Minimal, clean, 600px max width
  - Brand color from creator's theme
  - Logo from creator's org
  - Unsubscribe link (mandatory)
  - View in browser link
  - Responsive (mobile email clients)
```

---

## 7. SEARCH — Instant, Typo-Tolerant, Faceted

### 7.1 Typesense Configuration

```typescript
// Collection schema
const productCollection = {
  name: 'products',
  fields: [
    { name: 'id', type: 'string' },
    { name: 'name', type: 'string' },
    { name: 'description', type: 'string', optional: true },
    { name: 'type', type: 'string', facet: true },
    { name: 'category', type: 'string', facet: true },
    { name: 'tags', type: 'string[]', facet: true },
    { name: 'org_name', type: 'string' },
    { name: 'org_slug', type: 'string' },
    { name: 'price_cents', type: 'int32', facet: true },
    { name: 'currency', type: 'string' },
    { name: 'rating', type: 'float', facet: true },
    { name: 'total_sales', type: 'int32' },
    { name: 'review_count', type: 'int32' },
    { name: 'created_at', type: 'int64' },
    { name: 'published_at', type: 'int64' },
  ],
  default_sorting_field: 'total_sales',
};

// Search query
const results = await typesense.collections('products').documents().search({
  q: query,
  query_by: 'name,description,tags,org_name',
  filter_by: `type:=[${typeFilter}] && price_cents:>=${minPrice} && price_cents:<=${maxPrice} && rating:>=${minRating}`,
  sort_by: sortBy, // 'total_sales:desc', 'price_cents:asc', 'published_at:desc', '_text_match:desc'
  facet_by: 'type,category,tags',
  max_facet_values: 20,
  per_page: 20,
  page: page,
  typo_tokens_threshold: 3,
  highlight_full_fields: 'name,description',
});

// Index sync
// On product.published → add to Typesense
// On product.updated → update in Typesense
// On product.archived → remove from Typesense
// Full re-index: daily cron job (safety net)
```

### 7.2 Search UX

```
Features:
  - Instant results as you type (debounced 200ms)
  - Typo correction: "tempate" → "template"
  - Faceted filters (type, category, price range, rating)
  - Highlighted matches in results
  - Recent searches (localStorage)
  - Trending searches (Redis sorted set)
  - "No results" state with suggestions
  - Voice search (Web Speech API, Phase 2)

Command Palette (Dashboard):
  Cmd+K opens palette
  Search across: products, orders, customers, settings, actions
  Keyboard navigation: ↑↓ Enter Esc
  Recent items shown first
  Actions: "Create product", "View analytics", "Open settings"
```

---

## 8. ADVANCED CHECKOUT — Conversion Optimized

### 8.1 Checkout UX Best Practices

```
1. ONE page checkout (no multi-step)
2. Email first (pre-fill if returning buyer)
3. Apple Pay / Google Pay above card form
4. Auto-detect card type (Visa/MC/Amex)
5. Inline validation (not on submit)
6. Security badges: Stripe logo + lock icon
7. Social proof: "847 people bought this"
8. Urgency (if flash sale): countdown timer
9. Order bump below total (1-click add)
10. Trust: Money-back guarantee badge
11. Loading: Progress indicator during payment
12. Success: Confetti animation + immediate download

Conversion boosters:
  - Saved payment info (Stripe Link)
  - Express checkout (1 click for returning buyers)
  - Exit intent popup: "Wait! 10% off if you buy now"
  - Cart abandonment email (1h, 24h, 72h)
```

### 8.2 Multi-Currency Checkout

```
Flow:
  1. Detect buyer country (CF-IPCountry header)
  2. Show price in local currency (converted via Stripe)
  3. If PPP enabled: show discounted price + badge
  4. Checkout in buyer's currency
  5. Creator receives payout in their currency (Stripe handles FX)

Display:
  "US $19.00" (United States)
  "Rp 300.000" (Indonesia, with PPP: "Rp 135.000 — 🌍 Regional pricing")
  "¥2,800" (Japan)
  "R$95,00" (Brazil, with PPP: "R$57,00 — 🌍 Regional pricing")
```

---

## 9. DEVELOPER EXPERIENCE — API-First

### 9.1 API Design Principles

```
REST conventions:
  - Plural nouns: /products, /orders, /licenses
  - HTTP verbs: GET (read), POST (create), PATCH (update), DELETE (remove)
  - Nested resources: /orgs/:slug/products/:id/prices
  - Query params for filtering: ?status=active&type=download
  - Cursor-based pagination: ?cursor=xxx&limit=20

Response envelope:
  Success: { data: T }
  List: { data: T[], meta: { cursor, hasMore, total } }
  Error: { error: { code, message, details } }

Versioning:
  - URL prefix: /v1/...
  - Breaking changes get new version: /v2/...
  - Non-breaking additions: add fields (never remove)

Rate limit headers (every response):
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 997
  X-RateLimit-Reset: 1716400000
  Retry-After: 60 (on 429)
```

### 9.2 SDK Design

```typescript
// @teskel/sdk — TypeScript SDK

import { Teskel } from '@teskel/sdk';

const teskel = new Teskel({ apiKey: 'tskl_live_xxx' });

// Products
const products = await teskel.products.list({ status: 'active' });
const product = await teskel.products.create({ name: 'My Template', type: 'download', ... });
const updated = await teskel.products.update(productId, { name: 'New Name' });
await teskel.products.archive(productId);

// Checkout
const session = await teskel.checkout.createSession({
  productId: 'prod_xxx',
  priceId: 'price_xxx',
  successUrl: 'https://mysite.com/thanks',
  cancelUrl: 'https://mysite.com/cancel',
});
// → session.url (redirect buyer here)

// Licenses
const validation = await teskel.licenses.validate('TSKL-XXXX-XXXX-XXXX');
// → { valid: true, productName: 'My Plugin', features: {...} }

const activation = await teskel.licenses.activate('TSKL-XXXX-XXXX-XXXX', {
  fingerprint: 'machine-hash',
  label: 'MacBook Pro',
});

// Access
const access = await teskel.access.check({
  customerEmail: 'buyer@email.com',
  productId: 'prod_xxx',
});
// → { allowed: true, tier: 'pro', features: [...] }

// Webhooks (verification helper)
const isValid = teskel.webhooks.verify(requestBody, signatureHeader, secret);
```

### 9.3 Embed Widget API

```html
<!-- Checkout button -->
<button data-teskel-checkout="prod_xxx" data-teskel-price="price_xxx">
  Buy Now — $19
</button>

<!-- Product card -->
<div data-teskel-product="prod_xxx"></div>

<!-- Full store -->
<div data-teskel-store="creator-slug"></div>

<!-- Load TESKEL embed script -->
<script src="https://js.teskel.com/v1.js" async></script>

<!-- Configuration -->
<script>
  window.TeskelConfig = {
    theme: 'auto',          // light, dark, auto
    position: 'right',      // popup position
    locale: 'en',           // language
    onSuccess: (order) => { // callback after purchase
      console.log('Purchased!', order.id);
    },
  };
</script>
```

---

## 10. GROWTH & VIRAL MECHANICS

### 10.1 Built-in Virality

```
1. "Powered by TESKEL" badge on free-tier storefronts
   → Buyer sees TESKEL → potential creator signup

2. Success page share buttons
   → "I just bought [product]! [Share on Twitter]"
   → Social proof + discovery

3. Referral program (creator referral)
   → Creator refers another creator → both get 1 month Pro free
   → Creator referral link: teskel.com/r/creator-slug

4. Product launch notifications
   → Buyer follows creator → notification on new product
   → Re-engagement loop

5. Bundle cross-promotion
   → Creator A bundles with Creator B → both audiences exposed
   → Network growth

6. Marketplace trending
   → High-velocity products surface → social proof → more sales → more velocity
   → Flywheel
```

### 10.2 Creator Onboarding Optimization

```
First-sale activation:
  - Goal: Creator makes first sale within 7 days of signup
  - Onboarding email sequence (7 days):
    Day 0: Welcome + quick start guide
    Day 1: "Create your first product in 2 minutes"
    Day 2: "Share your store link on Twitter/LinkedIn"
    Day 3: "Set up a discount code to drive first sales"
    Day 5: "3 creators in your niche sold $X this week"
    Day 7: "Need help? Book a free 15-min setup call"

  - Dashboard nudges:
    "You haven't published a product yet. [Create one now →]"
    "Your store has no visits. Share it: [Copy link]"
    "Add a payment method to start receiving payouts"

  - Success metrics:
    - Time to first product: < 10 minutes
    - Time to first sale: < 7 days (for 30% of creators)
    - 30-day retention: > 60%
```

---

## 11. ACCESSIBILITY — WCAG 2.1 AA Compliance

```
Requirements:
  1. Color contrast ratio: 4.5:1 (text), 3:1 (large text, UI)
  2. Focus indicators: visible on all interactive elements
  3. Keyboard navigation: all features accessible without mouse
  4. Screen reader: proper ARIA labels, roles, live regions
  5. Motion: prefers-reduced-motion respected
  6. Text resize: content readable at 200% zoom
  7. Touch targets: minimum 44×44px on mobile
  8. Forms: labels associated, error messages linked, required indicated
  9. Images: alt text on all meaningful images
  10. Skip links: "Skip to main content" on every page

Testing:
  - Automated: axe-core in CI (fail build on violations)
  - Manual: keyboard-only navigation test per page
  - Screen reader: VoiceOver (Mac) / NVDA (Windows) spot checks
```

---

## 12. PWA & MOBILE — Modern Mobile Experience

```
PWA features:
  - manifest.json (name, icons, theme_color, start_url)
  - Service Worker (offline dashboard shell, cache API responses)
  - Install prompt (after 2 visits + engagement criteria)
  - Push notifications (via Service Worker)
  - Offline indicator banner

Mobile-specific UX:
  - Bottom navigation on mobile (instead of sidebar)
  - Pull-to-refresh on lists
  - Swipe actions on order/product rows
  - Haptic feedback on key actions (navigator.vibrate)
  - Share sheet integration (Web Share API)
  - Camera access for product image upload
  - Touch-optimized checkout (larger buttons, autofill)
```

---

## 13. FEATURE PRIORITY MATRIX

Which features make TESKEL "top global" vs "just another platform":

```
MUST-HAVE (differentiators):
  ✓ Multi-product-type support (13 types)
  ✓ License validation API (real infra, not just marketplace)
  ✓ Embed anywhere (commerce happens outside TESKEL)
  ✓ SDK/API-first (developer adoption)
  ✓ PPP pricing (true global access)
  ✓ Real-time dashboard (modern SaaS feel)
  ✓ Funnel builder (revenue optimization)
  ✓ Multi-currency (not just USD)
  ✓ Instant search (Typesense, not basic SQL LIKE)
  ✓ Beautiful default themes (no ugly stores)
  ✓ 1-click checkout (Apple Pay / Google Pay / Link)

SHOULD-HAVE (competitive advantage):
  ✓ Affiliate system (network growth)
  ✓ Bundle & collab (cross-creator revenue)
  ✓ Email sequences (retention)
  ✓ Custom domain (professional creators)
  ✓ Marketplace discovery (buyer acquisition)
  ✓ Webhook + event system (extensibility)
  ✓ Team management (agencies, studios)
  ✓ Analytics with cohorts (data-driven creators)
  ✓ Review & social proof (trust)
  ✓ Multi-language (global reach)

NICE-TO-HAVE (future moat):
  ○ AI pricing optimization
  ○ AI content generation
  ○ AI support agent
  ○ Live commerce
  ○ Web3 ownership tokens
  ○ Creator banking
  ○ Component marketplace
  ○ Protocol/federation
  ○ Agent commerce
```

---

## 14. COMPETITIVE POSITIONING — Why Choose TESKEL

```
┌──────────────────┬───────────┬──────────┬──────────┬───────────┬───────────┐
│ Feature          │ TESKEL    │ Gumroad  │ Whop     │ Lemon     │ Paddle    │
├──────────────────┼───────────┼──────────┼──────────┼───────────┼───────────┤
│ Product types    │ 13        │ 3        │ 5        │ 2         │ 1 (SaaS)  │
│ License API      │ ✅ Full   │ ❌       │ ❌       │ ✅ Basic  │ ✅ Full   │
│ Embed widget     │ ✅ Full   │ ❌       │ ❌       │ ✅ Basic  │ ✅ Basic  │
│ SDK/API          │ ✅ Full   │ ❌       │ ❌       │ ❌        │ ✅ Full   │
│ Funnel builder   │ ✅        │ ❌       │ ❌       │ ❌        │ ❌        │
│ Affiliate        │ ✅        │ ❌       │ ✅       │ ❌        │ ❌        │
│ PPP pricing      │ ✅ Auto   │ ❌       │ ❌       │ ❌        │ ❌        │
│ Multi-currency   │ ✅ 30+    │ ✅ USD   │ ✅ USD   │ ✅ 20+    │ ✅ 30+    │
│ Local payments   │ ✅ 50+    │ ❌       │ ❌       │ ✅ 10+    │ ✅ 20+    │
│ Custom domain    │ ✅        │ ✅       │ ✅       │ ❌        │ ❌        │
│ Real-time dash   │ ✅        │ ❌       │ ❌       │ ❌        │ ❌        │
│ Email sequences  │ ✅        │ ❌       │ ❌       │ ❌        │ ❌        │
│ Marketplace      │ ✅        │ ❌       │ ✅       │ ❌        │ ❌        │
│ Team roles       │ ✅ 4 roles│ ❌       │ ❌       │ ❌        │ ✅ 2 roles│
│ White-label      │ ✅        │ ❌       │ ❌       │ ❌        │ ❌        │
│ Platform fee     │ 1.5-5%   │ 10%      │ 5%       │ 5%        │ 5%+flat   │
│ Global-first     │ ✅        │ ❌       │ ❌       │ ✅ Partial│ ✅ EU     │
│ Open source      │ Partial   │ ❌       │ ❌       │ ❌        │ ❌        │
└──────────────────┴───────────┴──────────┴──────────┴───────────┴───────────┘
```

---

*This spec elevates TESKEL from "another digital product platform" to "world-class digital commerce OS" by implementing global-first i18n, design system, real-time features, advanced analytics, instant search, conversion-optimized checkout, developer-first API, accessibility, PWA, and viral growth mechanics.*
