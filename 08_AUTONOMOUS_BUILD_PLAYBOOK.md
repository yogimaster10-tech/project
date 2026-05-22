# TESKEL — Autonomous Build Playbook
## From Phase 0 to Final Startup Launch Without Getting Lost

> This file is the operating manual for an AI coding agent building TESKEL autonomously from an empty repository to a production-ready startup.

---

## 0. PRIME DIRECTIVE

Build TESKEL incrementally, safely, and verifiably.

The agent must not "just code everything." The agent must operate in loops:

```
Understand → Plan → Implement smallest slice → Verify → Fix → Commit → Continue
```

The project is complete only when:

1. The app can be deployed to production.
2. A creator can sign up, create a digital product, connect payout, publish store, accept payment, and deliver the product.
3. A buyer can discover/buy/download/manage purchases.
4. Admin/creator can view orders, customers, analytics, licenses, payouts, settings.
5. Tests, lint, typecheck, migrations, security checks, and smoke tests pass.
6. Observability, recovery, CI/CD, and deployment docs exist.

---

## 1. FILE READING ORDER FOR THE AI AGENT

Before coding, read these files in order:

```
1.  TESKEL_BUILD_INDEX.md
2.  00_README_BUILD_ORDER.md
3.  08_AUTONOMOUS_BUILD_PLAYBOOK.md
4.  11_BUILD_READINESS_AUDIT.md
5.  01_PRODUCT_OVERVIEW.md
6.  02_DATABASE_SCHEMA.md
7.  03_API_SPEC.md
8.  04_FRONTEND_UIUX_SPEC.md
9.  05_BUILD_DEPLOY_PLAN.md
10. 06_INFRA_SECURITY_DEVOPS.md
11. 07_ENGINEERING_DETAILS.md
12. 09_TOP_GLOBAL_FEATURES.md
13. 10_UIUX_MODERN_CLEAN.md
```

If any file conflicts with another:

1. `00_README_BUILD_ORDER.md` wins for coding behavior.
2. `08_AUTONOMOUS_BUILD_PLAYBOOK.md` wins for execution order.
3. `11_BUILD_READINESS_AUDIT.md` wins for maturity clarifications and resolved gaps.
4. `02_DATABASE_SCHEMA.md` wins for schema.
5. `03_API_SPEC.md` wins for backend contract.
6. `04_FRONTEND_UIUX_SPEC.md` wins for UI/page structure.
7. `06_INFRA_SECURITY_DEVOPS.md` wins for production/security/deploy.
8. `09_TOP_GLOBAL_FEATURES.md` wins for design system, i18n, search, UX quality.

---

## 2. AUTONOMOUS AGENT BEHAVIOR RULES

### 2.1 Continue Automatically Unless Blocked

The agent should continue from one task to the next without asking the user after every small step.

Allowed autonomous actions:

- Create project files.
- Add source code.
- Add tests.
- Run lint/typecheck/test/build commands.
- Fix failing tests caused by the agent's changes.
- Create migrations.
- Update docs when directly related to implemented code.
- Commit progress if version control is available.

Must stop and ask only when:

- A paid external account/API key is required and not present.
- A destructive action is needed, such as deleting a database, wiping files, force-pushing, or removing user code.
- A product/business decision is ambiguous and affects user-facing behavior.
- A third-party integration requires manual dashboard setup that cannot be inferred.
- Tests fail due to external service credentials or unavailable infrastructure.
- Continuing would require storing secrets in code.

### 2.2 Karpathy-Style Autonomy

Autonomy does not mean guessing. It means moving through verifiable goals.

For every feature:

```
Assumptions:
- State only assumptions that affect implementation.

Goal:
- One concrete outcome.

Implementation:
- Smallest possible slice.

Verification:
- Exact command/test/check that proves success.

If fails:
- Fix only the failing slice.
```

### 2.3 Never Build Future Complexity Too Early

Even though TESKEL's final vision is large, the build must be staged.

Do not start with:

- AI engine.
- Agent commerce.
- Web3 DOT ownership.
- Creator banking.
- Fractional ownership.
- Real-time live commerce.
- Component marketplace.
- Protocol federation.

These are future phases. The MVP must first prove:

```
Creator → Product → Checkout → Payment → Delivery → Buyer access
```

---

## 3. AUTONOMOUS WORK LOOP

Every task follows this exact loop:

```
1. Pick the next unchecked task from the current phase.
2. Read only files relevant to that task.
3. State assumptions in code comments only if needed; otherwise keep docs clean.
4. Implement the smallest working slice.
5. Add or update tests.
6. Run verification commands.
7. If verification fails, inspect failure and fix.
8. Re-run verification.
9. If green, mark task complete in BUILD_STATUS.md.
10. Commit if git is available.
11. Move to next task.
```

### 3.1 BUILD_STATUS.md

The agent must maintain a file named:

```
BUILD_STATUS.md
```

Format:

```markdown
# TESKEL Build Status

Current Phase: Phase 1 — Foundation
Current Task: Product CRUD API
Last Verified: 2026-05-22 23:15 UTC+7

## Completed
- [x] Monorepo initialized
- [x] Next.js app created
- [x] Hono API created

## In Progress
- [ ] Product CRUD API

## Blocked
None

## Verification Log
| Time | Command | Result |
|------|---------|--------|
| 23:10 | pnpm typecheck | pass |
| 23:11 | pnpm test | pass |
```

The agent updates this after every meaningful milestone.

---

## 4. PHASE 0 — REPOSITORY BOOTSTRAP

### Goal

Create a clean monorepo foundation that all later phases can build on.

### Tasks

#### 0.1 Initialize Monorepo

Create:

```
package.json
pnpm-workspace.yaml
turbo.json
tsconfig.base.json
.gitignore
.env.example
README.md
BUILD_STATUS.md
```

Root `package.json` scripts:

```json
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint",
    "typecheck": "turbo typecheck",
    "test": "turbo test",
    "test:e2e": "turbo test:e2e",
    "db:generate": "pnpm --filter @teskel/db db:generate",
    "db:migrate": "pnpm --filter @teskel/db db:migrate",
    "db:studio": "pnpm --filter @teskel/db db:studio"
  }
}
```

Verify:

```bash
pnpm install
pnpm build
pnpm lint
pnpm typecheck
```

Acceptance:

- All commands run.
- Empty monorepo does not error.
- Workspace packages resolve.

#### 0.2 Create App/Package Skeleton

Create folders:

```
apps/web
apps/api
apps/embed
packages/db
packages/shared
packages/sdk
packages/email
packages/cli
infra
.github/workflows
```

Verify:

```bash
pnpm -r list
```

Acceptance:

- All workspaces appear.

---

## 5. PHASE 1 — FOUNDATION MVP

### Goal

Build the minimum viable TESKEL:

```
Auth → Org → Product → Stripe Checkout → Order → Delivery
```

### Phase 1 Completion Criteria

A real user can:

1. Sign up.
2. Complete onboarding.
3. Create a `download` product.
4. Upload a file.
5. Publish a storefront.
6. Buyer purchases via Stripe test mode.
7. Buyer receives download link.
8. Creator sees order in dashboard.

### 5.1 Database Foundation

Implement only these tables first:

```
users
user_oauth_accounts
user_sessions
organizations
organization_members
products
product_prices
product_files
customers
orders
order_items
access_grants
deliveries
api_keys
```

Do not implement all 30 tables immediately unless needed.

Verify:

```bash
pnpm db:generate
pnpm db:migrate
pnpm db:studio
```

Tests:

- Insert user/org/member.
- Insert product/price/file.
- Insert customer/order/order_item.
- Assert foreign keys work.

Acceptance:

- Migration applies cleanly.
- Schema matches the Phase 1 subset of `02_DATABASE_SCHEMA.md`.

### 5.2 Auth MVP

Implement:

- Email/password signup.
- Login/logout.
- Google OAuth if credentials exist.
- GitHub OAuth if credentials exist.
- Session cookie.
- Protected dashboard routes.

Do not block progress if OAuth keys are missing. Implement code path and mark OAuth manual verification pending.

Pages:

```
/login
/signup
/forgot-password
/reset-password
/verify-email
/onboarding
```

Verify:

```bash
pnpm typecheck
pnpm test auth
```

Manual smoke:

```
1. Visit /signup.
2. Create account.
3. Redirect to /onboarding.
4. Logout.
5. Login again.
6. Visit /dashboard successfully.
```

Acceptance:

- Unauthenticated users cannot access `/dashboard`.
- Logged-in users cannot access `/login` and `/signup`.
- Session persists after refresh.

### 5.3 Organization & Onboarding

Implement onboarding steps:

1. Profile.
2. Store setup.
3. First product optional.
4. Stripe connect optional.

MVP behavior:

- After signup, create user.
- On onboarding store step, create organization.
- Set user as owner.
- Set `users.defaultOrgId`.

Verify:

- New user has exactly one organization.
- User role is `owner`.
- Dashboard loads org context.

Acceptance:

- User can complete onboarding without Stripe.
- Dashboard shows store name.

### 5.4 Product CRUD MVP

Product type: `download` only.

Backend endpoints:

```
GET    /v1/orgs/:orgSlug/products
POST   /v1/orgs/:orgSlug/products
GET    /v1/orgs/:orgSlug/products/:id
PATCH  /v1/orgs/:orgSlug/products/:id
DELETE /v1/orgs/:orgSlug/products/:id
POST   /v1/orgs/:orgSlug/products/:id/prices
POST   /v1/orgs/:orgSlug/products/:id/files
```

Frontend pages:

```
/dashboard/products
/dashboard/products/new
/dashboard/products/[id]
```

MVP UI fields:

- Name.
- Description.
- Price.
- Currency.
- Product image.
- File upload.
- Status: draft/active.

Tests:

- Owner can create product.
- Viewer cannot create product.
- Org A cannot read Org B product.
- Invalid product type returns 422.
- Delete archives product, does not hard delete.

Acceptance:

- Creator can create, edit, publish, archive a download product.
- Product appears on storefront when active.

### 5.5 File Upload MVP

Storage: Cloudflare R2.

If R2 credentials are missing:

- Implement local filesystem storage adapter for dev.
- Keep R2 adapter ready for production.
- Use same interface.

Storage interface:

```typescript
interface StorageAdapter {
  putObject(input: PutObjectInput): Promise<StoredObject>;
  getSignedUrl(key: string, expiresInSeconds: number): Promise<string>;
  deleteObject(key: string): Promise<void>;
}
```

Verify:

- Upload file.
- File metadata saved in DB.
- Signed URL generated.
- Download works.

Acceptance:

- No file is stored in Postgres.
- Large files do not pass through API response body.

### 5.6 Storefront MVP

Pages:

```
/[storeSlug]
/[storeSlug]/products/[productSlug]
```

MVP sections:

- Store header.
- Product grid.
- Product detail.
- Buy button.

Verify:

- Draft/private products hidden.
- Active public products visible.
- Product detail has SEO metadata.

Acceptance:

- Storefront loads without auth.
- Product buy button starts checkout.

### 5.7 Stripe Checkout MVP

Implement only one-time payment first.

Flow:

```
Buy Button → POST /checkout/sessions → Stripe Checkout → Webhook → Order → Delivery → Success page
```

Backend:

```
POST /v1/checkout/sessions
POST /v1/webhooks/stripe
GET  /v1/delivery/:orderId/files
GET  /v1/delivery/:orderId/files/:fileId
```

Webhook events MVP:

```
checkout.session.completed
checkout.session.expired
```

Idempotency:

- Before creating order, check `orders.stripeSessionId`.
- If exists, do nothing.

Tests:

- Create checkout session for valid product.
- Reject checkout for draft product.
- Webhook creates order once.
- Duplicate webhook does not duplicate order.
- Delivery only works for paid order.

Acceptance:

- Buyer can purchase in Stripe test mode.
- Order appears in dashboard.
- Buyer can download purchased file.

### 5.8 Dashboard Orders MVP

Pages:

```
/dashboard/orders
/dashboard/orders/[id]
```

MVP data:

- Order ID.
- Customer email.
- Product name.
- Total.
- Status.
- Created date.
- Delivery status.

Acceptance:

- Creator can see paid orders.
- Creator can inspect order detail.

---

## 6. PHASE 2 — PRODUCT TYPES & CREATOR TOOLS

### Goal

Expand from one product type to core digital commerce types.

### 6.1 Add License Product

Implement:

- `license_keys`.
- `license_activations`.
- License generation on purchase.
- Public validation endpoint.
- Activation/deactivation endpoint.
- Dashboard license management.

Acceptance:

- Purchase license product → key generated.
- Validate valid key → `valid: true`.
- Validate revoked key → `valid: false`.
- Activation respects `maxActivations`.

### 6.2 Add Prompt Pack

Prompt pack is a specialized download.

Additional metadata:

- Prompt count.
- Format: markdown/json/text.
- Example prompts.

Acceptance:

- Prompt pack product can be created.
- Buyer can download prompts after purchase.

### 6.3 Add Community Product

Start with Discord only.

Implement:

- Discord invite link field.
- Optional role ID.
- Delivery email with invite link.

Do not build full Discord bot until manual invite flow works.

Acceptance:

- Buyer gets invite link after purchase.

### 6.4 Add Service Product

Implement:

- Booking duration.
- Calendar link.
- Timezone.
- Delivery email with booking instructions.

Acceptance:

- Buyer gets service booking instructions.

### 6.5 Add Subscription Pricing

Implement:

- Stripe subscription checkout.
- `subscriptions` table.
- Webhooks:
  - `customer.subscription.created`.
  - `customer.subscription.updated`.
  - `customer.subscription.deleted`.
  - `invoice.payment_succeeded`.
  - `invoice.payment_failed`.

Acceptance:

- Subscription purchase grants access.
- Cancel subscription revokes access at period end.
- Failed payment marks `past_due`.

### 6.6 Discounts

Implement:

- Discount CRUD.
- Validation.
- Checkout apply/remove.
- Usage count.

Acceptance:

- Percentage and fixed discounts work.
- Max usage enforced.
- First-time buyer only enforced.

---

## 7. PHASE 3 — GROWTH SYSTEMS

### Goal

Add growth mechanics that increase creator revenue.

### 7.1 Email System

Implement:

- Transactional email templates.
- Subscriber capture.
- Email sequences.
- Abandoned checkout email.

Acceptance:

- Receipt email sends.
- Abandoned checkout email sends after configured delay.
- Unsubscribe works.

### 7.2 Funnels

Start simple, then visual.

MVP funnel:

```
Product page → Checkout → Success
```

Then add:

- Order bump.
- Upsell.
- Downsell.
- Funnel analytics.

Acceptance:

- Funnel order attribution works.
- Order bump adds item to order.
- Upsell creates second purchase.

### 7.3 Affiliates

Implement:

- Affiliate CRUD.
- Referral links.
- Cookie tracking.
- Commission calculation.
- Affiliate dashboard.

Acceptance:

- Referral purchase credits affiliate.
- Commission calculated correctly.

### 7.4 Bundles

Implement:

- Bundle product type.
- Bundle item editor.
- Bundle delivery for all child products.
- Revenue split calculation.

Acceptance:

- Purchase bundle → all child products delivered.

### 7.5 Marketplace v1

Implement:

- Product opt-in.
- Explore page.
- Search.
- Category filters.
- Product reviews.
- Creator profile.

Acceptance:

- Active opted-in product appears in marketplace.
- Search finds products.
- Buyer can buy from marketplace.

---

## 8. PHASE 4 — PLATFORM & DEVELOPER INFRA

### Goal

Make TESKEL usable outside the dashboard.

### 8.1 API Keys

Implement:

- Create/revoke API keys.
- Scope enforcement.
- Last used tracking.

Acceptance:

- API key can list products if scope includes `products:read`.
- API key cannot write without `products:write`.

### 8.2 Embed Widget

Build in order:

1. Buy button embed.
2. Checkout modal embed.
3. Product card embed.
4. Full store embed.

Acceptance:

- External HTML page can include script and sell product.

### 8.3 TypeScript SDK

Implement:

```typescript
const teskel = new Teskel({ apiKey: 'tskl_live_xxx' });

await teskel.products.list();
await teskel.checkout.createSession({ productId, priceId });
await teskel.licenses.validate(key);
await teskel.access.check({ customerEmail, productId });
```

Acceptance:

- SDK works in Node.
- Types are exported.
- README includes examples.

### 8.4 CLI

Implement minimal CLI:

```
teskel login
teskel products list
teskel products create
teskel licenses validate <key>
```

Acceptance:

- CLI authenticates and calls API.

---

## 9. PHASE 5 — PRODUCTION HARDENING

### Goal

Make TESKEL safe, observable, resilient, and deployable.

### 9.1 Security Hardening

Implement:

- CSP headers.
- Strict CORS.
- Rate limiting.
- CSRF protection.
- Input validation on every endpoint.
- PII log scrubbing.
- API key hashing.
- Secure cookies.
- Webhook signature verification.
- Stripe webhook idempotency.

Acceptance:

- Public endpoint rate limits return 429.
- API keys are never stored plaintext.
- Stripe webhook rejects invalid signature.

### 9.2 RLS / Multi-Tenant Protection

Implement application-level org filtering first.

Then add Postgres RLS policies when stable.

Acceptance:

- Automated test proves Org A cannot read/write Org B data for every org-scoped table.

### 9.3 Observability

Implement:

- Structured logs.
- Sentry.
- PostHog.
- Health endpoint.
- Metrics endpoint.
- Uptime monitor.

Acceptance:

- Unhandled error appears in Sentry.
- Health check returns OK.
- Logs include traceId/orgId/userId where applicable.

### 9.4 Backup & Recovery

Implement:

- DB backup script.
- R2 backup/versioning.
- Restore procedure doc.
- Incident runbook.

Acceptance:

- Restore from backup tested in staging.

### 9.5 Load Testing

Use k6.

Scenarios:

- 100 concurrent checkout session creations.
- 1000 license validations/min.
- 500 marketplace searches/min.
- 100 file download URL generations/min.

Acceptance:

- API p95 < 500ms for normal endpoints.
- License validation p95 < 50ms cached.
- Error rate < 1%.

---

## 10. PHASE 6 — LANDING, DOCS, LAUNCH

### Goal

Make TESKEL ready for real users.

### 10.1 Marketing Site

Pages:

```
/
/pricing
/docs
/docs/getting-started
/docs/api-reference
/docs/embed
/docs/sdk
/blog
/changelog
/about
/contact
/legal/terms
/legal/privacy
/legal/dpa
```

Acceptance:

- Landing page Lighthouse > 90.
- All CTA buttons route correctly.
- Pricing table matches billing logic.

### 10.2 Documentation

Docs must include:

- Quickstart.
- Product creation.
- Checkout integration.
- License validation.
- Webhook verification.
- API authentication.
- SDK examples.
- Embed examples.
- Error codes.

Acceptance:

- A developer can validate a license using docs only.

### 10.3 Production Deploy

Deploy:

- Web: Vercel.
- API: Fly.io.
- Embed: Cloudflare Workers.
- Storage: Cloudflare R2.
- DB: Neon.
- Redis: Upstash.
- Email: Resend.
- Search: Typesense.

Acceptance:

- `teskel.com` loads.
- `app.teskel.com` loads dashboard.
- `api.teskel.com/v1/health` returns OK.
- Test checkout succeeds in production Stripe live mode with real low-value product.

### 10.4 Launch Checklist

```
□ Production env vars configured
□ DNS configured
□ SSL valid
□ Stripe live mode enabled
□ Stripe webhook endpoint live
□ Resend domain verified (SPF, DKIM, DMARC)
□ Sentry production release active
□ PostHog production active
□ Backup enabled
□ Uptime monitor active
□ Error alert active
□ Load test passed
□ 10 beta creators onboarded
□ Feedback form available
□ Support email works
□ Legal pages published
□ Pricing published
□ Public changelog exists
□ Launch announcement drafted
```

---

## 11. PHASE 7 — POST-LAUNCH ITERATION

### Goal

Improve based on real usage, not speculation.

### Build only after usage evidence:

| Feature | Trigger to Build |
|---------|------------------|
| AI pricing | 100+ active products with pricing data |
| AI support agent | 500+ support tickets or repetitive questions |
| Live commerce | Creators ask for launch events |
| Component marketplace | 10+ third-party developers request integrations |
| Web3 DOT | Users request resale/ownership proof |
| Creator finance | $100K+ monthly GMV |
| Agent API | AI agents/devs ask for commerce access |
| DCP protocol | External platforms ask to interoperate |

Post-launch loop:

```
Collect data → Identify bottleneck → Write hypothesis → Build smallest experiment → Measure → Keep or revert
```

---

## 12. AUTONOMOUS FAILURE RECOVERY

### If TypeScript Fails

```
1. Read first error only.
2. Fix root cause.
3. Re-run typecheck.
4. Repeat.
5. Do not mass-edit unrelated files.
```

### If Tests Fail

```
1. Identify if failure is from your change.
2. If yes, fix.
3. If no, document as pre-existing and continue only if unrelated.
4. Never delete failing tests to make suite green.
```

### If Build Fails

```
1. Check import paths.
2. Check missing env vars.
3. Check server/client component boundary.
4. Check package exports.
5. Fix smallest issue.
```

### If Migration Fails

```
1. Do not edit generated migration blindly.
2. Inspect schema change.
3. If dev database only, reset allowed after confirmation.
4. If production, write forward-only migration.
5. Never drop data unless explicitly approved.
```

### If External Service Missing

```
1. Implement adapter interface.
2. Add mock/local adapter.
3. Mark real integration as pending in BUILD_STATUS.md.
4. Continue building features against interface.
```

Example:

```typescript
interface EmailProvider {
  send(input: SendEmailInput): Promise<void>;
}

class ResendEmailProvider implements EmailProvider { ... }
class ConsoleEmailProvider implements EmailProvider { ... }
```

---

## 13. AUTONOMOUS COMMIT STRATEGY

Commit after every verifiable milestone.

Commit examples:

```
feat(db): add core auth and organization schema
feat(auth): implement signup login and session middleware
feat(products): add product CRUD API and dashboard pages
feat(checkout): add stripe checkout session and webhook order creation
feat(delivery): add signed file download flow
```

Commit message format:

```
<type>(<scope>): <short description>

Verification:
- pnpm typecheck ✅
- pnpm test products ✅
- manual smoke: create product → publish ✅
```

Never commit:

- Broken build.
- Failing tests caused by current change.
- Secrets.
- `.env` files.
- Generated local cache.

---

## 14. FINAL ACCEPTANCE TEST — Startup Launch Ready

The final build is accepted only when this full script passes:

```
1. Founder visits teskel.com.
2. Clicks Get Started.
3. Signs up.
4. Completes onboarding.
5. Creates store: teskel.com/demo-creator.
6. Creates product: "Notion Finance Template".
7. Uploads file.
8. Sets price: $9.
9. Publishes product.
10. Opens public storefront in incognito.
11. Opens product detail.
12. Clicks Buy.
13. Pays via Stripe test/live mode.
14. Redirects to success page.
15. Downloads file.
16. Receives receipt email.
17. Creator dashboard shows order.
18. Creator dashboard shows revenue metric updated.
19. Buyer portal shows purchase.
20. License validation works for license product.
21. Discount code works.
22. Marketplace search finds product.
23. Embed button works on external HTML page.
24. API key can list products.
25. Webhook endpoint receives `order.paid`.
26. Rate limit returns 429 when exceeded.
27. Sentry captures test error.
28. Backup restore tested in staging.
29. Production deploy passes smoke tests.
30. All docs pages load.
```

Required command suite:

```bash
pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e
pnpm build
pnpm db:migrate
pnpm smoke:prod
```

All must pass.

---

## 15. STOP CONDITIONS

The agent should stop only if:

1. A secret/API key is required and cannot be mocked.
2. A destructive operation is required, including database wipe, data deletion, force push, or irreversible migration.
3. A paid account or manual dashboard configuration is required and no local/mock adapter can preserve progress.
4. A product decision changes architecture materially.
5. The user asked to pause.

Otherwise, continue.

---

## 16. ONE-SENTENCE INSTRUCTION FOR FUTURE AI AGENT

> Read `TESKEL_BUILD_INDEX.md`, `00_README_BUILD_ORDER.md`, `08_AUTONOMOUS_BUILD_PLAYBOOK.md`, and `11_BUILD_READINESS_AUDIT.md`, then build TESKEL phase by phase. For each task, implement the smallest working slice, verify with tests/commands, update `BUILD_STATUS.md`, persist progress according to the environment, and continue until the final acceptance test passes or a stop condition is reached.

---

*This playbook turns TESKEL from a concept into an autonomous buildable startup project.*
