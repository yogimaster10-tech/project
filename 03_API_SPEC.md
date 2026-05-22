# TESKEL — API Specification
## Hono API Server — All Endpoints, Auth, Flows

---

## 1. API STRUCTURE

```
apps/api/src/
├── index.ts                    # App entry, global middleware, route mounting
├── routes/
│   ├── auth.ts                 # Auth endpoints
│   ├── products.ts             # Product CRUD
│   ├── prices.ts               # Price management
│   ├── files.ts                # File upload
│   ├── checkout.ts             # Checkout sessions
│   ├── orders.ts               # Order management
│   ├── customers.ts            # Customer management
│   ├── licenses.ts             # License CRUD + public validation
│   ├── access.ts               # Access check + grants
│   ├── delivery.ts             # File download + delivery
│   ├── subscriptions.ts        # Subscription management
│   ├── discounts.ts            # Discount code CRUD + public validate
│   ├── funnels.ts              # Funnel CRUD
│   ├── affiliates.ts           # Affiliate CRUD
│   ├── analytics.ts            # Analytics queries
│   ├── marketplace.ts          # Public marketplace browse
│   ├── emails.ts               # Email template + sequence
│   ├── webhooks.ts             # Webhook endpoint CRUD
│   ├── settings.ts             # Org settings
│   └── agent.ts                # Agent API (future)
├── services/                   # Business logic (called by routes)
├── middleware/
│   ├── auth.ts                 # JWT + API key authentication
│   ├── rate-limit.ts           # Per-tenant rate limiting
│   ├── cors.ts                 # CORS headers
│   ├── error-handler.ts        # Global error handler
│   ├── request-logger.ts       # Request logging
│   └── tenant-resolver.ts      # Resolve org context from slug/api-key
├── workers/                    # Background job processors
│   ├── stripe-webhook.worker.ts
│   ├── delivery.worker.ts
│   ├── email.worker.ts
│   ├── payout.worker.ts
│   ├── analytics.worker.ts
│   ├── license-cleanup.worker.ts
│   └── webhook-retry.worker.ts
└── config/
    ├── env.ts                  # Zod-validated env vars
    └── stripe.ts               # Stripe client init
```

---

## 2. AUTHENTICATION

### 2.1 Methods

| Method | Header | Use Case |
|--------|--------|----------|
| Session Cookie | (auto) | Dashboard web app |
| API Key | `X-TESKEL-API-KEY: tskl_live_xxx` | SDK, external apps |
| Bearer Token | `Authorization: Bearer xxx` | Mobile, embed |

### 2.2 Auth Middleware Flow

```typescript
// middleware/auth.ts
export const authMiddleware = async (c: Context, next: Next) => {
  // 1. Check session cookie (web dashboard)
  const sessionToken = getSessionCookie(c);
  if (sessionToken) {
    const session = await validateSession(sessionToken);
    if (session) {
      c.set('userId', session.userId);
      c.set('orgId', session.currentOrgId);
      c.set('authMethod', 'session');
      return next();
    }
  }

  // 2. Check API key (SDK/external)
  const apiKey = c.req.header('X-TESKEL-API-KEY');
  if (apiKey) {
    const keyHash = sha256(apiKey);
    const keyRecord = await db.query.apiKeys.findFirst({
      where: eq(apiKeys.keyHash, keyHash),
    });
    if (keyRecord && keyRecord.status === 'active') {
      // Check scopes
      const requiredScope = getRouteScope(c.req.path, c.req.method);
      if (!keyRecord.scopes.includes(requiredScope) && !keyRecord.scopes.includes('*')) {
        return c.json({ error: 'Insufficient scope' }, 403);
      }
      c.set('userId', null);
      c.set('orgId', keyRecord.orgId);
      c.set('authMethod', 'apikey');
      c.set('apiKeyScopes', keyRecord.scopes);
      // Update last used
      await db.update(apiKeys).set({ lastUsedAt: new Date() }).where(eq(apiKeys.id, keyRecord.id));
      return next();
    }
  }

  // 3. Check Bearer token (mobile/embed)
  const bearer = c.req.header('Authorization')?.replace('Bearer ', '');
  if (bearer) {
    const decoded = verifyJWT(bearer);
    if (decoded) {
      c.set('userId', decoded.userId);
      c.set('orgId', decoded.orgId);
      c.set('authMethod', 'bearer');
      return next();
    }
  }

  return c.json({ error: 'Unauthorized' }, 401);
};
```

### 2.3 Rate Limiting

```typescript
// middleware/rate-limit.ts
// Per-tenant (org_id) rate limiting via Upstash Redis

const rateLimits = {
  // Route prefix → { limit, window }
  '/v1/licenses/validate': { limit: 100, window: 60 },     // 100/min public
  '/v1/marketplace':       { limit: 200, window: 60 },     // 200/min public
  '/v1/checkout':          { limit: 30, window: 60 },       // 30/min per org
  '/v1/orgs':              { limit: 100, window: 60 },      // 100/min per org (default)
};

// API key tier overrides
const tierLimits = {
  free:       { limit: 100, window: 60 },
  pro:        { limit: 1000, window: 60 },
  scale:      { limit: 5000, window: 60 },
  enterprise: { limit: 10000, window: 60 },
};
```

---

## 3. ALL ENDPOINTS — Request & Response

### 3.1 Auth

```
POST /v1/auth/signup
  Body: { email, password, name }
  Response: { user, session }
  Validations: email valid + unique, password ≥ 8 chars

POST /v1/auth/login
  Body: { email, password }
  Response: { user, session }
  Validations: email exists, password correct, email verified

POST /v1/auth/logout
  Headers: Session cookie
  Response: { ok: true }

POST /v1/auth/forgot-password
  Body: { email }
  Response: { ok: true } (always return ok to prevent email enumeration)

POST /v1/auth/reset-password
  Body: { token, password }
  Response: { ok: true }

GET /v1/auth/verify-email/:token
  Response: { ok: true }

POST /v1/auth/oauth/:provider
  Body: { code, redirectUri }
  Response: { user, session, isNewUser }
  Providers: google, github

GET /v1/auth/me
  Headers: Auth required
  Response: { user, memberships: [{ org, role }] }

PATCH /v1/auth/me
  Headers: Auth required
  Body: { name?, avatarUrl? }
  Response: { user }
```

### 3.2 Products

```
GET /v1/orgs/:orgSlug/products
  Auth: Required (member+)
  Query: ?status=active&type=download&page=1&limit=20&search=xxx&sort=createdAt:desc
  Response: { data: Product[], meta: { page, limit, total } }

POST /v1/orgs/:orgSlug/products
  Auth: Required (member+)
  Body: {
    name, type, pricingModel, description?,
    category?, tags?: string[],
    featuredImageUrl?, coverImageUrl?,
    visibility?: 'public' | 'unlisted' | 'private',
    metadata?: ProductMetadata,
    prices: [{ name, priceCents, currency?, interval?, intervalCount?, trialPeriodDays?, isDefault?, metadata? }],
    files?: [{ fileId }],  // pre-uploaded file IDs
  }
  Response: { product }
  Side effects:
    - Auto-generate slug from name
    - Create Stripe product + prices
    - If type=license, set default maxActivations=1

GET /v1/orgs/:orgSlug/products/:id
  Auth: Required (member+)
  Response: { product, prices, files, accessRules, bundleItems? }

PATCH /v1/orgs/:orgSlug/products/:id
  Auth: Required (member+)
  Body: { any product field }
  Response: { product }
  Side effects:
    - If name changed, regenerate slug
    - Sync Stripe product if price changed
    - If status changed to 'active', set publishedAt

DELETE /v1/orgs/:orgSlug/products/:id
  Auth: Required (admin+)
  Response: { ok: true }
  Side effects:
    - Set status to 'archived' (soft delete)
    - Do NOT delete from Stripe (keep for records)

POST /v1/orgs/:orgSlug/products/:id/prices
  Auth: Required (member+)
  Body: { name, priceCents, currency?, interval?, isDefault? }
  Response: { price }
  Side effects: Create Stripe price

PATCH /v1/orgs/:orgSlug/products/:id/prices/:priceId
  Auth: Required (member+)
  Body: { name?, isDefault?, metadata?, sortOrder? }
  Response: { price }
  Note: Cannot change amount (must create new price, archive old)

DELETE /v1/orgs/:orgSlug/products/:id/prices/:priceId
  Auth: Required (admin+)
  Response: { ok: true }
  Side effects: Archive in Stripe (set active=false)

POST /v1/orgs/:orgSlug/products/:id/files
  Auth: Required (member+)
  Body: multipart/form-data { file, priceId?, version? }
  Response: { file }
  Side effects:
    - Upload to R2: {org_id}/{product_id}/{file_id}/v{version}_{filename}
    - Calculate SHA-256 checksum
    - Store file metadata in DB

DELETE /v1/orgs/:orgSlug/products/:id/files/:fileId
  Auth: Required (admin+)
  Response: { ok: true }
  Side effects: Delete from R2 + DB
```

### 3.3 Checkout

```
POST /v1/checkout/sessions
  Auth: NOT required (public endpoint, buyer-facing)
  Body: {
    productId, priceId,
    quantity?: 1,
    discountCode?: string,
    customerEmail?: string,
    successUrl, cancelUrl,
    metadata?: {},
    // Funnel context (set by frontend)
    funnelId?, funnelStepId?,
    // Affiliate tracking (from cookie)
    affiliateCode?,
    // UTM (from URL params)
    utmSource?, utmMedium?, utmCampaign?, utmContent?,
  }
  Response: { sessionId, url }  // url = Stripe Checkout URL
  Side effects:
    - Validate product exists + is active
    - Validate price belongs to product
    - Validate discount code if provided
    - Calculate tax (if Stripe Tax enabled)
    - Create Stripe Checkout Session
    - Store session in Redis (TTL 30 min)
    - Log analytics event: checkout_started

GET /v1/checkout/sessions/:id
  Auth: NOT required
  Response: { status, customerEmail, lineItems, totalCents }

POST /v1/checkout/sessions/:id/apply-discount
  Body: { code }
  Response: { session } (updated with discount)

POST /v1/checkout/sessions/:id/remove-discount
  Response: { session } (discount removed)
```

### 3.4 Orders

```
GET /v1/orgs/:orgSlug/orders
  Auth: Required (member+)
  Query: ?status=paid&page=1&limit=20&from=2025-01-01&to=2025-12-31&search=xxx
  Response: { data: OrderWithItems[], meta }

GET /v1/orgs/:orgSlug/orders/:id
  Auth: Required (member+)
  Response: { order, items, customer, deliveries }

POST /v1/orgs/:orgSlug/orders/:id/refund
  Auth: Required (admin+)
  Body: { reason, items?: [{ orderItemId, quantity, amountCents? }], fullRefund?: boolean }
  Response: { order, refund }
  Side effects:
    - Process refund via Stripe
    - Revoke access grants
    - Revoke/suspend license keys
    - Cancel delivery links
    - If subscription, cancel subscription
    - Log analytics event
    - Fire webhook: order.refunded

GET /v1/orgs/:orgSlug/orders/export
  Auth: Required (admin+)
  Query: ?format=csv&from=xxx&to=xxx&status=paid
  Response: CSV file download
```

### 3.5 Customers

```
GET /v1/orgs/:orgSlug/customers
  Auth: Required (member+)
  Query: ?page=1&limit=20&search=xxx&sort=lifetimeSpend:desc
  Response: { data: Customer[], meta }

GET /v1/orgs/:orgSlug/customers/:id
  Auth: Required (member+)
  Response: { customer, purchases, subscriptions, accessGrants }

GET /v1/orgs/:orgSlug/customers/:id/purchases
  Auth: Required (member+)
  Response: { orders: Order[] }

GET /v1/orgs/:orgSlug/customers/:id/subscriptions
  Auth: Required (member+)
  Response: { subscriptions: Subscription[] }
```

### 3.6 Licenses

```
POST /v1/orgs/:orgSlug/licenses
  Auth: Required (admin+)
  Body: { productId, count: 10, maxActivations?, features?, expiresAt? }
  Response: { keys: string[] }  // Show ONCE, never again

GET /v1/orgs/:orgSlug/licenses/:key
  Auth: Required (member+)
  Response: { license, activations }

PATCH /v1/orgs/:orgSlug/licenses/:key
  Auth: Required (admin+)
  Body: { status?, maxActivations?, features?, expiresAt? }
  Response: { license }

POST /v1/orgs/:orgSlug/licenses/bulk
  Auth: Required (admin+)
  Body: { productId, count, maxActivations?, prefix? }
  Response: { keys: string[] }

GET /v1/licenses/:key/validate
  Auth: NOT required (PUBLIC, rate-limited 100/min)
  Response: {
    valid: boolean,
    productId: string,
    productName: string,
    status: 'active' | 'revoked' | 'expired',
    features: {},
    expiresAt: string | null,
    activationCount: number,
    maxActivations: number | null,
  }
  Note: This is the most-called endpoint. Must be cached in Redis (TTL 60s).
  Cache key: license:validate:{key}
  Invalidate on: license update, activation, revocation

POST /v1/licenses/:key/activate
  Auth: NOT required (PUBLIC, rate-limited 10/min)
  Body: { fingerprint, label?, userAgent? }
  Response: { success: boolean, activationId, remainingActivations }
  Validations:
    - License must be active
    - activationCount < maxActivations (or maxActivations is null)
    - Fingerprint not already activated for this key
  Side effects:
    - Increment activationCount
    - Create activation record
    - Update Redis cache
    - Fire webhook: license.activated

DELETE /v1/licenses/:key/activations/:activationId
  Auth: Required (member+) OR fingerprint match
  Response: { ok: true }
  Side effects:
    - Decrement activationCount
    - Mark activation as deactivated
    - Update Redis cache
```

### 3.7 Access

```
POST /v1/access/check
  Auth: NOT required (PUBLIC, rate-limited 50/min)
  Body: { customerEmail?, customerId?, productId, licenseKey? }
  Response: {
    allowed: boolean,
    grantType: string | null,
    tier: string | null,
    features: string[],
    expiresAt: string | null,
    upgradeOptions: { priceId, name, amountCents }[],
  }
  Logic:
    1. Find active access grants for customer+product
    2. If no grants found, check license key
    3. If still no access, return allowed=false with upgrade options
    4. Cache result in Redis (TTL 300s)

GET /v1/orgs/:orgSlug/access-grants
  Auth: Required (member+)
  Query: ?productId=xxx&customerId=xxx&status=active
  Response: { data: AccessGrant[], meta }

POST /v1/orgs/:orgSlug/access-grants
  Auth: Required (admin+)
  Body: { customerId, productId, grantType: 'manual', tier?, features?, expiresAt? }
  Response: { accessGrant }

DELETE /v1/orgs/:orgSlug/access-grants/:id
  Auth: Required (admin+)
  Response: { ok: true }
  Side effects: Set status='revoked'
```

### 3.8 Delivery

```
GET /v1/delivery/:orderId/files
  Auth: Required (buyer = customer of this order)
  Response: { files: [{ id, fileName, fileSize, version, downloadUrl }] }
  Note: downloadUrl = signed R2 URL, expires in 24h

GET /v1/delivery/:orderId/files/:fileId
  Auth: Required (buyer)
  Response: Redirect to signed R2 URL
  Validations:
    - Order is paid
    - Access grant exists and is active
    - Download count < max (if access rule has download_limit)
  Side effects:
    - Increment access count
    - Log analytics: product_downloaded

GET /v1/delivery/:orderId/licenses
  Auth: Required (buyer)
  Response: { licenses: [{ key, productName, status, features, expiresAt }] }
```

### 3.9 Subscriptions

```
GET /v1/orgs/:orgSlug/subscriptions
  Auth: Required (member+)
  Query: ?status=active&page=1&limit=20
  Response: { data: Subscription[], meta }

GET /v1/orgs/:orgSlug/subscriptions/:id
  Auth: Required (member+)
  Response: { subscription, customer, price, product }

PATCH /v1/orgs/:orgSlug/subscriptions/:id
  Auth: Required (admin+)
  Body: { cancelAtPeriodEnd?: boolean }
  Response: { subscription }
  Side effects:
    - Update Stripe subscription
    - If canceling, schedule access grant revocation at period end
    - Fire webhook: subscription.canceled (if cancelAtPeriodEnd=true)
```

### 3.10 Discounts

```
GET /v1/orgs/:orgSlug/discounts
  Auth: Required (member+)
  Response: { data: DiscountCode[], meta }

POST /v1/orgs/:orgSlug/discounts
  Auth: Required (admin+)
  Body: { code, type, value, maxUses?, maxUsesPerCustomer?, validFrom?, validUntil?, applicableProductIds?, firstTimeBuyerOnly?, autoApply? }
  Response: { discountCode }

PATCH /v1/orgs/:orgSlug/discounts/:id
  Auth: Required (admin+)
  Body: { any field }
  Response: { discountCode }

DELETE /v1/orgs/:orgSlug/discounts/:id
  Auth: Required (admin+)
  Response: { ok: true }

POST /v1/discounts/validate
  Auth: NOT required (PUBLIC)
  Body: { code, orgSlug, productId?, customerEmail? }
  Response: {
    valid: boolean,
    type: 'percentage' | 'fixed_amount',
    value: number,
    discountAmountCents: number,  // calculated for this product
    message: string,  // "20% off!" or "$5 off!"
  }
  Validations:
    - Code exists and belongs to org
    - Within valid date range
    - Not exceeded max uses
    - Customer hasn't exceeded per-customer limit
    - Product is in applicable list (or list is null)
    - First-time buyer check if enabled
```

### 3.11 Funnels

```
GET /v1/orgs/:orgSlug/funnels
  Auth: Required (member+)
  Response: { data: Funnel[], meta }

POST /v1/orgs/:orgSlug/funnels
  Auth: Required (member+)
  Body: { name, productId, steps: FunnelStepInput[] }
  Response: { funnel }

GET /v1/orgs/:orgSlug/funnels/:id
  Auth: Required (member+)
  Response: { funnel, steps }

PATCH /v1/orgs/:orgSlug/funnels/:id
  Auth: Required (member+)
  Body: { name?, status?, steps?: FunnelStepInput[] }
  Response: { funnel }
  Note: Steps are replaced entirely on update (not merged)

DELETE /v1/orgs/:orgSlug/funnels/:id
  Auth: Required (admin+)
  Response: { ok: true }

GET /v1/orgs/:orgSlug/funnels/:id/analytics
  Auth: Required (member+)
  Response: { steps: [{ stepType, views, conversions, conversionRate, revenueCents }], totalRevenueCents }
```

### 3.12 Affiliates

```
GET /v1/orgs/:orgSlug/affiliates
  Auth: Required (admin+)
  Response: { data: Affiliate[], meta }

POST /v1/orgs/:orgSlug/affiliates
  Auth: Required (admin+)
  Body: { code, commissionType, commissionValue, cookieDurationDays?, applicableProductIds? }
  Response: { affiliate }

PATCH /v1/orgs/:orgSlug/affiliates/:id
  Auth: Required (admin+)
  Body: { commissionType?, commissionValue?, status? }
  Response: { affiliate }

DELETE /v1/orgs/:orgSlug/affiliates/:id
  Auth: Required (admin+)
  Response: { ok: true }
```

### 3.13 Analytics

```
GET /v1/orgs/:orgSlug/analytics/summary
  Auth: Required (member+)
  Query: ?period=30d
  Response: {
    revenue: { totalCents, changePercent },
    orders: { total, changePercent },
    customers: { total, new, changePercent },
    mrr: { totalCents, changePercent },
    averageOrderValueCents,
    topProducts: [{ productId, name, revenueCents, salesCount }],
  }

GET /v1/orgs/:orgSlug/analytics/revenue
  Auth: Required (member+)
  Query: ?period=30d&granularity=day
  Response: { data: [{ date, revenueCents, orderCount }] }

GET /v1/orgs/:orgSlug/analytics/orders
  Auth: Required (member+)
  Query: ?period=30d
  Response: { total, byStatus, byProduct, byCountry }

GET /v1/orgs/:orgSlug/analytics/products
  Auth: Required (member+)
  Query: ?period=30d&sort=revenue:desc&limit=10
  Response: { data: [{ productId, name, revenueCents, salesCount, conversionRate }] }

GET /v1/orgs/:orgSlug/analytics/customers
  Auth: Required (member+)
  Query: ?period=30d
  Response: { total, newVsReturning, topBuyers, averageLifetimeValueCents }

GET /v1/orgs/:orgSlug/analytics/funnels
  Auth: Required (member+)
  Query: ?period=30d
  Response: { data: [{ funnelId, name, stepConversions, totalRevenueCents }] }
```

### 3.14 Marketplace (PUBLIC)

```
GET /v1/marketplace/products
  Auth: NOT required
  Query: ?category=templates&type=download&minPrice=0&maxPrice=5000&rating=4&page=1&limit=20&sort=trending
  Response: { data: MarketplaceProduct[], meta }
  Sort options: trending, newest, price_asc, price_desc, rating, popular

GET /v1/marketplace/products/:slug
  Auth: NOT required
  Response: { product, prices, creator, reviews, relatedProducts }

GET /v1/marketplace/categories
  Auth: NOT required
  Response: { categories: [{ name, slug, productCount }] }

GET /v1/marketplace/creators/:slug
  Auth: NOT required
  Response: { org, products, stats }

GET /v1/marketplace/search
  Auth: NOT required
  Query: ?q=notion+template&type=download&category=templates
  Response: { data: MarketplaceProduct[], meta, facets }
  Implementation: Typesense search

GET /v1/marketplace/trending
  Auth: NOT required
  Query: ?limit=10&period=7d
  Response: { products: MarketplaceProduct[] }

GET /v1/marketplace/featured
  Auth: NOT required
  Response: { products: MarketplaceProduct[], creators: Org[] }
```

### 3.15 Emails

```
GET /v1/orgs/:orgSlug/emails/templates
  Auth: Required (admin+)
  Response: { templates: [{ type, subject, bodyHtml, enabled }] }

PATCH /v1/orgs/:orgSlug/emails/templates/:type
  Auth: Required (admin+)
  Body: { subject?, bodyHtml?, enabled? }
  Response: { template }

GET /v1/orgs/:orgSlug/emails/sequences
  Auth: Required (admin+)
  Response: { data: EmailSequence[], meta }

POST /v1/orgs/:orgSlug/emails/sequences
  Auth: Required (admin+)
  Body: { name, trigger, steps: [{ delayHours, subject, bodyHtml, trackOpens?, trackClicks? }] }
  Response: { sequence }

PATCH /v1/orgs/:orgSlug/emails/sequences/:id
  Auth: Required (admin+)
  Body: { name?, status?, steps? }
  Response: { sequence }

GET /v1/orgs/:orgSlug/emails/subscribers
  Auth: Required (admin+)
  Query: ?page=1&limit=50&source=popup
  Response: { data: EmailSubscriber[], meta }

DELETE /v1/orgs/:orgSlug/emails/subscribers/:id
  Auth: Required (admin+)
  Response: { ok: true }
```

### 3.16 Webhooks

```
GET /v1/orgs/:orgSlug/webhooks
  Auth: Required (admin+)
  Response: { data: WebhookEndpoint[] }

POST /v1/orgs/:orgSlug/webhooks
  Auth: Required (admin+)
  Body: { url, events: string[] }
  Response: { endpoint, secret }  // secret shown ONCE

PATCH /v1/orgs/:orgSlug/webhooks/:id
  Auth: Required (admin+)
  Body: { url?, events?, active? }
  Response: { endpoint }

DELETE /v1/orgs/:orgSlug/webhooks/:id
  Auth: Required (admin+)
  Response: { ok: true }

GET /v1/orgs/:orgSlug/webhooks/:id/deliveries
  Auth: Required (admin+)
  Query: ?status=failed&page=1&limit=20
  Response: { data: WebhookDelivery[], meta }

POST /v1/orgs/:orgSlug/webhooks/:id/deliveries/:deliveryId/retry
  Auth: Required (admin+)
  Response: { ok: true }
```

### 3.17 Settings

```
GET /v1/orgs/:orgSlug/settings
  Auth: Required (member+)
  Response: { org, plan, stripeStatus, customDomain, theme }

PATCH /v1/orgs/:orgSlug/settings
  Auth: Required (admin+)
  Body: { name?, description?, logoUrl?, website?, theme?, announcementBar?, payoutSchedule? }
  Response: { org }

POST /v1/orgs/:orgSlug/settings/domain
  Auth: Required (admin+)
  Body: { domain: "store.mysite.com" }
  Response: { dnsRecords: [{ type, name, value }] }
  Side effects:
    - Validate domain format
    - Create Cloudflare for SaaS custom hostname
    - Return DNS records creator must add

DELETE /v1/orgs/:orgSlug/settings/domain
  Auth: Required (admin+)
  Response: { ok: true }

GET /v1/orgs/:orgSlug/settings/stripe-status
  Auth: Required (admin+)
  Response: { status, chargesEnabled, payoutsEnabled, requirements }

POST /v1/orgs/:orgSlug/settings/stripe-onboarding
  Auth: Required (admin+)
  Response: { url }  // Stripe Connect onboarding URL

GET /v1/orgs/:orgSlug/api-keys
  Auth: Required (admin+)
  Response: { data: [{ id, name, keyPrefix, scopes, lastUsedAt, status }] }

POST /v1/orgs/:orgSlug/api-keys
  Auth: Required (admin+)
  Body: { name, scopes: string[] }
  Response: { id, name, key, scopes }  // key shown ONCE

DELETE /v1/orgs/:orgSlug/api-keys/:id
  Auth: Required (admin+)
  Response: { ok: true }

GET /v1/orgs/:orgSlug/members
  Auth: Required (admin+)
  Response: { members: [{ userId, name, email, role, acceptedAt }] }

POST /v1/orgs/:orgSlug/members/invite
  Auth: Required (admin+)
  Body: { email, role }
  Response: { ok: true }
  Side effects: Send invitation email

PATCH /v1/orgs/:orgSlug/members/:userId
  Auth: Required (owner)
  Body: { role }
  Response: { ok: true }

DELETE /v1/orgs/:orgSlug/members/:userId
  Auth: Required (owner)
  Response: { ok: true }
```

### 3.18 Buyer Portal

```
GET /v1/me/purchases
  Auth: Required (buyer session)
  Response: { data: OrderWithItems[] }

GET /v1/me/purchases/:id
  Auth: Required (buyer = owner of this order)
  Response: { order, items, downloads, licenses }

GET /v1/me/subscriptions
  Auth: Required (buyer session)
  Response: { data: Subscription[] }

PATCH /v1/me/subscriptions/:id
  Auth: Required (buyer = owner of this subscription)
  Body: { cancelAtPeriodEnd?: boolean }
  Response: { subscription }

GET /v1/me/licenses
  Auth: Required (buyer session)
  Response: { data: LicenseKey[] }

PATCH /v1/me/settings
  Auth: Required (buyer session)
  Body: { name? }
  Response: { ok: true }
```

### 3.19 Inbound Webhooks

```
POST /v1/webhooks/stripe
  Auth: Stripe signature verification
  Events handled:
    checkout.session.completed → create order, customer, access grants, deliveries, fire webhooks
    checkout.session.expired → mark session expired, log analytics
    customer.subscription.created → create subscription, access grant
    customer.subscription.updated → update subscription
    customer.subscription.deleted → revoke access, mark subscription expired
    invoice.payment_succeeded → log renewal, extend access grant
    invoice.payment_failed → mark past_due, send dunning email
    charge.dispute.created → mark order disputed, notify creator
    charge.refunded → update order, revoke access
    account.updated → update Stripe Connect status
    payout.paid → log payout
  Response: 200 OK (must respond quickly, process async)

POST /v1/webhooks/resend
  Auth: Resend signature verification
  Events: email.delivered, email.bounced, email.complained
  Side effects: Update subscriber status on bounce/complaint
```

---

## 4. CORE BUSINESS FLOWS

### 4.1 Purchase Flow (End-to-End)

```
1. Buyer visits product page on storefront/marketplace
2. Buyer clicks "Buy Now"
3. Frontend calls POST /v1/checkout/sessions
4. API creates Stripe Checkout Session, returns URL
5. Frontend redirects buyer to Stripe Checkout
6. Buyer completes payment on Stripe
7. Stripe fires checkout.session.completed webhook
8. API webhook handler:
   a. Verify Stripe signature
   b. Extract session data (customer, line items, payment)
   c. Find or create Customer record
   d. Create Order + OrderItems
   e. For each OrderItem:
      - Create AccessGrant (purchase type)
      - If type=license: Generate LicenseKey
      - Create Delivery record based on product type
      - If type=community: Send invite via bot
      - If type=service: Create booking confirmation
   f. Calculate affiliate commission (if applicable)
   g. Create PayoutSchedule (creator payout)
   h. Send receipt email
   i. Fire creator webhooks (order.paid)
   j. Log analytics events
   k. Update denormalized product stats (totalSales, totalRevenueCents)
9. Buyer redirected to success page
10. Success page shows downloads, license keys, delivery info
11. If funnel active, show upsell offer
```

### 4.2 License Validation Flow

```
1. Developer's app calls GET /v1/licenses/:key/validate
2. API checks Redis cache first (key: license:validate:{key})
3. If cache miss, query database
4. Check: status=active, not expired, not revoked
5. Return validation result
6. Cache result (TTL 60s)
7. Developer's app grants/denies feature access based on response
```

---

## 5. WEBHOOK EVENT TYPES

```
// Creator-subscribable events
order.created
order.paid
order.refunded
order.disputed

customer.created
customer.updated

license.generated
license.activated
license.deactivated
license.revoked
license.expired

access.granted
access.revoked

subscription.created
subscription.renewed
subscription.past_due
subscription.canceled
subscription.expired

delivery.completed
delivery.failed

payout.initiated
payout.completed
payout.failed

product.published
product.updated

review.created
```

---

## 6. ERROR RESPONSES

```typescript
// All errors follow this format:
interface ApiError {
  error: {
    code: string;       // 'NOT_FOUND', 'VALIDATION_ERROR', 'UNAUTHORIZED', etc.
    message: string;     // Human-readable message
    details?: any;       // Additional context (e.g., validation errors)
  };
}

// HTTP Status Codes:
400 — Validation error, bad request
401 — Unauthorized (no auth)
403 — Forbidden (insufficient role/scope)
404 — Not found
409 — Conflict (duplicate, already exists)
422 — Unprocessable entity (business rule violation)
429 — Rate limited
500 — Internal server error
```

---

*Total: 80+ endpoints with full request/response specs, auth requirements, validations, side effects, and business flows.*
