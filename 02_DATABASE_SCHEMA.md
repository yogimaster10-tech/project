# TESKEL — Database Schema Specification
## Drizzle ORM + PostgreSQL (Neon)

> Semua tabel, kolom, tipe, relasi, index, constraint — lengkap.

---

## ENUMS

```typescript
// Semua pgEnum yang dipakai

export const productTypeEnum = pgEnum('product_type', [
  'download', 'license', 'prompt_pack', 'micro_saas',
  'service', 'community', 'course', 'bundle',
  'pwyw', 'tiered', 'link', 'collection', 'agent'
]);

export const pricingModelEnum = pgEnum('pricing_model', [
  'one_time', 'subscription', 'pwyw', 'tiered', 'free', 'usage_based'
]);

export const productStatusEnum = pgEnum('product_status', [
  'draft', 'active', 'archived', 'scheduled'
]);

export const productVisibilityEnum = pgEnum('product_visibility', [
  'public', 'unlisted', 'private', 'embed_only'
]);

export const orderStatusEnum = pgEnum('order_status', [
  'pending', 'paid', 'refunded', 'partially_refunded', 'disputed'
]);

export const licenseStatusEnum = pgEnum('license_status', [
  'active', 'revoked', 'expired', 'suspended'
]);

export const subscriptionStatusEnum = pgEnum('subscription_status', [
  'active', 'past_due', 'canceled', 'expired', 'trialing'
]);

export const accessGrantTypeEnum = pgEnum('access_grant_type', [
  'purchase', 'subscription', 'bundle', 'manual', 'affiliate', 'trial', 'gift'
]);

export const deliveryTypeEnum = pgEnum('delivery_type', [
  'file_download', 'license_key', 'account_access', 'invite_link',
  'booking_confirm', 'stream_access', 'webhook_trigger',
  'email_delivery', 'redirect'
]);

export const orgRoleEnum = pgEnum('org_role', [
  'owner', 'admin', 'member', 'viewer'
]);

export const discountTypeEnum = pgEnum('discount_type', [
  'percentage', 'fixed_amount', 'free_trial', 'buy_x_get_y'
]);

export const funnelStepTypeEnum = pgEnum('funnel_step_type', [
  'product_page', 'checkout', 'order_bump', 'upsell',
  'downsell', 'thank_you', 'wait', 'email', 'redirect', 'custom_html'
]);

export const affiliateCommissionTypeEnum = pgEnum('affiliate_commission_type', [
  'percentage', 'fixed_amount'
]);

export const emailSequenceTriggerEnum = pgEnum('email_sequence_trigger', [
  'purchase', 'email_signup', 'abandoned_checkout',
  'subscription_start', 'subscription_end'
]);

export const webhookDeliveryStatusEnum = pgEnum('webhook_delivery_status', [
  'pending', 'delivered', 'failed'
]);

export const marketplaceListingStatusEnum = pgEnum('marketplace_listing_status', [
  'active', 'paused', 'rejected', 'pending_review'
]);
```

---

## TABLES

### 1. users

```typescript
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }),
  avatarUrl: text('avatar_url'),
  emailVerified: boolean('email_verified').default(false),
  passwordHash: text('password_hash'),           // null untuk OAuth-only users
  mfaEnabled: boolean('mfa_enabled').default(false),
  mfaSecret: text('mfa_secret'),                  // TOTP secret
  defaultOrgId: uuid('default_org_id'),            // org yang aktif saat login
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
});

export const usersRelations = relations(users, ({ one, many }) => ({
  defaultOrg: one(organizations, {
    fields: [users.defaultOrgId],
    references: [organizations.id],
  }),
  oauthAccounts: many(userOAuthAccounts),
  sessions: many(userSessions),
  memberships: many(organizationMembers),
}));
```

**Notes:**
- `passwordHash` null berarti user hanya pakai OAuth (Google/GitHub)
- `defaultOrgId` menentukan org mana yang load pertama di dashboard
- Email harus unique — dipakai sebagai login identifier

---

### 2. user_oauth_accounts

```typescript
export const userOAuthAccounts = pgTable('user_oauth_accounts', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }).notNull(),
  provider: varchar('provider', { length: 50 }).notNull(),  // 'google', 'github'
  providerAccountId: varchar('provider_account_id', { length: 255 }).notNull(),
  accessToken: text('access_token'),
  refreshToken: text('refresh_token'),
  tokenExpiresAt: timestamp('token_expires_at', { withTimezone: true }),
  scope: text('scope'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  providerUniqueIdx: uniqueIndex('oauth_provider_unique_idx')
    .on(table.provider, table.providerAccountId),
  userIdx: index('oauth_user_idx').on(table.userId),
}));

export const userOAuthAccountsRelations = relations(userOAuthAccounts, ({ one }) => ({
  user: one(users, {
    fields: [userOAuthAccounts.userId],
    references: [users.id],
  }),
}));
```

---

### 3. user_sessions

```typescript
export const userSessions = pgTable('user_sessions', {
  id: uuid('id').defaultRandom().primaryKey(),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }).notNull(),
  token: text('token').notNull().unique(),
  expiresAt: timestamp('expires_at', { withTimezone: true }).notNull(),
  ipAddress: varchar('ip_address', { length: 45 }),
  userAgent: text('user_agent'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  tokenIdx: uniqueIndex('session_token_idx').on(table.token),
  userIdx: index('session_user_idx').on(table.userId),
  expiresIdx: index('session_expires_idx').on(table.expiresAt),
}));

export const userSessionsRelations = relations(userSessions, ({ one }) => ({
  user: one(users, {
    fields: [userSessions.userId],
    references: [users.id],
  }),
}));
```

**Notes:**
- Session dihapus otomatis saat expired (cron job cleanup)
- `token` = JWT refresh token, di-hash sebelum compare

---

### 4. organizations

```typescript
export const organizations = pgTable('organizations', {
  id: uuid('id').defaultRandom().primaryKey(),
  slug: varchar('slug', { length: 100 }).notNull().unique(),  // URL identifier
  name: varchar('name', { length: 255 }).notNull(),
  logoUrl: text('logo_url'),
  description: text('description'),
  website: text('website'),
  // Stripe Connect
  stripeAccountId: text('stripe_account_id'),
  stripeAccountStatus: varchar('stripe_account_status', { length: 50 })
    .default('pending'),  // pending, onboarding, active, restricted
  // Storefront
  customDomain: varchar('custom_domain', { length: 255 }),
  customDomainStatus: varchar('custom_domain_status', { length: 50 }).default('none'),
    // none, pending, active, failed
  theme: jsonb('theme').$type<OrgTheme>().default({
    primaryColor: '#6366f1',
    fontFamily: 'inter',
    borderRadius: 'lg',
    mode: 'system',  // light, dark, system
  }),
  announcementBar: jsonb('announcement_bar').$type<AnnouncementBar | null>().default(null),
  // Subscription plan
  plan: varchar('plan', { length: 50 }).default('free'),  // free, pro, scale, enterprise
  planStatus: varchar('plan_status', { length: 50 }).default('active'),
  stripeSubscriptionId: text('stripe_subscription_id'),
  // Payout
  payoutSchedule: varchar('payout_schedule', { length: 50 }).default('weekly'),
    // daily, weekly, monthly
  minimumPayoutCents: integer('minimum_payout_cents').default(1000),  // $10
  // Metadata
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  slugIdx: uniqueIndex('org_slug_idx').on(table.slug),
  customDomainIdx: index('org_custom_domain_idx').on(table.customDomain)
    .where(sql`${table.customDomain} is not null`),
}));

// Type definitions for JSONB columns
interface OrgTheme {
  primaryColor: string;
  fontFamily: string;
  borderRadius: 'none' | 'sm' | 'md' | 'lg' | 'full';
  mode: 'light' | 'dark' | 'system';
  coverImageUrl?: string;
  customCss?: string;
}

interface AnnouncementBar {
  text: string;
  link?: string;
  bgColor?: string;
  textColor?: string;
  active: boolean;
}

export const organizationsRelations = relations(organizations, ({ many }) => ({
  members: many(organizationMembers),
  products: many(products),
  orders: many(orders),
  customers: many(customers),
  affiliates: many(affiliates),
  discountCodes: many(discountCodes),
  funnels: many(funnels),
  webhookEndpoints: many(webhookEndpoints),
  apiKeys: many(apiKeys),
  emailSubscribers: many(emailSubscribers),
  emailSequences: many(emailSequences),
}));
```

---

### 5. organization_members

```typescript
export const organizationMembers = pgTable('organization_members', {
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }).notNull(),
  role: orgRoleEnum('role').notNull().default('member'),
  invitedByEmail: varchar('invited_by_email', { length: 255 }),
  invitedAt: timestamp('invited_at', { withTimezone: true }).defaultNow().notNull(),
  acceptedAt: timestamp('accepted_at', { withTimezone: true }),
}, (table) => ({
  pk: primaryKey({ columns: [table.orgId, table.userId] }),
  userIdx: index('org_member_user_idx').on(table.userId),
}));

export const organizationMembersRelations = relations(organizationMembers, ({ one }) => ({
  org: one(organizations, {
    fields: [organizationMembers.orgId],
    references: [organizations.id],
  }),
  user: one(users, {
    fields: [organizationMembers.userId],
    references: [users.id],
  }),
}));
```

**Role permissions:**
| Action | owner | admin | member | viewer |
|--------|-------|-------|--------|--------|
| Manage billing | ✅ | ❌ | ❌ | ❌ |
| Delete org | ✅ | ❌ | ❌ | ❌ |
| Manage team | ✅ | ✅ | ❌ | ❌ |
| Create/edit products | ✅ | ✅ | ✅ | ❌ |
| View orders | ✅ | ✅ | ✅ | ✅ |
| Issue refunds | ✅ | ✅ | ❌ | ❌ |
| Manage webhooks | ✅ | ✅ | ✅ | ❌ |
| View analytics | ✅ | ✅ | ✅ | ✅ |

---

### 6. products

```typescript
export const products = pgTable('products', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  slug: varchar('slug', { length: 150 }).notNull().unique(),
  name: varchar('name', { length: 255 }).notNull(),
  description: text('description'),
  type: productTypeEnum('type').notNull(),
  status: productStatusEnum('status').notNull().default('draft'),
  visibility: productVisibilityEnum('visibility').notNull().default('public'),
  pricingModel: pricingModelEnum('pricing_model').notNull(),
  // Media
  featuredImageUrl: text('featured_image_url'),
  coverImageUrl: text('cover_image_url'),
  previewImages: jsonb('preview_images').$type<string[]>().default([]),
  videoPreviewUrl: text('video_preview_url'),
  // SEO & Discovery
  category: varchar('category', { length: 100 }),
  tags: jsonb('tags').$type<string[]>().default([]),
  metaTitle: varchar('meta_title', { length: 255 }),
  metaDescription: text('meta_description'),
  // Type-specific config
  metadata: jsonb('metadata').$type<ProductMetadata>().default({}),
  // Sort & Featured
  sortOrder: integer('sort_order').default(0),
  isFeatured: boolean('is_featured').default(false),
  // Stats (denormalized for performance)
  totalSales: integer('total_sales').default(0),
  totalRevenueCents: integer('total_revenue_cents').default(0),
  averageRating: numeric('average_rating', { precision: 3, scale: 2 }).default('0'),
  reviewCount: integer('review_count').default(0),
  // Timestamps
  publishedAt: timestamp('published_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: index('product_org_idx').on(table.orgId),
  typeStatusIdx: index('product_type_status_idx').on(table.type, table.status),
  categoryIdx: index('product_category_idx').on(table.category),
  publishedIdx: index('product_published_idx').on(table.publishedAt),
}));

// Type-specific metadata
interface ProductMetadata {
  // download type
  downloadLimit?: number;          // max downloads per purchase
  downloadExpiryHours?: number;    // link expiry (default 24)
  // license type
  maxActivations?: number;         // max machine activations
  licenseFormat?: string;          // 'TSKL-XXXX-XXXX-XXXX'
  licenseFeatures?: Record<string, any>;
  // prompt_pack type
  promptCount?: number;
  promptFormat?: 'markdown' | 'json' | 'text';
  // micro_saas type
  provisionWebhookUrl?: string;
  accountAccessType?: 'api_key' | 'credentials' | 'oauth';
  // service type
  bookingDurationMinutes?: number;
  calendarLink?: string;
  timezone?: string;
  // community type
  platform?: 'discord' | 'telegram' | 'slack';
  inviteLink?: string;
  roleId?: string;                 // Discord role ID
  botToken?: string;               // Encrypted
  // course type
  contentUrls?: string[];
  dripSchedule?: { day: number; url: string }[];
  // bundle type — handled by product_bundle_items table
  // pwyw type
  minimumAmountCents?: number;     // default 0
  suggestedAmountsCents?: number[];  // [500, 1000, 2000]
  // tiered type — handled by product_prices with tier metadata
  // link type
  externalUrl?: string;
  // agent type
  agentEndpoint?: string;
  perRequestPriceCents?: number;
}

export const productsRelations = relations(products, ({ one, many }) => ({
  org: one(organizations, {
    fields: [products.orgId],
    references: [organizations.id],
  }),
  prices: many(productPrices),
  files: many(productFiles),
  bundleItems: many(productBundleItems, { relationName: 'bundle' }),
  accessRules: many(accessRules),
}));

// Slug generation: auto from name, lowercase, hyphens, unique per org
// Example: "My Template Kit" → "my-template-kit"
// If conflict: "my-template-kit-2"
```

---

### 7. product_prices

```typescript
export const productPrices = pgTable('product_prices', {
  id: uuid('id').defaultRandom().primaryKey(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  name: varchar('name', { length: 255 }).notNull(),   // "Standard", "Pro", "Enterprise"
  description: text('description'),
  priceCents: integer('price_cents').notNull(),         // 0 = free
  currency: varchar('currency', { length: 3 }).default('USD').notNull(),
  // Subscription fields (null = one-time)
  interval: varchar('interval', { length: 20 }),        // null, 'month', 'year'
  intervalCount: integer('interval_count').default(1),
  trialPeriodDays: integer('trial_period_days'),
  // Stripe sync
  stripePriceId: text('stripe_price_id'),
  // UI
  isDefault: boolean('is_default').default(false),
  sortOrder: integer('sort_order').default(0),
  // Tier metadata (for tiered pricing)
  metadata: jsonb('metadata').$type<PriceMetadata>().default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  productIdx: index('price_product_idx').on(table.productId),
}));

interface PriceMetadata {
  features?: string[];              // ["5 projects", "API access", "Priority support"]
  highlighted?: boolean;            // Show as recommended
  badge?: string;                   // "Most Popular", "Best Value"
  ctaText?: string;                 // "Get Started", "Start Free Trial"
}

export const productPricesRelations = relations(productPrices, ({ one }) => ({
  product: one(products, {
    fields: [productPrices.productId],
    references: [products.id],
  }),
}));
```

---

### 8. product_files

```typescript
export const productFiles = pgTable('product_files', {
  id: uuid('id').defaultRandom().primaryKey(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  priceId: uuid('price_id').references(() => productPrices.id, { onDelete: 'set null' }),
    // null = available for all prices
  fileName: varchar('file_name', { length: 255 }).notNull(),
  fileSize: bigint('file_size', { mode: 'number' }),     // bytes
  storageKey: text('storage_key').notNull(),               // R2 object key
  contentType: varchar('content_type', { length: 100 }),   // mime type
  version: varchar('version', { length: 50 }).default('1.0.0'),
  changelog: text('changelog'),
  checksumSha256: varchar('checksum_sha256', { length: 64 }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  productIdx: index('file_product_idx').on(table.productId),
}));

export const productFilesRelations = relations(productFiles, ({ one }) => ({
  product: one(products, {
    fields: [productFiles.productId],
    references: [products.id],
  }),
  price: one(productPrices, {
    fields: [productFiles.priceId],
    references: [productPrices.id],
  }),
}));
```

**R2 storage structure:**
```
teskel-files/
  └── {org_id}/
      └── {product_id}/
          └── {file_id}/
              └── v{version}_{filename}
```

---

### 9. product_bundle_items

```typescript
export const productBundleItems = pgTable('product_bundle_items', {
  bundleProductId: uuid('bundle_product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  itemProductId: uuid('item_product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  itemPriceId: uuid('item_price_id').references(() => productPrices.id, { onDelete: 'set null' }),
  ownerOrgId: uuid('owner_org_id').references(() => organizations.id),
  ownerSharePercent: numeric('owner_share_percent', { precision: 5, scale: 2 }),
    // Revenue split % for this item's creator. Null = same org (no split needed).
}, (table) => ({
  pk: primaryKey({ columns: [table.bundleProductId, table.itemProductId] }),
  bundleIdx: index('bundle_item_bundle_idx').on(table.bundleProductId),
}));

export const productBundleItemsRelations = relations(productBundleItems, ({ one }) => ({
  bundleProduct: one(products, {
    fields: [productBundleItems.bundleProductId],
    references: [products.id],
    relationName: 'bundle',
  }),
  itemProduct: one(products, {
    fields: [productBundleItems.itemProductId],
    references: [products.id],
    relationName: 'item',
  }),
  itemPrice: one(productPrices, {
    fields: [productBundleItems.itemPriceId],
    references: [productPrices.id],
  }),
  ownerOrg: one(organizations, {
    fields: [productBundleItems.ownerOrgId],
    references: [organizations.id],
  }),
}));
```

---

### 10. customers

```typescript
export const customers = pgTable('customers', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  name: varchar('name', { length: 255 }),
  stripeCustomerId: text('stripe_customer_id'),
  // Denormalized stats
  lifetimeSpendCents: integer('lifetime_spend_cents').default(0).notNull(),
  purchaseCount: integer('purchase_count').default(0).notNull(),
  activeSubscriptions: integer('active_subscriptions').default(0).notNull(),
  // Source tracking
  source: varchar('source', { length: 100 }),  // 'storefront', 'marketplace', 'embed'
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgEmailIdx: uniqueIndex('customer_org_email_idx').on(table.orgId, table.email),
  orgIdx: index('customer_org_idx').on(table.orgId),
  stripeIdx: index('customer_stripe_idx').on(table.stripeCustomerId),
}));

export const customersRelations = relations(customers, ({ one, many }) => ({
  org: one(organizations, {
    fields: [customers.orgId],
    references: [organizations.id],
  }),
  orders: many(orders),
  accessGrants: many(accessGrants),
  subscriptions: many(subscriptions),
}));
```

---

### 11. orders

```typescript
export const orders = pgTable('orders', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  customerId: uuid('customer_id').references(() => customers.id),
  customerEmail: varchar('customer_email', { length: 255 }).notNull(),
  customerName: varchar('customer_name', { length: 255 }),
  status: orderStatusEnum('status').notNull().default('pending'),
  // Pricing breakdown
  subtotalCents: integer('subtotal_cents').notNull(),
  discountCents: integer('discount_cents').default(0).notNull(),
  taxCents: integer('tax_cents').default(0).notNull(),
  processingFeeCents: integer('processing_fee_cents').default(0).notNull(),
  totalCents: integer('total_cents').notNull(),
  currency: varchar('currency', { length: 3 }).default('USD').notNull(),
  // Stripe
  stripeSessionId: text('stripe_session_id'),
  stripePaymentIntentId: text('stripe_payment_intent_id'),
  stripeChargeId: text('stripe_charge_id'),
  // Discount
  discountCodeId: uuid('discount_code_id'),
  // Funnel context
  funnelId: uuid('funnel_id'),
  funnelStepId: uuid('funnel_step_id'),
  orderBumpAccepted: boolean('order_bump_accepted').default(false),
  // Affiliate
  affiliateId: uuid('affiliate_id'),
  affiliateCommissionCents: integer('affiliate_commission_cents').default(0),
  // Tracking
  utmSource: varchar('utm_source', { length: 255 }),
  utmMedium: varchar('utm_medium', { length: 255 }),
  utmCampaign: varchar('utm_campaign', { length: 255 }),
  utmContent: varchar('utm_content', { length: 255 }),
  referrer: text('referrer'),
  ipAddress: varchar('ip_address', { length: 45 }),
  countryCode: varchar('country_code', { length: 2 }),  // buyer's country
  // Refund tracking
  refundedCents: integer('refunded_cents').default(0).notNull(),
  refundReason: text('refund_reason'),
  refundedAt: timestamp('refunded_at', { withTimezone: true }),
  // Metadata
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: index('order_org_idx').on(table.orgId),
  customerIdx: index('order_customer_idx').on(table.customerId),
  statusIdx: index('order_status_idx').on(table.status),
  createdIdx: index('order_created_idx').on(table.createdAt),
  stripeSessionIdx: index('order_stripe_session_idx').on(table.stripeSessionId),
}));

export const ordersRelations = relations(orders, ({ one, many }) => ({
  org: one(organizations, { fields: [orders.orgId], references: [organizations.id] }),
  customer: one(customers, { fields: [orders.customerId], references: [customers.id] }),
  items: many(orderItems),
  deliveries: many(deliveries),
  discountCode: one(discountCodes, { fields: [orders.discountCodeId], references: [discountCodes.id] }),
  affiliate: one(affiliates, { fields: [orders.affiliateId], references: [affiliates.id] }),
}));
```

---

### 12. order_items

```typescript
export const orderItems = pgTable('order_items', {
  id: uuid('id').defaultRandom().primaryKey(),
  orderId: uuid('order_id').references(() => orders.id, { onDelete: 'cascade' }).notNull(),
  productId: uuid('product_id').references(() => products.id).notNull(),
  priceId: uuid('price_id').references(() => productPrices.id).notNull(),
  productName: varchar('product_name', { length: 255 }).notNull(),   // snapshot
  priceName: varchar('price_name', { length: 255 }).notNull(),       // snapshot
  amountCents: integer('amount_cents').notNull(),
  quantity: integer('quantity').default(1).notNull(),
  // Populated after payment
  licenseKeyId: uuid('license_key_id').references(() => licenseKeys.id),
  // Snapshot of product type at time of purchase
  productType: productTypeEnum('product_type'),
  metadata: jsonb('metadata').default({}),
}, (table) => ({
  orderIdx: index('order_item_order_idx').on(table.orderId),
}));

export const orderItemsRelations = relations(orderItems, ({ one }) => ({
  order: one(orders, { fields: [orderItems.orderId], references: [orders.id] }),
  product: one(products, { fields: [orderItems.productId], references: [products.id] }),
  price: one(productPrices, { fields: [orderItems.priceId], references: [productPrices.id] }),
  licenseKey: one(licenseKeys, { fields: [orderItems.licenseKeyId], references: [licenseKeys.id] }),
}));
```

---

### 13. license_keys

```typescript
export const licenseKeys = pgTable('license_keys', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  orderItemId: uuid('order_item_id').references(() => orderItems.id),
  key: varchar('key', { length: 30 }).notNull().unique(),  // TSKL-XXXX-XXXX-XXXX
  status: licenseStatusEnum('status').notNull().default('active'),
  activationCount: integer('activation_count').default(0).notNull(),
  maxActivations: integer('max_activations'),  // null = unlimited
  features: jsonb('features').$type<Record<string, any>>().default({}),
  expiresAt: timestamp('expires_at', { withTimezone: true }),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  keyIdx: uniqueIndex('license_key_idx').on(table.key),
  orgIdx: index('license_org_idx').on(table.orgId),
  productIdx: index('license_product_idx').on(table.productId),
  statusIdx: index('license_status_idx').on(table.status),
}));

export const licenseKeysRelations = relations(licenseKeys, ({ one, many }) => ({
  org: one(organizations, { fields: [licenseKeys.orgId], references: [organizations.id] }),
  product: one(products, { fields: [licenseKeys.productId], references: [products.id] }),
  orderItem: one(orderItems, { fields: [licenseKeys.orderItemId], references: [orderItems.id] }),
  activations: many(licenseActivations),
}));
```

**Key format:** `TSKL-{4 alphanumeric}-{4 alphanumeric}-{4 alphanumeric}`
Generated with crypto.randomBytes, no ambiguous chars (0/O, 1/I/L removed).

---

### 14. license_activations

```typescript
export const licenseActivations = pgTable('license_activations', {
  id: uuid('id').defaultRandom().primaryKey(),
  licenseKeyId: uuid('license_key_id').references(() => licenseKeys.id, { onDelete: 'cascade' }).notNull(),
  fingerprint: varchar('fingerprint', { length: 255 }).notNull(),  // machine hash
  ipAddress: varchar('ip_address', { length: 45 }),
  userAgent: text('user_agent'),
  label: varchar('label', { length: 255 }),  // "MacBook Pro", "Windows Desktop"
  status: varchar('status', { length: 50 }).notNull().default('active'),  // active, deactivated
  activatedAt: timestamp('activated_at', { withTimezone: true }).defaultNow().notNull(),
  lastHeartbeat: timestamp('last_heartbeat', { withTimezone: true }),
}, (table) => ({
  licenseIdx: index('activation_license_idx').on(table.licenseKeyId),
  fingerprintIdx: index('activation_fingerprint_idx').on(table.licenseKeyId, table.fingerprint),
}));
```

---

### 15. access_rules

```typescript
export const accessRules = pgTable('access_rules', {
  id: uuid('id').defaultRandom().primaryKey(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  ruleType: varchar('rule_type', { length: 50 }).notNull(),
    // 'require_purchase', 'require_subscription', 'require_tier',
    // 'require_license', 'require_email', 'time_limited',
    // 'download_limit', 'ip_whitelist'
  condition: jsonb('condition').$type<Record<string, any>>().notNull(),
    // rule-specific params, e.g.:
    // { "max_downloads": 5 }
    // { "tier": "pro" }
    // { "hours": 72 }
  priority: integer('priority').default(0),  // evaluation order
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  productIdx: index('access_rule_product_idx').on(table.productId),
}));
```

---

### 16. access_grants

```typescript
export const accessGrants = pgTable('access_grants', {
  id: uuid('id').defaultRandom().primaryKey(),
  customerId: uuid('customer_id').references(() => customers.id, { onDelete: 'cascade' }).notNull(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  grantType: accessGrantTypeEnum('grant_type').notNull(),
  sourceId: uuid('source_id'),  // order_id / subscription_id / bundle_product_id
  scope: varchar('scope', { length: 50 }).default('full'),  // full, partial
  tier: varchar('tier', { length: 100 }),
  features: jsonb('features').$type<string[]>().default([]),
  status: varchar('status', { length: 50 }).notNull().default('active'),  // active, revoked, expired
  revokedReason: text('revoked_reason'),
  expiresAt: timestamp('expires_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  customerIdx: index('grant_customer_idx').on(table.customerId),
  productIdx: index('grant_product_idx').on(table.productId),
  statusIdx: index('grant_status_idx').on(table.status),
  expiresIdx: index('grant_expires_idx').on(table.expiresAt),
}));

export const accessGrantsRelations = relations(accessGrants, ({ one }) => ({
  customer: one(customers, { fields: [accessGrants.customerId], references: [customers.id] }),
  product: one(products, { fields: [accessGrants.productId], references: [products.id] }),
}));
```

---

### 17. subscriptions

```typescript
export const subscriptions = pgTable('subscriptions', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  customerId: uuid('customer_id').references(() => customers.id, { onDelete: 'cascade' }).notNull(),
  priceId: uuid('price_id').references(() => productPrices.id).notNull(),
  stripeSubscriptionId: text('stripe_subscription_id'),
  status: subscriptionStatusEnum('status').notNull().default('active'),
  currentPeriodStart: timestamp('current_period_start', { withTimezone: true }),
  currentPeriodEnd: timestamp('current_period_end', { withTimezone: true }),
  cancelAtPeriodEnd: boolean('cancel_at_period_end').default(false),
  canceledAt: timestamp('canceled_at', { withTimezone: true }),
  endedAt: timestamp('ended_at', { withTimezone: true }),
  trialStart: timestamp('trial_start', { withTimezone: true }),
  trialEnd: timestamp('trial_end', { withTimezone: true }),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: index('sub_org_idx').on(table.orgId),
  customerIdx: index('sub_customer_idx').on(table.customerId),
  statusIdx: index('sub_status_idx').on(table.status),
  stripeIdx: index('sub_stripe_idx').on(table.stripeSubscriptionId),
}));

export const subscriptionsRelations = relations(subscriptions, ({ one }) => ({
  org: one(organizations, { fields: [subscriptions.orgId], references: [organizations.id] }),
  customer: one(customers, { fields: [subscriptions.customerId], references: [customers.id] }),
  price: one(productPrices, { fields: [subscriptions.priceId], references: [productPrices.id] }),
}));
```

---

### 18. deliveries

```typescript
export const deliveries = pgTable('deliveries', {
  id: uuid('id').defaultRandom().primaryKey(),
  orderId: uuid('order_id').references(() => orders.id, { onDelete: 'cascade' }).notNull(),
  customerId: uuid('customer_id').references(() => customers.id, { onDelete: 'cascade' }).notNull(),
  productId: uuid('product_id').references(() => products.id).notNull(),
  deliveryType: deliveryTypeEnum('delivery_type').notNull(),
  payload: jsonb('payload').$type<DeliveryPayload>().notNull(),
  status: varchar('status', { length: 50 }).notNull().default('pending'),
    // pending, delivered, expired, revoked
  deliveredAt: timestamp('delivered_at', { withTimezone: true }),
  expiresAt: timestamp('expires_at', { withTimezone: true }),
  firstAccessedAt: timestamp('first_accessed_at', { withTimezone: true }),
  lastAccessedAt: timestamp('last_accessed_at', { withTimezone: true }),
  accessCount: integer('access_count').default(0),
}, (table) => ({
  orderIdx: index('delivery_order_idx').on(table.orderId),
  customerIdx: index('delivery_customer_idx').on(table.customerId),
}));

interface DeliveryPayload {
  // file_download
  fileUrl?: string;
  fileId?: string;
  // license_key
  licenseKey?: string;
  licenseKeyId?: string;
  // invite_link
  inviteUrl?: string;
  platform?: string;
  // account_access
  apiKey?: string;
  endpoint?: string;
  // booking_confirm
  calendarEventUrl?: string;
  bookingTime?: string;
  // webhook_trigger
  webhookUrl?: string;
  webhookPayload?: any;
  // redirect
  redirectUrl?: string;
}
```

---

### 19. discount_codes

```typescript
export const discountCodes = pgTable('discount_codes', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  code: varchar('code', { length: 100 }).notNull(),
  type: discountTypeEnum('type').notNull(),
  value: integer('value').notNull(),
    // percentage: 0-100, fixed: cents, free_trial: days, buy_x_get_y: see metadata
  maxUses: integer('max_uses'),
  usedCount: integer('used_count').default(0).notNull(),
  maxUsesPerCustomer: integer('max_uses_per_customer'),
  validFrom: timestamp('valid_from', { withTimezone: true }),
  validUntil: timestamp('valid_until', { withTimezone: true }),
  applicableProductIds: jsonb('applicable_product_ids').$type<string[] | null>(),
    // null = all products
  firstTimeBuyerOnly: boolean('first_time_buyer_only').default(false),
  autoApply: boolean('auto_apply').default(false),
  autoApplyConditions: jsonb('auto_apply_conditions'),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgCodeIdx: uniqueIndex('discount_org_code_idx').on(table.orgId, table.code),
  orgIdx: index('discount_org_idx').on(table.orgId),
}));

export const discountCodesRelations = relations(discountCodes, ({ one }) => ({
  org: one(organizations, { fields: [discountCodes.orgId], references: [organizations.id] }),
}));
```

---

### 20. funnels

```typescript
export const funnels = pgTable('funnels', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  productId: uuid('product_id').references(() => products.id),  // main product
  status: varchar('status', { length: 50 }).notNull().default('draft'),
    // draft, active, paused, archived
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: index('funnel_org_idx').on(table.orgId),
}));

export const funnelsRelations = relations(funnels, ({ one, many }) => ({
  org: one(organizations, { fields: [funnels.orgId], references: [organizations.id] }),
  product: one(products, { fields: [funnels.productId], references: [products.id] }),
  steps: many(funnelSteps),
}));
```

---

### 21. funnel_steps

```typescript
export const funnelSteps = pgTable('funnel_steps', {
  id: uuid('id').defaultRandom().primaryKey(),
  funnelId: uuid('funnel_id').references(() => funnels.id, { onDelete: 'cascade' }).notNull(),
  stepType: funnelStepTypeEnum('step_type').notNull(),
  productId: uuid('product_id').references(() => products.id),
  priceId: uuid('price_id').references(() => productPrices.id),
  condition: jsonb('condition'),
    // { "if": "upsell_rejected", "show": "downsell" }
    // { "if": "purchased_tier", "equals": "basic", "show": "pro_upgrade" }
  config: jsonb('config').$type<FunnelStepConfig>().default({}),
  sortOrder: integer('sort_order').default(0).notNull(),
  // Stats
  views: integer('views').default(0).notNull(),
  conversions: integer('conversions').default(0).notNull(),
  revenueCents: integer('revenue_cents').default(0).notNull(),
}, (table) => ({
  funnelIdx: index('funnel_step_funnel_idx').on(table.funnelId),
}));

interface FunnelStepConfig {
  title?: string;
  description?: string;
  ctaText?: string;
  discountPercent?: number;
  customHtml?: string;
  delayHours?: number;        // for 'wait' step
  emailTemplateId?: string;   // for 'email' step
  redirectUrl?: string;       // for 'redirect' step
}
```

---

### 22. affiliates

```typescript
export const affiliates = pgTable('affiliates', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  userId: uuid('user_id').references(() => users.id),
  code: varchar('code', { length: 100 }).notNull().unique(),
  commissionType: affiliateCommissionTypeEnum('commission_type').notNull(),
  commissionValue: integer('commission_value').notNull(),
  cookieDurationDays: integer('cookie_duration_days').default(30).notNull(),
  applicableProductIds: jsonb('applicable_product_ids').$type<string[] | null>(),
  // Stats
  totalReferrals: integer('total_referrals').default(0).notNull(),
  totalConversions: integer('total_conversions').default(0).notNull(),
  totalRevenueCents: integer('total_revenue_cents').default(0).notNull(),
  totalCommissionCents: integer('total_commission_cents').default(0).notNull(),
  // Payout
  pendingCommissionCents: integer('pending_commission_cents').default(0).notNull(),
  paidCommissionCents: integer('paid_commission_cents').default(0).notNull(),
  status: varchar('status', { length: 50 }).notNull().default('active'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  codeIdx: uniqueIndex('affiliate_code_idx').on(table.code),
  orgIdx: index('affiliate_org_idx').on(table.orgId),
}));
```

---

### 23. email_subscribers

```typescript
export const emailSubscribers = pgTable('email_subscribers', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  name: varchar('name', { length: 255 }),
  source: varchar('source', { length: 100 }),
    // 'product_page', 'popup', 'checkout', 'manual'
  productId: uuid('product_id').references(() => products.id),
  status: varchar('status', { length: 50 }).notNull().default('active'),
    // active, unsubscribed, bounced
  unsubscribedAt: timestamp('unsubscribed_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgEmailIdx: uniqueIndex('subscriber_org_email_idx').on(table.orgId, table.email),
}));
```

---

### 24. email_sequences + email_sequence_steps

```typescript
export const emailSequences = pgTable('email_sequences', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  trigger: emailSequenceTriggerEnum('trigger').notNull(),
  status: varchar('status', { length: 50 }).notNull().default('draft'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
});

export const emailSequenceSteps = pgTable('email_sequence_steps', {
  id: uuid('id').defaultRandom().primaryKey(),
  sequenceId: uuid('sequence_id').references(() => emailSequences.id, { onDelete: 'cascade' }).notNull(),
  delayHours: integer('delay_hours').notNull(),  // delay from trigger/previous step
  subject: varchar('subject', { length: 255 }).notNull(),
  bodyHtml: text('body_html').notNull(),
  trackOpens: boolean('track_opens').default(true),
  trackClicks: boolean('track_clicks').default(true),
  sortOrder: integer('sort_order').default(0).notNull(),
}, (table) => ({
  sequenceIdx: index('email_step_sequence_idx').on(table.sequenceId),
}));
```

---

### 25. webhook_endpoints + webhook_deliveries

```typescript
export const webhookEndpoints = pgTable('webhook_endpoints', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  url: text('url').notNull(),
  secret: varchar('secret', { length: 255 }).notNull(),
  events: jsonb('events').$type<string[]>().notNull(),
    // ['order.paid', 'license.activated', ...]
  active: boolean('active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
});

export const webhookDeliveries = pgTable('webhook_deliveries', {
  id: uuid('id').defaultRandom().primaryKey(),
  endpointId: uuid('endpoint_id').references(() => webhookEndpoints.id, { onDelete: 'cascade' }).notNull(),
  eventType: varchar('event_type', { length: 100 }).notNull(),
  payload: jsonb('payload').notNull(),
  status: webhookDeliveryStatusEnum('status').notNull().default('pending'),
  responseStatusCode: integer('response_status_code'),
  responseBody: text('response_body'),
  attempts: integer('attempts').default(0).notNull(),
  maxAttempts: integer('max_attempts').default(5).notNull(),
  nextRetryAt: timestamp('next_retry_at', { withTimezone: true }),
  deliveredAt: timestamp('delivered_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  endpointIdx: index('webhook_del_endpoint_idx').on(table.endpointId),
  statusIdx: index('webhook_del_status_idx').on(table.status),
  retryIdx: index('webhook_del_retry_idx').on(table.nextRetryAt),
}));
```

---

### 26. marketplace_listings

```typescript
export const marketplaceListings = pgTable('marketplace_listings', {
  id: uuid('id').defaultRandom().primaryKey(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull().unique(),
  category: varchar('category', { length: 100 }).notNull(),
  tags: jsonb('tags').$type<string[]>().default([]),
  isFeatured: boolean('is_featured').default(false),
  listingStatus: marketplaceListingStatusEnum('listing_status').default('pending_review'),
  approvedAt: timestamp('approved_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  categoryIdx: index('listing_category_idx').on(table.category),
  statusIdx: index('listing_status_idx').on(table.listingStatus),
}));
```

---

### 27. product_reviews

```typescript
export const productReviews = pgTable('product_reviews', {
  id: uuid('id').defaultRandom().primaryKey(),
  productId: uuid('product_id').references(() => products.id, { onDelete: 'cascade' }).notNull(),
  customerId: uuid('customer_id').references(() => customers.id, { onDelete: 'cascade' }).notNull(),
  rating: integer('rating').notNull(),  // 1-5
  content: text('content'),
  status: varchar('status', { length: 50 }).notNull().default('published'),
    // published, hidden, flagged
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  productIdx: index('review_product_idx').on(table.productId),
  uniqueReview: uniqueIndex('review_unique_idx').on(table.productId, table.customerId),
}));
```

---

### 28. analytics_events

```typescript
export const analyticsEvents = pgTable('analytics_events', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  eventType: varchar('event_type', { length: 100 }).notNull(),
  entityType: varchar('entity_type', { length: 50 }),
  entityId: uuid('entity_id'),
  metadata: jsonb('metadata').default({}),
  sessionId: varchar('session_id', { length: 255 }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgTypeIdx: index('analytics_org_type_idx').on(table.orgId, table.eventType),
  createdIdx: index('analytics_created_idx').on(table.createdAt),
}));

// Event types:
// page_view, checkout_started, checkout_completed, checkout_abandoned,
// product_viewed, product_downloaded, license_activated,
// subscription_created, subscription_renewed, subscription_canceled,
// access_granted, access_revoked, review_created,
// email_subscribed, email_sequence_triggered
```

---

### 29. api_keys

```typescript
export const apiKeys = pgTable('api_keys', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  keyHash: varchar('key_hash', { length: 64 }).notNull(),  // SHA-256 of full key
  keyPrefix: varchar('key_prefix', { length: 12 }).notNull(),  // "tskl_live_" / "tskl_test_"
  scopes: jsonb('scopes').$type<string[]>().default([]),
    // ["products:read", "products:write", "orders:read", "orders:write",
    //  "customers:read", "licenses:read", "licenses:write",
    //  "analytics:read", "webhooks:read", "webhooks:write"]
  lastUsedAt: timestamp('last_used_at', { withTimezone: true }),
  expiresAt: timestamp('expires_at', { withTimezone: true }),
  status: varchar('status', { length: 50 }).notNull().default('active'),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  keyHashIdx: uniqueIndex('apikey_hash_idx').on(table.keyHash),
  orgIdx: index('apikey_org_idx').on(table.orgId),
  prefixIdx: index('apikey_prefix_idx').on(table.keyPrefix),
}));
```

**Key generation:**
- Full key: `tskl_live_{32 random chars}` or `tskl_test_{32 random chars}`
- Store SHA-256 hash of full key in DB
- Show full key ONCE on creation, then never again
- Authenticate by: hash incoming key → compare with stored hash

---

### 30. payout_schedules

```typescript
export const payoutSchedules = pgTable('payout_schedules', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  orderId: uuid('order_id').references(() => orders.id),
  // Amounts
  grossAmountCents: integer('gross_amount_cents').notNull(),
  platformFeeCents: integer('platform_fee_cents').notNull(),
  processingFeeCents: integer('processing_fee_cents').notNull(),
  netAmountCents: integer('net_amount_cents').notNull(),
  // Splits (for multi-creator bundles)
  splits: jsonb('splits').$type<PayoutSplit[]>().default([]),
  // Status
  status: varchar('status', { length: 50 }).notNull().default('pending'),
    // pending, processing, completed, failed
  stripeTransferId: text('stripe_transfer_id'),
  scheduledAt: timestamp('scheduled_at', { withTimezone: true }).notNull(),
  completedAt: timestamp('completed_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: index('payout_org_idx').on(table.orgId),
  statusIdx: index('payout_status_idx').on(table.status),
  scheduledIdx: index('payout_scheduled_idx').on(table.scheduledAt),
}));

interface PayoutSplit {
  recipientOrgId: string;
  recipientStripeAccountId: string;
  amountCents: number;
  description: string;
  type: 'creator' | 'affiliate' | 'platform';
}
```

---

### 31. notifications

```typescript
export const notifications = pgTable('notifications', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }).notNull(),
  type: varchar('type', { length: 100 }).notNull(),
    // 'order.paid', 'review.created', 'payout.completed',
    // 'chargeback.opened', 'subscription.canceled', 'license.activated'
  title: varchar('title', { length: 255 }).notNull(),
  body: text('body'),
  link: text('link'),                         // In-app link: /dashboard/orders/xxx
  read: boolean('read').default(false).notNull(),
  readAt: timestamp('read_at', { withTimezone: true }),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  userReadIdx: index('notif_user_read_idx').on(table.userId, table.read),
  orgIdx: index('notif_org_idx').on(table.orgId),
  createdIdx: index('notif_created_idx').on(table.createdAt),
}));
```

---

### 32. notification_preferences

```typescript
export const notificationPreferences = pgTable('notification_preferences', {
  userId: uuid('user_id').references(() => users.id, { onDelete: 'cascade' }).notNull(),
  eventType: varchar('event_type', { length: 100 }).notNull(),
  inApp: boolean('in_app').default(true).notNull(),
  email: boolean('email').default(true).notNull(),
  push: boolean('push').default(false).notNull(),
}, (table) => ({
  pk: primaryKey({ columns: [table.userId, table.eventType] }),
}));
```

---

### 33. ppp_configs

```typescript
export const pppConfigs = pgTable('ppp_configs', {
  id: uuid('id').defaultRandom().primaryKey(),
  orgId: uuid('org_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  enabled: boolean('enabled').default(false).notNull(),
  maxDiscountPercent: integer('max_discount_percent').default(70),
  // Country-specific overrides (null = use global PPP table)
  countryOverrides: jsonb('country_overrides').$type<Record<string, number>>().default({}),
    // e.g. { "IN": 60, "BR": 40, "ID": 55 }
  blockedCountries: jsonb('blocked_countries').$type<string[]>().default([]),
  applicableProductIds: jsonb('applicable_product_ids').$type<string[] | null>(),
    // null = all products
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  orgIdx: uniqueIndex('ppp_org_idx').on(table.orgId),
}));
```

---

### 34. search_sync_log

```typescript
export const searchSyncLog = pgTable('search_sync_log', {
  id: uuid('id').defaultRandom().primaryKey(),
  entityType: varchar('entity_type', { length: 50 }).notNull(), // 'product'
  entityId: uuid('entity_id').notNull(),
  action: varchar('action', { length: 20 }).notNull(), // 'index', 'update', 'delete'
  status: varchar('status', { length: 20 }).notNull().default('pending'),
    // pending, synced, failed
  error: text('error'),
  syncedAt: timestamp('synced_at', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
}, (table) => ({
  statusIdx: index('search_sync_status_idx').on(table.status),
}));
```

---

## MIGRATION COMMANDS

```bash
# Generate migration
pnpm drizzle-kit generate

# Run migration
pnpm drizzle-kit migrate

# Push schema directly (dev only)
pnpm drizzle-kit push

# Studio (visual DB browser)
pnpm drizzle-kit studio
```

---

*Total: 34 tables, 15 enums, complete Drizzle ORM definitions with relations, indexes, and TypeScript types.*
