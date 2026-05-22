# TESKEL — Engineering Excellence Playbook
## Senior-Grade Practices for Day-to-Day Engineering

> Standar minimum agar setiap perubahan kode di TESKEL aman, jelas, dan reversible. Berlaku untuk human engineer dan AI agent.

---

## 1. Definition of Done (DoD)

Sebuah unit pekerjaan dianggap selesai HANYA jika seluruh hal berikut benar:

- Kode mengubah perilaku sesuai requirement, tidak lebih.
- `pnpm lint`, `pnpm typecheck`, dan test relevan hijau secara lokal.
- Tes baru / yang relevan ditulis: happy path + minimal 2 failure path.
- Multi-tenant isolation diuji untuk perubahan yang menyentuh data org.
- Semua secret tidak masuk repo, log, atau response.
- Migrasi DB forward-only, sudah diverifikasi pada DB dev.
- Observability ditambah untuk path kritis (metrik/log/trace).
- Dokumentasi yang langsung terkait di-update (`BUILD_STATUS.md`, ADR, README service jika ada).
- PR self-review checklist lulus sebelum minta review.

Jika satu pun gagal, tugas belum selesai.

---

## 2. Code Style and Tooling

### 2.1 Wajib Aktif Pada Repo

| Tool | Tujuan | Konfigurasi |
|------|--------|------------|
| TypeScript strict | Type safety | `strict: true`, `noUncheckedIndexedAccess: true`, `noImplicitOverride: true` |
| ESLint | Linter | Preset `@typescript-eslint/strict-type-checked` + project rules |
| Prettier | Format | 100 col, trailing commas, single quotes JS / double quotes JSX |
| Vitest / Jest | Test runner | Per package |
| Drizzle Kit | Migrasi | Diff-driven, manual review setiap migrasi |
| Husky + lint-staged | Pre-commit | lint, format staged, typecheck affected |
| Commitlint | Pesan commit | Conventional Commits |
| Renovate / Dependabot | Upgrade deps | Weekly minor/patch, manual major |
| Knip | Dead code | Berjalan di CI sebagai warning |
| Syncpack | Versi konsisten antar workspace | Berjalan di CI |

### 2.2 Naming

- File komponen React: `kebab-case.tsx`, ekspor `PascalCase`.
- Hook: `useXxx`, satu hook satu file.
- Service backend: `xxx.service.ts`, eksport named function.
- Schema Drizzle: `tableName` snake_case di DB, `camelCase` di TS.
- Test file: `xxx.test.ts` atau `xxx.spec.ts`, satu file per unit.
- Env var: `SCREAMING_SNAKE_CASE`, dengan prefix sumber (`STRIPE_*`, `R2_*`).

### 2.3 TypeScript Idiom

- Tidak ada `any`. Gunakan `unknown` lalu narrow.
- Tidak ada non-null assertion (`!`) tanpa runtime guard.
- Hindari interface untuk bentuk data utility — gunakan `type`.
- Gunakan `as const` untuk lookup table dan enum literal.
- Pakai `Result<T, E>` discriminated union saat banyak kemungkinan error path.

---

## 3. Git Workflow

### 3.1 Branching: Trunk-Based with Short-Lived Feature Branches

```
main  ── stable, deployable kapan pun, protected
  └── feat/<scope>-<task>     short-lived (<3 hari)
  └── fix/<scope>-<bug>       hotfix sementara hingga merged
  └── chore/<scope>-<task>    non-functional housekeeping
  └── docs/<scope>-<task>     dokumen only
  └── refactor/<scope>-<task> tanpa perubahan perilaku
```

Aturan:

- Branch life cycle maksimum 72 jam aktif.
- Wajib di-rebase ke `main` sebelum merge.
- Tidak ada force-push ke `main` (protected).
- Tag rilis: `vMAJOR.MINOR.PATCH` (semver). Pre-release: `-rc.N`.

### 3.2 Conventional Commits

Format:

```
<type>(<scope>): <subject>

<body — opsional, max 72 char per baris>

<footer — opsional: BREAKING CHANGE, refs>
```

Types: `feat`, `fix`, `perf`, `refactor`, `docs`, `chore`, `test`, `build`, `ci`, `style`, `revert`.

Subject ringkas, imperatif, lowercase, ≤72 karakter.

Contoh:

```
feat(checkout): handle expired stripe session by recreating intent

Verifies session before redirect.  Falls back to fresh session
when expired.  Adds integration test.

Refs: TES-204
```

### 3.3 Pull Request Standards

| Aturan | Wajib |
|--------|-------|
| Ukuran | ≤ 400 baris diff (di luar test, schema generated, lockfile) |
| Lifetime | Reviewed dalam 24 jam kerja |
| Deskripsi | Pakai template (lihat 3.4) |
| Linked issue | Wajib untuk feat/fix |
| Approval | ≥1 untuk small, ≥2 untuk multi-service |
| CI | Wajib hijau sebelum merge |
| Squash merge | Default untuk feature branches |

PR yang lebih besar dari 400 baris HARUS dipecah, kecuali secara teknis tidak mungkin (mis. migrasi schema besar). Saat itu, sertakan rationale.

### 3.4 PR Template

```markdown
## What
Ringkas perubahan dalam 2–3 kalimat.

## Why
Konteks business / bug / regresi.

## How
Pendekatan implementasi, tradeoff, alternatif yang ditolak.

## Verification
- [ ] pnpm lint
- [ ] pnpm typecheck
- [ ] pnpm test --filter=<scope>
- [ ] Tested manually: <skrip / langkah>

## Risk & Rollback
- Risk: <low | medium | high>
- Rollback strategy: <revert commit | feature flag off | data migration backwards script>

## Screenshots / Logs (jika UI / observability)

## Linked Issues
Closes #...
```

---

## 4. Code Review Checklist

### 4.1 Hal Wajib Diperiksa

**Korrectness**
- Apakah requirement terpenuhi seluruhnya tanpa scope creep?
- Apakah edge case (kosong, null, error provider, race condition) ditangani?
- Apakah behavior multi-tenant tetap aman (semua query menyertakan `org_id`)?
- Apakah idempotency dipertahankan pada write endpoint?

**Keamanan**
- Tidak ada secret di code/log.
- Validasi input dengan Zod (atau equivalen) sebelum bisnis logika.
- Authorization explicit, bukan implicit "kalau bisa load berarti boleh".
- Tidak ada SQL injection / template literal dengan input user.

**Performance**
- Tidak ada N+1 query.
- Index pada filter berat.
- Tidak melakukan blocking I/O di middleware kritis (gunakan worker).
- Batas waktu eksekusi reasonable, ada timeout/circuit breaker untuk dependency.

**Reliabilitas**
- Retry policy untuk dependency eksternal (Stripe, R2, email).
- Tidak menelan exception (`catch {}` kosong dilarang).
- Tidak melakukan operasi destruktif tanpa konfirmasi (drop, mass update tanpa scope).

**Observability**
- Path kritis memiliki structured log (`orgId`, `userId`, `requestId`, `latencyMs`).
- Metrik dasar (rate / error / latency) dipasang di route baru.
- Error custom mapping ke Sentry/observability layer.

**Maintainability**
- Naming jelas, fungsi <60 baris, parameter <5.
- Tidak ada copy-paste >10 baris yang bisa dijadikan helper.
- Komentar hanya untuk maksud non-obvious, bukan untuk what.
- Tidak menambah dependency tanpa justifikasi.

**Test**
- Test mencakup perubahan baru.
- Test reproducible, deterministik, tidak bergantung pada wall-clock.
- Test multi-tenant: orgA tidak bisa melihat data orgB.

**UX (jika frontend)**
- Loading skeleton ada.
- Error state ada (lihat `10_UIUX_MODERN_CLEAN.md` §4.3).
- Keyboard accessibility (Tab/Enter/Esc) berfungsi.
- Tidak ada layout shift saat data load.

### 4.2 Cara Memberi Review

- Berikan komentar dalam 4 kategori: `must`, `should`, `nit`, `praise`.
- `must` memblokir merge. `should` boleh ditolak dengan justifikasi. `nit` non-blocking. `praise` opsional tapi disarankan.
- Hindari "saya akan menulis berbeda" tanpa alasan teknis.
- Diskusi panjang >5 round-trip → switch ke voice/pair session, ringkas kesimpulan di PR.

---

## 5. Architecture Decision Records (ADR)

Setiap keputusan arsitektur yang sulit dibalikkan WAJIB punya ADR.

### 5.1 Kapan Bikin ADR

- Memilih library berat (DB, queue, auth provider, payment).
- Memilih pola arsitektur lintas service.
- Mengubah kontrak API publik.
- Mengubah model data utama (RLS, multi-tenant strategy, dll).
- Trade-off keamanan / privasi yang signifikan.

### 5.2 Lokasi dan Format

```
docs/adr/
  0001-use-hono-for-api.md
  0002-better-auth-over-nextauth.md
  0003-r2-for-asset-storage.md
```

Format ADR (MADR-lite):

```markdown
# ADR 0003 — Use R2 for Asset Storage

Date: 2026-05-22
Status: accepted   <!-- proposed | accepted | superseded | deprecated -->

## Context
Apa masalah, constraint, dan asumsi yang relevan?

## Decision
Apa yang dipilih, dalam 1–3 kalimat.

## Alternatives Considered
- Option A: ...
- Option B: ...
- Option C: ...

## Consequences
- Positive: ...
- Negative: ...
- Risk: ...

## References
- Link RFC, benchmark, vendor doc.
```

ADR yang di-supersede TIDAK dihapus. Set `Status: superseded by ADR-00xx`.

---

## 6. Technical Debt Management

### 6.1 Aturan

- Debt yang sengaja diambil HARUS dicatat di `docs/tech-debt.md` saat di-merge.
- Setiap entri punya: judul, dampak, owner, target resolusi, indikator yang akan memaksa resolusi.
- Maksimum 20% effort sprint dialokasikan untuk repayment, dievaluasi mingguan.

### 6.2 Format `docs/tech-debt.md`

```markdown
| ID | Title | Impact | Owner | Created | Target | Resolved |
|----|-------|--------|-------|---------|--------|----------|
| TD-001 | License key uses uuid v4, bukan ULID | Sortable index lemah | @yogi | 2026-05-22 | 2026-07-01 | — |
```

### 6.3 Anti-Pattern Tech Debt

- Membuka "TODO" tanpa ID dan owner.
- Komentar `// FIXME later` tanpa entry di backlog.
- Branch experiment yang menumpuk >2 minggu.
- Library yang masuk repo via copy-paste, bukan dependency.

---

## 7. Dependency Hygiene

- Pin versi semua dependency runtime dengan exact version. Range hanya pada devDependency.
- Renovate / Dependabot menjalankan PR mingguan untuk minor/patch.
- Major upgrade: manual, dengan ADR atau ringkasan di PR.
- Audit `pnpm audit` pada CI; vulnerability `high`/`critical` memblokir release.
- Lockfile WAJIB ter-commit dan ter-merge tanpa konflik (diselesaikan dengan regenerate, bukan edit manual).
- Tidak ada paket yang ditambah hanya untuk satu helper kecil (<50 baris). Tulis sendiri di `packages/shared`.

---

## 8. Documentation Standards

- Setiap package punya `README.md`: tujuan, instalasi, scripts, contoh penggunaan singkat.
- Setiap service punya runbook (lihat `14_OBSERVABILITY_AND_INCIDENTS.md`).
- Setiap public API endpoint terdokumentasi di `03_API_SPEC.md` dan OpenAPI generated.
- Internal docs di-edit melalui PR yang sama dengan kode perubahannya.
- Jangan tulis dokumentasi yang tidak terverifikasi. Setiap snippet code di dokumen WAJIB minimal type-check.

---

## 9. Pair / Mob Programming Triggers

Trigger wajib pair session:

- Perubahan migrasi schema kritis (orders, payments, license).
- Refactor lintas service.
- Implementasi pertama dari domain baru (mis. realtime, marketplace moderation).
- Bug produksi SEV1/SEV2 (lihat `14_OBSERVABILITY_AND_INCIDENTS.md`).
- Onboarding engineer baru (minggu pertama).

Format hemat waktu:

```
2 orang × 1.5 jam fokus → ringkasan keputusan ditulis di PR/ADR.
```

---

## 10. Working with AI Agents

AI agent dianggap sebagai pair partner, bukan junior tanpa supervisi.

### 10.1 Aturan

- Setiap PR yang dihasilkan AI WAJIB melalui code review human untuk perubahan menyentuh: auth, payments, migration, multi-tenant boundary, security headers, secrets.
- AI tidak boleh meng-edit file ADR yang sudah `accepted` tanpa human approval baru.
- AI tidak boleh menambahkan dependency tanpa human konfirmasi.
- AI tidak boleh menjalankan migration destructive (drop column, drop table) tanpa human.

### 10.2 Prompt Discipline

- Tugaskan dengan acceptance criteria, bukan langkah.
- Selalu tegaskan "buat test reproducer dulu sebelum fix".
- Minta diff kecil; bila respons >400 baris, hentikan dan pecah.
- Cek ulang assumption dengan tool/log, jangan percaya klaim tanpa bukti.

### 10.3 Logging Penggunaan AI

Setiap PR yang ditulis AI menyertakan footer:

```
Generated-By: <agent-name>
Reviewed-By: <human>
```

---

## 11. Onboarding Standard

Engineer / agent baru selesai onboarding ketika bisa:

1. Setup lokal: `pnpm install && pnpm dev` jalan dalam <10 menit.
2. Menjalankan test suite: `pnpm test` hijau.
3. Membuat PR docs trivial (typo fix) dan merged.
4. Membaca: `TESKEL_BUILD_INDEX.md`, `00_README_BUILD_ORDER.md`, `08_AUTONOMOUS_BUILD_PLAYBOOK.md`, `11_BUILD_READINESS_AUDIT.md`, file ini.
5. Mendemokan flow MVP (signup → product → checkout → delivery) di staging.

---

## 12. Anti-Patterns (Repeat Offenders)

Tolak tanpa diskusi panjang:

- "Quick fix" yang bypass type-check / lint.
- Force push ke branch shared.
- Commit gabungan (feat + refactor + style + chore) menjadi satu.
- Test yang `expect(true).toBe(true)` untuk "lulus".
- Mock yang memalsukan response tanpa contract.
- Komponen UI 1000+ baris tanpa decomposition.
- Komponen yang memegang state global tanpa boundary.
- Pemakaian `process.env.X` tersebar tanpa dilewatkan validator env.
- Logging secret/PII (bahkan di dev).
- Menulis migration manual SQL tanpa lewat drizzle-kit kecuali ada ADR.

---

## 13. Quality Gates per Phase

| Phase | Quality Gate Minimum |
|-------|---------------------|
| Phase 0 | Scaffold scripts, lint, typecheck, test, build sukses; CI runs |
| Phase 1 | MVP flow ter-test e2e; integration test untuk semua endpoint baru |
| Phase 2 | Test pyramid mencapai target awal (lihat `13_TESTING_STRATEGY.md`); error budget tracker aktif |
| Phase 3 | Observability lengkap (log/metric/trace); on-call rotation draft |
| Phase 4 | Pen test internal lulus; load test memenuhi SLO; runbook semua service |
| Phase 5 | Security audit pihak ketiga, GDPR readiness, postmortem berlatih |
| Phase 6 | GA launch checklist 100% (lihat `17_LAUNCH_AND_GROWTH.md`) |

---

*Dokumen ini melengkapi `00_README_BUILD_ORDER.md`. Bila bertentangan, `00_README_BUILD_ORDER.md` menang untuk perilaku coding low-level, dokumen ini menang untuk proses, review, dan kebijakan engineering.*
