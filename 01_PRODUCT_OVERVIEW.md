# TESKEL — Product Overview

> Baca `TESKEL_BUILD_INDEX.md` dulu. File ini menjelaskan positioning, stack, dan struktur target.

| File | Isi | Untuk Siapa |
|------|-----|-------------|
| `TESKEL_BUILD_INDEX.md` | Index utama dan urutan baca | Semua |
| `00_README_BUILD_ORDER.md` | Aturan coding, build order, verification discipline | Semua builder |
| `11_BUILD_READINESS_AUDIT.md` | Klarifikasi agar plan siap dieksekusi tanpa tebakan | Captain / Build agent |
| `12_ENGINEERING_EXCELLENCE.md` | Senior engineering practices (PR, review, ADR, debt) | Semua engineer |
| `13_TESTING_STRATEGY.md` | Pyramid test, contract, load, chaos, environment | QA + engineer |
| `14_OBSERVABILITY_AND_INCIDENTS.md` | SLI/SLO, alerting, on-call, postmortem, runbook | Ops + engineer |
| `15_SECURITY_AND_COMPLIANCE.md` | STRIDE, OWASP, GDPR, SOC 2 readiness | Security + engineer |
| `16_RELEASE_AND_PERFORMANCE.md` | Release pipeline, feature flag, perf budget | Engineering + DevOps |
| `17_LAUNCH_AND_GROWTH.md` | Launch stage, north star, pricing, GTM | Product + GTM |
| `18_LANDING_AND_MARKETING_SITE.md` | Halaman publik lengkap (landing, pricing, docs, legal, dll) | Frontend + Marketing |
| `19_BRAND_COPY_AND_VOICE.md` | Brand identity, voice/tone, microcopy library | Marketing + Designer |
| `20_EMAIL_TEMPLATES.md` | 47 template email + trigger + preference | Backend + Marketing |
| `21_ERROR_AND_STATE_CATALOG.md` | Katalog error API + UI state | Frontend + Backend |
| `22_GLOSSARY_AND_DATA_DICTIONARY.md` | Glosarium, status enum, format ID/slug | Semua |
| `23_FINAL_BUILD_CHECKLIST.md` | Master checklist Phase 0 → GA | Captain / Engineering Lead |
| `02_DATABASE_SCHEMA.md` | Schema lengkap semua tabel (Drizzle ORM) | Backend |
| `03_API_SPEC.md` | Semua endpoint + request/response + auth | Backend |
| `04_FRONTEND_UIUX_SPEC.md` | Semua page + component + flow + UI spec | Frontend + UI/UX |
| `05_BUILD_DEPLOY_PLAN.md` | Build phases, env, deploy, cost | DevOps + PM |

## Positioning

TESKEL = Operating System untuk digital commerce. Bukan marketplace. Bukan store builder.
Infra layer yang memungkinkan siapapun membangun bisnis produk digital di atasnya.

5 mode pakai: Marketplace → Storefront → Embed → API/SDK → White-label

## Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | Next.js 15 App Router + shadcn/ui + Tailwind |
| API | Hono (edge-first) |
| Database | Neon (Postgres) + Drizzle ORM |
| Cache/Queue | Upstash Redis |
| Storage | Cloudflare R2 |
| Auth | Better Auth |
| Payments | Stripe Connect |
| Email | Resend + React Email |
| Search | Typesense |
| Charts | Recharts |
| Funnel Builder | React Flow |
| Deploy | Vercel (web) + Fly.io (api) + CF Workers (embed) |
| CI/CD | GitHub Actions + Turborepo |
| Monitor | Sentry + PostHog |

## Monorepo Structure

```
teskel/
├── apps/
│   ├── web/          # Next.js (landing + dashboard + storefront + marketplace + checkout + buyer)
│   ├── api/          # Hono API server
│   └── embed/        # Embeddable checkout widget
├── packages/
│   ├── db/           # Shared DB client + schema
│   ├── shared/       # Types, validators, utils
│   ├── sdk/          # @teskel/sdk
│   ├── email/        # React Email templates
│   └── cli/          # @teskel/cli
├── infra/
│   └── terraform/    # IaC
├── .github/workflows/
├── turbo.json
├── pnpm-workspace.yaml
└── .env.example
```

## Page Count: ~55 pages
## API Endpoints: 80+
## Database Tables: 34
## Build Timeline: 12 weeks (4 phases)
