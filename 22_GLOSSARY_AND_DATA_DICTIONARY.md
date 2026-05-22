# TESKEL — Glossary & Data Dictionary
## One Vocabulary Across Engineering, Product, GTM, and Legal

> Setiap istilah TESKEL punya satu definisi resmi. ID/slug/status enum dicantumkan agar lintas dokumen tidak meleset.

---

## 1. Core Terminology

| Term | Definisi | Catatan |
|------|----------|---------|
| Creator | Individu atau tim yang menjual produk digital melalui TESKEL | Pengguna utama |
| Buyer | Orang/entitas yang membeli produk | Bisa juga jadi creator di org lain |
| Customer | Sinonim Buyer di konteks dashboard creator | Entity di DB: `customers` |
| Organization (Org) | Workspace tempat creator membangun bisnis | Mendukung tim/multi-user |
| Member | Pengguna yang punya role di org | Tabel `organization_members` |
| Storefront | Halaman publik dengan branding org | `{slug}.teskel.com` atau path |
| Marketplace | Direktori publik global di `/explore` | Konten moderasi |
| Embed widget | Komponen JS untuk pasang checkout di domain lain | Distribusi via `js.teskel.com` |
| API key | Token API untuk integrasi server-to-server | Format `tskl_<env>_<random>` |
| SDK | Library `@teskel/sdk` untuk JS/TS | Wrapping API publik |
| CLI | `@teskel/cli` untuk operasi org dari terminal | Local dev tooling |
| Funnel | Urutan langkah checkout/upsell | Tabel `funnels` |
| Discount | Kode promo / diskon dinamis | Tabel `discount_codes` |
| Bundle | Paket multi-produk | Tabel `product_bundle_items` |
| Affiliate | Promotor yang dapat komisi | Tabel `affiliates` |
| License key | Kunci yang membatasi penggunaan software | Tabel `license_keys` |
| Activation | Penggunaan license di device | Tabel `license_activations` |
| Subscription | Akses berulang (monthly/yearly) | Tabel `subscriptions` |
| Delivery | Proses menyerahkan file/akses ke buyer | Tabel `deliveries` |
| Access Grant | Hak akses buyer atas produk/community | Tabel `access_grants` |
| PWYW | Pay-What-You-Want, harga ditentukan pembeli | Min/max boleh diatur |
| Tiered | Produk dengan beberapa level harga | One-time + escalating value |
| Take rate | Persentase fee TESKEL dari transaksi | Variasi per plan |
| Plan | Tier subscription TESKEL untuk creator | Free, Pro, Scale, Enterprise |
| Payout | Pencairan dana ke creator | Tabel `payout_schedules` |
| Refund | Pengembalian dana ke buyer | Subset order |
| Webhook | HTTP callback dari TESKEL ke endpoint creator | Tabel `webhook_endpoints` |
| Sub-processor | Vendor pihak ketiga yang memproses data | Daftar di `/legal/subprocessors` |
| Org tier | Sinonim Plan saat referensi org-level | Hindari pemakaian campur |

---

## 2. Status Enums (Authoritative)

### 2.1 `product.status`

| Value | Definisi |
|-------|----------|
| `draft` | Belum dipublish |
| `active` | Live, bisa dibeli |
| `paused` | Sementara tidak bisa dibeli, masih visible |
| `archived` | Ditarik, link tetap bisa diakses owner saja |

### 2.2 `order.status`

| Value | Definisi |
|-------|----------|
| `pending` | Menunggu konfirmasi pembayaran |
| `paid` | Pembayaran sukses |
| `partially_refunded` | Sebagian refund |
| `refunded` | Refund penuh |
| `failed` | Pembayaran gagal terminal |
| `disputed` | Stripe dispute aktif |
| `canceled` | Dibatalkan sebelum capture |

### 2.3 `subscription.status`

| Value | Definisi |
|-------|----------|
| `trialing` | Dalam trial |
| `active` | Aktif berjalan |
| `past_due` | Renewal gagal, dalam grace period |
| `paused` | Dihentikan sementara |
| `canceled` | Dibatalkan, akses sampai period end |
| `expired` | Periode habis |

### 2.4 `license.status`

| Value | Definisi |
|-------|----------|
| `active` | Valid, bisa diaktivasi |
| `revoked` | Dimatikan oleh creator |
| `expired` | Lewat masa berlaku |

### 2.5 `payout.status`

| Value | Definisi |
|-------|----------|
| `scheduled` | Antri |
| `in_transit` | Diproses Stripe |
| `paid` | Sukses sampai bank |
| `failed` | Gagal, butuh retry |
| `canceled` | Dibatalkan |

### 2.6 `delivery.status`

| Value | Definisi |
|-------|----------|
| `pending` | Sedang disiapkan |
| `ready` | Tersedia |
| `delivered` | Sudah di-download / di-akses |
| `failed` | Gagal generate / tersampaikan |

### 2.7 `webhook_delivery.status`

| Value | Definisi |
|-------|----------|
| `pending` | Belum dikirim |
| `success` | 2xx |
| `retrying` | Sedang retry |
| `failed` | Maksimum retry tercapai |
| `disabled` | Endpoint dinonaktifkan |

### 2.8 `stripe_account.status` (per org)

| Value | Definisi |
|-------|----------|
| `not_connected` | Belum onboarding |
| `pending` | Stripe minta info tambahan |
| `active` | Bisa terima dana & payout |
| `restricted` | Terbatas (mis. butuh document review) |
| `disabled` | Stripe menutup akses |

### 2.9 `marketplace_listing.status`

| Value | Definisi |
|-------|----------|
| `pending` | Menunggu review |
| `approved` | Live di /explore |
| `rejected` | Ditolak moderasi |
| `unlisted` | Dicabut oleh creator |

### 2.10 `org_member.role`

| Value | Akses |
|-------|-------|
| `owner` | Full akses + billing + ownership transfer |
| `admin` | Semua operasi kecuali billing/ownership |
| `editor` | Manage produk, order, customer |
| `analyst` | Read-only + export |
| `viewer` | Read-only |

### 2.11 `currency`

ISO 4217 (mis. `USD`, `IDR`, `EUR`). Lihat `09_TOP_GLOBAL_FEATURES.md` §2.2.

### 2.12 `locale`

BCP-47 (mis. `en-US`, `id-ID`, `pt-BR`).

---

## 3. ID & Slug Format

### 3.1 Prefixed IDs (Stripe-style)

Untuk public-facing & log readability. Internal DB tetap UUID/ULID.

| Entity | Prefix |
|--------|--------|
| User | `usr_` |
| Organization | `org_` |
| Product | `prod_` |
| Price | `price_` |
| File | `file_` |
| Order | `ord_` |
| Order item | `oi_` |
| Customer | `cust_` |
| License | `lic_` |
| Activation | `act_` |
| Discount | `dsc_` |
| Funnel | `fnl_` |
| Affiliate | `aff_` |
| Subscription | `sub_` |
| Delivery | `del_` |
| Access grant | `acg_` |
| Webhook endpoint | `we_` |
| Webhook delivery | `wd_` |
| Email subscriber | `es_` |
| Email sequence | `eseq_` |
| Sequence step | `eseqs_` |
| Marketplace listing | `mlst_` |
| Product review | `rev_` |
| Analytics event | `ev_` |
| API key | `key_` (prefix display); secret `tskl_<env>_...` |
| Payout schedule | `po_` |
| Notification | `ntf_` |
| Notification pref | `ntfp_` |
| PPP config | `ppp_` |
| Search sync | `ssn_` |
| Request | `req_` |
| Trace | `trc_` |

### 3.2 Slugs

- Lowercase, kebab-case.
- 2–60 karakter.
- Char set: `[a-z0-9-]`.
- Tidak boleh berawalan/berakhiran `-`.
- Tidak boleh `--`.

Reserved slug global:

```
api, app, auth, blog, dashboard, docs, embed, help, login, marketplace,
explore, onboarding, pricing, sales, security, settings, signup, status,
support, terms, privacy, legal, www
```

### 3.3 Currency & Number Encoding

- Harga internal disimpan sebagai integer `*_cents` (`cents` walau IDR juga; gunakan smallest unit currency, untuk JPY = yen).
- Konversi via tabel rate (atau live via Stripe) dan cache 15 menit.
- Display: format ICU per locale.

### 3.4 Date/Time

- DB: `timestamptz` UTC.
- API: ISO 8601 with offset, mis. `2026-05-22T10:00:00Z`.
- UI: relative (`2 jam lalu`) untuk recent, absolute untuk distant.

---

## 4. Event Naming

Format: `<domain>.<entity>.<action>` lowercase + dot + snake.

Domain:

- `user`, `org`, `product`, `price`, `file`, `order`, `payment`, `subscription`, `license`, `access`, `delivery`, `email`, `webhook`, `affiliate`, `discount`, `funnel`, `bundle`, `marketplace`, `review`, `apikey`, `payout`, `notification`, `system`.

Actions: `created`, `updated`, `deleted`, `archived`, `published`, `paused`, `resumed`, `started`, `succeeded`, `failed`, `paid`, `refunded`, `revoked`, `activated`, `deactivated`, `exported`, `imported`, `viewed`, `clicked`, `delivered`, `bounced`.

Contoh:

```
order.created
order.paid
order.refunded
subscription.canceled
license.activated
delivery.failed
webhook.delivery.failed
user.email_verified
```

---

## 5. Permission Matrix Summary

Detail di `07_ENGINEERING_DETAILS.md` §3. Ringkas:

| Action | viewer | analyst | editor | admin | owner |
|--------|--------|---------|--------|-------|-------|
| View dashboard | ✓ | ✓ | ✓ | ✓ | ✓ |
| Export data | — | ✓ | ✓ | ✓ | ✓ |
| Manage products | — | — | ✓ | ✓ | ✓ |
| Manage orders & refund | — | — | ✓ | ✓ | ✓ |
| Manage members | — | — | — | ✓ | ✓ |
| Billing & plan | — | — | — | — | ✓ |
| Transfer ownership | — | — | — | — | ✓ |

---

## 6. Plan Entitlements Summary

Detail di `18_LANDING_AND_MARKETING_SITE.md` §3.2. Ringkas:

| Capability | Free | Pro | Scale | Enterprise |
|-----------|:----:|:---:|:-----:|:----------:|
| Storefront | ✓ | ✓ | ✓ | ✓ |
| Marketplace | ✓ | ✓ | ✓ | ✓ |
| Custom domain | — | ✓ | ✓ | ✓ |
| Embed widget | — | ✓ | ✓ | ✓ |
| API access | — | ✓ (limit) | ✓ | ✓ + SLA |
| Email broadcasts | 1k recipients | 10k | 100k | Negotiated |
| Members | 2 | 5 | 25 | Unlimited |
| White-label | — | — | — | ✓ |

---

## 7. Region / Compliance Tags

| Tag | Penanganan |
|-----|------------|
| `EU` | GDPR full, EU VAT support, data residency option |
| `UK` | UK GDPR + ICO |
| `US-CA` | CCPA + CPRA |
| `BR` | LGPD |
| `ID` | UU PDP |
| `JP` | APPI |
| `AU` | Privacy Act |
| `CN` | Tidak dilayani Phase awal |
| `IR` `KP` `SY` | Sanctioned, tidak dilayani |

---

## 8. Audit Log Event Examples

Format `<actor>.<verb>.<resource>` + payload:

```
admin.deleted.product       { product_id, reason }
member.updated.role         { member_id, old_role, new_role }
owner.transferred.ownership { from_user_id, to_user_id }
system.purged.user_data     { user_id, request_id }
creator.exported.orders     { range, format }
```

Audit log immutable, retention 1 tahun online + 7 tahun cold.

---

## 9. Performance Vocabulary

| Term | Definisi |
|------|----------|
| LCP | Largest Contentful Paint |
| INP | Interaction to Next Paint |
| CLS | Cumulative Layout Shift |
| TTFB | Time to First Byte |
| P50/P95/P99 | Percentile latency |
| Burn rate | Konsumsi error budget |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |
| SLA | Service Level Agreement (eksternal) |

---

## 10. Reading Aids

- "MVP" — milestone Phase 1, lihat `11_BUILD_READINESS_AUDIT.md` §7.
- "GA" — General Availability, lihat `17_LAUNCH_AND_GROWTH.md` §1 & §3.
- "ADR" — Architecture Decision Record, lihat `12_ENGINEERING_EXCELLENCE.md` §5.
- "DoD" — Definition of Done, lihat `12_ENGINEERING_EXCELLENCE.md` §1.
- "RACI" — Responsible, Accountable, Consulted, Informed (peran dalam project management).
- "BCP/DR" — Business Continuity Plan / Disaster Recovery, lihat `15_SECURITY_AND_COMPLIANCE.md` §20.

---

## 11. Anti-Patterns

Tolak:

- Memperkenalkan term baru tanpa entry di glosarium.
- Status enum dengan nilai berlainan antar tabel (selalu konsisten).
- ID tanpa prefix di endpoint publik.
- Slug yang melanggar reserved word.
- Currency yang disimpan sebagai float.
- Event naming yang berantakan (mis. `OrderCreated`, `order_created`, `order.create` di pilih sembarangan).

---

*Dokumen ini melengkapi `02_DATABASE_SCHEMA.md`, `03_API_SPEC.md`, dan `07_ENGINEERING_DETAILS.md`. Bila conflict, dokumen ini menang untuk definisi term, enum value, prefix ID, dan event name.*
