# TESKEL — Testing Strategy
## Production-Grade Quality Plan for a Global Commerce OS

> Setiap perubahan ke TESKEL diuji dengan tingkat keyakinan yang sesuai dampaknya. Dokumen ini mendefinisikan pyramid, gating, data, dan ritual pengujian.

---

## 1. Filosofi

```
Fast feedback first.  Confidence at scale second.  Realism third.
```

Konsekuensi:

- Unit test paling banyak; cepat (<1s), deterministik.
- Integration test fokus pada module boundary penting.
- E2E test menjaga critical path bisnis; jumlah kecil, stabilitas tinggi.
- Contract test menjaga kontrak antar service / dengan vendor.
- Load & chaos test menjaga production behavior.

Tidak ada feature yang dianggap "selesai" jika tidak diuji pada level yang sesuai.

---

## 2. Test Pyramid Target

| Layer | Target Jumlah | Target Coverage Statement | Target Run Time |
|------|---------------|---------------------------|-----------------|
| Unit | 70% test suite | 80% line / 75% branch (kode bisnis) | <60 detik total per package |
| Integration | 20% test suite | Setiap endpoint API + service path utama | <5 menit total |
| E2E | 5% test suite | 12–18 critical flows MVP, ~30 saat GA | <10 menit total |
| Contract | 3% test suite | Semua interface eksternal (Stripe, Resend, Typesense) | <2 menit |
| Load / chaos | 2% test suite | Smoke load nightly + targeted scenario | dijadwalkan |

Anti-target: jangan kejar coverage 100% — kejar test yang menangkap regresi nyata.

---

## 3. Unit Tests

### 3.1 Library

- Vitest (preferensi monorepo) atau Jest jika sudah ada legacy.
- React component test: Vitest + React Testing Library.
- Tidak boleh menggunakan `enzyme`.

### 3.2 Skop

- Pure function utility (kalkulasi fee, format harga, currency).
- Validator Zod (skema input/output).
- React component pure (presentational), tanpa fetch.
- Service business logic dengan dependency di-mock di boundary.

### 3.3 Aturan

- Maksimum 3 expectation per `it` block.
- Tidak boleh mengakses jaringan, filesystem, atau jam sistem nyata.
- Wajib menggunakan `vi.setSystemTime` untuk waktu deterministik.
- Snapshot test diizinkan HANYA untuk output statis kecil (<30 baris). Snapshot panjang dilarang.

Contoh:

```typescript
import { calculatePlatformFee } from './fee';

describe('calculatePlatformFee', () => {
  it('charges 5% for free plan', () => {
    expect(calculatePlatformFee(10_000, 'free')).toBe(500);
  });

  it('charges 0.5% for enterprise plan', () => {
    expect(calculatePlatformFee(10_000, 'enterprise')).toBe(50);
  });

  it('defaults to free fee for unknown plan', () => {
    expect(calculatePlatformFee(10_000, 'invalid')).toBe(500);
  });
});
```

---

## 4. Integration Tests

### 4.1 Skop

- Semua HTTP endpoint API: validasi input, auth/authz, business path, persist DB, side effects.
- Job worker (webhook handler, payout, email).
- Database layer dengan migration nyata.

### 4.2 Infrastruktur

- Postgres ephemeral via `pg-mem` (cepat) atau `testcontainers` (lebih realistic untuk migrasi).
- Redis ephemeral via `testcontainers/ioredis-mock`.
- S3-compat: MinIO via testcontainers.
- Stripe: gunakan `stripe-mock` atau fixture HTTP record/replay.
- Resend: in-process collector (lihat §10).

### 4.3 Test Pattern

```typescript
describe('POST /v1/orgs/:slug/products', () => {
  let app: TestApp;
  let admin: AuthContext;

  beforeAll(async () => {
    app = await createTestApp();
    admin = await app.factory.createAdminUser();
  });

  afterAll(async () => app.close());

  it('returns 201 with product payload for admin', async () => {
    const res = await app
      .request('/v1/orgs/demo/products', { method: 'POST', body: validProduct(), auth: admin });
    expect(res.status).toBe(201);
    const json = await res.json();
    expect(json.data.id).toBeDefined();
    expect(json.data.orgId).toBe(admin.orgId);
  });

  it('returns 403 for viewer role', async () => { /* ... */ });

  it('returns 422 for invalid product type', async () => { /* ... */ });

  it('returns 409 for duplicate slug within same org', async () => { /* ... */ });

  it('refuses to create product in another org', async () => {
    const other = await app.factory.createAdminUser({ orgSlug: 'other' });
    const res = await app
      .request('/v1/orgs/demo/products', { method: 'POST', body: validProduct(), auth: other });
    expect(res.status).toBe(403);
  });
});
```

Setiap suite WAJIB punya tes "tidak bisa lihat data org lain".

---

## 5. End-to-End Tests

### 5.1 Tooling

- Playwright (Chromium, Firefox, WebKit).
- Pakai test ID stabil: `data-testid="..."`. Jangan rely pada copy text yang berubah.

### 5.2 Flow Wajib Dipertahankan

| Flow | Phase Wajib |
|------|-------------|
| Creator signup → onboarding → org dibuat | Phase 1 |
| Create download product → upload file → publish | Phase 1 |
| Buyer view storefront → checkout → success | Phase 1 |
| Buyer download file via signed URL | Phase 1 |
| Creator melihat order baru di dashboard | Phase 1 |
| Buyer mengakses buyer portal | Phase 1 |
| Discount code apply | Phase 2 |
| License generate → validate → activate | Phase 2 |
| Subscription billing flow | Phase 2 |
| Marketplace search → buy | Phase 3 |
| Affiliate referral diatribut | Phase 3 |
| Embed widget di domain pihak ketiga | Phase 4 |
| SDK call dari project luar | Phase 4 |

### 5.3 Aturan

- Run di mode headless di CI, headed di lokal.
- Tidak boleh sleep arbitrer. Pakai `await expect(locator).toHaveText(...)` dengan timeout terbatas.
- Setiap test mencatat trace via `trace: 'on-first-retry'`.
- Reset state via API atau seed isolation per worker, bukan UI klik berulang.

### 5.4 Stability Policy

- Test flaky tiga kali berturut → di-quarantine ke `tests/quarantine/` dan ticket dibuka.
- Maksimum 5 test di quarantine pada saat yang sama. Lebih dari itu blok release.

---

## 6. Contract Tests

### 6.1 External Contract

Untuk dependency yang stabil tapi penting (Stripe webhook, Resend API, Typesense, OAuth provider), kita pakai dua lapis:

1. Schema test: response yang diharapkan dipersist sebagai JSON fixture, divalidasi Zod.
2. Probe nightly: hit endpoint sandbox vendor untuk memastikan schema tidak berubah. Failure → alert non-blocking, ticket otomatis.

### 6.2 Internal Contract (Consumer-Driven)

Antar service internal (apps/web ↔ apps/api ↔ apps/embed):

- API spec (`03_API_SPEC.md`) generate OpenAPI.
- Consumer membaca generated TypeScript client.
- Provider menjalankan contract verification step di CI sebelum merge.

---

## 7. Load and Performance Tests

### 7.1 Tooling

- k6 untuk HTTP load.
- Artillery untuk WebSocket / event-driven load (post-MVP).

### 7.2 Scenario Wajib

| Scenario | Frekuensi | Target |
|----------|-----------|--------|
| Smoke load (baseline) | Setiap PR ke main (5 RPS, 60s) | P95 latency tetap ≤ baseline + 10% |
| Spike load checkout | Mingguan | P95 < 800 ms, error <0.5% pada 200 RPS |
| Burst marketplace search | Mingguan | P95 < 300 ms pada 500 RPS |
| Soak test 4 jam | Bulanan | Tidak ada memory leak (RSS naik <2%/jam) |
| License validate | Mingguan | P99 < 100 ms pada 1k RPS |
| File download signed URL | Mingguan | P95 < 200 ms |

Hasil di-track di Grafana dashboard `perf-overview`. Regresi >10% memblok release.

### 7.3 Performance Budget

Lihat `16_RELEASE_AND_PERFORMANCE.md`. Test load yang melanggar budget melempar status non-success di CI.

---

## 8. Chaos & Failure Injection

Mulai diaktifkan pada Phase 4.

### 8.1 Skenario Wajib

| Skenario | Cara | Ekspektasi |
|----------|------|-----------|
| Stripe webhook delay 5s | Mock latency di test env | UI tetap responsif; pesan "processing" muncul; tidak ada double order |
| Redis down | Stop container | Aplikasi degraded mode (rate-limit fallback memory, cache miss penuh ke DB), tetap respons <2s |
| Postgres read-replica lag 30s | Toggle lag | Critical path tetap pakai primary, non-critical boleh stale |
| R2 timeout | Network fault | Retry 3× exponential, lalu pesan error jelas |
| Email provider down | Toggle | Email diqueue, dikirim ulang setelah recovery, log alarm |
| Currency conversion API down | Toggle | Fallback ke cache 24 jam terakhir |
| Stripe Connect account suspended mid-flow | Webhook | Order ditangguhkan; UI memberitahu creator |

### 8.2 Aturan

- Hanya jalan di staging atau test env yang isolated.
- Dijalankan terjadwal mingguan, hasil di-postingkan ke channel `#chaos-results`.
- Setiap kelemahan baru menjadi ticket dengan owner.

---

## 9. Security Tests

### 9.1 SAST

- ESLint security plugin (`eslint-plugin-security`).
- TypeScript strict tanpa `any`.
- `semgrep` config khusus TS/SQL/Drizzle, run di CI.

### 9.2 DAST

- ZAP baseline scan ke staging nightly.
- Targeted scan sebelum release besar.

### 9.3 Dependency Scan

- `pnpm audit` per CI. Vulnerability `high`/`critical` memblok.
- Snyk atau GitHub Advanced Security pada Phase 4+.

### 9.4 Penetration Test

- Internal pen test setiap minor release.
- Pen test pihak ketiga setiap major release / setahun sekali.

Lihat detail di `15_SECURITY_AND_COMPLIANCE.md`.

---

## 10. Test Data Management

### 10.1 Prinsip

- Data test deterministik, dapat diregenerasi.
- Tidak ada data produksi di staging tanpa anonimisasi.
- Setiap suite punya factory yang menghasilkan entitas independen.

### 10.2 Factories

```typescript
export const factory = {
  user: (overrides?) => ({
    id: uuid(),
    email: `user+${faker.string.alphanumeric(8)}@test.local`,
    name: faker.person.fullName(),
    ...overrides,
  }),
  org: (overrides?) => ({ /* ... */ }),
  product: (overrides?) => ({ /* ... */ }),
};
```

### 10.3 Seed Levels

- `seed:minimal` — 1 creator, 1 product (untuk smoke test).
- `seed:demo` — 5 creator, 20 product, 50 order (untuk demo & manual QA).
- `seed:scale` — 1k creator, 50k product, 500k order (untuk perf test).

Semua seed deterministik (fixed RNG seed).

### 10.4 Email Capture

Saat test, gunakan in-process collector:

```typescript
const inbox = await getInbox();
const email = await inbox.waitForFirst({ to: buyer.email, subject: /receipt/i });
expect(email.html).toContain(orderId);
```

---

## 11. Flaky Test Policy

1. Test flaky terdeteksi otomatis bila gagal lalu lulus tanpa code change dalam 24 jam.
2. Flaky test dipindahkan ke `tests/quarantine/` dan ticket dengan label `flaky-test` dibuat.
3. Quarantine maksimum 5 file. Lebih dari itu blok release.
4. Investigasi root cause (race, timing, data leak) dalam 5 hari kerja.
5. Hapus quarantine setelah fix terverifikasi 30 run berturut hijau.

Dilarang menambahkan `retry()` per test tanpa root cause analysis. Retry default sistem-level: `playwright.config` retries=2 di CI, 0 di local.

---

## 12. Coverage Targets

Coverage adalah indikator, bukan goal.

| Folder | Target |
|--------|--------|
| `apps/api/src/services/` | ≥ 85% line |
| `apps/api/src/routes/` | ≥ 75% line |
| `apps/web/src/components/` | ≥ 60% line (UI). Snapshot tidak dihitung. |
| `packages/shared/` | ≥ 90% line |
| `packages/db/` | ≥ 80% line |

Coverage di-track di Codecov. Penurunan >2% memblok merge (komen automation).

---

## 13. Mutation Testing (Post-MVP)

Mulai diaktifkan Phase 4 untuk module bisnis utama (`pricing`, `fee`, `discount`, `license`).

Tool: Stryker.

Mutation score target: ≥ 70% untuk module bisnis utama.

Mutation testing tidak di-CI mandatory (mahal), tapi dijalankan terjadwal mingguan.

---

## 14. Accessibility Tests

- Setiap PR yang menyentuh UI menjalankan `axe-core` pada storybook stories.
- Playwright e2e wajib menjalankan satu pass `injectAxe(); checkA11y();` pada landing, checkout, dan dashboard utama.
- Target: tidak ada violation `serious` atau `critical`.

Detail standar di `09_TOP_GLOBAL_FEATURES.md` §11 dan `10_UIUX_MODERN_CLEAN.md` §6.

---

## 15. Test Environments

| Env | Tujuan | Data | Akses |
|-----|--------|------|-------|
| local | dev iterasi | seed:minimal | dev only |
| ci | CI run | seed:minimal | automation |
| preview | per-PR ephemeral | seed:demo | tim |
| staging | rilis kandidat | seed:demo + anonymized prod sample | tim & QA |
| sandbox | demo eksternal | seed:demo (read-only) | sales/marketing |
| production | live | real | terbatas, audit log |

Aturan:

- Tidak boleh menjalankan migration destructive ke staging tanpa snapshot baru.
- Production hanya bisa dibaca melalui tools yang teregulasi (Metabase RBAC, dsb).

---

## 16. Continuous Testing in CI

CI memblokir merge jika salah satu gagal:

```
1. pnpm install --frozen-lockfile
2. pnpm lint
3. pnpm typecheck
4. pnpm test --filter=affected
5. pnpm test:integration --filter=affected
6. pnpm test:e2e:smoke
7. pnpm build
8. pnpm contract:verify (jika perubahan API spec)
9. pnpm audit (high/critical block)
10. semgrep --config p/owasp-top-ten
```

E2E penuh berjalan terjadwal nightly + sebelum release.

---

## 17. Quality Triage & Bug Severity

Ditangani sesuai severity (lihat `14_OBSERVABILITY_AND_INCIDENTS.md` untuk skala incident):

| Severity | Definisi | Respon |
|----------|---------|--------|
| SEV1 | Total outage critical path (checkout down) | Stop release, on-call dipanggil |
| SEV2 | Sebagian besar pengguna terdampak (delivery lambat, dashboard error) | Patch dalam 24 jam |
| SEV3 | Bug mengganggu tapi ada workaround | Fix dalam release berikutnya |
| SEV4 | Cosmetic / minor | Backlog |

Setiap bug yang muncul ke produksi memicu postmortem (lihat `14_OBSERVABILITY_AND_INCIDENTS.md`).

---

## 18. Test Anti-Patterns

Larangan:

- Test yang `console.log` lalu tidak assert.
- `it.skip` tanpa ticket dan tanggal review.
- Test order-dependent (suite gagal jika di-run acak).
- Mock yang menyembunyikan bug (`vi.fn().mockResolvedValue({ ok: true })` untuk path error).
- Test yang menulis ke produksi (env validation wajib menolak).
- Snapshot besar (lebih dari 30 baris).
- Mengabaikan promise (lupa `await`).

---

## 19. Release Acceptance Tests

Sebelum release di-tag, jalankan:

```bash
pnpm release:acceptance
```

Script ini menjalankan:

- Smoke E2E pada staging.
- Contract verify.
- Migration dry-run pada snapshot prod (anonymized).
- Load smoke pada 5 endpoint kritikal.
- Synthetic check signed URL R2.

Jika ada yang gagal, release ditunda dan bug ticket otomatis dibuka.

---

*Dokumen ini melengkapi `00_README_BUILD_ORDER.md` §1.5. Bila conflict, dokumen ini menang untuk metodologi dan kebijakan, sementara `00_README_BUILD_ORDER.md` menang untuk perilaku coding low-level.*
