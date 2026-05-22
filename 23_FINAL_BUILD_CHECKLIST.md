# TESKEL — Final Build Checklist
## The Single Source of "Have I Built Everything?"

> Setiap baris di sini adalah pekerjaan nyata. Jika seluruh item tercentang, TESKEL siap launch global. Mengikuti spec di file 00–22.

Gunakan format `- [ ]` saat memulai; `- [x]` saat selesai. Update `BUILD_STATUS.md` dengan reference checklist ini.

---

## 0. Repository Bootstrap (Phase 0)

- [ ] `package.json` root dengan scripts standar.
- [ ] `pnpm-workspace.yaml`.
- [ ] `turbo.json` dengan task graph (`build`, `lint`, `typecheck`, `test`, `dev`).
- [ ] `tsconfig.base.json` strict (lihat `12_ENGINEERING_EXCELLENCE.md` §2.1).
- [ ] `.gitignore`, `.editorconfig`, `.nvmrc`, `.npmrc`.
- [ ] `.env.example` lengkap (lihat `05_BUILD_DEPLOY_PLAN.md` §2).
- [ ] Husky + lint-staged + commitlint aktif.
- [ ] Renovate / Dependabot config.
- [ ] CI workflow di `.github/workflows/ci.yml`.
- [ ] `BUILD_STATUS.md` tersedia & up-to-date.
- [ ] License file `LICENSE` (default proprietary; SDK dapat MIT terpisah).
- [ ] Code of Conduct & Security policy (`SECURITY.md`).
- [ ] Pull request template (`.github/PULL_REQUEST_TEMPLATE.md`).
- [ ] Issue templates (bug, feature, docs).

Apps & Packages tersedia:

- [ ] `apps/web` (Next.js 15 App Router) booted (`pnpm dev` running).
- [ ] `apps/api` (Hono) booted, `/v1/status` returns ok.
- [ ] `apps/embed` (Cloudflare Workers / static) booted.
- [ ] `packages/db` skeleton + Drizzle config.
- [ ] `packages/shared` (types, Zod schemas, utils).
- [ ] `packages/email` skeleton (React Email).
- [ ] `packages/sdk` skeleton (publishable `@teskel/sdk`).
- [ ] `packages/cli` skeleton (`@teskel/cli`).
- [ ] `packages/config` (eslint, ts, tailwind preset).
- [ ] `infra/terraform` skeleton.

CI gates:

- [ ] `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build` hijau.
- [ ] `gitleaks` & `semgrep` step ada.
- [ ] Branch protection di GitHub.

---

## 1. Database (Phase 1 onward)

Wajib ada di Phase 1:

- [ ] `users`
- [ ] `user_oauth_accounts`
- [ ] `user_sessions`
- [ ] `organizations`
- [ ] `organization_members`
- [ ] `products`
- [ ] `product_prices`
- [ ] `product_files`
- [ ] `customers`
- [ ] `orders`
- [ ] `order_items`
- [ ] `deliveries`
- [ ] `access_rules`
- [ ] `access_grants`

Lanjutan Phase 2–4:

- [ ] `product_bundle_items`
- [ ] `license_keys`
- [ ] `license_activations`
- [ ] `subscriptions`
- [ ] `discount_codes`
- [ ] `funnels`
- [ ] `funnel_steps`
- [ ] `affiliates`
- [ ] `email_subscribers`
- [ ] `email_sequences`
- [ ] `email_sequence_steps`
- [ ] `webhook_endpoints`
- [ ] `webhook_deliveries`
- [ ] `marketplace_listings`
- [ ] `product_reviews`
- [ ] `analytics_events`
- [ ] `api_keys`
- [ ] `payout_schedules`
- [ ] `notifications`
- [ ] `notification_preferences`
- [ ] `ppp_configs`
- [ ] `search_sync_log`

Migration & data:

- [ ] Drizzle migrations ter-generate per fase.
- [ ] Seed scripts: `seed:minimal`, `seed:demo`, `seed:scale`.
- [ ] Backup config + restore drill di staging.
- [ ] RLS policies (Phase 5).

---

## 2. Backend — Hono API

### 2.1 Middleware

- [ ] `auth` (session/api-key/bearer)
- [ ] `tenant-resolver`
- [ ] `rate-limit`
- [ ] `cors`
- [ ] `request-logger`
- [ ] `error-handler`
- [ ] `idempotency`
- [ ] `webhook-signature-verify`

### 2.2 Routes (lihat `03_API_SPEC.md` §3)

Auth:

- [ ] `POST /v1/auth/signup`
- [ ] `POST /v1/auth/login`
- [ ] `POST /v1/auth/logout`
- [ ] `GET  /v1/auth/me`
- [ ] `POST /v1/auth/oauth/:provider/start`
- [ ] `POST /v1/auth/oauth/:provider/callback`
- [ ] `POST /v1/auth/magic-link`
- [ ] `POST /v1/auth/password-reset`
- [ ] `POST /v1/auth/mfa/setup`
- [ ] `POST /v1/auth/mfa/verify`

Org:

- [ ] `POST /v1/orgs`
- [ ] `GET  /v1/orgs/:slug`
- [ ] `PATCH /v1/orgs/:slug`
- [ ] `GET  /v1/orgs/:slug/members`
- [ ] `POST /v1/orgs/:slug/invites`
- [ ] `POST /v1/orgs/:slug/members/:id/role`
- [ ] `DELETE /v1/orgs/:slug/members/:id`

Products:

- [ ] `POST /v1/orgs/:slug/products`
- [ ] `GET  /v1/orgs/:slug/products`
- [ ] `GET  /v1/orgs/:slug/products/:id`
- [ ] `PATCH /v1/orgs/:slug/products/:id`
- [ ] `POST /v1/orgs/:slug/products/:id/publish`
- [ ] `POST /v1/orgs/:slug/products/:id/archive`
- [ ] `POST /v1/orgs/:slug/products/:id/duplicate`

Prices:

- [ ] `POST /v1/orgs/:slug/products/:id/prices`
- [ ] `PATCH /v1/orgs/:slug/products/:id/prices/:priceId`
- [ ] `DELETE /v1/orgs/:slug/products/:id/prices/:priceId`

Files:

- [ ] `POST /v1/orgs/:slug/files`
- [ ] `POST /v1/orgs/:slug/files/multipart/start` (Phase 3+)
- [ ] `POST /v1/orgs/:slug/files/multipart/part`
- [ ] `POST /v1/orgs/:slug/files/multipart/complete`

Checkout & orders:

- [ ] `POST /v1/checkout/sessions`
- [ ] `GET  /v1/orgs/:slug/orders`
- [ ] `GET  /v1/orgs/:slug/orders/:id`
- [ ] `POST /v1/orgs/:slug/orders/:id/refund`

Customers:

- [ ] `GET  /v1/orgs/:slug/customers`
- [ ] `GET  /v1/orgs/:slug/customers/:id`
- [ ] `POST /v1/orgs/:slug/customers/:id/notes`

Licenses:

- [ ] `POST /v1/orgs/:slug/licenses`
- [ ] `GET  /v1/orgs/:slug/licenses`
- [ ] `POST /v1/orgs/:slug/licenses/:id/revoke`
- [ ] `POST /v1/licenses/validate` (public)
- [ ] `POST /v1/licenses/activate`
- [ ] `POST /v1/licenses/deactivate`

Access & delivery:

- [ ] `GET  /v1/access/check`
- [ ] `GET  /v1/deliveries/:id/download`

Subscriptions:

- [ ] `GET  /v1/orgs/:slug/subscriptions`
- [ ] `POST /v1/orgs/:slug/subscriptions/:id/cancel`
- [ ] `POST /v1/orgs/:slug/subscriptions/:id/pause`
- [ ] `POST /v1/orgs/:slug/subscriptions/:id/resume`

Discounts:

- [ ] `POST /v1/orgs/:slug/discounts`
- [ ] `GET  /v1/orgs/:slug/discounts`
- [ ] `POST /v1/orgs/:slug/discounts/:id/disable`
- [ ] `POST /v1/discounts/validate` (public)

Funnels & bundles:

- [ ] `POST /v1/orgs/:slug/funnels`
- [ ] `GET  /v1/orgs/:slug/funnels`
- [ ] `POST /v1/orgs/:slug/bundles`

Affiliates:

- [ ] `POST /v1/orgs/:slug/affiliates`
- [ ] `GET  /v1/orgs/:slug/affiliates`
- [ ] `POST /v1/orgs/:slug/affiliates/:id/payout`

Analytics:

- [ ] `GET  /v1/orgs/:slug/analytics/summary`
- [ ] `GET  /v1/orgs/:slug/analytics/funnel`
- [ ] `GET  /v1/orgs/:slug/analytics/revenue`
- [ ] `GET  /v1/orgs/:slug/analytics/customers`

Marketplace (public):

- [ ] `GET  /v1/marketplace/products`
- [ ] `GET  /v1/marketplace/products/:slug`
- [ ] `GET  /v1/marketplace/categories`

Emails:

- [ ] `POST /v1/orgs/:slug/emails/sequences`
- [ ] `POST /v1/orgs/:slug/emails/broadcast`
- [ ] `GET  /v1/orgs/:slug/emails/subscribers`

Webhooks:

- [ ] `POST /v1/orgs/:slug/webhooks`
- [ ] `GET  /v1/orgs/:slug/webhooks`
- [ ] `POST /v1/orgs/:slug/webhooks/:id/test`

Settings & API keys:

- [ ] `GET  /v1/orgs/:slug/settings`
- [ ] `PATCH /v1/orgs/:slug/settings`
- [ ] `POST /v1/orgs/:slug/api-keys`
- [ ] `DELETE /v1/orgs/:slug/api-keys/:id`

Buyer portal:

- [ ] `GET  /v1/account/purchases`
- [ ] `GET  /v1/account/subscriptions`
- [ ] `POST /v1/account/export`
- [ ] `POST /v1/account/delete`

Inbound webhooks:

- [ ] `POST /v1/inbound/stripe`
- [ ] `POST /v1/inbound/resend`
- [ ] `POST /v1/inbound/typesense`

Realtime (Phase 4):

- [ ] `POST /v1/realtime/auth`

System:

- [ ] `GET  /v1/status`
- [ ] `GET  /v1/healthz`

### 2.3 Services

- [ ] `auth.service`
- [ ] `org.service`
- [ ] `product.service`
- [ ] `price.service`
- [ ] `file.service`
- [ ] `checkout.service`
- [ ] `order.service`
- [ ] `customer.service`
- [ ] `license.service`
- [ ] `access.service`
- [ ] `delivery.service`
- [ ] `subscription.service`
- [ ] `discount.service`
- [ ] `funnel.service`
- [ ] `affiliate.service`
- [ ] `analytics.service`
- [ ] `marketplace.service`
- [ ] `email.service`
- [ ] `webhook.service`
- [ ] `payout.service`
- [ ] `apikey.service`
- [ ] `audit.service`

### 2.4 Workers

- [ ] `stripe-webhook.worker`
- [ ] `delivery.worker`
- [ ] `email.worker`
- [ ] `payout.worker`
- [ ] `analytics.worker`
- [ ] `license-cleanup.worker`
- [ ] `webhook-retry.worker`
- [ ] `search-sync.worker`
- [ ] `data-export.worker`
- [ ] `data-deletion.worker`

---

## 3. Frontend — Next.js (apps/web)

### 3.1 Marketing & Public

Halaman wajib (`18_LANDING_AND_MARKETING_SITE.md`):

- [ ] `/`
- [ ] `/pricing`
- [ ] `/features`
- [ ] `/features/[slug]` template ready
- [ ] `/customers`
- [ ] `/customers/[slug]`
- [ ] `/changelog`
- [ ] `/changelog/[slug]`
- [ ] `/docs`
- [ ] `/docs/[section]/[slug]`
- [ ] `/blog`
- [ ] `/blog/[slug]`
- [ ] `/about`
- [ ] `/careers`
- [ ] `/careers/[slug]`
- [ ] `/contact`
- [ ] `/security`
- [ ] `/legal/terms`
- [ ] `/legal/privacy`
- [ ] `/legal/dpa`
- [ ] `/legal/cookie-policy`
- [ ] `/legal/refund-policy`
- [ ] `/legal/subprocessors`
- [ ] `/legal/acceptable-use`
- [ ] `/404`
- [ ] `/500`
- [ ] `/api/sitemap.xml`
- [ ] `/api/robots.txt`

### 3.2 Auth & Onboarding

- [ ] `/signup`
- [ ] `/login`
- [ ] `/forgot-password`
- [ ] `/reset-password/[token]`
- [ ] `/verify-email/[token]`
- [ ] `/mfa/setup`
- [ ] `/mfa/verify`
- [ ] `/invite/[token]`
- [ ] `/onboarding` (multi-step wizard 4 steps)

### 3.3 Dashboard (creator)

- [ ] `/dashboard` overview
- [ ] `/dashboard/products`
- [ ] `/dashboard/products/new`
- [ ] `/dashboard/products/[id]`
- [ ] `/dashboard/products/[id]/files`
- [ ] `/dashboard/products/[id]/pricing`
- [ ] `/dashboard/products/[id]/delivery`
- [ ] `/dashboard/products/[id]/marketplace`
- [ ] `/dashboard/orders`
- [ ] `/dashboard/orders/[id]`
- [ ] `/dashboard/customers`
- [ ] `/dashboard/customers/[id]`
- [ ] `/dashboard/licenses`
- [ ] `/dashboard/subscriptions`
- [ ] `/dashboard/discounts`
- [ ] `/dashboard/funnels`
- [ ] `/dashboard/funnels/[id]/builder`
- [ ] `/dashboard/bundles`
- [ ] `/dashboard/affiliates`
- [ ] `/dashboard/emails`
- [ ] `/dashboard/emails/sequences`
- [ ] `/dashboard/emails/subscribers`
- [ ] `/dashboard/analytics`
- [ ] `/dashboard/analytics/revenue`
- [ ] `/dashboard/analytics/funnels`
- [ ] `/dashboard/analytics/customers`
- [ ] `/dashboard/payouts`
- [ ] `/dashboard/team`
- [ ] `/dashboard/api-keys`
- [ ] `/dashboard/webhooks`
- [ ] `/dashboard/integrations`
- [ ] `/dashboard/settings/account`
- [ ] `/dashboard/settings/profile`
- [ ] `/dashboard/settings/store`
- [ ] `/dashboard/settings/domain`
- [ ] `/dashboard/settings/billing`
- [ ] `/dashboard/settings/notifications`
- [ ] `/dashboard/settings/security`
- [ ] `/dashboard/settings/data`

### 3.4 Storefront (public)

- [ ] `/[storeSlug]` storefront home
- [ ] `/[storeSlug]/products/[productSlug]`
- [ ] `/[storeSlug]/collections/[collectionSlug]`
- [ ] `/[storeSlug]/about`
- [ ] `/[storeSlug]/community`
- [ ] `/[storeSlug]/policy`

### 3.5 Marketplace

- [ ] `/explore`
- [ ] `/explore/[category]`
- [ ] `/explore/products/[slug]`
- [ ] `/explore/creators/[slug]`

### 3.6 Checkout & Success

- [ ] `/checkout/[sessionId]`
- [ ] `/success/[orderId]`
- [ ] `/failed/[sessionId]`

### 3.7 Buyer Portal

- [ ] `/account`
- [ ] `/account/purchases`
- [ ] `/account/purchases/[orderId]`
- [ ] `/account/subscriptions`
- [ ] `/account/licenses`
- [ ] `/account/profile`
- [ ] `/account/notifications`
- [ ] `/account/data`

### 3.8 Embed

- [ ] `/embed/[productId]`
- [ ] `/embed/checkout/[sessionId]`

### 3.9 Components

- [ ] Dashboard sidebar nav
- [ ] Top bar with org switcher + search
- [ ] Command palette (Cmd+K)
- [ ] Notification dropdown
- [ ] File uploader
- [ ] Pricing editor
- [ ] Theme customizer
- [ ] Funnel builder (React Flow)
- [ ] Email editor
- [ ] Discount editor
- [ ] License manager UI
- [ ] Order detail panel
- [ ] Stripe Connect onboarding wizard
- [ ] Domain setup wizard
- [ ] Analytics chart library (Recharts)
- [ ] Table primitives (sort, filter, pagination, bulk action)

### 3.10 Storybook / UI Library

- [ ] Setup Storybook untuk `packages/shared/ui` (jika dipisah).
- [ ] Story untuk Button, Card, Table, Form, Modal, Toast, Nav, Sidebar.
- [ ] A11y test (axe) di Storybook.

---

## 4. Email Templates (`20_EMAIL_TEMPLATES.md`)

Phase 1 wajib:

- [ ] `auth.verify-email`
- [ ] `auth.password-reset`
- [ ] `creator.onboarding-welcome`
- [ ] `creator.stripe-action-required`
- [ ] `creator.first-sale-celebration`
- [ ] `creator.payout-scheduled`
- [ ] `creator.payout-completed`
- [ ] `creator.payout-failed`
- [ ] `creator.sale-notification`
- [ ] `creator.refund-issued`
- [ ] `buyer.receipt`
- [ ] `buyer.download-ready`
- [ ] `buyer.refund-confirmation`

Phase 2+:

- [ ] `org.invitation`, `org.invitation-accepted`, `org.role-changed`
- [ ] `auth.magic-link`, `auth.suspicious-login`, `auth.mfa-backup-codes`
- [ ] `creator.first-product-nudge`
- [ ] `creator.plan-upgrade`, `creator.plan-renewal-reminder`, `creator.plan-renewal-failed`
- [ ] `creator.api-key-created`
- [ ] `buyer.license-key`, `buyer.subscription-*`, `buyer.access-revoked`
- [ ] `buyer.community-invite`
- [ ] `creator.review-received`, `creator.affiliate-commission`
- [ ] `creator.weekly-digest`
- [ ] `buyer.cart-abandoned`, `buyer.newsletter-confirmation`, `buyer.product-update`
- [ ] `marketing.launch-announcement`, `marketing.newsletter`
- [ ] `system.data-export-ready`, `system.account-deletion-confirmation`
- [ ] `system.security-incident-notice`, `system.status-incident-update`
- [ ] `system.payment-method-expiring`, `system.bounce-warning`
- [ ] `system.tos-update`
- [ ] `creator.api-key-rotation-reminder`

Operasional:

- [ ] Resend domain `mail.teskel.com` & `updates.teskel.com` configured.
- [ ] SPF, DKIM, DMARC verifikasi hijau.
- [ ] List-Unsubscribe header aktif.
- [ ] Preference center implemented.

---

## 5. Infrastructure & DevOps

- [ ] Vercel project untuk `apps/web` (preview + production).
- [ ] Fly.io app untuk `apps/api` di 2 region.
- [ ] Cloudflare Workers untuk `apps/embed`.
- [ ] Cloudflare R2 bucket `teskel-files` + custom domain.
- [ ] Neon Postgres database (primary + read replica).
- [ ] Upstash Redis instance.
- [ ] Typesense cluster (Phase 3+).
- [ ] Sentry project (web + api).
- [ ] PostHog project.
- [ ] Statuspage (status.teskel.com).
- [ ] DNS records for all subdomains.
- [ ] CDN config: cache rules, security headers.
- [ ] WAF rules Cloudflare.
- [ ] Bot management (Phase 4).
- [ ] Backup automation Neon snapshots.
- [ ] Cron jobs (worker scheduler, cleanup, digest).
- [ ] CI/CD pipelines (lint/typecheck/test/build/deploy).
- [ ] Secrets manager configured.
- [ ] Synthetic monitoring (status checks).

---

## 6. Observability & Operations

- [ ] Structured logging deployed on all services.
- [ ] OpenTelemetry traces (Phase 3).
- [ ] Dashboards: `slo-status`, `service-health`, `business-health`, `perf-overview`, `payment-pipeline`, `delivery-pipeline`, `email-pipeline`, `marketplace-search`, `auth-security`, `cost-tracking`.
- [ ] Alerts configured per severity (lihat `14_OBSERVABILITY_AND_INCIDENTS.md` §7).
- [ ] Runbooks tersedia untuk min. 10 alert critical.
- [ ] PagerDuty rotation aktif (atau alternatif).
- [ ] Status page automation: incident creation, public communication.
- [ ] Restore drill quarterly schedule.
- [ ] Postmortem repository (`docs/postmortems`) ready.

---

## 7. Security & Compliance

- [ ] Better Auth + MFA optional/wajib (Owner/Admin).
- [ ] Argon2id password hashing.
- [ ] Rate limit per IP/org/API key + IP fallback.
- [ ] CSP, HSTS, security headers semua aktif.
- [ ] Webhook signature verify + idempotency.
- [ ] Secrets in Vault/SecretManager, not in repo.
- [ ] SAST (semgrep, eslint-security) di CI.
- [ ] Dependency scan (pnpm audit + Snyk Phase 4).
- [ ] DAST (ZAP) nightly staging.
- [ ] Penetration test internal (Phase 4) + eksternal (Phase 5).
- [ ] GDPR/CCPA flows implemented (export + deletion).
- [ ] DPA, ToS, Privacy, Subprocessor list publik.
- [ ] PCI SAQ-A attestation.
- [ ] SOC 2 readiness: Vanta/Drata onboard, evidence collection live.
- [ ] Bug bounty private aktif (Phase 4).
- [ ] Threat model di-review setiap quarter.

---

## 8. Localization & Global

- [ ] i18n architecture aktif (`next-intl`).
- [ ] Currency conversion service (Stripe / Open Exchange Rates).
- [ ] PPP table seeded (Phase 5+).
- [ ] Locale switcher di footer.
- [ ] RTL support diuji (Arabic).
- [ ] Languages: EN ready, ID + ES + PT-BR + DE + FR + JA + AR roadmap Phase 5+.
- [ ] Tax: Stripe Tax setup US/EU/UK; PT/JP/AU expansion roadmap.
- [ ] Local payment methods (Phase 3+): iDEAL, Bancontact, SEPA, OXXO, Boleto, PromptPay, GrabPay, GoPay, etc.

---

## 9. Performance & Quality

- [ ] Web Vitals memenuhi budget (`16_RELEASE_AND_PERFORMANCE.md` §10.1).
- [ ] API P95/P99 sesuai target.
- [ ] Bundle size budget tidak terlampaui.
- [ ] Lighthouse score ≥ 95 landing + dashboard.
- [ ] Axe a11y violation tidak ada `serious/critical`.
- [ ] Synthetic checks 5 menit interval.
- [ ] Load test 5 endpoint kritikal hijau.
- [ ] Chaos test scenario terjadwal (Phase 4).
- [ ] Mutation testing aktif untuk module bisnis utama.

---

## 10. Testing

- [ ] Unit test coverage targets terpenuhi per folder.
- [ ] Integration test untuk semua API endpoint + multi-tenant case.
- [ ] E2E Playwright untuk 12+ critical flows.
- [ ] Contract test untuk Stripe, Resend, Typesense.
- [ ] Load test (k6) baseline & spike scenario.
- [ ] Accessibility test Playwright + axe pass.
- [ ] Email template snapshot + Litmus render.
- [ ] Flaky quarantine bersih (≤5).
- [ ] Release acceptance script (`pnpm release:acceptance`) hijau.

---

## 11. Product Surfaces

- [ ] Onboarding 4 langkah selesai untuk creator.
- [ ] Onboarding 2 langkah untuk buyer (post-purchase).
- [ ] Storefront customizer (theme, color, logo).
- [ ] Marketplace listing flow (creator submit → review → approved).
- [ ] Embed widget CDN <30 KB gzipped, init <50 ms.
- [ ] SDK TypeScript published (`@teskel/sdk`).
- [ ] CLI published (`@teskel/cli`).
- [ ] Webhook tester endpoint + UI.
- [ ] API explorer (Phase 3+) di `/docs`.
- [ ] Embeddable status badge (`https://status.teskel.com/badge`).

---

## 12. Customer & Growth Ops

- [ ] Helpdesk (Intercom / Pylon / Plain) integrated.
- [ ] Knowledge base `/docs` + `/help` populated min. 30 artikel.
- [ ] Onboarding email sequence active.
- [ ] Lifecycle email sequence (D2, D7, D14, D30) active.
- [ ] Drip campaign A/B (Phase 5).
- [ ] Public roadmap publik (Featurebase/Canny).
- [ ] Pricing page A/B tested baseline.
- [ ] North star tracking dashboard (`MGV/AC`).
- [ ] Conversion funnel dashboard (Land → Sign Up → Org → Product → Publish → Sale).
- [ ] Activation cohort dashboards.
- [ ] NPS bulanan email.
- [ ] Referral / affiliate program live (Phase 3).

---

## 13. Launch Readiness (Phase 6)

`17_LAUNCH_AND_GROWTH.md` §3 satu-per-satu:

- [ ] T-7 freeze, comms internal, copy final.
- [ ] T-1 rehearsal, support rotation, vendor notif.
- [ ] Launch day deploy + multi-channel comms.
- [ ] T+24h mini-postmortem.
- [ ] T+7d metric review.

Public assets siap:

- [ ] Hero video (≤2 menit) + transkrip.
- [ ] Product Hunt page + thumbnail.
- [ ] Press kit page + zip.
- [ ] Founder LinkedIn / Twitter posts queued.
- [ ] Status page entry "GA launch".
- [ ] Statuspage subscribed for high-traffic.

---

## 14. Continuous Improvement (Post-GA)

- [ ] Weekly metric review meeting.
- [ ] Monthly cohort & retention report.
- [ ] Quarterly OKR set & review.
- [ ] Quarterly security review + threat model update.
- [ ] Quarterly cost engineering audit.
- [ ] Quarterly brand & content audit.
- [ ] Annual SOC 2 Type II audit (Phase 6).
- [ ] Annual external pen test.
- [ ] Bug bounty public (Phase 6).
- [ ] Roadmap update setiap quarter.

---

## 15. Documentation Hygiene (Ongoing)

- [ ] `BUILD_STATUS.md` selalu reflektif kondisi terbaru.
- [ ] ADR aktif untuk semua keputusan arsitektur.
- [ ] Postmortem repository menerima semua SEV1/SEV2.
- [ ] Runbook untuk setiap alert.
- [ ] Changelog publik mengikuti rilis.
- [ ] Onboarding doc engineer baru < 30 menit produktif.
- [ ] Docs (public) di-update saat fitur baru launch.

---

## 16. Final Startup Acceptance Test (cross-document)

Lulus jika seluruh skenario `08_AUTONOMOUS_BUILD_PLAYBOOK.md` §14 + `17_LAUNCH_AND_GROWTH.md` §3 hijau, observability tetap dalam SLO selama 14 hari pertama GA, dan tidak ada SEV1 open.

---

*Dokumen ini adalah `single source of truth` untuk "what's left to build". Diperbarui bersama `BUILD_STATUS.md` setiap kali sebuah item selesai. Bila konflik dengan dokumen lain, kembali ke dokumen sumber yang relevan dan update di sini.*
