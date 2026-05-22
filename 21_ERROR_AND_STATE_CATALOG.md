# TESKEL — Error & State Catalog
## Every API Error, Every Form Validation, Every UI State

> Satu sumber kebenaran untuk error code, validation message, dan UI state. Engineer dan AI agent menulis sekali, pakai konsisten di mana saja.

---

## 0. Conventions

### 0.1 API Error Envelope

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Product not found",
    "details": { "id": "prod_123" },
    "requestId": "req_abc123",
    "docsUrl": "https://docs.teskel.com/errors/RESOURCE_NOT_FOUND"
  }
}
```

- `code` SCREAMING_SNAKE, unik global.
- `message` aman untuk end-user (English default).
- `details` opsional, struktur tergantung domain.
- `requestId` & `docsUrl` selalu ada.

### 0.2 HTTP Status Mapping

| Status | Pemakaian |
|--------|-----------|
| 200 | OK |
| 201 | Created |
| 202 | Accepted (async) |
| 204 | No content |
| 400 | Bad request (parse/validation umum) |
| 401 | Unauthorized (no auth) |
| 403 | Forbidden (auth tapi tidak boleh) |
| 404 | Not found (resource atau route) |
| 409 | Conflict (duplicate, state mismatch) |
| 410 | Gone (resource pernah ada, dihapus permanen) |
| 422 | Unprocessable entity (validasi bisnis) |
| 423 | Locked (subscription suspended) |
| 425 | Too early (resource belum siap, mis. delivery preparing) |
| 429 | Rate limited |
| 500 | Internal server error |
| 502 | Bad gateway (vendor) |
| 503 | Service unavailable (maintenance) |
| 504 | Gateway timeout |

### 0.3 Locale & i18n

- Default `en-US`. Locale lain via key di `packages/email/locales`.
- Format placeholder ICU `{var}`.

---

## 1. Auth Errors

| Code | HTTP | Message |
|------|------|---------|
| `AUTH_INVALID_CREDENTIALS` | 401 | The email or password is incorrect. |
| `AUTH_EMAIL_NOT_VERIFIED` | 403 | Verify your email to continue. |
| `AUTH_SESSION_EXPIRED` | 401 | Your session has expired. Please sign in again. |
| `AUTH_MFA_REQUIRED` | 401 | Enter your two-factor code to continue. |
| `AUTH_MFA_INVALID` | 401 | The code didn't match. Try again. |
| `AUTH_MFA_BACKUP_USED` | 401 | That backup code is no longer valid. |
| `AUTH_RATE_LIMITED` | 429 | Too many sign-in attempts. Try again in {minutes} minutes. |
| `AUTH_ACCOUNT_LOCKED` | 423 | This account is temporarily locked for security. |
| `AUTH_OAUTH_FAILED` | 400 | We couldn't complete sign-in with {provider}. |
| `AUTH_PASSWORD_WEAK` | 422 | Choose a stronger password (at least 12 characters). |
| `AUTH_INVITE_INVALID` | 400 | This invite link is invalid or expired. |
| `AUTH_INVITE_EXPIRED` | 410 | This invite has expired. Ask your admin to resend. |

---

## 2. Account / Org Errors

| Code | HTTP | Message |
|------|------|---------|
| `ORG_NOT_FOUND` | 404 | We couldn't find that organization. |
| `ORG_SLUG_TAKEN` | 409 | The handle "{slug}" is already taken. |
| `ORG_INVITE_DUPLICATE` | 409 | This person already has an invite. |
| `ORG_MEMBER_LIMIT_REACHED` | 422 | You've reached the team member limit for your plan. |
| `ORG_OWNER_REMOVAL_BLOCKED` | 422 | The org must always have at least one owner. |
| `ORG_PLAN_DOWNGRADE_BLOCKED` | 422 | Downgrade isn't possible while {reason}. |
| `ORG_ROLE_INSUFFICIENT` | 403 | You need {required_role} permission for this action. |

---

## 3. Product Errors

| Code | HTTP | Message |
|------|------|---------|
| `PRODUCT_NOT_FOUND` | 404 | Product not found. |
| `PRODUCT_SLUG_TAKEN` | 409 | A product with this URL already exists. |
| `PRODUCT_TYPE_LOCKED` | 422 | Product type can't be changed after the first sale. |
| `PRODUCT_PUBLISH_BLOCKED` | 422 | Add a price and a file before publishing. |
| `PRODUCT_ARCHIVED` | 410 | This product has been archived. |
| `PRODUCT_FILE_LIMIT_EXCEEDED` | 422 | The file is larger than your plan allows. |
| `PRODUCT_UNSUPPORTED_FILE_TYPE` | 422 | This file type isn't allowed. |
| `PRODUCT_BUNDLE_REQUIRES_TWO_ITEMS` | 422 | A bundle needs at least two products. |
| `PRODUCT_PRICE_INVALID` | 422 | Set a price greater than zero or mark as Pay-What-You-Want. |
| `PRODUCT_COVER_REQUIRED` | 422 | Upload a cover image to publish. |

---

## 4. Checkout / Order Errors

| Code | HTTP | Message |
|------|------|---------|
| `CHECKOUT_PRODUCT_INACTIVE` | 422 | This product is no longer available. |
| `CHECKOUT_OUT_OF_STOCK` | 422 | This product is out of stock. |
| `CHECKOUT_REGION_BLOCKED` | 403 | This product can't ship to your region. |
| `CHECKOUT_CURRENCY_NOT_SUPPORTED` | 422 | We can't accept payment in this currency yet. |
| `CHECKOUT_PAYMENT_REQUIRED` | 402 | Payment didn't go through. Please try again. |
| `CHECKOUT_CARD_DECLINED` | 402 | Your card was declined. Try another payment method. |
| `CHECKOUT_FRAUD_BLOCKED` | 403 | This purchase was blocked for security reasons. |
| `CHECKOUT_SESSION_EXPIRED` | 410 | This checkout link has expired. Try again. |
| `CHECKOUT_IDEMPOTENCY_CONFLICT` | 409 | Order already processed. |
| `ORDER_NOT_FOUND` | 404 | Order not found. |
| `ORDER_REFUND_WINDOW_CLOSED` | 422 | The refund window for this order has passed. |
| `ORDER_REFUND_ALREADY_ISSUED` | 409 | This order has already been refunded. |
| `ORDER_PARTIAL_REFUND_INVALID` | 422 | Partial refunds must be less than the original amount. |

---

## 5. License Errors

| Code | HTTP | Message |
|------|------|---------|
| `LICENSE_NOT_FOUND` | 404 | License not found. |
| `LICENSE_INVALID` | 400 | This license key is invalid. |
| `LICENSE_EXPIRED` | 410 | This license has expired. |
| `LICENSE_REVOKED` | 403 | This license has been revoked. |
| `LICENSE_DEVICE_LIMIT_REACHED` | 422 | You've reached the activation limit for this license. |
| `LICENSE_ALREADY_ACTIVATED` | 409 | This license is already activated on this device. |
| `LICENSE_RATE_LIMITED` | 429 | Validation rate limit hit. Try again in {seconds} seconds. |

---

## 6. Subscription Errors

| Code | HTTP | Message |
|------|------|---------|
| `SUBSCRIPTION_NOT_FOUND` | 404 | Subscription not found. |
| `SUBSCRIPTION_ALREADY_ACTIVE` | 409 | You're already subscribed to this product. |
| `SUBSCRIPTION_PAYMENT_PENDING` | 425 | Your subscription is waiting for payment confirmation. |
| `SUBSCRIPTION_CANCELED` | 410 | This subscription has been canceled. |
| `SUBSCRIPTION_PAUSED` | 423 | This subscription is paused. |
| `SUBSCRIPTION_RENEWAL_FAILED` | 402 | Renewal failed. Update your payment method. |

---

## 7. Delivery / Access Errors

| Code | HTTP | Message |
|------|------|---------|
| `DELIVERY_NOT_READY` | 425 | Your download is being prepared. Try again in a moment. |
| `DELIVERY_URL_EXPIRED` | 410 | This download link has expired. |
| `DELIVERY_ACCESS_DENIED` | 403 | You don't have access to this file. |
| `DELIVERY_FILE_MISSING` | 404 | This file isn't available anymore. |
| `ACCESS_GRANT_NOT_FOUND` | 404 | Access grant not found. |
| `ACCESS_GRANT_REVOKED` | 403 | Your access to this product has been revoked. |

---

## 8. Discount / Affiliate / Marketing Errors

| Code | HTTP | Message |
|------|------|---------|
| `DISCOUNT_INVALID` | 400 | This discount code is invalid. |
| `DISCOUNT_EXPIRED` | 410 | This discount has expired. |
| `DISCOUNT_USAGE_LIMIT_REACHED` | 422 | This discount can no longer be used. |
| `DISCOUNT_NOT_APPLICABLE` | 422 | This discount doesn't apply to your cart. |
| `AFFILIATE_NOT_FOUND` | 404 | Affiliate not found. |
| `AFFILIATE_LINK_DISABLED` | 410 | This affiliate link is no longer active. |
| `AFFILIATE_SELF_REFERRAL` | 422 | You can't refer yourself. |

---

## 9. Webhook / Integration Errors

| Code | HTTP | Message |
|------|------|---------|
| `WEBHOOK_SIGNATURE_INVALID` | 400 | Webhook signature did not match. |
| `WEBHOOK_TIMESTAMP_TOO_OLD` | 400 | Webhook timestamp is too old. |
| `WEBHOOK_DUPLICATE` | 200 | Already processed (returned with 200 by design). |
| `WEBHOOK_ENDPOINT_NOT_FOUND` | 404 | Webhook endpoint not found. |
| `WEBHOOK_DELIVERY_FAILED` | 502 | Endpoint returned an error. |
| `STRIPE_API_ERROR` | 502 | We couldn't reach our payment processor. |
| `STRIPE_CONNECT_NOT_LINKED` | 422 | Connect your Stripe account to receive payouts. |

---

## 10. File / Upload Errors

| Code | HTTP | Message |
|------|------|---------|
| `FILE_TOO_LARGE` | 413 | The file exceeds the maximum size of {max} MB. |
| `FILE_TYPE_NOT_ALLOWED` | 422 | File type "{type}" isn't supported. |
| `FILE_UPLOAD_FAILED` | 500 | Upload didn't finish. Try again. |
| `FILE_VIRUS_DETECTED` | 422 | This file failed our security scan. |
| `FILE_MULTIPART_INVALID` | 400 | One or more parts of the upload are missing. |

---

## 11. API Key / Rate Limit Errors

| Code | HTTP | Message |
|------|------|---------|
| `APIKEY_INVALID` | 401 | API key invalid. |
| `APIKEY_REVOKED` | 401 | This API key has been revoked. |
| `APIKEY_SCOPE_INSUFFICIENT` | 403 | This API key doesn't have permission for this action. |
| `APIKEY_RATE_LIMITED` | 429 | Rate limit exceeded. Retry after {seconds} seconds. |
| `IP_RATE_LIMITED` | 429 | Too many requests from your IP. Try again in {seconds} seconds. |

---

## 12. Marketplace / Search Errors

| Code | HTTP | Message |
|------|------|---------|
| `MARKETPLACE_LISTING_NOT_FOUND` | 404 | Listing not found. |
| `MARKETPLACE_LISTING_PENDING` | 423 | This product is pending review. |
| `MARKETPLACE_LISTING_REJECTED` | 410 | This product was removed from the marketplace. |
| `SEARCH_QUERY_TOO_SHORT` | 422 | Search at least 2 characters. |
| `SEARCH_BACKEND_UNAVAILABLE` | 503 | Search is temporarily down. We're on it. |

---

## 13. Compliance / Account Lifecycle Errors

| Code | HTTP | Message |
|------|------|---------|
| `ACCOUNT_DELETION_PENDING` | 423 | This account is being deleted. |
| `ACCOUNT_SUSPENDED` | 423 | This account is suspended. Contact support. |
| `TERMS_NOT_ACCEPTED` | 403 | Accept the updated Terms to continue. |
| `EXPORT_NOT_READY` | 425 | Your export is still being prepared. |
| `GDPR_REQUEST_DUPLICATE` | 409 | A privacy request is already in progress. |

---

## 14. Generic / Fallback Errors

| Code | HTTP | Message |
|------|------|---------|
| `BAD_REQUEST` | 400 | Bad request. |
| `UNPROCESSABLE_ENTITY` | 422 | We couldn't process this request. |
| `NOT_FOUND` | 404 | Resource not found. |
| `CONFLICT` | 409 | This request conflicts with the current state. |
| `INTERNAL_ERROR` | 500 | Something went wrong on our side. We're looking into it. |
| `MAINTENANCE_MODE` | 503 | TESKEL is in maintenance. Back in a moment. |
| `UPSTREAM_TIMEOUT` | 504 | A dependency timed out. Please try again. |
| `FEATURE_FLAG_DISABLED` | 403 | This feature isn't enabled for your account yet. |

---

## 15. Form Validation Messages

Pesan untuk client-side Zod + Server-side validation.

| Field type | Rule | Message |
|------------|------|---------|
| String required | Empty | This field is required. |
| String length | Min/max | Use between {min} and {max} characters. |
| Email | Invalid | Enter a valid email. |
| URL | Invalid | Enter a valid URL starting with http(s). |
| Slug | Invalid | Use lowercase letters, numbers, and dashes only. |
| Password | Weak | Use at least 12 characters and mix letters, numbers, and symbols. |
| Confirm password | Mismatch | Passwords don't match. |
| Number | Min/Max | Enter a number between {min} and {max}. |
| Currency | Negative | Amount must be positive. |
| Date | In past | Choose a future date. |
| File | Size | File must be under {max} MB. |
| File | Type | Allowed types: {types}. |
| Phone | Invalid | Enter a valid phone number with country code. |
| Country | Required | Choose a country. |

---

## 16. UI State Catalog

### 16.1 Empty States

| Surface | H2 | Body | CTA |
|---------|----|------|-----|
| Products list | No products yet | Create your first downloadable product, license, or service. | Create product |
| Orders list | No orders yet | Orders will land here as soon as your first sale comes in. | Share your store |
| Customers list | No customers yet | Once people buy, you'll see their details here. | Invite test buyer |
| Discounts list | No discounts yet | Promotions boost conversion. Create your first code. | New discount |
| Funnels list | No funnels yet | Funnels turn visitors into customers in steps. | New funnel |
| Affiliates list | No affiliates yet | Invite people to share your store and earn commission. | Add affiliate |
| Analytics | No data yet | Numbers appear after your first sale. | Visit your store |
| Webhook endpoints | No endpoints yet | Wire TESKEL events into your stack. | Add endpoint |
| Marketplace search | No products match "{query}" | Try a different term or browse categories. | Browse all |
| API keys | No API keys yet | Generate keys to talk to TESKEL from your code. | Create API key |
| Notifications | All caught up | We'll let you know when something needs your attention. | — |
| Buyer purchases | No purchases yet | When you buy something, it'll show up here. | Explore marketplace |

### 16.2 Loading States

- Skeleton untuk list/grid (default).
- Inline spinner untuk in-button.
- Full-page splash hanya untuk first paint.
- Copy supportive bila >2 detik:
  - `Preparing your dashboard…`
  - `Crunching numbers…`
  - `Sending receipt…`

### 16.3 Error States (UI)

| Surface | Headline | Body | Actions |
|---------|----------|------|---------|
| Page error | Something went wrong | We've logged this incident. Try refreshing or come back in a moment. | Refresh / Go home |
| Stripe disconnected | Connect Stripe to keep selling | Your Stripe account isn't connected. Reconnect to receive payouts. | Connect Stripe |
| Plan limit reached | You've hit your plan limit | Upgrade to {next_plan} to keep {feature} active. | Upgrade |
| Permission denied | You don't have access | Ask an admin in {org} to grant you the {role} role. | Contact admin |
| Maintenance | We're upgrading TESKEL | We'll be back in a few minutes. Track progress on status.teskel.com. | Open status page |
| Network offline | You're offline | Check your connection. We'll keep your changes locally. | Retry |
| Webhook signature mismatch | We couldn't verify this request | The signature doesn't match. Check your webhook secret. | View docs |

### 16.4 Success States

- Toast: `Saved.`, `Sent.`, `Refund issued.`.
- Full page (rare): first sale celebration.
- Inline ✓ icon + `Saved · 12s ago` muncul setelah autosave.

### 16.5 Confirmation Modals

Pattern:

```
Title: <Action> <Resource>?
Body : <One-liner consequence>
Buttons: [Cancel] [<Action> <Resource>] (destructive: red)
```

Contoh:

| Action | Title | Body |
|--------|-------|------|
| Delete product | Delete product? | This will also archive all related orders summaries. This can't be undone. |
| Refund order | Refund order #{n}? | The buyer will see the refund within 5–10 business days. |
| Revoke license | Revoke license? | The user will lose access immediately. |
| Cancel subscription | Cancel subscription? | The buyer keeps access until {end_date}. |
| Delete API key | Delete API key? | Integrations using this key will stop working immediately. |
| Delete org | Delete organization? | All products, orders, and members will be permanently removed. |

---

## 17. Status Indicators

Semua status di UI menggunakan dot + label.

| Status | Warna | Pemakaian |
|--------|-------|-----------|
| `active` | success | Product, license, subscription |
| `draft` | muted | Product not published |
| `paused` | warning | Subscription pause |
| `expired` | muted-foreground | License/subscription end |
| `revoked` | destructive | Manually disabled |
| `pending` | warning | Awaiting action |
| `failed` | destructive | Payout/webhook |
| `processing` | info (blue) | In-progress |

---

## 18. Documentation Pattern

- Setiap error code punya halaman `docs.teskel.com/errors/{CODE}`.
- Halaman memuat: explanation, common causes, suggested fix, related endpoints.
- Auto-generated dari katalog ini melalui pipeline build.

---

## 19. Localization Keys

Format: `errors.<code_lowercase>.message`, `errors.<code_lowercase>.helper`.

Contoh `packages/email/locales/en-US/errors.json`:

```json
{
  "errors": {
    "auth_invalid_credentials": {
      "message": "The email or password is incorrect."
    },
    "checkout_card_declined": {
      "message": "Your card was declined. Try another payment method."
    }
  }
}
```

---

## 20. Anti-Patterns

Tolak:

- Mengembalikan stack trace ke client.
- Pesan error tanpa actionable guidance.
- `code` yang berubah antar versi.
- Toast yang menghilang sebelum 4 detik untuk error.
- Modal konfirmasi tanpa pemisahan visual antara cancel & destructive.
- Empty state generic ("No data") tanpa CTA.
- Loading state >2 detik tanpa indikator.

---

*Dokumen ini melengkapi `03_API_SPEC.md` §6 (error responses), `10_UIUX_MODERN_CLEAN.md` §4 (empty/loading/error), dan `19_BRAND_COPY_AND_VOICE.md`. Bila conflict, dokumen ini menang untuk error code dan UI state copy.*
