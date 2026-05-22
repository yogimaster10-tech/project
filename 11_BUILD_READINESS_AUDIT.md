# TESKEL — Build Readiness Audit
## Plan Maturity Notes Before Implementation

This document hardens the existing plan so a future build agent can move from specification to production-quality implementation without guessing.

---

## 1. Current Repository State

The repository is specification-only. There is no application code yet.

Existing docs define:

- Product vision and positioning.
- Monorepo structure.
- Database schema with 34 tables.
- Hono API contract.
- Next.js UI/page map.
- Deployment, security, observability, and recovery plans.
- Autonomous build playbook.
- Modern UI/UX and global feature standards.

Implementation must begin with Phase 0 in `08_AUTONOMOUS_BUILD_PLAYBOOK.md`.

---

## 2. Authoritative Source of Truth

When documents overlap, use this priority order:

1. `00_README_BUILD_ORDER.md` — coding behavior, verification discipline, simplicity rules.
2. `08_AUTONOMOUS_BUILD_PLAYBOOK.md` — execution order.
3. `11_BUILD_READINESS_AUDIT.md` — maturity clarifications and gap closures.
4. `12_ENGINEERING_EXCELLENCE.md` — engineering process, PR, review, ADR, tech debt.
5. `13_TESTING_STRATEGY.md` — testing methodology, quality gates.
6. `14_OBSERVABILITY_AND_INCIDENTS.md` — observability, alerting, on-call, incident.
7. `15_SECURITY_AND_COMPLIANCE.md` — security, privacy, regulatory decisions.
8. `16_RELEASE_AND_PERFORMANCE.md` — release management, performance budgets.
9. `17_LAUNCH_AND_GROWTH.md` — go-to-market, pricing, support, brand.
10. `18_LANDING_AND_MARKETING_SITE.md` — marketing surface, IA, sections.
11. `19_BRAND_COPY_AND_VOICE.md` — brand identity, voice, microcopy.
12. `20_EMAIL_TEMPLATES.md` — email behavior and content.
13. `21_ERROR_AND_STATE_CATALOG.md` — error codes, validation messages, UI states.
14. `22_GLOSSARY_AND_DATA_DICTIONARY.md` — terminology, enums, IDs, events.
15. `23_FINAL_BUILD_CHECKLIST.md` — master TODO across the whole startup.
16. `02_DATABASE_SCHEMA.md` — schema names, relationships, indexes.
17. `03_API_SPEC.md` — external API contract.
18. `04_FRONTEND_UIUX_SPEC.md` and `10_UIUX_MODERN_CLEAN.md` — page structure and visual behavior.
19. `06_INFRA_SECURITY_DEVOPS.md` — production security, scaling, deployment, recovery.
20. `09_TOP_GLOBAL_FEATURES.md` — post-MVP global polish.

If two docs conflict, do not silently pick one. Record the decision in `BUILD_STATUS.md`.

---

## 3. MVP Cut Line

The MVP is not the whole vision. The first production-quality slice is:

```text
signup → onboarding → organization → downloadable product → file upload → public storefront → Stripe checkout → webhook → order → access grant → signed download → receipt email → creator order dashboard
```

Everything else is post-MVP unless it is required to keep this flow secure and deployable.

### Build Now

- Monorepo scaffold.
- Strict TypeScript.
- Database package and initial migrations.
- Better Auth session auth.
- Organization and member model.
- Download product CRUD.
- Product file upload.
- Public storefront and product detail.
- Checkout session creation.
- Stripe webhook processing.
- Order, customer, order item, delivery, and access grant creation.
- Signed download URL.
- Minimal receipt email.
- Dashboard order list and product list.
- Integration and e2e tests for the full purchase loop.

### Defer Until After MVP

- Marketplace.
- Funnels.
- Affiliates.
- SDK and CLI.
- Embed widget.
- Custom domains.
- WebSocket real-time.
- AI insights.
- PPP pricing.
- Full i18n rollout.
- Advanced tax automation.
- White-label mode.

---

## 4. Resolved Implementation Decisions

### 4.1 API Boundary

Hono owns the public `/v1` API. Next.js owns UI rendering and dashboard UX.

Dashboard mutations may use Server Actions, but those actions must call shared service functions or a typed internal API client. Do not duplicate business logic separately in Next.js route handlers.

Rule:

```text
Business logic lives in service modules.
Hono routes validate/authorize/call services.
Server Actions validate UI forms/authorize/call the same services or typed client.
```

### 4.2 Authentication and Tenant Context

Use Better Auth for dashboard sessions. API keys authenticate external API consumers.

Every org-scoped read/write must resolve:

```text
userId
orgId
role
authMethod
```

No org-scoped query may execute without `org_id`.

For subdomains and API domain boundaries, prefer cookie domain `.teskel.com` only after local MVP works. Local development can use one origin first.

### 4.3 Realtime

MVP uses polling for dashboards. WebSocket/Pusher-compatible realtime is Phase 4+.

When realtime is implemented, private channels require a signed auth endpoint:

```text
POST /v1/realtime/auth
Auth: session or API key
Body: { socketId, channelName }
Checks: channel orgId matches authenticated orgId
Returns: provider-specific signed auth payload
```

### 4.4 File Upload

MVP upload path:

```text
dashboard form → server validation → R2 object write → product_files row
```

Limits:

- MVP max file size: 100 MB.
- Allowed types: PDF, ZIP, PNG, JPG, MP4, TXT, CSV, JSON.
- Files are private by default.
- Downloads use short-lived signed URLs.

Post-MVP large file upload:

```text
POST /v1/orgs/:slug/files/multipart/start
POST /v1/orgs/:slug/files/multipart/part
POST /v1/orgs/:slug/files/multipart/complete
```

### 4.5 Stripe, Tax, and External Credentials

Phase 1 must work with Stripe test mode.

If Stripe credentials are missing:

1. Keep payment provider behind an interface.
2. Add a local fake payment adapter for tests and UI smoke flows.
3. Mark real Stripe setup pending in `BUILD_STATUS.md`.

Tax automation is deferred. MVP supports tax disabled or a simple explicit tax placeholder. Stripe Tax becomes production hardening after the checkout loop is stable.

### 4.6 Email

MVP requires only:

- Buyer receipt.
- Download/access email.
- Creator sale notification.

If Resend credentials are missing, use a console/local email adapter and assert emails in tests via captured messages.

### 4.7 Search

Typesense is not required for MVP. Use database search for dashboard/product basics. Add Typesense only for marketplace/search scale.

### 4.8 Rate Limiting

Use deterministic Redis keys:

```text
ratelimit:{scope}:{identifier}:{routeGroup}
```

Examples:

```text
ratelimit:ip:203.0.113.10:checkout
ratelimit:org:org_123:api
ratelimit:apikey:key_456:licenses_validate
```

Public checkout and license validation must have IP-based fallback limits when no org context is available.

### 4.9 Seed Data

Phase 0 must include a deterministic seed command:

```bash
pnpm db:seed
```

Seed must create:

- One creator user.
- One organization.
- One download product.
- One test customer.
- One paid test order.

No seed file may contain real secrets.

### 4.10 Migration Safety

Development may reset a local database only with explicit confirmation.

Production migrations are forward-only:

- No destructive column/table drops without a backup and approval.
- Prefer additive schema changes.
- Add backfills as separate scripts.
- Record migration verification in `BUILD_STATUS.md`.

### 4.11 Custom Domains and TLS

Custom domains are post-MVP.

When implemented:

1. Creator adds domain.
2. App returns required DNS records.
3. Background worker verifies DNS.
4. Cloudflare for SaaS issues certificate.
5. Storefront routing activates only after certificate is active.

Do not block MVP on wildcard/custom-domain infrastructure.

### 4.12 Marketplace Moderation

Marketplace is post-MVP and must not auto-list every product.

Required workflow:

```text
creator submits listing → automated checks → pending review → approved/rejected → public listing
```

Minimum checks:

- Product has description, price, preview image, refund policy.
- File is present for download products.
- No banned words from moderation list.
- Creator account is verified.

---

## 5. Build Quality Gates

A task is complete only when:

- Code is implemented in the smallest working slice.
- Tests cover the happy path and at least two important failure paths.
- Multi-tenant isolation is tested where org data is involved.
- `pnpm lint`, `pnpm typecheck`, relevant tests, and build checks pass or the exact blocker is documented.
- `BUILD_STATUS.md` is updated.

Do not mark a phase complete because files exist. Mark it complete only when the user-facing flow works.

---

## 6. Phase 0 Acceptance Criteria

Phase 0 is complete when:

- `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.base.json`, `.gitignore`, `.env.example`, and `BUILD_STATUS.md` exist.
- `apps/web`, `apps/api`, `apps/embed`, `packages/db`, `packages/shared`, `packages/email`, `packages/sdk`, and `packages/cli` exist.
- `pnpm install` succeeds.
- `pnpm lint`, `pnpm typecheck`, `pnpm test`, and `pnpm build` run successfully, even if tests are initially minimal.
- CI workflow runs the same command suite.
- No real secrets are committed.

---

## 7. Phase 1 Acceptance Criteria

Phase 1 is complete when this exact flow passes in automated or manually documented smoke testing:

```text
creator signup
creator onboarding creates org
creator creates download product
creator uploads file
creator publishes product
buyer opens storefront
buyer opens product page
buyer starts checkout
payment succeeds in test mode or fake adapter
webhook/fake event creates order and access grant
buyer sees success page
buyer downloads file through signed URL
receipt email is captured or sent
creator dashboard shows order and revenue
```

This is the first real startup checkpoint. Advanced features can wait; revenue flow cannot.
