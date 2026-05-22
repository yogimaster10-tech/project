# TESKEL — Build Documentation Index
## Read This First

Dokumen ini adalah index utama untuk membangun TESKEL dari nol sampai launch.

---

## Execution Reading Order

AI agent / developer wajib membaca file berikut secara berurutan:

| Order | File | Purpose |
|------:|------|---------|
| 00 | `TESKEL_BUILD_INDEX.md` | Index utama dan urutan baca |
| 01 | `00_README_BUILD_ORDER.md` | Karpathy-inspired coding guideline, build behavior, anti-patterns, verification rules |
| 02 | `08_AUTONOMOUS_BUILD_PLAYBOOK.md` | Autonomous build execution, phases, stop conditions, final acceptance test |
| 03 | `11_BUILD_READINESS_AUDIT.md` | Matured implementation decisions, MVP cut line, resolved gaps, Phase 0/1 acceptance criteria |
| 04 | `12_ENGINEERING_EXCELLENCE.md` | Senior engineering practices: DoD, branching, code review, ADR, tech debt, AI-pair workflow |
| 05 | `13_TESTING_STRATEGY.md` | Test pyramid, integration/E2E/contract/load/chaos, flaky policy, coverage targets, test environments |
| 06 | `14_OBSERVABILITY_AND_INCIDENTS.md` | Golden signals, SLI/SLO, alerting, on-call, incident severity, postmortem, runbook |
| 07 | `15_SECURITY_AND_COMPLIANCE.md` | STRIDE, OWASP, auth/API security, secrets, supply chain, GDPR/CCPA, SOC 2 readiness |
| 08 | `16_RELEASE_AND_PERFORMANCE.md` | Release pipeline, feature flags, canary, rollback, performance budgets, capacity, cost engineering |
| 09 | `17_LAUNCH_AND_GROWTH.md` | Launch stages, north star, activation funnel, pricing, content, support, partnerships |
| 10 | `18_LANDING_AND_MARKETING_SITE.md` | Complete marketing site spec: landing, pricing, features, docs, blog, legal, 404/500, sitemap |
| 11 | `19_BRAND_COPY_AND_VOICE.md` | Brand essence, voice/tone, microcopy library, accessibility copy, anti-patterns |
| 12 | `20_EMAIL_TEMPLATES.md` | 47 email templates: triggers, payload, subject/preheader, deliverability, preference center |
| 13 | `21_ERROR_AND_STATE_CATALOG.md` | API error codes, form validation, empty/loading/error UI states, confirmations |
| 14 | `22_GLOSSARY_AND_DATA_DICTIONARY.md` | Terminology, status enums, ID/slug formats, event naming, plan entitlements |
| 15 | `23_FINAL_BUILD_CHECKLIST.md` | Master checklist: repo, db, api, frontend, email, infra, security, launch, ops |
| 16 | `01_PRODUCT_OVERVIEW.md` | Product positioning, stack summary, monorepo overview |
| 17 | `02_DATABASE_SCHEMA.md` | Database schema, tables, enums, relations, indexes |
| 18 | `03_API_SPEC.md` | Backend API endpoints, auth, request/response, business flows |
| 19 | `04_FRONTEND_UIUX_SPEC.md` | Landing, dashboard, storefront, checkout, buyer portal, UI/UX components |
| 20 | `05_BUILD_DEPLOY_PLAN.md` | 12-week build plan, env vars, deploy, CI/CD, costs, testing |
| 21 | `06_INFRA_SECURITY_DEVOPS.md` | Cloud architecture, security, RLS, caching, scaling, observability, recovery |
| 22 | `07_ENGINEERING_DETAILS.md` | Backend service logic, frontend component behavior, edge cases, auth permissions |
| 23 | `09_TOP_GLOBAL_FEATURES.md` | Design system, i18n, multi-currency, PPP, real-time, analytics, SEO, search, checkout UX, SDK, growth, PWA, accessibility, competitive positioning |
| 24 | `10_UIUX_MODERN_CLEAN.md` | Page-by-page visual spec, component specs (buttons/cards/tables/forms/modals/nav/toast), layout system, color rules, icon system, charts, responsive behavior, dark mode, empty/loading/error states, interaction patterns, reference UIs |

---

## Recommended Instruction for Future AI Agent

Use this prompt:

```text
Read TESKEL_BUILD_INDEX.md first.
Then read 00_README_BUILD_ORDER.md, 08_AUTONOMOUS_BUILD_PLAYBOOK.md, 11_BUILD_READINESS_AUDIT.md, the senior guideline pack 12_ENGINEERING_EXCELLENCE.md, 13_TESTING_STRATEGY.md, 14_OBSERVABILITY_AND_INCIDENTS.md, 15_SECURITY_AND_COMPLIANCE.md, 16_RELEASE_AND_PERFORMANCE.md, 17_LAUNCH_AND_GROWTH.md, and the surface/content/reference pack 18_LANDING_AND_MARKETING_SITE.md, 19_BRAND_COPY_AND_VOICE.md, 20_EMAIL_TEMPLATES.md, 21_ERROR_AND_STATE_CATALOG.md, 22_GLOSSARY_AND_DATA_DICTIONARY.md, 23_FINAL_BUILD_CHECKLIST.md.
Build TESKEL phase by phase.
For each task:
1. Implement the smallest working slice.
2. Add or update tests.
3. Run verification commands.
4. Fix failures caused by your changes.
5. Update BUILD_STATUS.md.
6. Tick the matching items in 23_FINAL_BUILD_CHECKLIST.md.
7. Commit progress if git is available.
8. Continue until the final acceptance test passes or a stop condition is reached.
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

### Senior Engineering Guideline Pack

- `12_ENGINEERING_EXCELLENCE.md`
- `13_TESTING_STRATEGY.md`
- `14_OBSERVABILITY_AND_INCIDENTS.md`
- `15_SECURITY_AND_COMPLIANCE.md`
- `16_RELEASE_AND_PERFORMANCE.md`
- `17_LAUNCH_AND_GROWTH.md`

### Surface, Content & Reference Pack

- `18_LANDING_AND_MARKETING_SITE.md`
- `19_BRAND_COPY_AND_VOICE.md`
- `20_EMAIL_TEMPLATES.md`
- `21_ERROR_AND_STATE_CATALOG.md`
- `22_GLOSSARY_AND_DATA_DICTIONARY.md`
- `23_FINAL_BUILD_CHECKLIST.md`

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
