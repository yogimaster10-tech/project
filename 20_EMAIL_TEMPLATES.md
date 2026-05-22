# TESKEL — Email Template Library
## Every Transactional, Lifecycle, and System Email TESKEL Sends

> Email adalah produk juga. Setiap template di sini punya pemicu, audiens, kontrak data, dan kontrak visual yang seragam.

---

## 0. Stack

- Provider: **Resend** untuk pengiriman.
- Renderer: **React Email** components di package `@teskel/email`.
- Fallback: text/plain auto-generated dari MJML-like component.
- Transport: setiap email lewat queue → worker idempoten → Resend.
- Test inbox: `inbucket` di local; `MailHog` di CI.

Setiap email punya kontrak TypeScript:

```typescript
export interface EmailContract<Payload> {
  slug: string;
  subject: (p: Payload, locale: string) => string;
  preheader: (p: Payload, locale: string) => string;
  render: (p: Payload, locale: string) => React.ReactNode;
  textFallback: (p: Payload, locale: string) => string;
  tags: { template: string; org_id?: string; user_id?: string };
}
```

---

## 1. Design System

Layout dasar:

```
[hidden preheader 50-110 char]
[Header logo + tagline kecil]
[Hero block (jika perlu)]
[Body markdown-like (max 90 char/line)]
[Primary CTA (full-width button)]
[Detail block (table/key-value)]
[Secondary action / footer info]
[Footer: address, unsubscribe, help link]
```

Aturan:

- Max width 600 px.
- Background `#FFFFFF` light / `#0B0E14` dark (auto via `prefers-color-scheme`).
- Font fallback: `Inter, "Helvetica Neue", Arial, sans-serif`.
- CTA tombol: `text-base font-medium`, rounded 12 px, padding 12×20.
- Tabular numbers untuk angka.
- Logo terisi dengan SVG inline + PNG fallback 2× retina.

---

## 2. Template List Overview

| # | Slug | Tipe | Audience | Trigger | Required |
|--:|------|------|----------|---------|----------|
| 01 | `auth.verify-email` | Transactional | New user | Sign up | MVP |
| 02 | `auth.magic-link` | Transactional | User | Magic-link request | Phase 1 |
| 03 | `auth.password-reset` | Transactional | User | Reset request | MVP |
| 04 | `auth.suspicious-login` | Security | User | New device/geo | Phase 2 |
| 05 | `auth.mfa-backup-codes` | Security | User | MFA enabled / regenerate | Phase 2 |
| 06 | `org.invitation` | Transactional | Invitee | Team invite | Phase 1 |
| 07 | `org.invitation-accepted` | Notification | Inviter | Invite accepted | Phase 1 |
| 08 | `org.role-changed` | Notification | Member | Role updated | Phase 2 |
| 09 | `creator.onboarding-welcome` | Lifecycle | Creator | After signup | Phase 1 |
| 10 | `creator.first-product-nudge` | Lifecycle | Creator (no product) | Day 2 inactive | Phase 2 |
| 11 | `creator.first-sale-celebration` | Lifecycle | Creator | First paid order | Phase 1 |
| 12 | `creator.payout-scheduled` | Transactional | Creator | Payout queued | Phase 1 |
| 13 | `creator.payout-completed` | Transactional | Creator | Payout sent | Phase 1 |
| 14 | `creator.payout-failed` | Transactional | Creator | Payout failed | Phase 1 |
| 15 | `creator.plan-upgrade` | Transactional | Creator | Plan changed | Phase 2 |
| 16 | `creator.plan-renewal-reminder` | Lifecycle | Creator | 5 days before renew | Phase 2 |
| 17 | `creator.plan-renewal-failed` | Transactional | Creator | Card decline | Phase 2 |
| 18 | `creator.stripe-action-required` | Security | Creator | Stripe Connect requirement | Phase 1 |
| 19 | `creator.sale-notification` | Notification | Creator | New paid order | Phase 1 |
| 20 | `creator.refund-issued` | Notification | Creator | Refund processed | Phase 1 |
| 21 | `creator.weekly-digest` | Lifecycle | Creator | Monday 09:00 local | Phase 3 |
| 22 | `creator.review-received` | Notification | Creator | New product review | Phase 3 |
| 23 | `creator.affiliate-commission` | Notification | Creator | Commission accrued | Phase 3 |
| 24 | `creator.api-key-created` | Security | Creator | API key generated | Phase 2 |
| 25 | `creator.api-key-rotation-reminder` | Security | Creator | 30 days before expiry | Phase 4 |
| 26 | `buyer.receipt` | Transactional | Buyer | Payment success | MVP |
| 27 | `buyer.download-ready` | Transactional | Buyer | Delivery available | MVP |
| 28 | `buyer.license-key` | Transactional | Buyer | License generated | Phase 2 |
| 29 | `buyer.subscription-active` | Transactional | Buyer | Sub started | Phase 2 |
| 30 | `buyer.subscription-renewed` | Transactional | Buyer | Renewal success | Phase 2 |
| 31 | `buyer.subscription-payment-failed` | Transactional | Buyer | Renewal fail | Phase 2 |
| 32 | `buyer.subscription-canceled` | Transactional | Buyer | Cancellation | Phase 2 |
| 33 | `buyer.refund-confirmation` | Transactional | Buyer | Refund processed | Phase 1 |
| 34 | `buyer.access-revoked` | Notification | Buyer | Access revoked | Phase 2 |
| 35 | `buyer.community-invite` | Notification | Buyer | Community access | Phase 3 |
| 36 | `buyer.cart-abandoned` | Lifecycle | Buyer | 60 menit cart no checkout | Phase 3 |
| 37 | `buyer.newsletter-confirmation` | Lifecycle | Subscriber | Sign up via popup | Phase 3 |
| 38 | `buyer.product-update` | Lifecycle | Buyer (owner) | Creator new version | Phase 3 |
| 39 | `marketing.launch-announcement` | Marketing | Waitlist | GA | Phase 6 |
| 40 | `marketing.newsletter` | Marketing | Subscriber | Weekly/bi-weekly | Phase 3 |
| 41 | `system.data-export-ready` | Transactional | User | GDPR export | Phase 3 |
| 42 | `system.account-deletion-confirmation` | Transactional | User | GDPR delete | Phase 3 |
| 43 | `system.security-incident-notice` | System | Affected users | Incident | Phase 4 |
| 44 | `system.status-incident-update` | System | Status subscribers | Status page update | Phase 4 |
| 45 | `system.payment-method-expiring` | Transactional | Creator/Buyer | 14 days before exp | Phase 2 |
| 46 | `system.bounce-warning` | System | Creator | High bounce rate | Phase 3 |
| 47 | `system.tos-update` | System | All users | Policy change | Phase 3 |

Implementasi MVP fokus pada label `MVP`/`Phase 1`. Sisanya per fase.

---

## 3. Template Specs (Tier 1 — MVP)

### 3.1 `auth.verify-email`

- **Trigger**: `event.user.registered` saat email belum diverifikasi.
- **Audience**: New user.
- **Subject**: `Verify your email to start using TESKEL`
- **Preheader**: `One click confirms your address. The link expires in 24 hours.`
- **Body sections**:
  1. Greeting `Hi {{first_name}},`
  2. Reason `Confirm this email so we can keep your account safe.`
  3. CTA button `Verify email`
  4. Fallback URL plain text
  5. Help footer: `Didn't sign up? Ignore this email or contact support@teskel.com.`
- **Payload**:

```ts
type Payload = { firstName: string; verifyUrl: string; expiresAt: ISODate };
```

- **Idempotency**: token unik per request; ulang kirim hanya kalau expired.
- **Tracking**: `verify_email_sent`, `verify_email_clicked`.

### 3.2 `auth.password-reset`

- **Subject**: `Reset your TESKEL password`
- **Preheader**: `Link expires in 30 minutes. If you didn't ask, ignore.`
- Body identik pola §3.1 dengan CTA `Reset password`.

### 3.3 `creator.onboarding-welcome`

- **Trigger**: After org provisioning (post §3.1 verified).
- **Subject**: `Welcome to TESKEL, {{first_name}}`
- **Preheader**: `Three steps to your first sale. We'll guide you.`
- **Sections**:
  1. Greeting
  2. Quick start checklist (3 items dengan link)
  3. Founder note
  4. Resources block (docs, community, status page)
- **Footer**: legal + unsubscribe.

### 3.4 `creator.stripe-action-required`

- **Trigger**: Stripe Connect onboarding requirement / refresh.
- **Subject**: `Action needed to receive payouts`
- **Preheader**: `Stripe needs a quick update before your next payout.`
- Body sections: explainer + button `Complete Stripe setup`.

### 3.5 `creator.first-sale-celebration`

- **Trigger**: `event.order.paid` pertama untuk org.
- **Subject**: `🎉 Your first sale on TESKEL`
- **Preheader**: `{{amount}} for {{product_name}}. Here's what's next.`
- Body:
  - Hero illustration.
  - Order summary mini.
  - Next-step list (publish, share, learn).
  - Quote founder atau testimonial.

### 3.6 `creator.payout-scheduled`

- **Subject**: `Payout of {{amount}} scheduled for {{date}}`
- **Preheader**: `Funds usually arrive within 2 business days.`
- Sections: jumlah, currency, destination bank (masked), tanggal, link to dashboard.

### 3.7 `creator.payout-completed`

- **Subject**: `Payout of {{amount}} sent`
- Body: konfirmasi pengiriman + reference id + estimated arrival.

### 3.8 `creator.payout-failed`

- **Subject**: `Payout failed — action required`
- Reason summary + button "Resolve in dashboard".

### 3.9 `creator.sale-notification`

- **Subject**: `New sale: {{product_name}} — {{amount}}`
- **Preheader**: `Customer in {{country}}. Tap to view order.`
- Body: order summary (price, fee TESKEL, net), buyer location (masked), link order detail.
- Frequency: setiap order; bisa di-batch (digest) jika >5 per jam (lihat §5).

### 3.10 `creator.refund-issued`

- Subject + body sederhana + summary.

### 3.11 `buyer.receipt`

- **Subject**: `Your purchase from {{store_name}}`
- **Preheader**: `Receipt for order #{{order_number}}. Total {{amount}}.`
- Body:
  - Greeting `Hi {{first_name}},`
  - Thank-you line
  - Order detail table (line items, fees, total, tax)
  - Download/access CTA jika applicable
  - Help footer (refund, support)
- Wajib mengikuti regulasi penerimaan elektronik (PDF terlampir jika negara mensyaratkan).

### 3.12 `buyer.download-ready`

- **Subject**: `Your download is ready: {{product_name}}`
- **Preheader**: `Link expires in {{expires_human}}.`
- Body: CTA Download, secondary "Access portal", text fallback URL.

### 3.13 `buyer.refund-confirmation`

- **Subject**: `Refund of {{amount}} confirmed`
- Body: jelaskan kapan dana muncul.

---

## 4. Template Specs (Tier 2 & 3)

Detail singkat. Implementasi mengikuti pattern §3 + payload spesifik.

| Slug | Subject Pattern | Audience | Highlights |
|------|-----------------|----------|-----------|
| `auth.magic-link` | `Your TESKEL sign-in link` | User | Single-use link 15 menit |
| `auth.suspicious-login` | `New sign-in from {{city}}` | User | Device, IP masked, "Was this you?" |
| `auth.mfa-backup-codes` | `Save your TESKEL backup codes` | User | List code, simpan offline |
| `org.invitation` | `{{inviter}} invited you to {{org}}` | Invitee | Accept CTA; expires 7 hari |
| `org.invitation-accepted` | `{{name}} joined {{org}}` | Inviter | Profile snippet |
| `org.role-changed` | `Your role in {{org}} is now {{role}}` | Member | Permission summary |
| `creator.first-product-nudge` | `Ready to publish your first product?` | Creator | Checklist visual |
| `creator.plan-upgrade` | `You're on TESKEL {{plan}} now` | Creator | New entitlement |
| `creator.plan-renewal-reminder` | `Your {{plan}} renews on {{date}}` | Creator | Cards summary |
| `creator.plan-renewal-failed` | `We couldn't renew your TESKEL {{plan}}` | Creator | Card update CTA |
| `creator.weekly-digest` | `This week on TESKEL — {{week_range}}` | Creator | KPI strip + top product |
| `creator.review-received` | `New ⭐{{rating}} review on {{product}}` | Creator | Snippet review |
| `creator.affiliate-commission` | `You earned {{amount}} from affiliates` | Creator | Top performers |
| `creator.api-key-created` | `New API key created in {{org}}` | Creator (Owner/Admin) | Scope list + IP |
| `creator.api-key-rotation-reminder` | `Rotate {{key_name}} before {{date}}` | Creator | CTA rotate |
| `buyer.license-key` | `Your license keys for {{product}}` | Buyer | Code blok, activate link |
| `buyer.subscription-active` | `You're subscribed to {{product}}` | Buyer | Renewal info |
| `buyer.subscription-renewed` | `Renewal confirmed — {{amount}}` | Buyer | Next renew |
| `buyer.subscription-payment-failed` | `Update your payment for {{product}}` | Buyer | Grace period info |
| `buyer.subscription-canceled` | `Subscription canceled` | Buyer | Final access date |
| `buyer.access-revoked` | `Access to {{product}} ended` | Buyer | Why + how to restore |
| `buyer.community-invite` | `You have access to {{community}}` | Buyer | Discord/Slack invite |
| `buyer.cart-abandoned` | `You left something in your cart` | Buyer | Product card + CTA |
| `buyer.newsletter-confirmation` | `You're in, {{first_name}}` | Subscriber | Frequency promise |
| `buyer.product-update` | `{{product}} just got better` | Owners | Changelog snippet |
| `marketing.launch-announcement` | `TESKEL is live` | Waitlist | Hero, value props |
| `marketing.newsletter` | `{{benefit_headline}}` | Subscriber | Curated content |
| `system.data-export-ready` | `Your data is ready to download` | User | Link 7 hari |
| `system.account-deletion-confirmation` | `Your TESKEL account has been deleted` | User | Final confirmation |
| `system.security-incident-notice` | `[TESKEL] Security update` | Affected | Plain factual |
| `system.status-incident-update` | `[Status] {{system}} — {{state}}` | Status subs | Summary + status link |
| `system.payment-method-expiring` | `Update your card before {{date}}` | Creator/Buyer | Card last4 |
| `system.bounce-warning` | `Your campaign bounce rate is high` | Creator | Threshold info |
| `system.tos-update` | `Updates to our Terms` | All users | Diff summary |

---

## 5. Sending & Throttling Rules

- Setiap email punya `delivery-channel` (transactional/lifecycle/marketing/system).
- **Transactional**: instant, tidak boleh di-batch atau di-suppress kecuali user delete account.
- **Lifecycle**: tunduk pada preference user (opt-out, frequency cap 4/minggu).
- **Marketing**: only ke user yang opt-in eksplisit; CAN-SPAM + GDPR compliant.
- **System**: kirim ke seluruh user/aff terdampak, tidak bisa di-opt-out untuk security/legal.
- Batching: jika >5 `creator.sale-notification` dalam 1 jam, masuk digest hourly.
- Throttle per recipient: 50 email per hari.

---

## 6. Preference Center

UI di dashboard `/settings/notifications`:

```
□ Sales summary (recommended)
   ○ Instant   ○ Hourly digest   ○ Daily digest
□ Plan & billing
□ Security alerts (cannot opt-out)
□ Product updates
□ Tips & best practices
□ Newsletter
```

Buyer portal `/account/notifications` setup serupa.

---

## 7. Localization

- Subject + body diterjemahkan via ICU files `packages/email/locales/<lang>/<slug>.json`.
- Fallback: EN.
- RTL: layout otomatis flip; CTA tombol kanan-kiri swap.
- Date/time: format locale; sertakan UTC reference jika kritikal (security).

---

## 8. Compliance & Headers

- Setiap email memuat:

  - Physical address (footer).
  - Unsubscribe link (List-Unsubscribe header) untuk lifecycle & marketing.
  - One-click unsubscribe (`List-Unsubscribe-Post: List-Unsubscribe=One-Click`).
  - SPF, DKIM, DMARC (p=reject) dikonfigurasi.

- Wajib: GDPR consent untuk marketing; Article 6(1)(a) basis.
- CAN-SPAM: identitas pengirim jelas; alamat kantor; opt-out diproses 10 hari.
- CASL (Canada): explicit consent + identification + unsubscribe.

---

## 9. Deliverability Practices

- Domain warmup bertahap.
- Pisah domain untuk transactional (`mail.teskel.com`) dan marketing (`updates.teskel.com`).
- Limit bounce rate < 2%, complaint rate < 0.1%.
- Suppress bounced/complaint > 1× otomatis.
- Monitoring: dashboard `email-pipeline` (lihat `14_OBSERVABILITY_AND_INCIDENTS.md` §6).

---

## 10. Testing

- Snapshot HTML/text per template, di-commit.
- Litmus / Email on Acid render test sebelum push template baru.
- Lokal: setiap template wajib bisa di-render via `pnpm email:preview <slug>`.
- E2E `buyer.receipt` divalidasi pada e2e test signup→checkout→delivery.

---

## 11. Versioning

- Slug dipertahankan stabil.
- Perubahan major konten = `slug@v2` (subject berubah signifikan, CTA berubah).
- Versi lama tetap dipertahankan 90 hari untuk audit.

---

## 12. Anti-Patterns

Tolak:

- Email tanpa text fallback.
- CTA tombol berbeda nama di subject vs body.
- Penggunaan `Reply-To: noreply@teskel.com` untuk transactional (gunakan `support@`).
- Single CTA tapi banyak link non-prioritas (max 3 link per email).
- Spam keyword (FREE!!!, BUY NOW).
- Embed gambar tanpa `alt`.
- Trigger ulang pada idempotency yang sama (dedupe per request).
- Mengirim email saat user dalam state `delete-pending`.

---

*Dokumen ini melengkapi `09_TOP_GLOBAL_FEATURES.md` §6 dan `15_SECURITY_AND_COMPLIANCE.md` §13. Bila conflict, dokumen ini menang untuk seluruh email behavior dan content.*
