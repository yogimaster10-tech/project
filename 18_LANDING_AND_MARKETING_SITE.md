# TESKEL — Landing & Marketing Site Specification
## Every Public Page That Sells, Educates, and Earns Trust

> Setiap halaman publik TESKEL ditulis sekali, dipakai berulang. Spec ini menetapkan route, struktur, copy direction, komponen, SEO, dan performa untuk seluruh marketing surface.

---

## 0. Site Map

```
/
/pricing
/features
/features/[slug]
/customers
/customers/[slug]
/changelog
/changelog/[slug]
/docs
/docs/[section]
/docs/[section]/[slug]
/blog
/blog/[slug]
/about
/careers
/careers/[slug]
/contact
/security
/legal/terms
/legal/privacy
/legal/dpa
/legal/cookie-policy
/legal/refund-policy
/legal/subprocessors
/legal/acceptable-use
/api/sitemap.xml
/api/robots.txt
/404
/500
```

External links (tetap dijaga konsistensi visual):

- `status.teskel.com` — public statuspage
- `dashboard.teskel.com` — app dashboard
- `docs.teskel.com` — boleh subdomain atau path, lihat §13

---

## 1. Global Marketing Layout

### 1.1 Structure

```
<MarketingLayout>
  <PromoBanner />          (opsional, dismissible, A/B-able)
  <TopNav />
  <main>{children}</main>
  <Footer />
  <CookieConsent />
</MarketingLayout>
```

### 1.2 Top Nav

```
[TESKEL logo]  Features ▾   Solutions ▾   Pricing   Customers   Docs   Changelog        [Sign in] [Get Started →]
```

- Sticky on scroll dengan blur background `bg-background/80 backdrop-blur`.
- Mega menu untuk Features & Solutions (lihat §2.5).
- Mobile: hamburger → drawer.

### 1.3 Footer

7 kolom max (desktop), reflow ke 2 di tablet, accordion di mobile.

```
Product             Solutions            Resources         Developers       Company        Legal              Stay Updated
- Features          - Creators           - Blog            - API docs       - About        - Terms            - Email input
- Pricing           - Agencies           - Customer        - SDK            - Careers      - Privacy          - "Join 4k+"
- Marketplace       - SaaS               - Help center     - CLI            - Press kit    - DPA              - Social icons
- Embed             - Communities        - Changelog       - Embed          - Brand        - Cookies          
- API               - Education          - Security        - Status         - Contact      - Subprocessors    
```

Bottom strip: `© 2026 TESKEL, Inc. — Made for creators, everywhere.` + region selector (currency/lang) + theme toggle (light/dark).

### 1.4 Promo Banner

- Dipakai untuk: announcement launch, Black Friday, product update.
- Max 70 karakter copy + 1 CTA chip.
- Dismissible (cookie 14 hari).
- Tinggi 36 px desktop / 44 px mobile.

---

## 2. `/` — Home / Landing

### 2.1 Goal

Mengubah pengunjung baru → signup (primary) atau request demo (enterprise secondary).

### 2.2 Section Order

1. Hero
2. Logo Wall (social proof)
3. Problem framing
4. Product modes (5 use cases)
5. Feature matrix (13 product types)
6. Live demo / interactive
7. Stripe Connect & payouts strip
8. Marketplace teaser
9. Developer block (SDK + API + Embed)
10. Pricing snapshot
11. Testimonials
12. Security & compliance bar
13. Final CTA

### 2.3 Section Details

**Hero**

```
H1: The Operating System for Digital Commerce.
SUB: One platform. 13 product types. Stripe Connect, instant payouts,
     marketplace, API, embed widget. Free to start.
CTA: [Get Started Free →]   [Watch 2-min demo]
HERO VISUAL: animated mock dashboard with live orders ticking
```

- Above-the-fold LCP image ≤ 100 KB AVIF/WebP.
- Background subtle radial gradient (light/dark).
- Trust seal: `SOC 2 Type II in progress · GDPR ready · PCI SAQ-A`.

**Logo Wall**

12 logo grayscale; `Used by indie builders, agencies, and creators in 40+ countries.`

**Problem Framing**

3 kolom:

- "Selling digital is fragmented" — Gumroad bundle, Lemon checkout, Stripe link.
- "Operating is painful" — refunds, license, delivery, tax.
- "Scaling means rebuild" — multi-currency, embed, white-label.

**Product Modes** (5 cards)

| Mode | One-liner | CTA |
|------|-----------|-----|
| Marketplace | Be discovered on `/explore` | "Browse marketplace" |
| Storefront | Your own branded `you.teskel.com` | "See an example" |
| Embed | Sell on your own site | "Try embed" |
| API & SDK | Build your own commerce experience | "Read docs" |
| White-label | Power your platform's commerce | "Talk to sales" |

**Feature Matrix**

Grid 4×4 menampilkan 13 product types (icon + nama), 3 sisanya untuk "+ more soon".

**Live Demo / Interactive**

Embed-able mock checkout — pengunjung bisa klik produk demo, pilih harga, lihat success page (fake) tanpa daftar. Tetap punya CTA `Try with your own product`.

**Stripe Connect Strip**

`Powered by Stripe Connect — payouts in 45+ countries. Tax + receipts handled. Refunds in one click.`

**Marketplace Teaser**

Carousel 6 produk featured (real dari production setelah Phase 5; sebelum itu pakai curated demo).

**Developer Block**

3 kolom: API, SDK, Embed. Tiap kolom kode sample 5 baris + tombol `Copy`.

**Pricing Snapshot**

Tabel 4 tier (Free, Pro, Scale, Enterprise) dengan 1 row utama + link `Compare all features →`.

**Testimonials**

3 quote panjang (avatar + nama + role + org link). Carousel di mobile.

**Security Bar**

```
[SOC 2 logo] [GDPR] [PCI SAQ-A] [Stripe Verified Partner] [Cloudflare] [Neon SOC 2]
```

**Final CTA**

```
H2: Ready to sell anything digital?
SUB: 2-minute setup. Free until you make $1,000 in sales.
CTA: [Create your store →]   [Talk to sales]
```

### 2.4 Behavior

- Scroll-driven animation: hero parallax, feature grid stagger entrance.
- Reduced-motion: animasi dimatikan via `prefers-reduced-motion`.
- A/B variants:
  - Headline A: "The Operating System for Digital Commerce."
  - Headline B: "Sell anything digital. Get paid everywhere."
- Track event: `landing.cta_click`, `landing.section_view{section_id}`.

### 2.5 Mega Menu

- Features: 13 product types ditampilkan dengan 2 kolom + "Browse all features" link.
- Solutions: 5 audience (Creators, Agencies, SaaS, Communities, Education) + "Industry stories".

### 2.6 SEO

- Title: `TESKEL — The Operating System for Digital Commerce`
- Meta description: 155 char.
- Schema.org `Organization`, `WebSite`, `BreadcrumbList`.
- Canonical: `https://teskel.com/`.
- OG image: 1200×630, generated dari template.

### 2.7 Performance Targets

- LCP ≤ 2.0 s mobile 4G.
- INP ≤ 200 ms.
- Bundle initial JS ≤ 150 KB gzipped.

---

## 3. `/pricing`

### 3.1 Layout

```
H1: Simple pricing, fair to creators.
SUB: Pay less as you grow.
PLAN TOGGLE: [Monthly] [Yearly — save 20%]
GRID: 4 plan cards (Free, Pro, Scale, Enterprise)
COMPARISON TABLE: full feature matrix (sticky header)
USAGE CALCULATOR: pricing simulator with sliders (GMV, products, members)
FAQ: 10 entries
TESTIMONIAL: 2 quotes
CTA: secondary CTA strip
```

### 3.2 Plan Cards

| | Free | Pro | Scale | Enterprise |
|--|------|-----|-------|-----------|
| Price | $0 | $19/mo | $99/mo | Custom |
| Take rate | 5% | 3% | 1.5% | 0.5% |
| Sales cap | $1k/mo | Unlimited | Unlimited | Unlimited |
| Storefront | ✓ | ✓ | ✓ | ✓ |
| Marketplace | ✓ | ✓ | ✓ | ✓ |
| Custom domain | — | ✓ | ✓ | ✓ |
| Embed widget | — | ✓ | ✓ | ✓ |
| API access | — | Limited | Full | Full + SLA |
| White-label | — | — | — | ✓ |
| Support | Community | Email 24h | Priority 8h | Dedicated CSM |

### 3.3 Calculator

Inputs: estimated monthly GMV, number of products, team seats.
Output: total cost (subscription + take rate), savings vs competitor (Gumroad).

### 3.4 FAQ

Topik wajib:

- Apakah ada biaya tersembunyi?
- Bisakah upgrade/downgrade kapan saja?
- Apakah TESKEL menyimpan informasi kartu?
- Berapa lama payout?
- Apakah ada free trial untuk Pro/Scale?
- Bagaimana refund ditangani?
- Bisa keluar (export data) kapan saja?
- Apakah Pro plan punya bandwidth limit?
- Bagaimana dukungan multi-currency?
- Apakah ada diskon non-profit / education?

### 3.5 SEO & Tracking

- Schema.org `Product`, `Offer`.
- Track event: `pricing.plan_click{plan}`, `pricing.calculator_input`.

---

## 4. `/features` & `/features/[slug]`

### 4.1 `/features`

Hub page:

- Hero: "Everything you need to ship commerce."
- 13 product type cards (each with 1 hero gif/lottie).
- Feature category nav: Commerce · Payments · Delivery · Growth · Developer · Platform.

### 4.2 `/features/[slug]`

Template:

```
<HeroFeature />           (problem → solution one-liner)
<KeyCapabilityList />     (3–5 bullets)
<Screenshots />           (3 max)
<UseCases />              (3 personas)
<Pricing tier badge />    (di tier mana fitur ini)
<RelatedFeatures />       (3 cards)
<CTA />
```

Slug daftar awal:

- `download-products`
- `licenses`
- `prompt-packs`
- `micro-saas`
- `services`
- `communities`
- `courses`
- `bundles`
- `pay-what-you-want`
- `tiered-access`
- `links`
- `collections`
- `agents`
- `marketplace`
- `embed-widget`
- `api-and-sdk`
- `funnels`
- `email`
- `affiliates`
- `analytics`
- `multi-currency`
- `white-label`

---

## 5. `/customers` & `/customers/[slug]`

### 5.1 Hub

- 12 case studies grid (logo + headline KPI: "+34% revenue in 60 days").
- Filter: industry, plan, region.

### 5.2 Detail

Story arc:

1. Hero quote + creator avatar.
2. Background (siapa & apa mereka jual).
3. Challenge.
4. Why TESKEL.
5. Results (3 KPI cards).
6. Quote besar.
7. CTA `Start your store`.

Schema.org `Article` + `Person` untuk quote.

---

## 6. `/changelog` & `/changelog/[slug]`

- Hub: timeline (year + month grouping).
- Setiap entry: tanggal, semver, headline, body markdown, screenshot/GIF, attribution author.
- RSS feed `/changelog/rss.xml` + atom.
- Section toggle: All · Product · Developer · Security.
- Subscribe form: email atau RSS.

---

## 7. `/docs`

### 7.1 Information Architecture

```
Getting Started
  Quickstart
  Concepts
  Installation
  Setup checklist

Building
  Products
  Storefront
  Marketplace
  Funnels
  Email
  Discounts
  Affiliates
  Bundles
  Community

Developers
  API reference
  SDK
  CLI
  Embed widget
  Webhooks
  Rate limits
  Idempotency

Operations
  Account
  Payouts
  Tax
  Refunds
  Security
  Compliance
  Backups

Resources
  Migration guides
  Cookbook
  FAQ
  Troubleshooting
```

### 7.2 Layout

Three-column:

- Left: sidebar TOC.
- Center: content (max-w-3xl prose).
- Right: page outline + last updated + "Edit on GitHub".

Komponen wajib:

- Callout (note, warning, tip, danger).
- Code tabs (TS, JS, curl, Python).
- API explorer (try it now) — Phase 3+.
- Snippet `<Copy />`.
- Versioning badge.
- Search (Cmd+K) — Algolia DocSearch.

### 7.3 SEO

- Schema.org `TechArticle`.
- Breadcrumb yang konsisten dengan IA.
- Canonical untuk i18n.

---

## 8. `/blog` & `/blog/[slug]`

### 8.1 Hub

- Hero post (featured) + 8 cards.
- Filter kategori: Engineering, Growth, Customer Stories, Product Updates.
- Newsletter signup di bawah grid.

### 8.2 Post Layout

```
<Breadcrumb />
<Category />
<H1 />
<AuthorMeta /> (avatar, name, role, date, reading time)
<HeroImage />
<TOC />        (sticky for long posts)
<MdxContent />
<ShareBar />
<RelatedPosts />
<NewsletterCTA />
```

Komponen MDX yang tersedia:

- `<Callout />`, `<Pullquote />`, `<CodeBlock />`, `<Compare />`, `<Embed type="tweet|youtube|figma" />`, `<Chart />`.

Schema.org `BlogPosting` + `Person` author.

---

## 9. `/about`

Sections:

1. Mission statement (1 paragraf).
2. Story (3 paragraf, foundational moment).
3. Values (5 cards).
4. Team grid (foto + role + 1 sentence).
5. Investors (logo wall opsional).
6. Press mention (logo wall).
7. CTA: `We're hiring →` / `Talk to us →`.

---

## 10. `/careers` & `/careers/[slug]`

### 10.1 Hub

- Mission statement singkat.
- Operating principles (5 cards) — refer ke `12_ENGINEERING_EXCELLENCE.md` & `17_LAUNCH_AND_GROWTH.md`.
- Benefits grid.
- Open roles list (cards).
- Process timeline.

### 10.2 Detail

```
<JobMeta /> (location, dept, time zone, comp band, equity range)
<AboutRole />
<Responsibilities />
<Qualifications />
<NiceToHave />
<HiringProcess />
<ApplyCTA /> (Greenhouse/Ashby embed)
```

---

## 11. `/contact`

- Form: name, email, company, role, message, region.
- Anti-spam: Cloudflare Turnstile.
- Auto-route: support@, sales@, security@, press@ berdasarkan dropdown.
- Confirmation: instant on-page success + email konfirmasi.

---

## 12. `/security`

Sections:

1. Mission.
2. Compliance badges (SOC 2 status, GDPR, PCI SAQ-A).
3. Encryption (transit + rest).
4. Auth & access (Better Auth, MFA, JIT).
5. Data residency (region availability roadmap).
6. Sub-processors (link ke /legal/subprocessors).
7. Vulnerability disclosure (link ke `security.txt` + email).
8. Pen test report (gated form).
9. Security FAQ.

Referensi ke `15_SECURITY_AND_COMPLIANCE.md`.

---

## 13. `/docs.*` Subdomain or Path

Pilih path (`teskel.com/docs/...`) untuk SEO benefit awal. Migrasi ke `docs.teskel.com` saat traffic dan tim Doc cukup besar (Phase 5+).

---

## 14. Legal Pages

Semua legal page wajib:

- Disertai tanggal "Last updated".
- Tersedia versi PDF download.
- Toggle bahasa (EN/ID minimal).

| Slug | Konten Utama |
|------|--------------|
| `/legal/terms` | Term of service, acceptable use ringkas, dispute resolution |
| `/legal/privacy` | Data yang dikumpulkan, tujuan, sub-processor link, hak GDPR/CCPA |
| `/legal/dpa` | Standard Contractual Clauses, controller-processor, breach notification |
| `/legal/cookie-policy` | Daftar cookie + tujuan + retention |
| `/legal/refund-policy` | Kebijakan refund, perlakuan per product type |
| `/legal/subprocessors` | Tabel sub-processor (nama, peran, lokasi data, SOC 2/ISO link) |
| `/legal/acceptable-use` | Aturan konten (no fraud, no hate, no IP infringement) |

---

## 15. `/404` & `/500`

### 15.1 `/404`

- Illustration (sederhana, on-brand).
- H1: "We can't find that page."
- Search box.
- Useful links (Home, Marketplace, Docs, Status).
- Track event `error.404{path}`.

### 15.2 `/500`

- Illustration.
- H1: "Something went wrong."
- Status page link.
- Retry CTA.
- Hidden incident id (untuk support).

---

## 16. `/api/sitemap.xml` & `/api/robots.txt`

Dynamic sitemap:

- Static page list.
- Blog posts (recent first).
- Customer stories.
- Changelog entries.
- Docs (auto from generated metadata).
- Marketplace top 1k products (post-launch).

Robots:

```
User-agent: *
Allow: /
Sitemap: https://teskel.com/api/sitemap.xml
Disallow: /dashboard
Disallow: /checkout
Disallow: /onboarding
```

---

## 17. Marketing Performance Engineering

- Pakai Next.js App Router + Server Components default.
- Hero image: `priority`, AVIF/WebP, `sizes` proper.
- Lottie: lazy-load + reduced-motion fallback.
- Embedded YouTube: `youtube-nocookie` + thumbnail click-to-load.
- Fonts: variable Inter / IBM Plex, subset latin + latin-ext, `font-display: swap`.
- Critical CSS dipertahankan via Tailwind purge.
- Page weight target: ≤ 600 KB total assets (gzipped) per page marketing.

---

## 18. Tracking & Experiments

- Analytics provider: PostHog (lihat `09_TOP_GLOBAL_FEATURES.md` §4).
- Required events per page:
  - `page_view{path}`
  - `cta_click{cta_id, page}`
  - `section_view{section_id, page}` (intersection observer)
  - `nav_click{item}`
- Experiment toggle via PostHog feature flag.
- Conversion goal: `signup_completed`.

---

## 19. Internationalization

- Awal: EN saja. Architecture siap multi-lang.
- Pakai `next-intl` atau routing `app/[locale]/...`.
- Marketing page wajib siap `/id`, `/es`, `/pt-br`, `/de`, `/fr`, `/ja`, `/ar` (Phase 5+).
- Hreflang & x-default tag siap.

---

## 20. Accessibility Baseline

- Semua landing page lulus `axe-core` tanpa violation `serious`/`critical`.
- Color contrast ≥ 4.5:1 untuk teks normal.
- Keyboard navigation lengkap (Tab, Shift+Tab, Enter, Esc).
- Focus ring visible (CSS variable `--ring`).
- Skip-to-content link di atas Nav.
- Hero animations menghormati `prefers-reduced-motion`.

---

## 21. Marketing Page Quality Gate

Sebelum publish:

- [ ] Copy direview oleh marketing lead.
- [ ] Hero image dioptimisasi (AVIF/WebP + LCP <2s mobile).
- [ ] SEO meta + schema valid (`schema.org` validator).
- [ ] OG image generated & terisi.
- [ ] A11y lint hijau.
- [ ] Performance budget tidak naik >10% baseline.
- [ ] Tracking event terdaftar di analytics dictionary.
- [ ] Tested di Chrome, Firefox, Safari (desktop + mobile).
- [ ] Diuji `prefers-reduced-motion` + dark mode.

---

## 22. Anti-Patterns

Tolak:

- Pop-up modal pada landing < 30 detik kunjungan.
- Video autoplay dengan suara.
- "Limited time offer" yang permanen.
- Klaim tanpa data (referensi source di tooltip).
- Carousel hero (kecuali kasus khusus dengan A/B).
- Form yang minta info berlebih untuk demo (≤ 4 field).
- Dark patterns (default opt-in newsletter, unsubscribe hidden).

---

*Dokumen ini melengkapi `04_FRONTEND_UIUX_SPEC.md` §4.1 dan `10_UIUX_MODERN_CLEAN.md` §3.1. Bila conflict, dokumen ini menang untuk seluruh marketing site spec.*
