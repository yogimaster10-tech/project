# TESKEL — Infrastructure, Security, Scaling, DevOps Specification
## Production-Grade Engineering Detail

---

## 1. CLOUD ARCHITECTURE — Complete

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                           GLOBAL EDGE LAYER                                 │
│                    Cloudflare (300+ PoP worldwide)                          │
│                                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐     │
│  │   CDN       │  │  WAF        │  │  DDoS       │  │  Workers     │     │
│  │  (Static    │  │  (OWASP     │  │  Protection │  │  (Embed +    │     │
│  │   Assets)   │  │   Rules)    │  │  (L3-L7)    │  │   Edge API) │     │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘     │
│         │                │                │                 │               │
│         └────────────────┴────────────────┴─────────────────┘              │
│                                    │                                       │
│                     ┌──────────────┼──────────────┐                       │
│                     │              │              │                        │
│              ┌──────▼──────┐ ┌────▼───────┐ ┌───▼────────┐              │
│              │  Americas   │ │  Europe    │ │ Asia-Pacific│              │
│              │  Region     │ │  Region    │ │  Region     │              │
│              │             │ │            │ │             │              │
│              │ Vercel:     │ │ Vercel:   │ │ Vercel:    │              │
│              │  us-east-1  │ │  eu-west-1 │ │  ap-south-1 │              │
│              │  us-west-2  │ │  eu-central│ │  ap-ne-1    │              │
│              │             │ │            │ │  ap-southeast│             │
│              │ Fly.io:     │ │ Fly.io:   │ │ Fly.io:    │              │
│              │  sjc        │ │  fra       │ │  sin        │              │
│              │  ord        │ │  ams       │ │  hkg        │              │
│              │  iad        │ │  lhr       │ │  nrt        │              │
│              └──────┬──────┘ └────┬───────┘ └───┬────────┘              │
│                     │              │              │                        │
│                     └──────────────┼──────────────┘                       │
│                                    │                                       │
│                     ┌──────────────▼──────────────┐                       │
│                     │       DATA LAYER             │                       │
│                     │                              │                       │
│                     │  Neon: Regional + Read Repl  │                       │
│                     │  Redis: Global (Upstash)     │                       │
│                     │  R2: Auto-replicate regions   │                       │
│                     │  ClickHouse: Analytics OLAP  │                       │
│                     └──────────────────────────────┘                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.1 Compute Sizing

| Service | Instance Type | CPU | RAM | Disk | Region | Count | Auto-scale |
|---------|-------------|-----|-----|------|--------|-------|-------------|
| **Vercel** | Serverless Functions | - | - | - | Global | Auto | ✅ (auto) |
| **Fly.io API** | shared-cpu-1x | 1 vCPU | 256MB | - | sjc,fra,sin | 2/region | ✅ min=1 max=5 |
| **Fly.io Workers** | shared-cpu-1x | 1 vCPU | 256MB | - | sjc | 1 | ❌ |
| **CF Workers (embed)** | Workers Paid | - | 128MB | - | Global | Auto | ✅ |
| **Neon** | 0.5 CU | 0.5 vCPU | 2GB | 10GB | us-east-2 | 1 + 1 repl | ✅ |
| **Upstash Redis** | Pay-per-request | - | - | 1GB | Global | Auto | ✅ |
| **R2** | - | - | - | Unlimited | Global | Auto | ✅ |
| **Typesense** | 2 vCPU / 4GB | 2 | 4GB | 50GB | us-east-1 | 1 | ❌ |

**Auto-scale triggers:**
- Fly.io: CPU > 70% for 5 min → scale up; CPU < 20% for 15 min → scale down
- Neon: Auto-resume on connection, auto-suspend after 5 min idle
- Vercel: Auto-scale per request (serverless)

---

## 2. SECURITY — Defense in Depth

### 2.1 Network Security

```
LAYER 1: Cloudflare WAF
  - Managed rules: OWASP Top 10
  - Bot management: Challenge suspicious requests
  - Rate limiting at edge (before hitting origin)
  - Geo-blocking: Block sanctioned countries (OFAC)
  - IP reputation: Block known malicious IPs

LAYER 2: TLS/SSL
  - TLS 1.3 minimum (TLS 1.2 for legacy API clients)
  - HSTS: max-age=31536000; includeSubDomains; preload
  - Certificate: Cloudflare Origin CA (origin) + Edge CA (client)
  - Certificate pinning: for mobile SDK (future)

LAYER 3: Application Security
  - CORS: Strict origin allowlist
  - CSP: default-src 'self'; script-src 'self' js.teskel.com; style-src 'self' 'unsafe-inline'
  - X-Frame-Options: DENY (except embed widget)
  - X-Content-Type-Options: nosniff
  - Referrer-Policy: strict-origin-when-cross-origin
  - Permissions-Policy: camera=(), microphone=(), geolocation=()

LAYER 4: Data Security
  - Encryption at rest: AES-256 (Neon, R2 handle this)
  - Encryption in transit: TLS 1.3
  - PII field-level encryption: 
    - customers.email: AES-256-GCM with org-specific key
    - users.email: AES-256-GCM with platform key
    - Keys stored in Cloudflare KV (encrypted with master key)
  - Database connection: SSL required (sslmode=require)
```

### 2.2 Authentication Security

```
Password Policy:
  - Min 8 characters
  - Max 128 characters
  - Bcrypt with cost factor 12
  - Breached password check (HaveIBeenPwned API, k-anonymity)
  - No password reuse (check last 5)

Session Security:
  - JWT access token: 15 min expiry, RS256 signed
  - Refresh token: 30 day expiry, httpOnly + Secure + SameSite=Strict cookie
  - Session bound to IP + User-Agent (revalidate on change)
  - Concurrent session limit: 5 per user
  - Session revocation: immediate on password change, logout all

MFA (Phase 2):
  - TOTP (Google Authenticator, Authy)
  - Backup codes (10 single-use)
  - Required for: payout changes, API key creation, team member invite

OAuth Security:
  - PKCE flow for all OAuth
  - State parameter with CSRF token
  - Nonce validation
  - Only allow verified emails from OAuth providers
```

### 2.3 API Security

```
Input Validation:
  - ALL inputs validated with Zod schemas
  - SQL injection: Impossible (Drizzle parameterized queries)
  - XSS: React auto-escapes + DOMPurify for rich text
  - CSRF: SameSite cookies + CSRF token for state-changing requests
  - SSRF: URL allowlist for webhook endpoints, no internal URLs
  - File upload:
    - Max 500MB per file
    - Allowed types: whitelist (zip, pdf, fig, notion, md, txt, json, png, jpg, mp4)
    - Virus scan: ClamAV on upload (async, quarantine if positive)
    - MIME type verification (not just extension)

Output Security:
  - No sensitive data in responses (password hash, stripe keys, etc.)
  - API keys: only show prefix in list, full key only on creation
  - PII masking in logs: email → e***@example.com

Rate Limiting (detailed):
  ┌──────────────────────────────────┬───────────┬──────────┐
  │ Endpoint                         │ Limit     │ Window   │
  ├──────────────────────────────────┼───────────┼──────────┤
  │ POST /v1/auth/login             │ 5         │ 15 min   │
  │ POST /v1/auth/signup            │ 3         │ 60 min   │
  │ POST /v1/auth/forgot-password   │ 3         │ 60 min   │
  │ GET /v1/licenses/:key/validate  │ 100       │ 1 min    │
  │ POST /v1/licenses/:key/activate │ 10        │ 1 min    │
  │ POST /v1/checkout/sessions      │ 30/org    │ 1 min    │
  │ GET /v1/marketplace/*           │ 200       │ 1 min    │
  │ GET /v1/orgs/:slug/* (default)  │ 100/org   │ 1 min    │
  │ POST /v1/orgs/:slug/* (writes)  │ 30/org    │ 1 min    │
  │ POST /v1/webhooks/stripe        │ 1000      │ 1 min    │
  │ POST /v1/discounts/validate     │ 20        │ 1 min    │
  │ POST /v1/access/check           │ 50        │ 1 min    │
  └──────────────────────────────────┴───────────┴──────────┘
  
  API Key tier overrides:
  - free: 100 req/min
  - pro: 1,000 req/min
  - scale: 5,000 req/min
  - enterprise: 10,000 req/min

  Implementation: Upstash Ratelimit (sliding window)
  Headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset
  429 response: { error: { code: "RATE_LIMITED", message: "...", retryAfter: 60 } }
```

### 2.4 Row-Level Security (RLS)

```sql
-- Neon PostgreSQL RLS policies
-- Ensures data isolation between organizations

ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE license_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE discount_codes ENABLE ROW LEVEL SECURITY;
ALTER TABLE funnels ENABLE ROW LEVEL SECURITY;
ALTER TABLE affiliates ENABLE ROW LEVEL SECURITY;
ALTER TABLE webhook_endpoints ENABLE ROW LEVEL SECURITY;
ALTER TABLE api_keys ENABLE ROW LEVEL SECURITY;
ALTER TABLE email_subscribers ENABLE ROW LEVEL SECURITY;
ALTER TABLE email_sequences ENABLE ROW LEVEL SECURITY;
ALTER TABLE payout_schedules ENABLE ROW LEVEL SECURITY;

-- Example policy: products
CREATE POLICY "org_members_can_read_own_products" ON products
  FOR SELECT USING (
    org_id IN (
      SELECT om.org_id FROM organization_members om
      WHERE om.user_id = current_setting('app.current_user_id')::UUID
      AND om.role IN ('owner', 'admin', 'member', 'viewer')
    )
  );

CREATE POLICY "org_members_can_write_own_products" ON products
  FOR ALL USING (
    org_id IN (
      SELECT om.org_id FROM organization_members om
      WHERE om.user_id = current_setting('app.current_user_id')::UUID
      AND om.role IN ('owner', 'admin', 'member')
    )
  );

-- Public read for published products (storefront/marketplace)
CREATE POLICY "public_can_read_published_products" ON products
  FOR SELECT USING (
    status = 'active' AND visibility IN ('public', 'unlisted')
  );

-- Same pattern for all org-scoped tables
-- Application sets current_setting before each query:
-- SET app.current_user_id = 'uuid-here';
```

### 2.5 Data Privacy & GDPR

```
Data Classification:
  ┌──────────────────┬──────────────┬──────────────────────────┐
  │ Category         │ Examples     │ Handling                 │
  ├──────────────────┼──────────────┼──────────────────────────┤
  │ PII (Sensitive)  │ email, name  │ Encrypted at rest,       │
  │                  │              │ masked in logs,          │
  │                  │              │ deletable on request     │
  ├──────────────────┼──────────────┼──────────────────────────┤
  │ Financial        │ card tokens  │ Never stored by TESKEL   │
  │                  │              │ (Stripe handles)         │
  ├──────────────────┼──────────────┼──────────────────────────┤
  │ Business         │ orders,      │ Org-owned, retained      │
  │                  │ revenue      │ per retention policy     │
  ├──────────────────┼──────────────┼──────────────────────────┤
  │ Analytics        │ page views,  │ Anonymized after 90d,    │
  │                  │ clicks       │ aggregated only          │
  ├──────────────────┼──────────────┼──────────────────────────┤
  │ System           │ logs, errors │ Auto-purged after 30d    │
  └──────────────────┴──────────────┴──────────────────────────┘

GDPR Compliance:
  - Right to Access: GET /v1/auth/me returns all user data
  - Right to Erasure: DELETE /v1/auth/me (soft delete, hard after 30d)
    - Anonymize PII in orders (keep financial records for tax)
    - Delete email subscribers
    - Revoke all sessions + API keys
  - Right to Portability: GET /v1/auth/me/export → JSON download
  - Consent Management:
    - Email subscribers: double opt-in
    - Marketing emails: separate consent from transactional
    - Cookie consent: banner with categories (necessary, analytics, marketing)
  - Data Processing Agreement: auto-generated per org
  - DPO: designated contact in legal page
  - Breach notification: 72h to authorities, users notified

Data Retention:
  - User data: until account deletion + 30d
  - Order data: 7 years (tax requirement)
  - Analytics: 90 days raw, then aggregated
  - Logs: 30 days
  - Sessions: until expiry
  - Email subscribers: until unsubscribe or account deletion
```

---

## 3. CACHING & CDN — Strategy

### 3.1 CDN (Cloudflare)

```
Static Assets:
  - Next.js static files: /_next/static/* → Cache 1 year (immutable)
  - Images: /_next/image → Cache 7 days
  - Fonts: Cache 1 year
  - R2 public files: files.teskel.com → Cache 24h

Dynamic Pages:
  - Landing page: Cache 1h, revalidate on deploy
  - Storefront: Cache 5min, purge on product update
  - Marketplace: Cache 5min, purge on listing update
  - Product page: Cache 5min, purge on product update
  - Dashboard: NO cache (auth-required, real-time)

Cache Purging:
  - Product updated → purge /{storeSlug}/* and /products/{slug}
  - Product published → purge /explore/* and /{storeSlug}/*
  - Review added → purge /products/{slug}
  - Theme changed → purge /{storeSlug}/*

Implementation:
  - Cloudflare Cache API for programmatic purging
  - Next.js ISR (Incremental Static Regeneration) for storefront
  - revalidateTag() for targeted revalidation
```

### 3.2 Application Cache (Redis)

```
┌──────────────────────────────────┬──────────┬──────────────────────────┐
│ Cache Key                        │ TTL       │ Invalidation             │
├──────────────────────────────────┼──────────┼──────────────────────────┤
│ license:validate:{key}           │ 60s       │ On license update/active │
│ access:check:{email}:{product}   │ 300s      │ On access grant change   │
│ discount:validate:{code}:{org}   │ 300s      │ On discount update       │
│ product:public:{slug}            │ 300s      │ On product update        │
│ org:settings:{slug}              │ 600s      │ On settings update       │
│ marketplace:search:{query}       │ 60s       │ On product publish       │
│ marketplace:trending             │ 300s      │ On schedule (5min)       │
│ analytics:summary:{org}:{period} │ 900s      │ On new order             │
│ checkout:session:{id}            │ 1800s     │ On session expire/compete│
│ rate-limit:{org}:{endpoint}      │ Window    │ Sliding window auto      │
└──────────────────────────────────┴──────────┴──────────────────────────┘

Cache Strategy:
  - Write-through: Update cache on DB write
  - Cache-aside: Read cache first, miss → DB → cache
  - No cache for: Dashboard CRUD, order management, payout processing
```

### 3.3 Database Query Cache

```
Materialized Views (refreshed periodically):
  - mv_product_stats: product_id, total_sales, total_revenue, avg_rating
    Refresh: every 5 min (pg_cron)
  - mv_org_revenue_daily: org_id, date, revenue_cents, order_count
    Refresh: every 1 hour
  - mv_marketplace_trending: product_id, score (velocity * rating)
    Refresh: every 15 min

Query Optimization:
  - All queries use indexes (defined in 01_DATABASE.md)
  - EXPLAIN ANALYZE on slow queries (>100ms)
  - Connection pooling: PgBouncer (transaction mode) via Neon
  - Read replicas for analytics queries
  - Partitioning: analytics_events by month (pg_partman)
```

---

## 4. LOAD BALANCING & SCALING

### 4.1 Load Balancing

```
Global:
  Cloudflare → routes to nearest region (anycast DNS)

Regional:
  Fly.io internal load balancer → distributes across machines in region

API:
  Fly.io machines (2 per region, auto-scale 1-5)
  Health check: GET /v1/health → 200 OK
  Health check interval: 10s
  Unhealthy threshold: 3 failures → remove from pool
  Healthy threshold: 2 successes → add back to pool

Web:
  Vercel Edge Network → auto-distributes to nearest edge function
  No manual load balancing needed
```

### 4.2 Auto-Scaling Rules

```
Fly.io API Machines:
  Scale-up trigger:
    - CPU > 70% for 5 consecutive minutes
    - Request queue depth > 50
    - p95 latency > 500ms
  Scale-down trigger:
    - CPU < 20% for 15 consecutive minutes
    - Request queue depth < 5
  Min machines: 1 per region (always-on)
  Max machines: 5 per region
  Cooldown: 5 min between scale events

Neon Database:
  Auto-resume: On first connection after suspend
  Auto-suspend: After 5 min idle (free tier), never (paid)
  Compute scaling: 0.25 CU → 2 CU based on load (paid)
  Read replica: Auto-added when read queries > 70% of total

Vercel:
  Serverless: Auto-scale per request (no config needed)
  Edge functions: Auto-replicate globally
  Limit: 1000 concurrent serverless functions (Pro plan)
```

### 4.3 Database Scaling Strategy

```
Phase 1 (0-500 creators): Single Neon instance + 1 read replica
Phase 2 (500-2000): Neon Scale plan + 2 read replicas + PgBouncer
Phase 3 (2000-10000): Neon Business + connection pooler + partitioning
Phase 4 (10000+): Evaluate CockroachDB / Citus for horizontal sharding

Partitioning:
  - analytics_events: Partition by month (drop old partitions)
  - orders: Partition by year (archive old years)
  - webhook_deliveries: Partition by month (drop after 90d)

Read Split:
  - Write: Primary Neon instance
  - Analytics reads: Read replica (dedicated)
  - Dashboard reads: Primary (needs consistency)
  - Marketplace reads: Read replica (eventual consistency OK)
```

---

## 5. ERROR TRACKING & LOGGING

### 5.1 Structured Logging

```typescript
// Log format (JSON, one line per log)
interface LogEntry {
  timestamp: string;       // ISO 8601
  level: 'debug' | 'info' | 'warn' | 'error' | 'fatal';
  service: string;         // 'api' | 'web' | 'worker'
  traceId: string;         // OpenTelemetry trace ID
  spanId: string;          // OpenTelemetry span ID
  orgId?: string;          // Tenant context
  userId?: string;         // User context
  message: string;
  metadata?: Record<string, any>;
  error?: {
    name: string;
    message: string;
    stack?: string;        // Only in dev/staging
  };
}

// Example:
{"timestamp":"2025-05-22T14:30:00.000Z","level":"info","service":"api","traceId":"abc123","spanId":"def456","orgId":"org-uuid","userId":"user-uuid","message":"checkout.session.completed","metadata":{"orderId":"ord-uuid","totalCents":1900,"currency":"USD"}}
```

### 5.2 Error Tracking (Sentry)

```
Setup:
  - Frontend: @sentry/nextjs (auto-capture React errors)
  - Backend: @sentry/node (auto-capture unhandled exceptions)
  - Performance: Sentry Performance (trace slow requests)

Configuration:
  - Environment: production | staging | development
  - Release: git SHA (auto-tagged by CI)
  - Sample rate: 100% errors, 10% transactions
  - Before send: Scrub PII (email → [email]), remove password fields
  - Sourcemaps: Auto-upload on deploy (Vercel/Fly integration)

Alert Rules:
  - Error rate > 1% of requests → Slack #alerts
  - New error (first occurrence) → Slack #alerts
  - Error in checkout flow → PagerDuty (critical)
  - Error in webhook handler → Slack #alerts (high priority)
  - Error in payout worker → PagerDuty (critical)
```

### 5.3 Application Monitoring

```
Metrics (exposed via /v1/metrics, Prometheus format):
  - teskel_requests_total{method, path, status}
  - teskel_request_duration_seconds{method, path}
  - teskel_checkout_created_total{org_id}
  - teskel_checkout_completed_total{org_id}
  - teskel_license_validations_total{valid}
  - teskel_stripe_webhook_processed_total{event_type}
  - teskel_email_sent_total{template}
  - teskel_payout_processed_total{status}
  - teskel_active_subscriptions{org_id}
  - teskel_redis_cache_hit_ratio

Dashboards (Grafana):
  - API Overview: Request rate, latency p50/p95/p99, error rate
  - Business: GMV, orders/hr, new customers/hr
  - Infrastructure: CPU, memory, disk, connections
  - Per-Org: Revenue, orders, top products

Alerting:
  - p99 latency > 2s → Slack
  - Error rate > 5% → PagerDuty
  - Database connections > 80% pool → Slack
  - Disk usage > 80% → Slack
  - Queue depth > 1000 → Slack
```

---

## 6. AVAILABILITY & RECOVERY

### 6.1 Availability Targets

```
SLA Target: 99.9% uptime (8.76h downtime/year)

Component Availability:
  ┌──────────────────────┬───────────┬─────────────────────────┐
  │ Component           │ Target    │ Strategy                │
  ├──────────────────────┼───────────┼─────────────────────────┤
  │ Landing page         │ 99.99%    │ Static + CDN            │
  │ Storefront           │ 99.9%     │ ISR + CDN + fallback    │
  │ Checkout             │ 99.95%    │ Stripe-hosted           │
  │ API                  │ 99.9%     │ Multi-region + auto-scale│
  │ License validation   │ 99.99%    │ Edge + Redis cache      │
  │ Dashboard            │ 99.5%     │ Acceptable for internal │
  │ File delivery        │ 99.9%     │ R2 + signed URLs        │
  │ Email delivery       │ 99.0%     │ Queue + retry           │
  └──────────────────────┴───────────┴─────────────────────────┘
```

### 6.2 Backup & Recovery

```
Database (Neon):
  - Point-in-time recovery: 7 days (free), 30 days (paid)
  - Branching: Create branch before schema changes
  - Daily logical backup: pg_dump to R2 (cron job)
  - Backup retention: 30 days
  - RPO (Recovery Point Objective): <1 hour
  - RTO (Recovery Time Objective): <15 minutes

File Storage (R2):
  - Versioning: Enabled (keep last 10 versions per object)
  - Replication: Cross-region (us-east → eu-west → ap-south)
  - RPO: <1 hour
  - RTO: <5 minutes

Redis (Upstash):
  - Persistence: AOF every 1 second
  - No backup needed (cache only, rebuildable from DB)

Configuration:
  - All infra as code (Terraform)
  - Config in git (version controlled)
  - Secrets in: Fly.io secrets, Vercel env, Cloudflare KV
  - RPO: 0 (git)
  - RTO: <30 minutes (terraform apply)
```

### 6.3 Disaster Recovery Plan

```
Scenario 1: API server down (1 region)
  1. Cloudflare routes to healthy regions
  2. Fly.io auto-restarts machine
  3. If persistent: manual intervention, check logs
  4. RTO: <5 min (automatic)

Scenario 2: Database outage
  1. Neon auto-failover to replica
  2. If total outage: promote read replica
  3. If data loss: restore from PITR
  4. RTO: <15 min

Scenario 3: R2 outage
  1. File downloads fail → show error + retry button
  2. New uploads queued in Redis → process when R2 recovers
  3. RTO: depends on Cloudflare (usually <5 min)

Scenario 4: Stripe outage
  1. Checkout fails → show maintenance message
  2. Webhooks queue in Redis → process when Stripe recovers
  3. Existing access continues (no impact on current customers)
  4. RTO: depends on Stripe (usually <15 min)

Scenario 5: Total region failure
  1. Cloudflare routes to other regions
  2. API: Fly.io machines in other regions serve traffic
  3. Database: Neon cross-region replica promoted
  4. RTO: <30 min
```

---

## 7. CI/CD & VERSION CONTROL

### 7.1 Git Branching Strategy

```
main ──────────────────────────────────────────────► (production)
  │                                                    │
  ├─── develop ──────────────────────────────────────► (staging)
  │      │                                             │
  │      ├─── feature/TESK-123-product-crud ──►       │
  │      ├─── feature/TESK-124-checkout ──────►       │
  │      ├─── bugfix/TESK-125-license-bug ────►       │
  │      │                                             │
  │      └─── release/v0.1.0 ────────────────────────►│
  │                                                    │
  └─── hotfix/TESK-126-critical-bug ─────────────────►│

Rules:
  - main: protected, only merge via PR (1 approval required)
  - develop: protected, auto-deploy to staging
  - feature/*: auto-deploy to preview (Vercel + Neon branch)
  - hotfix/*: fast-track PR to main
  - release/*: merge to both main and develop

Commit Convention:
  feat: add product CRUD
  fix: license validation cache invalidation
  docs: update API documentation
  refactor: extract checkout service
  test: add order service tests
  chore: upgrade Next.js 15.1
  ci: add Fly.io deploy workflow

PR Template:
  ## What
  ## Why
  ## How to test
  ## Checklist
  - [ ] Tests pass
  - [ ] No breaking changes
  - [ ] Migration included (if schema change)
  - [ ] Env vars documented (if new)
```

### 7.2 CI Pipeline (Complete)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  # Job 1: Code Quality
  quality:
    runs-on: ubuntu-latest
    steps:
      - checkout
      - setup pnpm
      - pnpm install --frozen-lockfile
      - pnpm lint              # ESLint
      - pnpm format:check     # Prettier
      - pnpm typecheck        # TypeScript

  # Job 2: Unit + Integration Tests
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: POSTGRES_DB=test, POSTGRES_USER=test, POSTGRES_PASSWORD=test
      redis:
        image: redis:7
    steps:
      - checkout
      - setup pnpm
      - pnpm install --frozen-lockfile
      - pnpm db:migrate:test   # Run migrations against test DB
      - pnpm test              # Vitest unit + integration
      - upload coverage to Codecov

  # Job 3: E2E Tests
  e2e:
    runs-on: ubuntu-latest
    needs: [quality, test]
    if: github.event_name == 'pull_request'
    steps:
      - checkout
      - setup pnpm
      - pnpm install --frozen-lockfile
      - pnpm build
      - pnpm test:e2e          # Playwright
      - upload artifacts (screenshots on failure)

  # Job 4: Deploy Staging
  deploy-staging:
    needs: [quality, test]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - deploy web to Vercel (preview)
      - deploy api to Fly.io (staging)
      - create Neon branch (preview DB)
      - run smoke tests

  # Job 5: Deploy Production
  deploy-production:
    needs: [quality, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - deploy web to Vercel (production)
      - deploy api to Fly.io (production)
      - run migrations (production DB)
      - create Sentry release
      - run smoke tests
      - notify Slack #deployments
```

---

## 8. FRAMEWORK CHOICES — Justification

| Choice | Why | Alternatives Considered | Why Not |
|--------|-----|------------------------|---------|
| **Next.js 15** | SSR+SSG+ISR, App Router, RSC, edge runtime, Vercel integration | Remix | Less ecosystem, fewer components |
| **Hono** | Edge-first, 3x faster than Express, multi-runtime, type-safe | Fastify | Not edge-native, bigger bundle |
| **Drizzle ORM** | Edge-compatible, type-safe, SQL-like, lightweight | Prisma | Rust engine slow at edge, heavy |
| **Neon** | Serverless PG, branching, read replicas, connection pooling | Supabase | More vendor lock-in, less flexible |
| **Upstash Redis** | Global, edge-accessible, pay-per-request | Redis Cloud | Not edge-native, fixed pricing |
| **Cloudflare R2** | Zero egress, S3-compatible, global | AWS S3 | Egress fees kill file delivery |
| **Better Auth** | Self-hosted, open-source, multi-provider, session management | Clerk | Vendor lock-in, pricing at scale |
| **Stripe Connect** | Split payments, KYC, multi-currency, 40+ countries | Paddle | Less flexible, higher fees |
| **Resend** | Developer-friendly, React Email, good deliverability | SendGrid | Worse DX, more setup |
| **Typesense** | Instant search, typo-tolerance, easy self-host | Algolia | Expensive at scale, vendor lock-in |
| **shadcn/ui** | Copy-paste, customizable, accessible, beautiful | MUI | Harder to customize, heavier |
| **Tailwind CSS** | Utility-first, tree-shakeable, design system | CSS Modules | No design system, more boilerplate |
| **Turborepo** | Monorepo, caching, parallel builds | Nx | More complex, overkill for this size |
| **Fly.io** | Global deploy, Docker-native, auto-scale | Railway | Less regions, less control |
| **Vercel** | Next.js optimized, edge functions, preview deploys | Netlify | Less Next.js integration |

---

## 9. ADDITIONAL REFERENCES

### 9.1 Architecture References

| Reference | URL | Why |
|-----------|-----|-----|
| Next.js App Router | nextjs.org/docs/app | Routing, layouts, server components |
| Hono Docs | hono.dev | API framework, middleware, edge |
| Drizzle Docs | orm.drizzle.team | Schema, queries, migrations |
| Neon Docs | neon.tech/docs | Serverless PG, branching, pooling |
| Stripe Connect | stripe.com/connect | Multi-party payments |
| Better Auth | better-auth.com | Auth setup, providers, sessions |
| Cloudflare R2 | developers.cloudflare.com/r2 | Object storage, S3 API |
| Upstash Redis | upstash.com/docs | Edge Redis, rate limiting |
| shadcn/ui | ui.shadcn.com | Component library, theming |
| React Flow | reactflow.dev | Funnel builder, visual editor |
| Resend | resend.com/docs | Email API, React Email |
| Typesense | typesense.org/docs | Search engine, instant results |

### 9.2 Design References

| Reference | For What |
|-----------|----------|
| Stripe Dashboard | Analytics layout, metric cards, charts |
| Gumroad Dashboard | Product list, order management |
| Linear | Command palette, keyboard shortcuts, sidebar |
| Vercel Dashboard | Deployment list, clean minimal UI |
| Notion | Rich text editor, block-based |
| Figma Community | Product card design, storefront |
| Apple Store | Checkout flow, trust badges, payment |

### 9.3 API Design References

| Reference | For What |
|-----------|----------|
| Stripe API | Response format, pagination, error format |
| GitHub REST API | Rate limiting headers, list endpoints |
| Shopify Admin API | GraphQL patterns, filtering |
| Vercel API | Edge API patterns, simple REST |

### 9.4 Security References

| Reference | For What |
|-----------|----------|
| OWASP Top 10 | Web application security baseline |
| OWASP API Security | API-specific security |
| PCI DSS | Payment card security (handled by Stripe) |
| GDPR | Data privacy compliance |
| SOC 2 Type II | Security audit framework (roadmap) |

---

*This document covers: cloud architecture, compute sizing, security (WAF/TLS/CSP/RLS/GDPR), rate limiting per endpoint, caching strategy (CDN/Redis/DB), load balancing, auto-scaling, error tracking, logging, availability SLA, backup/DR, CI/CD pipeline, git strategy, framework justifications, and references.*
