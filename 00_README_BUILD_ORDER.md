# TESKEL — Build Guideline (Karpathy-Inspired)
## The CLAUDE.md-Equivalent for Building TESKEL

> *"LLMs are exceptionally good at looping until they meet specific goals. Don't tell it what to do, give it success criteria and watch it go."* — Andrej Karpathy

---

## 0. KARPATHY PRINCIPLES — Applied to TESKEL

These 4 principles govern EVERY line of code written for TESKEL. Violation = rewrite.

### Principle 1: Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before writing ANY code for TESKEL:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

**TESKEL-specific assumptions to always state:**
- Which org context? (multi-tenant: every query needs org_id)
- Which auth method? (session cookie vs API key vs public endpoint)
- Which product type? (13 types with different delivery logic)
- Is this a write path? (needs idempotency, transaction, webhook)
- Is this a read path? (can use cache, read replica, eventual consistency)

### Principle 2: Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

**TESKEL-specific simplicity rules:**
- Don't build all 13 product types at once. Build `download` first, then add types one by one.
- Don't build the full funnel editor on day 1. Build a simple funnel with 3 fixed steps first.
- Don't add caching until you have a performance problem. Measure first.
- Don't add RLS policies until you have multiple orgs. Start with application-level org_id filtering.
- Don't add Stripe Tax integration until a creator asks for it.
- Don't build the embed widget until the core checkout works perfectly.
- Don't build the SDK until the API is stable and documented.
- Don't add AI features until the basic product CRUD + checkout + delivery flow works end-to-end.

**The test:** Would a senior engineer say this is overcomplicated? If yes, simplify.

### Principle 3: Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing TESKEL code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

**The test:** Every changed line should trace directly to the user's request.

### Principle 4: Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add product CRUD" → "Write tests for create/read/update/delete, then make them pass"
- "Fix checkout bug" → "Write a test that reproduces it, then make it pass"
- "Add license validation" → "Test: valid key returns 200, invalid returns 404, rate-limited after 100 req/min"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

---

## 1. TESKEL-SPECIFIC CODING RULES

### 1.1 TypeScript Rules

```typescript
// ALWAYS: strict mode
// tsconfig.json: { "strict": true, "noUncheckedIndexedAccess": true }

// ALWAYS: explicit return types on exported functions
export function calculatePlatformFee(amountCents: number, plan: string): number {
  const fees: Record<string, number> = { free: 0.05, pro: 0.03, scale: 0.015, enterprise: 0.005 };
  return Math.round(amountCents * (fees[plan] ?? fees.free));
}

// ALWAYS: Zod validation on ALL API inputs
const createProductSchema = z.object({
  name: z.string().min(1).max(255),
  type: z.enum(['download', 'license', 'prompt_pack', 'micro_saas', 'service', 'community', 'course', 'bundle', 'pwyw', 'tiered', 'link', 'collection', 'agent']),
  pricingModel: z.enum(['one_time', 'subscription', 'pwyw', 'tiered', 'free', 'usage_based']),
  description: z.string().optional(),
  category: z.string().max(100).optional(),
  tags: z.array(z.string()).default([]),
});

// NEVER: any type
// NEVER: non-null assertion (!) without a guard
// NEVER: console.log in production code (use logger)
// NEVER: catch (error) {} with empty handler
// NEVER: async function without error handling
```

### 1.2 Database Rules

```typescript
// ALWAYS: org_id filter on every org-scoped query
// CORRECT:
const products = await db.query.products.findMany({
  where: and(eq(products.orgId, orgId), eq(products.status, 'active')),
});

// WRONG (no org_id filter = data leak between orgs):
const products = await db.query.products.findMany({
  where: eq(products.status, 'active'),
});

// ALWAYS: Use Drizzle ORM, never raw SQL
// ALWAYS: Use sql`` template for increments (atomic)
await db.update(products)
  .set({ totalSales: sql`${products.totalSales} + 1` })
  .where(eq(products.id, productId));

// NEVER: SELECT * — always specify columns
// NEVER: N+1 queries — use with: {} for relations or batch queries
// NEVER: Store plaintext secrets — hash API keys, encrypt tokens
```

### 1.3 API Rules

```typescript
// ALWAYS: Validate input with Zod before processing
// ALWAYS: Return consistent error format
interface ApiError {
  error: { code: string; message: string; details?: any };
}

// ALWAYS: Set rate limit headers
// X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

// ALWAYS: Idempotency on write endpoints
// Check if operation already processed before executing
// Use stripe_session_id for checkout, idempotency_key for other writes

// ALWAYS: Log org_id + user_id on every request (structured logging)

// NEVER: Return password hashes, API key values, or Stripe secrets in responses
// NEVER: Trust client-side data without validation
// NEVER: Process Stripe webhooks synchronously — acknowledge fast, process async
```

### 1.4 Frontend Rules

```typescript
// ALWAYS: Use server components by default, client components only when needed
// ALWAYS: 'use client' directive at top of client components
// ALWAYS: Use Server Actions for mutations (not fetch to own API)
// ALWAYS: Optimistic updates for simple mutations (with rollback on error)
// ALWAYS: Loading skeletons (not spinners) for async content
// ALWAYS: Error boundaries at route level
// ALWAYS: Toast notifications (sonner) for action feedback

// NEVER: Fetch data in client components when server component can do it
// NEVER: Store sensitive data in localStorage (use httpOnly cookies)
// NEVER: Build custom form components — use shadcn/ui Form + Zod
// NEVER: Inline styles — use Tailwind classes
// NEVER: Hardcode colors — use CSS variables from shadcn theme
```

### 1.5 Testing Rules

```typescript
// ALWAYS: Write test FIRST for bug fixes (reproduce → fix → verify)
// ALWAYS: Integration test for every API endpoint
// ALWAYS: Test the happy path + 2 error paths per endpoint
// ALWAYS: Test multi-tenant isolation (org A can't see org B's data)

// Test structure:
describe('POST /v1/orgs/:slug/products', () => {
  it('creates a product with valid input', async () => { /* ... */ });
  it('returns 401 without auth', async () => { /* ... */ });
  it('returns 403 for viewer role', async () => { /* ... */ });
  it('returns 422 for invalid product type', async () => { /* ... */ });
  it('returns 409 for duplicate slug', async () => { /* ... */ });
});
```

---

## 2. BUILD ORDER — Karpathy-Style with Verification

### Phase 1: Foundation (Week 1-3)

**Rule: Build the simplest end-to-end flow FIRST. One product type. One payment. One delivery.**

```
Step 1: Project scaffold
  → verify: pnpm install succeeds, pnpm build succeeds, pnpm test passes (empty)

Step 2: Database schema (users, organizations, organization_members ONLY)
  → verify: drizzle-kit generate succeeds, drizzle-kit migrate succeeds, drizzle-kit studio shows tables

Step 3: Auth (Better Auth: email+password, Google, GitHub)
  → verify: Can signup, login, logout. Session persists. Middleware redirects unauthenticated.

Step 4: Onboarding (4-step wizard)
  → verify: New user completes onboarding → org created → redirected to dashboard

Step 5: Database schema (products, product_prices, product_files)
  → verify: Migration succeeds. Can insert product with prices and files.

Step 6: Product CRUD API (POST, GET, PATCH, DELETE /v1/orgs/:slug/products)
  → verify: Integration tests pass for all CRUD operations. org_id filter enforced.

Step 7: Product CRUD Dashboard UI (list, new, edit)
  → verify: Can create product, see it in list, edit it, archive it. Files upload to R2.

Step 8: Stripe Connect onboarding
  → verify: Creator clicks "Connect Stripe" → completes onboarding → stripeAccountStatus = 'active'

Step 9: Checkout flow (download type ONLY)
  → verify: 
    - POST /v1/checkout/sessions returns Stripe URL
    - Stripe webhook (checkout.session.completed) creates order + customer + access grant + delivery
    - Success page shows download link
    - Download link works (signed R2 URL)
    - Receipt email received

Step 10: Order list + detail in dashboard
  → verify: Can see orders, click detail, see line items, see customer info

*** MILESTONE: End-to-end purchase flow works for "download" product type ***
*** This is the MINIMUM VIABLE PRODUCT. Everything else builds on top. ***
```

### Phase 2: Expand (Week 4-6)

**Rule: Add features one at a time. Test each before moving on.**

```
Step 11: License product type
  → verify: Purchase license product → key generated → validate API works → activate works → dashboard shows keys

Step 12: Prompt pack product type
  → verify: Purchase prompt pack → download works → same as download but type=prompt_pack

Step 13: Community product type (Discord only first)
  → verify: Purchase community product → invite link generated → Discord bot sends invite

Step 14: Subscription pricing
  → verify: Purchase subscription product → Stripe subscription created → access grant created → renewal webhook extends access → cancel revokes access

Step 15: Discount codes
  → verify: Create discount code → apply in checkout → order shows discount → usage count increments

Step 16: Storefront
  → verify: Visit /[storeSlug] → see products → click product → buy → checkout → download

Step 17: Custom domain
  → verify: Add custom domain → DNS records shown → verify DNS → store loads on custom domain

Step 18: Theme customization
  → verify: Change primary color → storefront updates → change font → storefront updates

*** MILESTONE: Creator can sell download, license, prompt_pack, community, subscription products with discounts and custom storefront ***
```

### Phase 3: Growth (Week 7-9)

**Rule: Each feature is independently testable and deployable.**

```
Step 19: Funnel builder (simple: 3 fixed steps — product page → checkout → thank you)
  → verify: Create funnel → add steps → visit funnel URL → walk through steps → order attributed to funnel

Step 20: Order bump
  → verify: Funnel with order bump → checkout shows bump → accept bump → order includes bump item

Step 21: Upsell
  → verify: Funnel with upsell → success page shows upsell → accept upsell → second order created

Step 22: Affiliate system
  → verify: Create affiliate → share referral link → purchase with link → commission calculated → affiliate dashboard shows stats

Step 23: Bundle product type
  → verify: Create bundle with 3 items → purchase bundle → all 3 items delivered → split payout calculated

Step 24: Email sequences (abandoned checkout only first)
  → verify: Start checkout → abandon → 1 hour later → email received → click email → return to checkout

Step 25: Marketplace v1
  → verify: Product opted into marketplace → appears on /explore → search finds it → buy from marketplace → same flow as storefront

Step 26: Analytics dashboard
  → verify: Dashboard shows revenue chart → order count → top products → data matches actual orders

*** MILESTONE: Full creator toolkit — funnels, affiliates, bundles, emails, marketplace, analytics ***
```

### Phase 4: Launch (Week 10-12)

**Rule: Polish what exists. Don't add new features.**

```
Step 27: Embed widget (checkout popup only)
  → verify: Embed script tag → click buy button → popup checkout → payment → success

Step 28: SDK v1 (TypeScript)
  → verify: npm install @teskel/sdk → create client → list products → create checkout → validate license

Step 29: Landing page
  → verify: All sections render → mobile responsive → Lighthouse >90 → pricing toggle works → CTAs link to signup

Step 30: Security audit
  → verify: Rate limiting on all public endpoints → CSRF protection → input validation → no PII in logs → RLS check

Step 31: Performance audit
  → verify: Lighthouse >90 → API p95 <200ms → license validation <50ms → storefront <2s FCP

Step 32: Production deploy
  → verify: 
    - DNS resolves (teskel.com, app.teskel.com, api.teskel.com)
    - SSL certificates valid
    - Stripe live mode works
    - Email deliverability (SPF, DKIM, DMARC pass)
    - Sentry captures errors
    - Monitoring alerts configured
    - Backup verified
    - Load test: 100 concurrent checkouts succeed

*** MILESTONE: LAUNCH 🚀 ***
```

---

## 3. FILE STRUCTURE — What to Create, In Order

### 3.1 Phase 1 Files (Create in This Exact Order)

```
1.  package.json                    — Root monorepo config
2.  pnpm-workspace.yaml             — Workspace definition
3.  turbo.json                      — Turborepo config
4.  .gitignore
5.  .env.example                    — All env vars documented
6.  tsconfig.base.json              — Shared TS config

7.  packages/db/package.json
8.  packages/db/client.ts           — Neon serverless client
9.  packages/db/schema/index.ts     — Re-export all schemas
10. packages/db/schema/users.ts
11. packages/db/schema/organizations.ts
12. packages/db/schema/products.ts
13. packages/db/schema/orders.ts
14. packages/db/schema/customers.ts
15. packages/db/schema/licenses.ts
16. packages/db/schema/access.ts
17. packages/db/schema/subscriptions.ts
18. packages/db/schema/deliveries.ts
19. packages/db/schema/discounts.ts
20. packages/db/schema/funnels.ts
21. packages/db/schema/affiliates.ts
22. packages/db/schema/emails.ts
23. packages/db/schema/webhooks.ts
24. packages/db/schema/marketplace.ts
25. packages/db/schema/analytics.ts
26. packages/db/schema/api-keys.ts
27. packages/db/schema/payouts.ts
28. packages/db/migrations/         — Generated by drizzle-kit

29. packages/shared/package.json
30. packages/shared/types/          — Shared TypeScript types
31. packages/shared/validators/     — Shared Zod schemas
32. packages/shared/utils/          — format-currency, slugify, generate-license-key, signing

33. apps/web/package.json
34. apps/web/next.config.ts
35. apps/web/tailwind.config.ts
36. apps/web/styles/globals.css
37. apps/web/middleware.ts           — Auth redirect
38. apps/web/app/layout.tsx         — Root layout
39. apps/web/app/not-found.tsx      — 404

    # Auth pages
40. apps/web/app/(auth)/layout.tsx
41. apps/web/app/(auth)/login/page.tsx
42. apps/web/app/(auth)/signup/page.tsx
43. apps/web/app/(auth)/forgot-password/page.tsx
44. apps/web/app/(auth)/reset-password/page.tsx
45. apps/web/app/(auth)/verify-email/page.tsx

    # Onboarding
46. apps/web/app/(auth)/onboarding/layout.tsx
47. apps/web/app/(auth)/onboarding/page.tsx          — Step 1: Profile
48. apps/web/app/(auth)/onboarding/store/page.tsx    — Step 2: Store
49. apps/web/app/(auth)/onboarding/product/page.tsx  — Step 3: First product
50. apps/web/app/(auth)/onboarding/stripe/page.tsx   — Step 4: Stripe Connect

    # Dashboard
51. apps/web/app/(app)/layout.tsx
52. apps/web/app/(app)/dashboard/layout.tsx         — Sidebar + topbar
53. apps/web/app/(app)/dashboard/page.tsx            — Overview
54. apps/web/app/(app)/dashboard/products/page.tsx  — Product list
55. apps/web/app/(app)/dashboard/products/new/page.tsx
56. apps/web/app/(app)/dashboard/products/[id]/page.tsx
57. apps/web/app/(app)/dashboard/orders/page.tsx
58. apps/web/app/(app)/dashboard/orders/[id]/page.tsx
59. apps/web/app/(app)/dashboard/customers/page.tsx
60. apps/web/app/(app)/dashboard/customers/[id]/page.tsx
61. apps/web/app/(app)/dashboard/settings/page.tsx
62. apps/web/app/(app)/dashboard/settings/team/page.tsx
63. apps/web/app/(app)/dashboard/settings/payouts/page.tsx
64. apps/web/app/(app)/dashboard/settings/domain/page.tsx
65. apps/web/app/(app)/dashboard/settings/theme/page.tsx
66. apps/web/app/(app)/dashboard/settings/webhooks/page.tsx
67. apps/web/app/(app)/dashboard/settings/api-keys/page.tsx

    # Storefront
68. apps/web/app/(store)/[storeSlug]/layout.tsx
69. apps/web/app/(store)/[storeSlug]/page.tsx
70. apps/web/app/(store)/[storeSlug]/products/[productSlug]/page.tsx

    # Checkout
71. apps/web/app/(checkout)/checkout/[sessionId]/page.tsx
72. apps/web/app/(checkout)/success/[orderId]/page.tsx

    # Buyer portal
73. apps/web/app/(buyer)/purchases/page.tsx
74. apps/web/app/(buyer)/purchases/[id]/page.tsx
75. apps/web/app/(buyer)/subscriptions/page.tsx
76. apps/web/app/(buyer)/licenses/page.tsx

    # API routes (webhooks only)
77. apps/web/app/api/webhooks/stripe/route.ts

    # Components (create as needed, not all upfront)
78. apps/web/components/ui/          — shadcn/ui init, add components as needed
79. apps/web/components/dashboard/  — Create per page, not all at once
80. apps/web/components/store/
81. apps/web/components/checkout/
82. apps/web/components/shared/

    # Lib
83. apps/web/lib/auth.ts
84. apps/web/lib/api.ts
85. apps/web/lib/utils.ts
86. apps/web/lib/validators.ts

    # Hooks
87. apps/web/hooks/use-auth.ts
88. apps/web/hooks/use-debounce.ts

89. apps/api/package.json
90. apps/api/tsconfig.json
91. apps/api/Dockerfile
92. apps/api/fly.toml
93. apps/api/src/index.ts              — Hono app entry
94. apps/api/src/config/env.ts         — Zod-validated env
95. apps/api/src/config/stripe.ts      — Stripe client init
96. apps/api/src/middleware/auth.ts
97. apps/api/src/middleware/rate-limit.ts
98. apps/api/src/middleware/cors.ts
99. apps/api/src/middleware/error-handler.ts
100. apps/api/src/middleware/tenant-resolver.ts
101. apps/api/src/routes/auth.ts
102. apps/api/src/routes/products.ts
103. apps/api/src/routes/prices.ts
104. apps/api/src/routes/files.ts
105. apps/api/src/routes/checkout.ts
106. apps/api/src/routes/orders.ts
107. apps/api/src/routes/customers.ts
108. apps/api/src/routes/licenses.ts
109. apps/api/src/routes/access.ts
110. apps/api/src/routes/delivery.ts
111. apps/api/src/routes/subscriptions.ts
112. apps/api/src/routes/discounts.ts
113. apps/api/src/routes/funnels.ts
114. apps/api/src/routes/affiliates.ts
115. apps/api/src/routes/analytics.ts
116. apps/api/src/routes/marketplace.ts
117. apps/api/src/routes/emails.ts
118. apps/api/src/routes/webhooks.ts
119. apps/api/src/routes/settings.ts
120. apps/api/src/services/checkout.service.ts
121. apps/api/src/services/product.service.ts
122. apps/api/src/services/license.service.ts
123. apps/api/src/services/access.service.ts
124. apps/api/src/services/delivery.service.ts
125. apps/api/src/services/payout.service.ts
126. apps/api/src/services/discount.service.ts
127. apps/api/src/services/funnel.service.ts
128. apps/api/src/services/affiliate.service.ts
129. apps/api/src/services/analytics.service.ts
130. apps/api/src/services/email.service.ts
131. apps/api/src/services/webhook.service.ts
132. apps/api/src/services/search.service.ts
133. apps/api/src/workers/stripe-webhook.worker.ts
134. apps/api/src/workers/delivery.worker.ts
135. apps/api/src/workers/email.worker.ts
136. apps/api/src/workers/payout.worker.ts
137. apps/api/src/workers/analytics.worker.ts
138. apps/api/src/workers/webhook-retry.worker.ts

    # Email templates
139. packages/email/package.json
140. packages/email/templates/purchase-receipt.tsx
141. packages/email/templates/license-delivery.tsx
142. packages/email/templates/subscription-confirmed.tsx
143. packages/email/templates/community-invite.tsx
144. packages/email/templates/abandoned-checkout.tsx
145. packages/email/templates/welcome.tsx
146. packages/email/sender.ts

    # CI/CD
147. .github/workflows/ci.yml
148. .github/workflows/deploy-api.yml
149. .github/workflows/deploy-web.yml
```

---

## 4. DECISION RECORD — Assumptions Made Explicit

Every assumption that could go wrong, documented here:

| # | Assumption | If Wrong | Mitigation |
|---|-----------|----------|------------|
| 1 | Better Auth can handle 10K users | Need to migrate to Clerk | Better Auth is self-hosted, can scale with DB |
| 2 | Stripe Connect supports all target countries | Some countries unsupported | Start with US/EU/ID, expand later |
| 3 | Neon serverless can handle 100 concurrent connections | Need connection pooler | PgBouncer built into Neon |
| 4 | R2 signed URLs are sufficient for file delivery | Need CDN for high-traffic files | R2 has built-in CDN via custom domain |
| 5 | Upstash Redis is fast enough for rate limiting | Need local Redis | Upstash has <10ms latency globally |
| 6 | Typesense can handle 100K product search | Need Algolia | Typesense scales horizontally |
| 7 | Hono on Fly.io can handle 1000 req/s | Need more machines | Auto-scale configured |
| 8 | Next.js ISR is sufficient for storefront caching | Need custom CDN purge | Cloudflare Cache API available |
| 9 | Resend can deliver 10K emails/day | Need SendGrid fallback | Resend Pro = 100K/month |
| 10 | Stripe Tax handles international tax | Some jurisdictions unsupported | Start US-only for tax, expand later |

---

## 5. ANTI-PATTERNS — What NOT to Do

```
❌ DON'T: Build all 13 product types at once
✅ DO: Build download type first, add others when needed

❌ DON'T: Build the funnel visual editor before basic funnel CRUD works
✅ DO: Build funnel as a simple form first, add visual editor later

❌ DON'T: Add Redis caching on day 1
✅ DO: Add caching when you measure a slow query (>100ms)

❌ DON'T: Create abstract base classes for services
✅ DO: Write plain functions until you see a repeated pattern, then extract

❌ DON'T: Add WebSocket real-time updates on day 1
✅ DO: Use polling (5s interval) first, upgrade to WebSocket when needed

❌ DON'T: Build a custom rich text editor
✅ DO: Use TipTap (existing, well-maintained)

❌ DON'T: Implement your own payment processing
✅ DO: Use Stripe for everything payment-related

❌ DON'T: Write 500-line service files
✅ DO: Keep services <200 lines. Extract when they grow.

❌ DON'T: Add features "because competitors have them"
✅ DO: Add features when creators ask for them

❌ DON'T: Deploy everything to production at once
✅ DO: Deploy one feature at a time, verify, then next

❌ DON'T: Skip writing tests "to move faster"
✅ DO: Write tests first for bug fixes, integration tests for API endpoints

❌ DON'T: Use Drizzle relations for complex queries
✅ DO: Use explicit joins for complex queries, relations for simple ones

❌ DON'T: Store files in the database
✅ DO: Store files in R2, store metadata (storage key) in database

❌ DON'T: Process Stripe webhooks synchronously
✅ DO: Acknowledge immediately (200), process in background worker

❌ DON'T: Trust any data from the client
✅ DO: Validate ALL inputs with Zod on the server
```

---

## 6. VERIFICATION CHECKLIST — Per Feature

Every feature must pass ALL of these before being considered "done":

```
□ Code compiles (pnpm typecheck passes)
□ Lint passes (pnpm lint passes)
□ Unit tests pass (if applicable)
□ Integration test pass (for API endpoints)
□ Multi-tenant isolation verified (org A can't access org B's data)
□ Error handling: happy path + 2 error paths tested
□ No PII in logs
□ Rate limiting applied (if public endpoint)
□ Auth check applied (if protected endpoint)
□ Cache invalidation handled (if cached data)
□ Webhook fired (if applicable)
□ Analytics event logged (if applicable)
□ Email sent (if applicable)
□ Mobile responsive (if UI)
□ Dark mode works (if UI)
□ Loading state shown (if async UI)
□ Error state shown (if async UI)
□ Empty state shown (if list UI)
```

---

## 7. ARCHITECTURE DECISIONS — Why, Not Just What

| Decision | Why | Alternative Rejected | When to Revisit |
|----------|-----|---------------------|-----------------|
| Monorepo | Shared types between web/api/sdk | Polyrepo | When team >10 people |
| Next.js App Router | RSC, ISR, edge runtime | Pages Router | Never |
| Hono for API | Edge-native, 3x faster than Express | Express, Fastify | Never |
| Drizzle ORM | Edge-compatible, type-safe, SQL-like | Prisma | If Prisma adds edge support |
| Neon (serverless PG) | Branching, auto-scale, read replicas | Supabase | When need >500 concurrent connections |
| Better Auth | Self-hosted, no vendor lock-in | Clerk, Auth0 | When need enterprise SSO |
| Stripe Connect | Split payments, KYC, 40+ countries | Paddle, LemonSqueezy | When need crypto payments |
| Cloudflare R2 | Zero egress, S3-compatible | AWS S3 | Never (egress fees are dealbreaker) |
| Upstash Redis | Edge-accessible, global | Redis Cloud | When need >1GB cache |
| shadcn/ui | Copy-paste, customizable, accessible | MUI, Chakra | Never |
| Tailwind CSS | Utility-first, tree-shakeable | CSS Modules, Styled | Never |
| Resend | React Email, good deliverability | SendGrid, Postmark | When deliverability drops |
| Typesense | Instant search, typo-tolerance | Algolia | When need >1M documents |
| Fly.io | Global deploy, Docker-native | Railway, Render | When need >10 machines/region |
| Vercel | Next.js optimized, preview deploys | Netlify, AWS | Never |

---

## 8. COMMIT CONVENTION

```
feat: add product CRUD API
feat: add product list dashboard page
fix: license validation cache not invalidated on activation
fix: checkout session not created for subscription products
docs: update API documentation for licenses endpoint
refactor: extract discount calculation to shared util
test: add integration tests for checkout flow
chore: upgrade Next.js to 15.1
ci: add Fly.io deploy workflow

Format: type(scope): description

Types: feat, fix, docs, refactor, test, chore, ci
Scope: api, web, db, sdk, embed, email, infra
```

---

## 9. ENVIRONMENT SETUP — First Commands

```bash
# 1. Create project
mkdir teskel && cd teskel
pnpm init

# 2. Setup Turborepo
pnpm add -D turbo
npx create-turbo@latest

# 3. Setup apps/web
pnpm create next-app apps/web --typescript --tailwind --eslint --app --src-dir=false --import-alias="@/*"

# 4. Setup apps/api
mkdir -p apps/api/src
cd apps/api && pnpm init
pnpm add hono drizzle-orm @neondatabase/serverless stripe resend
pnpm add -D drizzle-kit typescript @types/node vitest

# 5. Setup packages/db
mkdir -p packages/db/schema
cd packages/db && pnpm init
pnpm add drizzle-orm @neondatabase/serverless
pnpm add -D drizzle-kit

# 6. Setup shadcn/ui
cd apps/web
pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button card dialog dropdown-menu form input select table tabs toast badge avatar skeleton separator sheet switch

# 7. Setup Better Auth
cd apps/web
pnpm add better-auth

# 8. Generate first migration
cd packages/db
pnpm drizzle-kit generate
pnpm drizzle-kit migrate

# 9. Run dev
cd ../..
pnpm dev
```

---

*This guideline is the CLAUDE.md-equivalent for TESKEL. No separate CLAUDE.md is required unless the implementation environment specifically needs one. Every AI agent building TESKEL must follow these rules before touching code.*
