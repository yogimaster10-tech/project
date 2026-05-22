# TESKEL — Brand, Voice & Copy Library
## Sound the Same Everywhere

> Semua copy publik TESKEL — landing, dashboard, email, error, push — bersuara konsisten. Dokumen ini menetapkan brand identity, voice/tone, dan library copy siap pakai.

---

## 1. Brand Essence

- **Mission**: Menjadikan menjual produk digital semudah menulis tweet.
- **Vision**: Setiap creator di mana pun memiliki commerce stack yang setara perusahaan terbesar.
- **Promise**: Stack yang lengkap, cepat, jujur, dan dimiliki bersama creator.

Tagline kandidat:

1. *The Operating System for Digital Commerce.* (default)
2. *Sell anything digital. Get paid everywhere.*
3. *Commerce primitives for builders and creators.*

---

## 2. Voice & Tone

### 2.1 Voice (selalu)

- **Pragmatis** — fokus hasil, bukan janji.
- **Ringkas** — kalimat pendek, kata-kata sederhana.
- **Tech-native** — boleh teknis, tapi tidak intimidating.
- **Inklusif** — semua creator, semua negara.
- **Jujur** — tidak menjual mimpi yang tidak bisa kami penuhi.

### 2.2 Tone Modulasi

| Konteks | Tone |
|---------|------|
| Landing hero | Confident, sharp |
| Pricing | Transparent, plain-spoken |
| Onboarding | Encouraging, low-friction |
| Error message | Calm, owning, instructive |
| Incident communication | Honest, factual, no spin |
| Marketing email | Helpful, value-first |
| Sales email | Direct, business-like |
| Changelog | Engineering-proud, brief |
| Customer success | Warm, empathetic |
| Press release | Formal, KPI-led |

### 2.3 What We Avoid

- Hype words: "revolutionary", "game-changing", "10x".
- Emoji bombs di copy formal.
- "We're sorry for the inconvenience" tanpa konteks.
- Idiom culture-specific yang sulit diterjemahkan.
- Klaim privasi tanpa bukti ("100% private", "we never store anything").

---

## 3. Vocabulary

### 3.1 Always Use

| Use | Don't Use |
|-----|-----------|
| Creator | User (kecuali konteks teknis) |
| Buyer / Customer | Visitor (kecuali pre-checkout) |
| Storefront | Shop page (kecuali blog/SEO) |
| Marketplace | Mall / Directory |
| Order | Purchase (kecuali blog) |
| Payout | Withdrawal |
| Plan | Subscription tier |
| Org / Workspace | Account (gunakan untuk personal user) |
| Webhook | Hook |
| API key | Token (untuk auth bearer ya) |
| Refund | Cancellation (refund spesifik) |

### 3.2 Tone in Numbers

- Currency: format ISO (`$19.00`, `Rp 199.000`, `€19,00`).
- Persen: `3%`, bukan `0.03`.
- Rentang: `2–5 hari`, bukan `2-5 days`.
- Singkatan: `2 mnt baca`, `5K creators`, `$1.2M GMV`.

### 3.3 Capitalization

- Title case: nav, button, page title, modal title.
- Sentence case: body, helper, microcopy.
- All caps: hanya badge kecil ("BETA", "NEW") max 6 char.

---

## 4. Color Tokens

Lihat detail di `09_TOP_GLOBAL_FEATURES.md` §1.1 & `10_UIUX_MODERN_CLEAN.md` §6.

Ringkas wajib:

| Token | Light | Dark | Pemakaian |
|-------|-------|------|-----------|
| `primary` | `oklch(0.55 0.18 264)` | `oklch(0.72 0.16 264)` | Tombol utama, CTA |
| `accent` | `oklch(0.95 0.04 264)` | `oklch(0.25 0.04 264)` | Hover bg, badge subtle |
| `success` | `oklch(0.70 0.17 145)` | `oklch(0.78 0.16 145)` | Status active, confirmation |
| `warning` | `oklch(0.80 0.16 80)` | `oklch(0.78 0.16 80)` | Limit dekat, attention |
| `destructive` | `oklch(0.55 0.22 27)` | `oklch(0.70 0.20 27)` | Delete, error |
| `muted-foreground` | `oklch(0.45 0 0)` | `oklch(0.65 0 0)` | Helper text |

Aturan:

- Maks 2 warna brand utama per halaman (di luar netral & state).
- Status warna selalu disertai ikon (untuk a11y color-blind).

---

## 5. Logo & Mark Usage

- Mark primary: monogram "T" geometric.
- Wordmark: "teskel" lowercase.
- Clear space: minimum 0.5× tinggi mark di semua sisi.
- Minimum size: 16 px mark, 60 px wordmark.
- Boleh di atas: putih, dark background, abu netral.
- Dilarang di atas: gambar low contrast, pattern padat, warna saturated penuh.
- File: `logo.svg`, `logo-dark.svg`, `mark.svg`, `mark-dark.svg`, `og-default.png`.

---

## 6. Typography

- Display: **Inter Variable** (subset latin + latin-ext).
- Mono: **JetBrains Mono Variable**.
- Heading scale: 12 / 14 / 16 / 18 / 20 / 24 / 30 / 36 / 48 / 60 (px).
- Body line-height: 1.5; heading 1.2.
- Tabular nums on every numeric display (`font-variant-numeric: tabular-nums`).

---

## 7. Photography & Illustration

- Photography: real creators, candid, natural light, diverse representation.
- Illustration: minimal geometric, gradient subtle, max 3 warna brand.
- No stock photo cliché (people pointing at laptop).
- No AI-generated photo people tanpa tagging "Illustration".

---

## 8. Iconography

- Library default: Lucide icons.
- Style: 1.5 px stroke, rounded join.
- Sizes: 12 / 14 / 16 / 20 / 24.
- Tidak boleh emoji untuk navigation/state icon.
- Pakai emoji hanya pada konten user-generated, marketing celebratory, atau changelog playful (max 1 per paragraf).

---

## 9. Motion

- Default ease: `cubic-bezier(0.22, 1, 0.36, 1)`.
- Duration scale: 100 / 150 / 200 / 300 / 500 ms.
- Heavy motion (pop, scale): hanya untuk affordance feedback, bukan dekorasi.
- Reduced-motion: matikan parallax, autoplay, decorative animation.

---

## 10. Copy Patterns

### 10.1 Button Labels

- Verb + noun jelas: `Save changes`, `Create product`, `Send invite`.
- Hindari: `Click here`, `Submit`, `OK` (kecuali konfirmasi sederhana).
- Pakai future-state untuk progressive: `Saving…`, `Sending…`, `Connecting…`.
- Destructive: `Delete product` + dialog konfirmasi (`This can't be undone.`).

### 10.2 Empty States

Pola:

```
[icon]
H2: <Apa yang belum ada>
P: <Penjelasan singkat (1 kalimat)>
CTA: [Verb action]   [Learn more]
```

Contoh "Products":

```
H2: No products yet
P: Create your first downloadable product, license, or service.
CTA: [Create product]   [See examples]
```

### 10.3 Loading States

- Skeleton tetap utama (lihat `10_UIUX_MODERN_CLEAN.md` §4.2).
- Loading copy yang bersifat suspense lama:

```
- Generating preview…
- Crunching numbers…
- Sending receipt…
```

Hindari "Please wait" kosong.

### 10.4 Error States (UI)

Pola:

```
[icon error]
H2: Something blocked this action
P: <Apa yang terjadi (factual)> <Apa yang harus dilakukan user>
CTA: [Try again]   [Contact support]
```

Contoh "File upload failed":

```
H2: Upload didn't finish
P: The connection dropped at 64%. Resume to keep the upload where it left off.
CTA: [Resume upload]   [Cancel]
```

### 10.5 Success Confirmation

- Toast singkat (max 70 char): "Product saved.", "Invite sent to {{email}}.".
- Modal sukses panjang hanya untuk milestone besar (first sale).

### 10.6 Destructive Confirmation

```
Modal Title: Delete <noun>?
Body: This will permanently remove <noun>. <Side effect 1 sentence.>
Checkbox optional: "I understand this can't be undone."
Buttons: [Cancel] [Delete <noun>]
```

### 10.7 Form Helper

- Helper di bawah label menjelaskan format: `https://yoursite.com`, `4–32 characters, lowercase`.
- Pesan sukses inline tidak diperlukan kecuali form panjang.

---

## 11. Microcopy Library

| Konteks | Copy |
|---------|------|
| Sign up button | `Get started — it's free` |
| Login button | `Sign in` |
| Logout | `Sign out` |
| Save state | `Saved · just now` (relative time) |
| Save failed | `Couldn't save. We kept your changes locally.` |
| Connect Stripe (CTA) | `Connect Stripe to receive payouts` |
| Stripe connected | `Stripe connected — Payouts active` |
| Plan upgrade nudge | `Pro users get a custom domain — Upgrade` |
| Hit free cap | `You've reached the $1,000 free monthly cap. Upgrade to keep selling.` |
| 2FA enabled | `Two-factor authentication is on.` |
| Session expired | `For your security, you've been signed out.` |
| Empty marketplace search | `No products match "{{query}}". Try a different term or browse categories.` |
| Empty orders | `Orders will land here as soon as your first sale comes in.` |
| Pending payout | `Payout scheduled for {{date}}. Funds usually arrive within 2 business days.` |
| Refund issued | `Refund of {{amount}} issued. Your customer will see it within 5–10 business days.` |
| Webhook delivery failed | `We've retried 4 times. Update your endpoint to receive future events.` |
| API key copied | `Copied. Store this somewhere safe — we can't show it again.` |

---

## 12. Email Subject Patterns

Lihat detail di `20_EMAIL_TEMPLATES.md`. Pola umum:

| Tipe | Pola | Contoh |
|------|------|--------|
| Transaksional buyer | `[TESKEL] {{verb-noun}}` | `[TESKEL] Your purchase from Yogi Studio` |
| Notifikasi creator | `New {{event}}: {{name}}` | `New sale: Notion Finance Template — $19` |
| Lifecycle | `{{verb-question}}` | `Ready to publish your first product?` |
| Security | `Action required: {{event}}` | `Action required: confirm your new sign-in` |
| Marketing | `{{benefit-headline}}` | `Cut your refund rate by 40% with policy presets` |
| Incident | `[Status] {{system}} - {{state}}` | `[Status] Checkout — Investigating` |

---

## 13. Push & In-App Notification Copy

Pola:

```
TITLE  : ≤ 30 char, lead with subject.
BODY   : ≤ 120 char, actionable verb.
ACTION : 1–2 button max.
```

Contoh:

```
TITLE: New sale on Prompt Pack
BODY: $19 from a customer in Germany. Tap to view order.
ACTIONS: [View] [Snooze 1h]
```

---

## 14. Localization Notes

- Hindari idiom culture-specific.
- Tanggal: format ISO untuk konten teknis (`2026-05-22`), lokal untuk UI (`May 22, 2026` EN, `22 Mei 2026` ID).
- Mata uang: ikuti locale + symbol resmi.
- Pluralization: gunakan ICU `{count, plural, one {# product} other {# products}}`.
- Variable: `{{first_name}}`, `{{product_name}}`, hindari interpolasi gramatikal kompleks.

---

## 15. Accessibility Copy

- Setiap ikon-only button punya `aria-label`.
- Image dekoratif: `alt=""`. Image bermakna: deskripsi <120 char.
- Avoid sensory-only references ("Click the green button" — sebutkan label).
- Form label visible; placeholder bukan pengganti label.
- Notifikasi sukses/error juga muncul sebagai `aria-live="polite"` (success) / `assertive` (error).

---

## 16. Press / PR Boilerplate

```
About TESKEL
TESKEL is the operating system for digital commerce. The platform powers
storefronts, marketplaces, embeds, APIs, and white-label commerce for
creators and teams selling digital products in 40+ countries. Founded
in 2026 and headquartered globally-distributed, TESKEL is backed by
[investor list] and serves [N] active creators. Learn more at teskel.com.
```

Boilerplate maks 90 kata.

---

## 17. Approval Workflow

- Microcopy < 30 kata: bisa diputus engineer/PM.
- Page hero / blog post: minimum 1 reviewer marketing.
- Legal / privacy / refund: legal lead WAJIB.
- Press release: founder + marketing lead.

---

## 18. Copy Anti-Patterns

Tolak:

- `Click here`, `Submit`, `Read more` tanpa kontekstual.
- Disclaimer paksa dalam huruf 8 px.
- Title case yang berlebihan ("Click The Button Now To Start").
- Pernyataan yang tidak bisa dibuktikan.
- Permintaan maaf yang tidak menjelaskan apa-apa.
- Slang regional di copy global.
- Penggunaan exclamation mark > 1 per paragraf.

---

*Dokumen ini melengkapi `10_UIUX_MODERN_CLEAN.md` (visual) dan `17_LAUNCH_AND_GROWTH.md` §15 (brand voice). Bila conflict, dokumen ini menang untuk semua keputusan brand, voice, dan copy.*
