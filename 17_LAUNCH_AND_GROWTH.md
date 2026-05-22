# TESKEL — Launch & Growth Playbook
## From Closed Beta to Sustainable Global Scale

> Produk yang hebat tetap mati tanpa rencana go-to-market yang sederajat. Dokumen ini menyusun jalur peluncuran dan loop pertumbuhan yang teruji untuk TESKEL.

---

## 1. Launch Stages

| Stage | Audience | Tujuan | Gate Keluar |
|-------|----------|--------|------------|
| Internal Alpha | Tim TESKEL + invited friend | Smoke produk; validasi flow utama | Tidak ada SEV2 dalam 14 hari, Definition of Done §1 (`12_ENGINEERING_EXCELLENCE.md`) hijau, dan Quality Gate Phase 1 §13 terpenuhi |
| Closed Beta | 20–50 creator under waitlist | Validasi product-market fit, friction discovery | NPS ≥ 30, completion rate signup-to-first-sale ≥ 25%, churn 30-hari ≤ 15% |
| Open Beta | Publik, traffic terbatas | Stress test, conversion calibration | Uptime ≥ 99.9% selama 14 hari, support SLA terpenuhi |
| GA Launch | Publik penuh | Awareness + growth | Kriteria pasca-launch (lihat §3) |
| Scale | Skala global, partnership | Sustainability, expansion | Quarter on quarter growth target tercapai |

---

## 2. Pre-Launch Quality Gate

Wajib dilewati sebelum **Open Beta**:

- [ ] MVP flow end-to-end teruji (lihat `11_BUILD_READINESS_AUDIT.md` §7).
- [ ] SOC 2 readiness inisiasi (Vanta/Drata onboard).
- [ ] Pen test internal hijau (no SEV1/SEV2 open).
- [ ] Load smoke pada 5 endpoint kritikal lulus.
- [ ] Synthetic monitoring 24/7 aktif.
- [ ] Status page publik live.
- [ ] Postmortem template & runbook minimum 10 file siap.
- [ ] Support ticket pipeline siap (lihat §11).
- [ ] Legal: ToS, Privacy Policy, DPA, Refund Policy, Subprocessor list publikasi.
- [ ] Pricing & checkout uji A/B baseline siap.
- [ ] Onboarding 5-langkah lulus moderated UX test (≥ 80% completion).

---

## 3. GA Launch Day Plan

### 3.1 T-7 Days

- Freeze feature merges; bug fix saja.
- Pre-launch comms internal: roles + responsibility.
- Update copy / landing dengan messaging final.
- Smoke E2E lulus 5 hari berturut.

### 3.2 T-1 Day

- Full staging rehearsal: deploy, rollback drill, incident simulation.
- Confirm support team rotation 12-jam.
- Notif vendor (Stripe, Resend, Cloudflare) untuk window tinggi.

### 3.3 Launch Day

- 09:00 local: deploy GA artifact, observe 30 menit.
- 09:30: publish blog post + Product Hunt + Twitter/X + LinkedIn + Hacker News (no automation).
- 10:00: email waitlist (signup CTA + onboarding link).
- Live monitoring crew (3 orang) untuk 8 jam pertama.
- Status update internal setiap 2 jam.

### 3.4 T+24 Jam

- Mini-postmortem: yang berjalan, yang tidak.
- Public update jika ada incident.
- Customer follow-up untuk friend-of-creator referrals.

### 3.5 T+7 Hari

- Review metric: signup, activation, conversion to paid, NPS.
- Adjust onboarding & pricing berdasarkan data.
- Plan next 30 hari (komunikasi, content, partner).

---

## 4. Marketing & PR Alignment

### 4.1 Channel Priority

1. Owned: blog, docs, changelog, newsletter.
2. Community: Discord/Slack, Twitter/X, LinkedIn, Indie Hackers, Product Hunt.
3. Partnerships: indie creator influencers, vertical-specific Newsletters (Notion/AI/Indie SaaS), API directory listings.
4. Paid (post-GA, only if CAC unit economics positif).

### 4.2 Press Kit

- Logo (light/dark/mark).
- Hero screenshots & video.
- Founder bio + headshot.
- Pricing & feature one-pager.
- Quotes & customer stories (post beta).
- Contact form.

### 4.3 Announcement Templates

**Launch tweet/X**

```
TESKEL is live.

The Operating System for digital commerce.

– 13 product types, one platform.
– Stripe Connect, instant payouts.
– Marketplace + storefront + API.

Free tier. No credit card needed.

teskel.com
```

**Launch blog headline**

```
Building TESKEL: A Global-First Commerce OS for the Creator Economy
```

Body memuat: problem, why now, what we built (3 hero capability), customer quotes, pricing, what's next.

---

## 5. North Star Metric

Primary north star: **Monthly Gross Merchandise Volume per Active Creator (MGV/AC)**.

Komponen:

- Activated creator: organisasi dengan ≥1 sale dalam 30 hari.
- Bulanan GMV / activated creator.

Target lintas tahap:

| Phase | MGV/AC bulanan |
|-------|----------------|
| Closed Beta | ≥ $200 |
| Open Beta | ≥ $350 |
| GA | ≥ $500 |
| Scale | ≥ $1,200 |

Pendukung KPIs:

- Activation rate (signup → first sale dalam 14 hari) ≥ 25%.
- 90-day creator retention ≥ 60%.
- Take rate efektif sesuai plan (free 5%, pro 3%, scale 1.5%, enterprise 0.5%).
- Gross margin operasi (excl. payments) ≥ 70%.

---

## 6. Activation Funnel

```
Land     → /  Landing page visitor
Sign Up  → /signup completed
Org      → org created via onboarding
Product  → first product created
Publish  → product published
Visit    → first storefront view (own + organic)
Buy      → first sale
Re-buy   → second sale
Pro upgrade → paid plan
```

Target conversion (median):

| Step | Conversion |
|------|-----------|
| Land → Sign Up | 6–10% |
| Sign Up → Org | 90% |
| Org → Product | 60% |
| Product → Publish | 80% |
| Publish → First Sale | 35% |
| First Sale → Repeat | 50% within 30 hari |

Optimasi setiap drop besar via 6 minggu sprint dedicated.

---

## 7. Retention Cohorts

- Buat cohort weekly creator signup.
- Track aktif (login, edit product, sale) D7, D14, D30, D60, D90.
- Bandingkan dengan cohort sebelumnya; identifikasi feature yang berkorelasi retention positif.
- Aksi: customer success outreach untuk cohort di bawah baseline minggu 2.

---

## 8. Customer Feedback Loops

### 8.1 Sumber

- In-app feedback widget (`?`-icon).
- NPS bulanan via email.
- Public roadmap (Featurebase / Canny).
- Founder weekly 1:1 video call (5 creator).
- Support ticket tags otomatis (clustering bulanan).

### 8.2 Process

- Triage mingguan: top 10 friction → backlog candidate.
- Tema bulanan: 1–2 tema produk untuk diselesaikan.
- Kustomer enterprise: dedicated SA + quarterly business review.

---

## 9. Pricing Strategy

### 9.1 Prinsip

- Free tier substantif (≥ produk pertama, $1k GMV/bulan tanpa fee tetap).
- Take rate menurun seiring upgrade.
- Pro plan: $19/bulan flat + 3% take rate.
- Scale plan: $99/bulan + 1.5% take rate.
- Enterprise: kontrak (mulai $1k/bulan + 0.5% take rate).
- Currency localization (lihat `09_TOP_GLOBAL_FEATURES.md` §2.2 & PPP §2.3 untuk pelanggan emerging market di plan tertentu).

### 9.2 Tes & Iterasi

- A/B uji pricing copy & anchor.
- Eksperimen value-based menu pada Pro (mis. 1k email subscribers free).
- Hindari diskon agresif kecuali kohort awal (closed beta).

### 9.3 Anti-Pattern

- "Hidden fees".
- Trial yang berlanjut otomatis tanpa email reminder 3 hari sebelumnya.
- Migrasi paksa tanpa grandfathering >12 bulan.

---

## 10. Sales-Assisted Motion (Enterprise)

### 10.1 Qualification

Trigger SQL (Sales Qualified Lead):

- Org melewati threshold GMV $50k/bulan.
- Permintaan compliance khusus (SSO, SCIM, DPA tailored).
- Customer dengan compliance regulated (financial, education).

### 10.2 Process

1. SDR outreach → discovery call.
2. Solution Engineer demo (focus integrasi & SLA).
3. Security review (SOC 2, pen test, DPA).
4. Pricing & contract.
5. Onboarding plan (4 minggu) + dedicated CSM.

Output: case study, internal champion mapping, expansion playbook.

---

## 11. Customer Support Readiness

### 11.1 Tools

- Helpdesk: Intercom / Pylon / Plain.
- Knowledge base: docs site `docs.teskel.com`.
- Status page: `status.teskel.com`.

### 11.2 SLA

| Tier | First response | Resolution Target |
|------|----------------|--------------------|
| Free | 48 jam | Best effort |
| Pro | 24 jam | 5 hari kerja |
| Scale | 8 jam | 2 hari kerja |
| Enterprise | 4 jam (24/5), 1 jam (24/7 add-on) | Sesuai kontrak |

### 11.3 Workflow

- Tag ticket dengan area (auth, checkout, delivery, license, billing, account).
- Critical (SEV2+) di-route ke on-call.
- Resolution mingguan diaudit; top 5 root cause → backlog produk.
- Macro library untuk respon standar (≥ 30 macro Phase 4).

---

## 12. Content & SEO Foundation

### 12.1 Content Pillars

1. Build in public — engineering deep-dives.
2. Creator economy playbook — pricing, retention, growth.
3. Operational excellence — tax, payouts, compliance.
4. Comparison & migration guides (Gumroad, Lemon Squeezy, Whop, Sellfy).

### 12.2 SEO Baseline

- Sitemap & robots.txt benar.
- Canonical, OG, Twitter card meta.
- Schema.org: Product, Offer, BreadcrumbList, Organization.
- Core Web Vitals memenuhi budget (`16_RELEASE_AND_PERFORMANCE.md` §10).
- Internal linking architecture: hub-spoke (pricing, marketplace, docs).
- Hreflang setelah multi-language siap.

### 12.3 Editorial Cadence

- 2 posts technical/bulan.
- 1 post growth/bulan.
- 1 customer story/bulan post-GA.
- Changelog otomatis dari release notes.

---

## 13. Partnership & Ecosystem

- Stripe directory listing.
- Notion / Figma / Framer integrasi.
- Affiliate network (PartnerStack) pasca Phase 3.
- API directory: Postman, RapidAPI.
- Open-source spotlight via plugin ecosystem.

---

## 14. International Expansion

- Bahasa prioritas: EN → ID → ES → PT (BR) → DE → FR → JA → AR.
- Pertimbangan: payment methods lokal (lihat `09_TOP_GLOBAL_FEATURES.md` §2.4), kebijakan pajak, content localization.
- Hindari launch global day-1 tanpa support staffing.

---

## 15. Brand Voice

Karakter:

- Pragmatis, ringkas, technical-friendly.
- Tegas tanpa hype.
- Customer-stories driven.
- Inklusif, no dark patterns.

Tidak boleh:

- Klaim "AI-powered" tanpa fungsi nyata.
- Klaim privasi yang tidak bisa dibuktikan.
- Klaim "fastest" tanpa benchmark publik.

---

## 16. Risk & Communication Playbook

| Risk | Mitigasi |
|------|----------|
| Vendor outage (Stripe/Cloudflare) | Status page transparan, kompensasi kredit, fallback komunikasi via email |
| Fraud wave | Auto-freeze risky org, manual review, refund flow yang lancar |
| Misinformasi di sosial media | Respon resmi via blog + status page, screenshot bukti |
| Regulasi baru (mis. EU VAT, US sales tax expansion) | Roadmap update di docs, FAQ, deadline komunikasi 90 hari |
| Major security incident | Lihat `15_SECURITY_AND_COMPLIANCE.md` §13 & 16 |

---

## 17. Team & Org Scaling

| Stage | Team Minimum |
|-------|--------------|
| Closed Beta | 1 founding engineer, 1 product/designer, founder do GTM |
| Open Beta | + 2 engineer, + 1 designer, + 1 ops/support |
| GA | + tech lead, + DevRel/marketing, + sales engineer (enterprise prep) |
| Scale | + sec, + data, + customer success leadership |

Kultur: bias to action, document decisions, no-meetings Wednesday, written-first.

---

## 18. Post-GA Iteration Loop

```
Weekly:
  - Metric review (north star + funnel).
  - Support trend triage.
  - Top 3 customer asks.

Monthly:
  - Cohort retention analysis.
  - Pricing performance.
  - Roadmap re-prioritization.

Quarterly:
  - OKR set & review.
  - Pen test internal + security review.
  - Cost engineering audit.
  - Brand & content audit.
```

---

## 19. Long-Term Bets

Hanya dieksekusi setelah scale terbukti & MVP stabil >12 bulan:

- AI engine (`09_TOP_GLOBAL_FEATURES.md` §4.3).
- Agent commerce / autonomous storefronts.
- Component marketplace (plugin economy).
- Creator banking partner.
- Protocol federation (multi-tenant federation).
- Web3 ownership tokenization (optional, market-driven).

Setiap bet butuh ADR + business case + risk review.

---

## 20. Launch Anti-Patterns

Tolak:

- Launch day yang berbarengan dengan SEV2 yang belum diselesaikan.
- Janji fitur yang belum di-PR.
- Diskon "lifetime" kecuali strategic & limited.
- Spam waitlist tanpa segmentasi.
- Press release tanpa demo dukungan.
- Demo video > 3 menit untuk landing page.
- Onboarding video tanpa caption + transkrip.

---

*Dokumen ini melengkapi `09_TOP_GLOBAL_FEATURES.md`, `11_BUILD_READINESS_AUDIT.md`, dan `16_RELEASE_AND_PERFORMANCE.md`. Bila conflict, dokumen ini menang untuk GTM, growth, dan brand decision.*
