# TESKEL — Product Overview

> Baca `TESKEL_BUILD_INDEX.md` dulu. File ini menjelaskan positioning, stack, dan struktur target.

| File | Isi | Untuk Siapa |
|------|-----|-------------|
| `TESKEL_BUILD_INDEX.md` | Index utama dan urutan baca | Semua |
| `00_README_BUILD_ORDER.md` | Aturan coding, build order, verification discipline | Semua builder |
| `11_BUILD_READINESS_AUDIT.md` | Klarifikasi agar plan siap dieksekusi tanpa tebakan | Captain / Build agent |
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
