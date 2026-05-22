# TESKEL — Security & Compliance Playbook
## Threat Model, Controls, Privacy, and Regulatory Posture

> Keamanan dan privasi bukan fitur tambahan. Mereka adalah default. Dokumen ini menetapkan model ancaman, kontrol, dan playbook regulasi untuk TESKEL.

---

## 1. Security Principles

1. Defense in depth — banyak lapis, asumsikan satu lapis bobol.
2. Least privilege — akses minimum untuk tugas, di-rotate dan di-audit.
3. Secure by default — fitur baru aman tanpa konfigurasi tambahan.
4. Zero trust — verifikasi tiap permintaan, jangan asumsi network internal aman.
5. Audit everything that matters — auth, payments, admin actions, data export.
6. Encrypt everything in motion and at rest.
7. Fail closed for security decisions.

---

## 2. STRIDE Threat Model (Top-Level)

| Threat | Contoh di TESKEL | Mitigasi |
|--------|------------------|----------|
| Spoofing | Login dengan kredensial curian, fake webhook Stripe, fake API key | Better Auth + MFA, signature verification Stripe + Resend, hashed API key + scopes |
| Tampering | Modifikasi harga di client, ubah body checkout | Server-side recompute, Zod validation, signed cart state |
| Repudiation | Klaim "tidak membeli" / "tidak refund" | Audit log immutable, Stripe ledger sebagai source of truth |
| Information Disclosure | Data org A bocor ke org B, leak signed URL | Multi-tenant scoping wajib, short-lived signed URL, RLS Phase 5+ |
| Denial of Service | Spam checkout, scraping marketplace | Rate limit per IP / org / API key, WAF, captcha bila perlu |
| Elevation of Privilege | Viewer mengubah produk, dev key dipakai produksi | RBAC enforced di service, scope API key, env separation, key rotation |

Threat model di-review setiap quarter dan setelah perubahan arsitektur material.

---

## 3. OWASP Top 10 (2021) Coverage Map

| # | OWASP | Mitigasi di TESKEL |
|---|-------|---------------------|
| A01 | Broken Access Control | RBAC eksplisit di service layer; tes multi-tenant wajib; RLS pada Phase 5 |
| A02 | Cryptographic Failures | TLS everywhere; encryption at rest (Neon, R2); password hashing argon2id; secret manager |
| A03 | Injection | Drizzle ORM, no raw SQL kecuali audit; Zod validate sebelum eksekusi; CSP strict |
| A04 | Insecure Design | ADR untuk perubahan kritis; threat model wajib untuk feature payments/auth/data |
| A05 | Security Misconfiguration | IaC review; security headers (Helmet/Hono middleware); environment lockfile |
| A06 | Vulnerable & Outdated Components | Renovate; pnpm audit CI; SCA scan; supply chain pin |
| A07 | Identification & Authentication Failures | Better Auth + MFA; rate-limit login; CAPTCHA pada abuse |
| A08 | Software & Data Integrity Failures | Signed artifacts, dependency pin, CI provenance (SLSA-level 2 target) |
| A09 | Security Logging & Monitoring Failures | Audit log + Sentry + SIEM Phase 5; alert pada login anomaly |
| A10 | Server-Side Request Forgery | Outbound HTTP via allowlist; deny private IP fetch dari user-provided URL |

---

## 4. Authentication Security

### 4.1 User Auth (Better Auth)

- Email + password: argon2id, minimum 12 karakter, weak password check terhadap HIBP API offline mirror.
- OAuth: Google + GitHub; PKCE mandatory.
- MFA: TOTP wajib untuk role Owner/Admin, optional untuk lainnya. Backup codes 10 buah, di-hash.
- Session: signed cookie httpOnly, secure, sameSite=lax, lifetime 14 hari sliding, max 90 hari absolute.
- Logout di semua device: tombol "Sign out everywhere" yang invalidate semua session token.
- Brute-force protection: 5 percobaan gagal dalam 5 menit → soft lock 15 menit + CAPTCHA.
- Suspicious login: IP/geo/device fingerprint berubah → kirim email notifikasi + minta re-MFA.

### 4.2 API Key

- Format: `tskl_<env>_<random base32 24+ chars>`. Stored sebagai hash sha256.
- Scope eksplisit (`products:read`, `orders:write`, dll), default deny.
- Last-used timestamp di-update async.
- Rotasi: API key bisa di-rotate; old key tetap valid 24 jam (grace) lalu di-revoke.
- Display sekali setelah create; tidak boleh ditampilkan lagi.

### 4.3 Service-to-Service

- Antar service internal: mTLS atau signed JWT pendek (5 menit) dengan audience check.
- Tidak boleh menggunakan API key produksi untuk komunikasi internal.

---

## 5. API Security

- Rate limit: per IP, per org, per API key. Public endpoint selalu punya IP-based fallback.
- Request body limit: 1 MB default; raise hanya pada endpoint upload.
- CORS: deny by default; allowlist per origin.
- Security headers:

```
Content-Security-Policy: default-src 'self'; ...
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY  (kecuali embed widget yang explicit allow)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=(), interest-cohort=()
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-site
```

- CSRF: dashboard menggunakan double-submit cookie token; API key endpoint exempt (state ditentukan oleh key).
- Webhook inbound: signature verify (Stripe `Stripe-Signature`, Resend, etc); idempotency key wajib.

---

## 6. Secrets Management

### 6.1 Sumber Truth

- Vault provider: 1Password Connect (early), kemudian HashiCorp Vault / AWS Secrets Manager pada Phase 5+.
- Lokal dev: `.env.local` tidak ter-commit; template di `.env.example`.

### 6.2 Aturan

- Tidak ada secret di repo.
- Tidak ada secret di log / Sentry / analytics.
- Tidak ada secret di issue tracker / Slack tanpa channel khusus akses terbatas.
- Rotasi:

| Secret | Cadence |
|--------|---------|
| Better Auth signing key | 12 bulan |
| Stripe webhook secret | Saat env baru / kompromi |
| API key internal | 6 bulan |
| Database password | 90 hari |
| R2 access key | 6 bulan |
| Resend API key | 6 bulan |

- Setiap rotasi: tested di staging dulu, lalu produksi dalam window low-traffic.
- Setiap secret yang bocor: rotasi <1 jam, postmortem wajib (SEV1 atau SEV2 tergantung dampak).

### 6.3 Detection

- `gitleaks` di pre-commit + CI.
- GitHub Advanced Security secret scanning aktif.
- TruffleHog scan terhadap commit history secara berkala.

---

## 7. Supply Chain Security

- Pin versi pasti pada semua dependency runtime; lockfile committed.
- `pnpm audit` di CI, gate pada high/critical.
- SCA (Snyk / Dependabot) Phase 4+.
- Provenance: tagged release menggunakan signed artifact (npm provenance / Sigstore cosign Phase 5+).
- Build environment: GitHub Actions dengan ephemeral runner, secret terbatas, OIDC ke cloud provider.
- Tidak boleh `npm install` tanpa lockfile; tidak boleh `--no-frozen-lockfile` di CI.

Risk register untuk vendor pihak ketiga: Stripe, Resend, Cloudflare, Neon, Upstash, Typesense, PostHog, Sentry, Vercel, Fly.io. Setiap vendor punya:

- DPA (Data Processing Agreement) ditandatangani.
- SOC 2 atau ISO 27001 cert yang diverifikasi.
- Sub-processor list yang dipantau.

---

## 8. SAST / DAST / Dependency Scan

| Tool | Pemicu | Action |
|------|--------|--------|
| ESLint + `eslint-plugin-security` | Pre-commit + CI | Block warn-as-error |
| Semgrep | CI | Block pada `error` |
| TypeScript strict | CI | Block error |
| pnpm audit | CI | Block high/critical |
| OWASP ZAP baseline | Nightly staging | Slack alert |
| Trivy scan container | Build | Block high/critical |
| gitleaks | Pre-commit + CI | Block |

---

## 9. Penetration Testing

- Internal pen test: setiap minor release oleh tim platform / red-team mini.
- Pen test pihak ketiga: setiap 12 bulan + sebelum SOC 2 audit, sebelum enterprise customer onboard.
- Bug bounty program (HackerOne / Intigriti): aktif setelah Phase 5 / SOC 2 Type I.

---

## 10. Data Classification

| Class | Contoh | Penanganan |
|-------|--------|-----------|
| Public | Marketing copy, landing page | Tidak ada batasan |
| Internal | Konfigurasi, dashboards | Akses internal only |
| Confidential | Identitas creator, isi produk | Encrypted, akses RBAC |
| Sensitive | Email pelanggan, pesanan, kunci lisensi | Encrypted + audit log |
| Regulated | Pajak ID, info pembayaran (token Stripe), data PII GDPR | Encrypted, redaksi log, retention policy ketat |

Semua data minimal Confidential ke atas wajib dienkripsi at-rest dan in-transit.

---

## 11. Data Retention & Deletion

| Data | Default Retention | Hard Delete |
|------|-------------------|-------------|
| Active user/customer profile | Selama akun aktif | 90 hari setelah delete request |
| Orders | 7 tahun (regulasi pajak) | Hard delete bila wajib hukum, anonimisasi setelah retention |
| Audit logs | 1 tahun online, 7 tahun cold | Tidak dihapus selama retention |
| Webhook deliveries | 90 hari | Auto-purge |
| Analytics events | 24 bulan | Auto-purge sesuai region |
| Email content/log | 90 hari | Auto-purge |
| Backup database | 30 hari rolling | Auto-purge |

Permintaan hapus (GDPR/CCPA) diproses melalui job `data.delete.request`:

1. Verifikasi identitas requester.
2. Generate report data yang akan dihapus.
3. Konfirmasi pengguna (mis. via email link 24 jam).
4. Eksekusi delete + anonimisasi + audit entry.
5. Konfirmasi selesai dalam 30 hari kalender (legal SLA).

---

## 12. PII Handling

- Tidak menyimpan kartu kredit. Semua via Stripe.
- Email pelanggan disimpan; ditampilkan masked di analytics dashboard (mis. `j***@gmail.com`).
- IP address di-mask `203.0.113.10 → 203.0.113.0/24` setelah 30 hari (kecuali fraud investigation).
- Foto profil dan upload dari pengguna: scan virus (ClamAV), validasi MIME, simpan di bucket privat.
- Field `name`, `address` di-encrypt dengan KMS (envelope encryption).

---

## 13. GDPR / CCPA Workflows

### 13.1 Hak Pengguna

| Hak | Mekanisme |
|-----|-----------|
| Access (data portability) | Endpoint `/v1/account/export` mengirim email link ke ZIP berisi semua data, JSON + CSV |
| Rectification | Edit di settings + log audit |
| Erasure (right to be forgotten) | Workflow §11 |
| Restrict processing | Toggle di settings `pause-marketing` & `pause-analytics` |
| Object | Tombol opt-out di footer email |
| Automated decision | TESKEL tidak menggunakan automated decision-making yang signifikan; bila ada, opt-in eksplisit |

### 13.2 DPA & Sub-processor

- Setiap creator (data controller) menandatangani DPA dengan TESKEL (data processor).
- Sub-processor (Stripe, Resend, Neon, R2) dipublikasikan di `/legal/subprocessors`.
- Notifikasi 30 hari sebelum penambahan sub-processor.

### 13.3 Breach Notification

- SEV1 yang melibatkan data pribadi:

```
1. Tim hukum & DPO diberitahu <1 jam.
2. Otoritas (mis. ICO/CNIL) <72 jam.
3. Pengguna terdampak <72 jam (bila high risk).
4. Postmortem publik dipublikasi (sanitized) <14 hari.
```

---

## 14. PCI DSS Posture

- TESKEL TIDAK menyimpan/proses PAN (Primary Account Number).
- Semua pembayaran melalui Stripe Checkout / Elements; integration mengikuti SAQ-A.
- Memastikan tidak ada iframe override / XSS yang bisa intercept Stripe Elements.
- Annual self-assessment SAQ-A + attestasi ke partner enterprise.

---

## 15. SOC 2 Readiness

Target: SOC 2 Type I pada Phase 5, Type II pada Phase 6.

Common Criteria yang sudah/akan dipenuhi:

- CC1 Control Environment: kode etik, struktur tim.
- CC2 Communication: dokumen ini + onboarding materi.
- CC3 Risk Assessment: tahunan + ad-hoc per perubahan material.
- CC4 Monitoring: observability + audit trail.
- CC5 Control Activities: SDLC controls (PR review, CI gates).
- CC6 Logical Access: RBAC, MFA, key rotation.
- CC7 System Operations: change management, incident response.
- CC8 Change Management: ADR, code review, release ritual.
- CC9 Risk Mitigation: vendor management, BCP/DR.

Vendor: Vanta / Drata.

---

## 16. Incident Disclosure Policy

- Public disclosure dalam 14 hari kerja untuk insiden yang berdampak pengguna.
- Format: blog post sanitized + status page entry, link ke postmortem internal yang di-redact.
- Pengguna enterprise mendapatkan brief lebih detail dengan NDA.

---

## 17. Abuse Prevention

- Velocity check: lonjakan order/refund/login → flag account untuk review manual.
- Email reputation: monitor bounce/complaint rate; suspend campaign org bila >2% bounce.
- Fraud signals: BIN risk via Stripe Radar; geo mismatch; refund-shop pattern.
- IP reputation: integrasi MaxMind atau Cloudflare bot management.
- CAPTCHA pada signup & checkout untuk traffic tinggi-risiko.
- Marketplace moderation: lihat `11_BUILD_READINESS_AUDIT.md` §4.12.

---

## 18. Secure SDLC Checkpoints

| Checkpoint | Output |
|-----------|--------|
| Backlog grooming | Tag feature dengan label `security-impact: yes/no`; tambah threat model bila yes |
| Design review | ADR + threat model addendum |
| Code review | Checklist `12_ENGINEERING_EXCELLENCE.md` §4 |
| Pre-merge CI | SAST, dependency scan, secret scan, unit/integration test |
| Pre-release | DAST staging, smoke security test, sign-off security lead untuk perubahan high-impact |
| Post-release | Sentry/alerts monitored 24 jam pertama; rollback bila anomaly |

---

## 19. Access Control

- Production access: just-in-time (JIT) via PAM tooling (StrongDM / Teleport) Phase 4+.
- Database production read: read-only role, audit log per query.
- Database production write: forbidden manual — semua via migration / scripts ber-PR.
- Cloud console: SSO + MFA; akses production di-restrict ke role `prod-operator`.
- Quarterly access review (deprovisioning eks-karyawan, vendor, role).

---

## 20. Backup, Continuity, & Recovery

- RPO: 5 menit (Neon PITR).
- RTO: 60 menit (failover region).
- Backup encrypted; restore drill quarterly (lihat `14_OBSERVABILITY_AND_INCIDENTS.md` §15).
- BCP: dokumen `docs/bcp.md` (Phase 5+) mencakup skenario region down, vendor down, ransomware, key compromise.

---

## 21. Security Anti-Patterns

Tolak:

- Menambah role `god` tanpa scope.
- Membuat endpoint admin tanpa audit log.
- Endpoint debug yang aktif di production.
- Logging seluruh request body / response untuk "troubleshooting" tanpa redaksi.
- Storing `process.env` keseluruhan ke log.
- Memberikan akses production "sementara" tanpa PAM / audit.
- Mengganti library kripto dengan implementasi sendiri.
- Mengabaikan signature verification webhook.
- Membypass authorization checks "karena admin pasti boleh" tanpa cek role.

---

## 22. Maturity Roadmap

| Phase | Capability |
|-------|------------|
| Phase 1 | Better Auth + MFA optional + Zod + ratelimit |
| Phase 2 | Audit log + security headers + secret scanning |
| Phase 3 | DAST + dependency scan + threat model awal |
| Phase 4 | Pen test internal + Bug bounty private + SOC 2 prep |
| Phase 5 | SOC 2 Type I + pen test external + JIT access + RLS production |
| Phase 6 | SOC 2 Type II + ISO 27001 prep + bug bounty public |

---

*Dokumen ini melengkapi `06_INFRA_SECURITY_DEVOPS.md` §2 dan §6. Bila conflict, dokumen ini menang untuk kebijakan keamanan/privasi/regulasi, sementara `06_INFRA_SECURITY_DEVOPS.md` menang untuk arsitektur jaringan & cloud.*
