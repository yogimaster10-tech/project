# TESKEL — Release Management & Performance Engineering
## Ship Fast, Recover Faster, Stay Fast

> Rilis sering, kecil, terkontrol; performa diukur sejak commit, bukan setelah keluhan pelanggan.

---

## 1. Release Cadence

| Tipe | Frekuensi | Konten | Risk |
|------|-----------|--------|------|
| Continuous (trunk) | Setiap merge ke `main` | Auto-deploy ke staging | Rendah, otomatis |
| Canary production | Harian (business hours window) | Subset feature flags / canary cluster | Sedang |
| Production GA | 2–5× minggu | Setelah canary stabil 24 jam | Sedang |
| Hotfix | On-demand | Bug fix kritis | Tinggi (jalur khusus) |
| Major release | Per quarter | Feature flag turn-on global, marketing aligned | Tinggi |

Tidak ada "release party" yang menumpuk semua perubahan satu minggu.

---

## 2. Deployment Pipeline

```
PR merged → CI green → build artifact → push image / static bundle →
  staging deploy (auto) → smoke test → canary deploy (manual gate) →
  production rollout (progressive) → post-deploy verification → done
```

Setiap tahap memiliki gate yang otomatis menahan rilis bila gagal.

### 2.1 Tahap Detail

1. **CI Build**
   - Lockfile frozen, lint, typecheck, test, build, contract verify.
   - Artifact ditandatangani (provenance attestation).
2. **Staging Deploy (auto)**
   - Deploy ke env `staging` setelah merge.
   - DB migration dijalankan dengan dry-run + apply.
   - Smoke E2E (Playwright subset) berjalan otomatis.
3. **Canary Production**
   - Subset traffic (5–10%) atau cluster terpisah.
   - Watch dashboard `service-health` & `business-health` 30 menit.
   - Auto-halt bila error rate / P95 > 1.5× baseline.
4. **Progressive Rollout**
   - 10% → 25% → 50% → 100% dengan jeda 15–30 menit.
   - Halting rule sama dengan canary.
5. **Post-Deploy Verification**
   - Synthetic check kritikal lulus.
   - Sentry tidak menampilkan regression baru.
   - Business metric (orders/min) tidak turun >20% vs baseline.

---

## 3. Feature Flags

### 3.1 Vendor / Library

- Flag service: ConfigCat / Statsig / LaunchDarkly. Mulai dengan adapter agar swappable.
- Lokal dev: file `feature-flags.local.json` overrides.

### 3.2 Flag Types

| Type | Tujuan | Lifetime |
|------|-------|----------|
| Release | Ship dark, turn-on bertahap | ≤ 90 hari, lalu cleanup |
| Experiment | A/B test | ≤ 60 hari |
| Ops | Kill-switch fitur produksi | Permanen, audit triwulan |
| Permission | Gate by org/role | Permanen |

### 3.3 Aturan

- Setiap flag punya owner, default value, expected sunset date.
- Flag default OFF untuk fitur baru.
- Kode di-merge dengan flag, ON saat akan rilis.
- Setelah rilis stabil, flag dihapus + flag-related kode dibersihkan.
- Audit triwulan: flag yang lewat sunset → ticket cleanup atau perpanjangan dengan justifikasi.

### 3.4 Anti-Pattern

- Flag bertumpuk tanpa cleanup.
- Logika percabangan dalam-dalam berdasarkan banyak flag (>2 di satu fungsi).
- Flag tanpa observability (tidak tahu siapa yang melihat varian apa).

---

## 4. Canary & Blue-Green

### 4.1 Canary (default)

- Aktif untuk service stateless (apps/web, apps/api).
- Routing: 5% by user-org hash → 25% → 50% → 100%.
- Promosi otomatis bila SLO tidak burn 30 menit.
- Rollback otomatis bila SLO burn 2× selama 5 menit.

### 4.2 Blue-Green (khusus migration berat)

- Dua environment paralel (`prod-blue`, `prod-green`).
- Drainase traffic via load balancer / DNS TTL pendek.
- Smoke test green sebelum switch.
- Setelah switch, blue di-hold sebagai rollback target 24 jam.

### 4.3 DB Migrations

- Pakai pola expand → contract:

```
1. Expand: tambah column baru nullable.
2. Backfill: skrip async.
3. Dual-write: code menulis ke kolom lama & baru.
4. Switch read.
5. Contract: hapus kolom lama (PR terpisah, minimal 1 minggu setelah switch).
```

- Tidak ada perubahan schema yang membutuhkan downtime tanpa announcement 7 hari.

---

## 5. Rollback Playbook

### 5.1 Mode

| Mode | Kapan |
|------|-------|
| Flag off (instant) | Bug pada fitur di-gate flag |
| Re-deploy previous artifact | Bug pada layer aplikasi |
| Database rollback (forward-fix only) | Bug akibat migrasi |
| Disaster: full env restore | Outage region |

### 5.2 Aturan

- Rollback adalah default; debugging di production dilarang.
- Bukti rollback (commit SHA, time, who) tercatat di `#deploys`.
- Setelah rollback, root cause dianalisis dan postmortem dibuat (lihat `14_OBSERVABILITY_AND_INCIDENTS.md`).

---

## 6. Hotfix Policy

```
Severity SEV1/SEV2 → branch dari main → fix kecil → CI green → canary fast (5 menit) → 100%.
```

- Hotfix harus minimal: hanya fix.
- Tidak ada refactor opportunistic.
- PR review 1 reviewer (cukup), tapi WAJIB.
- Hotfix yang menyentuh schema dilarang kecuali tidak ada alternatif. Jika harus, tim wajib pair + sign-off engineering lead.

---

## 7. Release Notes Template

File: `docs/releases/vX.Y.Z.md`.

```markdown
# Release vX.Y.Z — YYYY-MM-DD

## Highlights
- (Top 3 perubahan visible bagi pelanggan)

## Added
- ...

## Changed
- ...

## Fixed
- ...

## Removed / Deprecated
- ...

## Security
- ...

## Breaking Changes
- (Ada/Tidak. Jika ada, sertakan migration guide & deprecation timeline.)

## Migration Notes
- DB migration: forward-only, expand → contract.
- API changes: tidak ada / lihat deprecation policy.

## Known Issues
- ...

## Credits
- Author, reviewer, QA.
```

---

## 8. Versioning Strategy

- API publik: `/v1`. Versi baru hanya saat ada breaking change.
- Library publik (`@teskel/sdk`, `@teskel/cli`): semver ketat.
- App internal: tag `vYYYY.MM.DD.N` untuk rilis (calendar version) — boleh juga semver, konsistensi tim.

---

## 9. Deprecation Policy

| Komponen | Notice Period | Komunikasi |
|----------|---------------|-----------|
| API endpoint publik | 12 bulan | Header `Deprecation` + `Sunset`, changelog, email enterprise |
| SDK method | 6 bulan | Console warning + docs |
| Webhook event | 6 bulan | Notifikasi dashboard + email |
| Database field eksternal | 12 bulan | Dual-write, change log |
| Public URL/slug | 12 bulan | 301 redirect + email |

Tidak ada penghapusan tanpa data telemetry membuktikan penggunaan rendah.

---

## 10. Performance Budget

### 10.1 Web Vitals (Storefront & Marketing)

| Metric | Target |
|--------|--------|
| LCP | ≤ 2.0 s (p75 mobile 4G) |
| INP | ≤ 200 ms (p75) |
| CLS | ≤ 0.05 |
| TTFB | ≤ 600 ms |
| FCP | ≤ 1.5 s |

### 10.2 Dashboard

| Metric | Target |
|--------|--------|
| First meaningful render | ≤ 1.5 s (median desktop) |
| Route transition (cached) | ≤ 200 ms |
| Heavy panel (analytics) | ≤ 2.5 s |
| Bundle initial JS | ≤ 150 KB gzipped landing, ≤ 250 KB gzipped dashboard |
| Bundle CSS | ≤ 30 KB gzipped |
| Image weight per page | ≤ 600 KB (compressed) |

### 10.3 API

| Endpoint Group | P50 | P95 | P99 |
|----------------|-----|-----|-----|
| Auth | 80 ms | 200 ms | 400 ms |
| Products | 100 ms | 300 ms | 600 ms |
| Checkout create session | 200 ms | 600 ms | 1.2 s |
| Webhook processing | 100 ms | 500 ms | 1.0 s |
| License validate (public) | 30 ms | 100 ms | 200 ms |
| Marketplace search | 80 ms | 300 ms | 600 ms |
| File signed URL gen | 60 ms | 200 ms | 400 ms |
| Stripe webhook ACK | 50 ms | 200 ms | 400 ms |

Budget regression >10% memblok release.

### 10.4 Resource Budget

| Resource | Target |
|----------|--------|
| Apps/web cold start | ≤ 1.5 s |
| API container cold start | ≤ 800 ms |
| DB connection pool utilization | ≤ 70% baseline, ≤ 90% spike |
| Redis hit rate (cache) | ≥ 85% |
| R2 download time global p95 | ≤ 800 ms (1 MB file) |

---

## 11. Capacity Planning

### 11.1 Forecast Triggers

- Sign-up growth >30% MoM → cek headroom.
- Order volume >20% WoW → cek webhook/worker scaling.
- Average org size meningkat → cek DB IO.

### 11.2 Headroom Goals

- CPU steady ≤ 50%.
- Memory ≤ 60%.
- DB CPU ≤ 60% steady.
- Queue depth ≤ 1k message baseline.

### 11.3 Cost-Per-Active-Org Target

| Phase | Cost/active org/bulan |
|-------|----------------------|
| Pre-launch | ≤ $0.50 |
| Phase 4 Scale | ≤ $1.20 |
| Phase 6 GA Global | ≤ $1.00 |

Bila melebihi: investigasi, optimisasi (caching, image, S3 lifecycle), atau renegosiasi vendor.

---

## 12. Load Testing Methodology

Lihat `13_TESTING_STRATEGY.md` §7. Praktik unik di sini:

- Setiap perubahan menyentuh checkout/webhook/license validate WAJIB menjalankan k6 scenario terkait di PR.
- Result diunggah sebagai PR artifact.
- Comparison vs baseline disimpan di Grafana (`perf-overview`).
- Regression >10% → comment otomatis di PR + label `perf-regression`.

---

## 13. Performance Engineering Practices

### 13.1 Frontend

- Server Components by default.
- Streaming (RSC stream) + `loading.tsx`.
- Code-splitting per route + per heavy widget.
- Image `next/image` dengan `priority` hanya untuk hero.
- Font: self-hosted variable font, preload subset.
- CSS: Tailwind purged; no inline styles.
- Avoid client state global yang re-render pohon besar; gunakan signals / atomic state.

### 13.2 Backend

- Query DB di-traced; setiap N+1 = bug.
- Cache layered: in-memory (LRU) → Redis → DB.
- Pre-warm cache untuk hot data setelah deploy (storefront popular).
- Streaming response untuk download besar (bukan buffer in-memory).
- Background heavy work via worker (bukan request thread).

### 13.3 Database

- Index design dipikir saat schema (`02_DATABASE_SCHEMA.md`).
- Auto vacuum settings sesuai workload.
- Partitioning untuk `orders`, `analytics_events` setelah threshold.
- Read replica untuk dashboard analitik.

### 13.4 Network

- HTTP/2 + Brotli/Gzip.
- Edge caching agresif pada storefront publik (Cloudflare).
- Avoid cookie-less requests untuk asset CDN.

---

## 14. Cost Engineering

### 14.1 Praktik

- Tag setiap resource cloud dengan `service`, `env`, `team`.
- Dashboard `cost-tracking` ditinjau mingguan.
- Setiap fitur baru estimate cost impact di ADR.
- Setiap vendor punya monthly cap & alert.

### 14.2 Levers Cepat

- TTL cache lebih agresif untuk read-heavy.
- Compress assets, image sizes appropriate.
- Region pricing arbitrase (R2 = no egress).
- DB right-size + pgbouncer.
- Otomatis arsipkan data lama ke cold tier (analytics_events >24 bulan).

---

## 15. Release Anti-Patterns

Tolak:

- "Big bang" release menumpuk perubahan 1 minggu.
- Deploy Jumat sore.
- Hotfix tanpa test.
- Rilis sambil canary di-skip karena "ini perubahan kecil".
- Flag yang dilupakan setelah GA.
- Deploy tanpa announcement bila memengaruhi customer enterprise.
- Skema migration besar tanpa expand→contract.
- Mendelegasikan keputusan rollback ke individu non-on-call.

---

## 16. Release Quality Gates per Phase

| Phase | Gate |
|-------|------|
| Phase 1 | CI hijau + smoke E2E lulus sebelum tag release |
| Phase 2 | + perf smoke lulus (no regression >10%) |
| Phase 3 | + canary 30 menit observasi sebelum 100% |
| Phase 4 | + chaos test mingguan hijau, error budget tracker aktif |
| Phase 5 | + SOC 2 controls aktif, restore drill lulus quarter terakhir |
| Phase 6 | + status page public, SLA monitoring otomatis |

---

*Dokumen ini melengkapi `05_BUILD_DEPLOY_PLAN.md`, `09_TOP_GLOBAL_FEATURES.md` §5, dan `14_OBSERVABILITY_AND_INCIDENTS.md`. Bila conflict, dokumen ini menang untuk kebijakan rilis & budget performa.*
