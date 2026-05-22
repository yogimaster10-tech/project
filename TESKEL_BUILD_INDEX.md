# TESKEL — Build Documentation Index
## Read This First

Dokumen ini adalah index utama untuk membangun TESKEL dari nol sampai launch.

---

## Reading Order

AI agent / developer wajib membaca file berikut secara berurutan:

| Order | File | Purpose |
|------:|------|---------|
| 00 | `00_README_BUILD_ORDER.md` | Karpathy-inspired coding guideline, build behavior, anti-patterns, verification rules |
| 01 | `01_PRODUCT_OVERVIEW.md` | Product positioning, stack summary, monorepo overview |
| 02 | `02_DATABASE_SCHEMA.md` | Database schema, tables, enums, relations, indexes |
| 03 | `03_API_SPEC.md` | Backend API endpoints, auth, request/response, business flows |
| 04 | `04_FRONTEND_UIUX_SPEC.md` | Landing, dashboard, storefront, checkout, buyer portal, UI/UX components |
| 05 | `05_BUILD_DEPLOY_PLAN.md` | 12-week build plan, env vars, deploy, CI/CD, costs, testing |
| 06 | `06_INFRA_SECURITY_DEVOPS.md` | Cloud architecture, security, RLS, caching, scaling, observability, recovery |
| 07 | `07_ENGINEERING_DETAILS.md` | Backend service logic, frontend component behavior, edge cases, auth permissions |
| 08 | `08_AUTONOMOUS_BUILD_PLAYBOOK.md` | Autonomous build execution, phases, stop conditions, final acceptance test |
| 09 | `09_TOP_GLOBAL_FEATURES.md` | Design system, i18n, multi-currency, PPP, real-time, analytics, SEO, search, checkout UX, SDK, growth, PWA, accessibility, competitive positioning |
| 10 | `10_UIUX_MODERN_CLEAN.md` | Page-by-page visual spec, component specs (buttons/cards/tables/forms/modals/nav/toast), layout system, color rules, icon system, charts, responsive behavior, dark mode, empty/loading/error states, interaction patterns, reference UIs |
| 11 | `11_BUILD_READINESS_AUDIT.md` | Matured implementation decisions, MVP cut line, resolved gaps, Phase 0/1 acceptance criteria |

---

## Recommended Instruction for Future AI Agent

Use this prompt:

```text
Read TESKEL_BUILD_INDEX.md first.
Then read 00_README_BUILD_ORDER.md, 08_AUTONOMOUS_BUILD_PLAYBOOK.md, and 11_BUILD_READINESS_AUDIT.md.
Build TESKEL phase by phase.
For each task:
1. Implement the smallest working slice.
2. Add or update tests.
3. Run verification commands.
4. Fix failures caused by your changes.
5. Update BUILD_STATUS.md.
6. Commit progress if git is available.
7. Continue until the final acceptance test passes or a stop condition is reached.
```

---

## Document Roles

### Strategy / Product

- `01_PRODUCT_OVERVIEW.md`

### Build Rules / Agent Behavior

- `00_README_BUILD_ORDER.md`
- `08_AUTONOMOUS_BUILD_PLAYBOOK.md`

### Readiness / Build Control

- `11_BUILD_READINESS_AUDIT.md`
- `BUILD_STATUS.md`

### Core Engineering Specs

- `02_DATABASE_SCHEMA.md`
- `03_API_SPEC.md`
- `04_FRONTEND_UIUX_SPEC.md`
- `07_ENGINEERING_DETAILS.md`

### Global / Modern / UX

- `09_TOP_GLOBAL_FEATURES.md`
- `10_UIUX_MODERN_CLEAN.md`

### Deployment / Production

- `05_BUILD_DEPLOY_PLAN.md`
- `06_INFRA_SECURITY_DEVOPS.md`

---

## Final Startup Acceptance Summary

TESKEL dianggap selesai jika:

- Creator bisa signup, onboarding, create store, create product, upload file, publish product.
- Buyer bisa buka storefront, checkout, bayar, download product, dan menerima receipt email.
- Creator bisa melihat orders, customers, revenue, product analytics.
- License product bisa generate, validate, activate, revoke.
- Discount, affiliate, bundle, funnel, email, marketplace, embed, API key, webhook berjalan.
- Lint, typecheck, tests, e2e, build, migration, and smoke test pass.
- Production deploy live dengan monitoring, logging, backup, rate limit, security headers, and recovery plan.

---

## Important Principle

Jangan build semua hal sekaligus.

Build order paling aman:

```text
Auth → Org → Product Download → File Upload → Storefront → Checkout → Order → Delivery → Dashboard → Product Types → Growth Tools → Platform API → Production Hardening → Launch
```

---

*This index is the entry point for the TESKEL build documentation.*
