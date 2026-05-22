# TESKEL — Build & Deploy Specification
## Phases, Environment, Deployment, Costs

---

## 1. BUILD PHASES — 12 Weeks to Launch

### PHASE 1: Foundation (Week 1-3)

**Week 1: Project Setup**
```
Day 1-2:
  □ pnpm create turbo monorepo
  □ Setup apps/web (Next.js 15 + App Router + Tailwind + shadcn/ui init)
  □ Setup apps/api (Hono + Drizzle)
  □ Setup packages/db, packages/shared, packages/sdk
  □ Setup pnpm-workspace.yaml + turbo.json
  □ Setup .env.example + .gitignore
  □ git init + first commit

Day 3-4:
  □ Setup Neon database (free tier)
  □ Write all Drizzle schemas (01_DATABASE.md)
  □ Run drizzle-kit generate + migrate
  □ Setup Better Auth (email+password, Google, GitHub)
  □ Auth pages: /login, /signup, /forgot-password, /reset-password, /verify-email
  □ Session management + middleware.ts

Day 5:
  □ Setup Cloudflare R2 bucket
  □ Setup Upstash Redis
  □ Setup Resend account
  □ Setup Stripe test mode
  □ Setup GitHub Actions CI (lint + typecheck + test)
  □ Deploy staging: Vercel (web) + Fly.io (api)
```

**Week 2: Core Primitives**
```
Day 1-2: Product CRUD
  □ API routes: products, prices, files
  □ Services: product.service, file.service
  □ Dashboard: product list page, new product wizard, product editor (details tab)
  □ R2 file upload with presigned URLs + progress
  □ Stripe product + price sync on create

Day 3-4: Organization & Settings
  □ API routes: settings, members
  □ Dashboard: settings pages (general, team, api-keys)
  □ Stripe Connect onboarding flow
  □ API key generation + hash storage

Day 5: Seed + Test
  □ Seed script (demo org + products + orders)
  □ Integration tests for product CRUD
  □ Manual testing of all flows
```

**Week 3: Checkout & Delivery**
```
Day 1-2: Checkout
  □ API route: checkout sessions
  □ Stripe Checkout session creation
  □ Checkout page UI (/checkout/[sessionId])
  □ Discount code input in checkout

Day 3-4: Webhooks & Orders
  □ Stripe webhook handler (checkout.session.completed)
  □ Order creation + order items
  □ Customer auto-creation/find
  □ Access grant creation
  □ Delivery record creation
  □ Receipt email (Resend + React Email template)

Day 5: Success & Delivery
  □ Success page (/success/[orderId]) with downloads
  □ Signed R2 download URLs (24h expiry)
  □ Delivery API endpoints
  □ Order list + detail pages in dashboard
```

---

### PHASE 2: Creator Experience (Week 4-6)

**Week 4: Dashboard Polish**
```
Day 1-2: Overview + Metrics
  □ Dashboard overview page (4 metric cards + revenue chart)
  □ Recharts integration
  □ Recent orders widget
  □ Top products widget
  □ Analytics event tracking setup

Day 3-4: Product Editor Complete
  □ Pricing tab (multi-tier editor)
  □ Files tab (upload + manage)
  □ Access rules tab
  □ Delivery config tab
  □ Analytics tab
  □ Settings tab (slug, publish, delete)

Day 5: UX Polish
  □ Command palette (cmdk)
  □ Toast notifications on all actions
  □ Skeleton loaders
  □ Empty states with CTAs
  □ Error boundaries
```

**Week 5: Storefront**
```
Day 1-2: Store Pages
  □ Storefront page (/[storeSlug]) — product grid
  □ Product detail page — full layout
  □ Buy button → checkout flow
  □ Store header + footer (creator-themed)

Day 3: Customization
  □ Theme customizer (colors, fonts, mode)
  □ Announcement bar
  □ Email capture popup
  □ Custom domain setup (Cloudflare for SaaS)

Day 4-5: SEO & Performance
  □ SSR with proper meta tags
  □ OG image generation (@vercel/og)
  □ JSON-LD structured data
  □ Dynamic sitemap
  □ Image optimization (next/image)
  □ Performance audit (Lighthouse >90)
```

**Week 6: License System**
```
Day 1-2: License API
  □ License key generation (on purchase for license-type)
  □ License validation API (GET /v1/licenses/:key/validate)
  □ Redis caching for validation (TTL 60s)
  □ Rate limiting (100/min)

Day 3-4: License Management
  □ License activation + deactivation
  □ Machine fingerprinting
  □ License management in dashboard
  □ Bulk generation
  □ License display on success page + buyer portal

Day 5: Testing
  □ End-to-end test: purchase → license → validate → activate
  □ Load test validation endpoint (k6)
  □ Fix bugs from week 4-6
```

---

### PHASE 3: Growth Features (Week 7-9)

**Week 7: Funnels & Email**
```
Day 1-3: Funnel Builder
  □ Funnel CRUD API
  □ Visual funnel builder (React Flow)
  □ Funnel step types: product_page, checkout, order_bump, upsell, downsell, thank_you
  □ Order bump component in checkout
  □ Upsell offer on success page
  □ Funnel analytics

Day 4-5: Email Engine
  □ Email template editor (TipTap)
  □ Email sequence builder
  □ Email subscriber management
  □ Abandoned checkout email (auto-trigger after 1h)
  □ Post-purchase email sequence
  □ Test email sending
```

**Week 8: Affiliate & Bundle**
```
Day 1-2: Affiliate System
  □ Affiliate CRUD API + dashboard
  □ Referral code tracking (cookie + URL param)
  □ Commission calculation on order
  □ Affiliate stats dashboard

Day 3-4: Bundle & Split Payment
  □ Bundle product type
  □ Bundle editor (add products from own or other orgs)
  □ Bundle delivery (recursive: trigger delivery for each item)
  □ Split payment calculation (owner_share_percent)
  □ Payout schedule creation

Day 5: Payout Processing
  □ Payout worker (process scheduled payouts via Stripe Transfer)
  □ Payout dashboard (pending, completed)
  □ Minimum payout threshold enforcement
```

**Week 9: Analytics & Marketplace v1**
```
Day 1-2: Analytics Dashboard
  □ Revenue chart (with time range selector)
  □ Orders analytics
  □ Product performance table
  □ Customer analytics
  □ Funnel analytics

Day 3-5: Marketplace
  □ Marketplace browse page (/explore)
  □ Category browsing
  □ Typesense integration for search
  □ Marketplace product detail page
  □ Creator profile page
  □ Product reviews + ratings
  □ Marketplace listing management in dashboard
```

---

### PHASE 4: Launch Prep (Week 10-12)

**Week 10: Embed & SDK**
```
Day 1-3: Embed Widget
  □ Rollup build (UMD + ESM)
  □ Checkout embed (popup modal)
  □ Product card embed
  □ Store embed
  □ CSS scoping (shadow DOM or data attributes)
  □ Deploy to Cloudflare Workers (js.teskel.com)
  □ Documentation page for embed

Day 4-5: SDK v1
  □ @teskel/sdk package
  □ Client class + resource classes
  □ Type-safe request/response
  □ Error handling
  □ README + usage examples
  □ Publish to npm
```

**Week 11: Polish & Security**
```
Day 1-2: Landing Page
  □ Full marketing site (/)
  □ Pricing page (/pricing)
  □ Documentation site (/docs)
  □ Blog setup (/blog)
  □ Legal pages (terms, privacy, dpa)

Day 3: Security
  □ Rate limiting on all public endpoints
  □ CSRF protection
  □ Input sanitization (XSS prevention)
  □ SQL injection prevention (Drizzle parameterized queries)
  □ Content Security Policy headers
  □ CORS configuration
  □ API key scope enforcement

Day 4-5: Performance & Accessibility
  □ Image optimization audit
  □ Bundle size analysis
  □ Mobile responsive polish
  □ Accessibility audit (WCAG 2.1 AA)
  □ Loading states + error pages
  □ 404 page
```

**Week 12: LAUNCH**
```
Day 1-2: Production Setup
  □ Neon production database
  □ Vercel production deployment
  □ Fly.io production deployment
  □ Cloudflare R2 production bucket
  □ Stripe live mode
  □ DNS: teskel.com, app.teskel.com, api.teskel.com
  □ SSL certificates (auto via Vercel/Cloudflare)

Day 3: Email & Monitoring
  □ Resend production domain + DNS (SPF, DKIM, DMARC)
  □ Sentry production project
  □ PostHog production project
  □ Uptime monitoring (Better Stack or similar)
  □ Alert rules (error rate >1%, response time >2s)

Day 4: Testing & Backup
  □ Full end-to-end test in production
  □ Load test (k6): 100 concurrent checkouts
  □ Database backup strategy (Neon PITR)
  □ R2 backup strategy
  □ Incident response runbook

Day 5: LAUNCH 🚀
  □ Beta creator onboarding (10 creators)
  □ Product hunt submission
  □ Twitter/X announcement
  □ Indie Hackers post
  □ Monitor closely for 48h
```

---

## 2. ENVIRONMENT VARIABLES

```bash
# ═══ DATABASE ═══
DATABASE_URL=postgresql://user:pass@ep-xxx.us-east-2.aws.neon.tech/teskel?sslmode=require
DATABASE_URL_DIRECT=postgresql://user:pass@ep-xxx.us-east-2.aws.neon.tech/teskel

# ═══ REDIS ═══
REDIS_URL=redis://default:xxx@us1-xxx-12345.upstash.io:6379

# ═══ AUTH ═══
BETTER_AUTH_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
BETTER_AUTH_URL=https://app.teskel.com

# ═══ OAUTH ═══
GOOGLE_CLIENT_ID=123456789.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxxxxxxxxxxxxx
GITHUB_CLIENT_ID=Iv1.xxxxxxxxxxxxxxxx
GITHUB_CLIENT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# ═══ STRIPE ═══
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxx
STRIPE_PUBLISHABLE_KEY=pk_live_xxxxxxxxxxxxxxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
STRIPE_CONNECT_CLIENT_ID=ca_xxxxxxxxxxxxxxxxxxxxxxxx

# ═══ STORAGE ═══
R2_ACCOUNT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
R2_ACCESS_KEY_ID=xxxxxxxxxxxxxxxxxxxx
R2_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
R2_BUCKET_NAME=teskel-files
R2_PUBLIC_URL=https://files.teskel.com

# ═══ EMAIL ═══
RESEND_API_KEY=re_xxxxxxxxxxxxxxxxxxxxxxxx
EMAIL_FROM=noreply@teskel.com
EMAIL_FROM_NAME="TESKEL"

# ═══ SEARCH ═══
TYPESENSE_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TYPESENSE_HOST=xxxxxxxx.xxx.typesense.net
TYPESENSE_PORT=443
TYPESENSE_PROTOCOL=https

# ═══ MONITORING ═══
SENTRY_DSN=https://xxxxxxxx@o123456.ingest.sentry.io/123456
NEXT_PUBLIC_POSTHOG_KEY=phc_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com

# ═══ APP URLS ═══
NEXT_PUBLIC_APP_URL=https://app.teskel.com
NEXT_PUBLIC_API_URL=https://api.teskel.com
NEXT_PUBLIC_EMBED_URL=https://js.teskel.com
NEXT_PUBLIC_STRIPE_PK=pk_live_xxxxxxxxxxxxxxxxxxxxxxxx

# ═══ NODE ═══
NODE_ENV=production
```

---

## 3. DEPLOYMENT

### 3.1 Vercel (Frontend — apps/web)

```
Project: teskel-web
Framework: Next.js
Build: pnpm build --filter=web
Output: .next
Domains:
  - teskel.com (marketing)
  - app.teskel.com (dashboard + all app routes)
  - *.teskel.com (creator storefronts — catch-all)

Environment: All NEXT_PUBLIC_* vars + DATABASE_URL + BETTER_AUTH_*

Deploy: Auto on push to main (via GitHub integration)
Preview: Auto on PR (with Neon branch)
```

### 3.2 Fly.io (API — apps/api)

```
App: teskel-api
Region: sjc (US West) + fra (EU) + sin (Asia)
Machine: shared-cpu-1x (256MB RAM)
Scale: 2 machines per region
Dockerfile: apps/api/Dockerfile

fly.toml:
  [deploy]
  release_command = "pnpm db:migrate"
  
  [http_service]
  internal_port = 3001
  force_https = true
  auto_stop_machines = "stop"
  auto_start_machines = true
  min_machines_running = 1

Domains:
  - api.teskel.com

Environment: All non-NEXT_PUBLIC vars
Secrets: STRIPE_SECRET_KEY, etc (via fly secrets)
```

### 3.3 Cloudflare Workers (Embed — apps/embed)

```
Worker: teskel-embed
Route: js.teskel.com/*
Build: rollup (UMD + ESM)
KV: None needed (stateless)

wrangler.toml:
  name = "teskel-embed"
  main = "dist/index.js"
  routes = [{ pattern = "js.teskel.com/*", zone_name = "teskel.com" }]
```

### 3.4 Cloudflare R2 (File Storage)

```
Bucket: teskel-files
Public access: via files.teskel.com (custom domain on R2)
CORS: Allow GET from *.teskel.com and custom domains
Lifecycle: None (files kept permanently)
```

### 3.5 Neon (Database)

```
Project: teskel-prod
Region: aws-us-east-2
Compute: 0.25 CU (auto-suspend 5min)
Branching: Enabled (preview branches for PRs)
PITR: 7 days (free tier)
Read replica: 1 (for analytics queries)
```

---

## 4. CI/CD PIPELINE

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  lint-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - uses: actions/setup-node@v4
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install --frozen-lockfile
      - run: pnpm test
      - run: pnpm test:e2e

  deploy-web:
    needs: [lint-typecheck, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}

  deploy-api:
    needs: [lint-typecheck, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: superfly/flyctl-actions@v1
        with:
          args: deploy apps/api
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

---

## 5. COST ESTIMATE

### Pre-Launch (Development)

| Service | Plan | Monthly Cost |
|---------|------|-------------|
| Vercel | Pro | $20 |
| Fly.io | Hobby | $0-10 |
| Neon | Launch | $19 |
| Upstash | Pay-as-you-go | $0-5 |
| Cloudflare R2 | Free tier | $0 |
| Cloudflare Workers | Free tier | $0 |
| Resend | Free tier (100 emails/day) | $0 |
| Typesense | Development | $0 |
| Sentry | Team | $26 |
| PostHog | Free tier | $0 |
| GitHub Actions | Free tier | $0 |
| Domain | .com | $12/yr |
| **Total** | | **~$75-85/mo** |

### Post-Launch (Scale to 500 creators)

| Service | Plan | Monthly Cost |
|---------|------|-------------|
| Vercel | Pro | $20 |
| Fly.io | 3 machines × 3 regions | $60-100 |
| Neon | Scale | $69 |
| Upstash | Pro | $30 |
| Cloudflare R2 | ~100GB | $1.50 |
| Cloudflare Workers | Paid | $5 |
| Resend | Pro | $20 |
| Typesense | Standard | $30 |
| Sentry | Team | $26 |
| PostHog | Free tier | $0 |
| **Total** | | **~$260-300/mo** |

---

## 6. KEY DEPENDENCY VERSIONS

```json
{
  "dependencies": {
    "next": "15.x",
    "react": "19.x",
    "react-dom": "19.x",
    "hono": "4.x",
    "drizzle-orm": "0.36.x",
    "@neondatabase/serverless": "0.10.x",
    "@upstash/redis": "1.x",
    "@upstash/ratelimit": "2.x",
    "better-auth": "1.x",
    "stripe": "17.x",
    "resend": "4.x",
    "@react-email/components": "0.x",
    "typesense": "1.x",
    "@tanstack/react-table": "8.x",
    "recharts": "2.x",
    "@xyflow/react": "12.x",
    "cmdk": "1.x",
    "sonner": "1.x",
    "tiptap": "2.x",
    "zod": "3.x",
    "tailwindcss": "4.x",
    "class-variance-authority": "0.7.x",
    "clsx": "2.x",
    "tailwind-merge": "2.x",
    "lucide-react": "0.x"
  },
  "devDependencies": {
    "drizzle-kit": "0.28.x",
    "typescript": "5.x",
    "eslint": "9.x",
    "prettier": "3.x",
    "turbo": "2.x",
    "vitest": "2.x",
    "playwright": "1.x"
  }
}
```

---

## 7. FILE COUNT ESTIMATE

| Category | Files | Lines (approx) |
|----------|-------|----------------|
| Database schema | 1 | 800 |
| API routes | 20 | 3,000 |
| API services | 15 | 4,000 |
| API middleware | 6 | 500 |
| API workers | 7 | 1,500 |
| Frontend pages | 55 | 5,000 |
| Frontend components | 65 | 8,000 |
| Frontend hooks/lib | 20 | 1,500 |
| Server actions | 5 | 500 |
| Shared types/validators | 10 | 1,000 |
| SDK | 8 | 1,200 |
| Embed widget | 5 | 800 |
| Email templates | 10 | 500 |
| CLI | 5 | 400 |
| Config files | 15 | 300 |
| **TOTAL** | **~247** | **~29,000** |

---

## 8. TESTING STRATEGY

```
Unit Tests (vitest):
  - All service functions
  - Utility functions
  - Validators (Zod schemas)
  - License key generation
  - Price calculation (discounts, tax, splits)

Integration Tests (vitest + test DB):
  - API endpoint tests (with test database)
  - Stripe webhook handler
  - Checkout flow
  - License validation
  - Access check

E2E Tests (Playwright):
  - Signup → onboarding → create product → publish
  - Buyer: browse → checkout → download
  - Dashboard: all CRUD operations
  - Storefront: product page → buy
  - License: purchase → validate → activate

Load Tests (k6):
  - License validation: 1000 req/s
  - Checkout creation: 100 concurrent
  - Marketplace search: 500 concurrent
```

---

## 9. MONITORING & ALERTS

```
Sentry:
  - Error tracking (frontend + backend)
  - Performance monitoring
  - Release tracking

PostHog:
  - Feature usage
  - Funnel conversion
  - User flows

Uptime (Better Stack):
  - api.teskel.com (ping every 60s)
  - app.teskel.com (ping every 60s)
  - Alert on 3 consecutive failures

Alert Rules:
  - Error rate > 1% → Slack alert
  - p95 response time > 2s → Slack alert
  - Checkout success rate < 90% → PagerDuty
  - Database connection pool > 80% → Slack alert
```

---

*This is the complete build specification. Read 00 → 01 → 02 → 03 → 04 in order, then start building Phase 1 Week 1.*
