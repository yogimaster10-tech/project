# TESKEL — Observability & Incident Response
## Detect, Diagnose, Recover — Without Heroics

> Sistem yang tidak teramati adalah sistem yang tidak ada. Dokumen ini menetapkan SLI/SLO, signal, alerting, dan ritual respons insiden untuk TESKEL berskala global.

---

## 1. Golden Signals (Per Service)

Setiap service di TESKEL WAJIB memiliki 4 signal ini terkirim ke observability backend (Sentry + Grafana/PostHog/Logflare/Datadog tergantung tahap):

| Signal | Definisi | Target Visualisasi |
|--------|----------|--------------------|
| Latency | P50 / P95 / P99 per endpoint | Time-series + histogram |
| Traffic | Request per detik, per endpoint / per org | Time-series + heatmap |
| Errors | Rate error 5xx, error rate 4xx tertentu | Time-series + log link |
| Saturation | CPU, memory, connection pool, queue depth | Gauge + alert |

Dashboards minimal: `service-health`, `business-health`, `perf-overview`, `slo-status`.

---

## 2. SLI / SLO / SLA Catalog

### 2.1 Top-Level Business SLO

| Indicator | SLI | SLO 30-day | Reaksi |
|-----------|-----|-----------|--------|
| Checkout availability | `count(success) / count(attempt)` di `/v1/checkout/sessions` | ≥ 99.9% | Burn rate >2× → SEV2 |
| Payment confirm latency | `webhook.checkout_completed → order created` | P95 ≤ 8s | Burn rate >2× → SEV3 |
| File delivery success | `count(success signed URL) / count(grant)` | ≥ 99.95% | Burn rate >2× → SEV2 |
| Storefront uptime | `count(2xx) / count(total)` di `/[storeSlug]` | ≥ 99.95% | Burn rate >2× → SEV1 |
| Marketplace search latency | P95 search query | ≤ 300 ms | Burn rate >2× → SEV3 |
| Email delivery | `delivered / sent` 24h | ≥ 99.5% | Bounce spike → SEV3 |
| Background job lag | order webhook processing | P95 ≤ 30s | Lag >5min → SEV2 |
| License validation latency | P99 latency | ≤ 100 ms | Burn rate >2× → SEV3 |

### 2.2 Error Budget Policy

- Error budget = 100% − SLO.
- Bila error budget 50% terpakai dalam 7 hari → freeze feature work, fokus reliabilitas.
- Bila 100% terpakai → halt rilis non-fix selama 7 hari, postmortem wajib.

### 2.3 SLA Eksternal

Komunikasi publik (dokumentasi):

- Free tier: best effort, tanpa SLA.
- Pro: 99.5% uptime monthly. Kompensasi: kredit pro-rata.
- Scale: 99.9% uptime monthly. Kompensasi: kredit + extended support.
- Enterprise: 99.95% uptime monthly + dedicated SLA per kontrak.

SLA selalu LEBIH RENDAH dari SLO internal. Internal harus aman dari SLA.

---

## 3. Logging Standards

### 3.1 Format

Structured JSON only. Tidak boleh `console.log` plain string di production code.

Field wajib di setiap log:

```json
{
  "ts": "2026-05-22T10:00:00.000Z",
  "level": "info",
  "msg": "checkout_session_created",
  "service": "api",
  "env": "production",
  "region": "us-east",
  "requestId": "req_xxx",
  "traceId": "trace_xxx",
  "userId": "usr_xxx",
  "orgId": "org_xxx",
  "route": "POST /v1/checkout/sessions",
  "latencyMs": 142,
  "statusCode": 201
}
```

### 3.2 Aturan

- Tidak boleh log PII mentah (email, nama, alamat). Pakai hash 8-char untuk korelasi.
- Tidak boleh log secret (API key, password, Stripe token).
- Setiap request HTTP otomatis di-log oleh middleware.
- Setiap operasi DB lintas batas write (insert/update/delete) di-log.
- Setiap webhook eksternal yang diterima di-log dengan idempotency key.

### 3.3 Level

| Level | Pakai untuk |
|-------|-------------|
| `error` | Failure yang perlu attention |
| `warn` | Indikasi potential issue (retry exhausted, rate limit hit) |
| `info` | State transition penting (order created, payout sent) |
| `debug` | Local dev only, off di production |

Log retention: 30 hari online, 1 tahun cold (S3 + lifecycle).

---

## 4. Metrics

### 4.1 RED Method per Endpoint

- Rate: requests per second.
- Errors: count of failed requests.
- Duration: latency distribution.

### 4.2 USE Method per Resource

- Utilization: CPU, memory, disk, network %.
- Saturation: queue depth, pending connections.
- Errors: error rate.

### 4.3 Business Metrics

| Metric | Source | Use |
|--------|--------|-----|
| `orders_created_total{org_id, type}` | service | Dashboard, alert anomaly |
| `revenue_cents_total{currency}` | service | Business dashboard |
| `licenses_issued_total{org_id}` | service | Anomaly detection |
| `refunds_initiated_total{reason}` | service | Trend analysis |
| `signup_total{plan}` | service | Growth funnel |
| `email_sent_total{template, status}` | worker | Deliverability |

---

## 5. Tracing

Distributed tracing wajib di Phase 3+.

- OpenTelemetry SDK pada `apps/web`, `apps/api`, `apps/embed`, workers.
- Trace context propagation via `traceparent` header.
- Sampling: 100% pada path checkout/webhook/delivery, 10% selainnya, 100% pada error.
- Backend: Sentry Performance atau Tempo + Grafana.

Setiap trace memuat span:

```
HTTP → middleware (auth, ratelimit) → service → DB → external (stripe/r2/email)
```

---

## 6. Dashboards Inventory

Wajib hadir sebelum GA:

1. `slo-status` — uptime, latency, error budget per SLI.
2. `service-health` — golden signals per service.
3. `business-health` — orders, revenue, refunds, churn signals.
4. `perf-overview` — load test results, latency budget burn.
5. `payment-pipeline` — Stripe webhook lag, retries, failure.
6. `delivery-pipeline` — file download success, R2 latency.
7. `email-pipeline` — sent, delivered, bounced, complaint.
8. `marketplace-search` — query rate, P95, hit rate.
9. `auth-security` — login success/fail, MFA usage, anomaly.
10. `cost-tracking` — daily cost per provider, per env, per org tier.

---

## 7. Alerting Policy

### 7.1 Skala Severity Alert

| Severity | Trigger | Routing | Response time |
|----------|---------|---------|---------------|
| SEV1 | Critical path total outage, data loss/leak | Pager (PagerDuty), all on-call | 15 menit |
| SEV2 | Major degradation (P95 >2× SLO, partial outage) | Pager | 30 menit |
| SEV3 | Performance regresi <SLO, anomalous metrics | Slack `#ops-alerts` | 4 jam (business hours) |
| SEV4 | Informational anomalies | Slack `#ops-info` | Next business day |

### 7.2 Alert Hygiene

- Setiap alert wajib mencantumkan: runbook URL, owner team, severity, trigger metric.
- Alert flaky dimute selama investigasi, ticket dibuka.
- Tidak ada alert tanpa runbook (alert noise discipline).
- Setiap quarter, audit alert: hapus yang tidak pernah berbunyi 90 hari & low value.

### 7.3 Contoh Alert Definition

```yaml
- alert: CheckoutErrorRateHigh
  expr: sum(rate(http_requests_total{route="/v1/checkout/sessions",status=~"5.."}[5m])) /
        sum(rate(http_requests_total{route="/v1/checkout/sessions"}[5m])) > 0.01
  for: 5m
  labels:
    severity: SEV2
    service: api
    owner: payments-team
  annotations:
    summary: Checkout error rate > 1% for 5m
    runbook: https://runbooks.teskel.com/checkout-error-rate
```

---

## 8. On-Call Rotation

### 8.1 Struktur

- Primary on-call: 1 engineer.
- Secondary on-call: 1 engineer (escalation).
- Rotasi mingguan.
- Maksimal 1 minggu on-call per 4 minggu.

### 8.2 Aturan

- Primary harus respon ≤15 menit ke pager SEV1, ≤30 menit SEV2.
- Setiap shift handover: brief 15 menit di Slack channel `#ops-handover` membahas open incident, deployments terakhir, anomaly.
- On-call compensation diatur HR (di luar scope dokumen ini, namun WAJIB ada).

### 8.3 Tools

- PagerDuty / OpsGenie.
- Slack: `#incident`, `#ops-alerts`, `#ops-handover`.
- Statuspage publik: `status.teskel.com`.

---

## 9. Incident Severity Matrix

| SEV | Definisi | Komunikasi Eksternal | Postmortem |
|-----|----------|----------------------|------------|
| SEV1 | Outage critical path, atau security incident dengan dampak pelanggan | Status page update <15 menit, email pelanggan terdampak <2 jam | Wajib, dalam 5 hari kerja |
| SEV2 | Degradasi major (sebagian besar pengguna terdampak) | Status page update <30 menit | Wajib, dalam 7 hari kerja |
| SEV3 | Gangguan minor, workaround tersedia | Status page jika berlangsung >1 jam | Opsional, ringkas |
| SEV4 | Tidak terlihat oleh pelanggan | Tidak | Tidak |

---

## 10. Incident Response Playbook

### 10.1 Roles

| Role | Tugas |
|------|------|
| Incident Commander (IC) | Pemimpin koordinasi, satu suara keputusan |
| Communications Lead | Update status page, email, internal Slack |
| Scribe | Mencatat timeline di Slack pin / Notion |
| Operations Lead | Eksekusi mitigasi teknis |
| Subject Matter Experts | Joined ad-hoc |

Satu orang bisa memegang >1 role di insiden kecil.

### 10.2 Workflow

```
1. Detect (alert atau report).
2. Acknowledge: IC ditunjuk dalam 5 menit.
3. Open Slack channel: #inc-YYYYMMDD-<slug>.
4. Bridge: voice call jika SEV1/SEV2.
5. Status page: post "Investigating".
6. Mitigate: prioritas kembalikan service, root cause belakangan.
7. Communicate: status update setiap 30 menit (SEV1) atau 60 menit (SEV2).
8. Stabilize: monitor 30 menit setelah mitigasi.
9. Status page: post "Resolved".
10. Schedule postmortem.
```

### 10.3 Eskalasi

```
Primary → 15 menit no ack → Secondary.
Secondary → 15 menit no ack → Engineering lead.
Engineering lead → 15 menit no ack → CTO.
```

---

## 11. Postmortem Template

File: `docs/postmortems/YYYY-MM-DD-<slug>.md`.

```markdown
# Postmortem — <Incident Title>

Date: <YYYY-MM-DD>
Severity: SEV1 / SEV2 / SEV3
Duration: HH:MM
Author: @owner
Status: draft / in review / final

## Summary
2–3 kalimat ringkasan: apa yang terjadi, dampak, durasi.

## Impact
- Customer impact: jumlah org, jenis flow terganggu.
- Revenue impact: estimasi USD.
- SLA impact: jumlah customer melanggar SLA.

## Timeline (UTC)
| Time | Event |
|------|-------|
| 09:42 | Alert "CheckoutErrorRateHigh" fired |
| 09:43 | IC acknowledged |
| 09:48 | Identified Stripe webhook signature mismatch after env rotation |
| 09:55 | Restored previous webhook secret |
| 10:05 | Error rate kembali normal |
| 10:35 | Postmortem dijadwalkan |

## Detection
Bagaimana incident terdeteksi? Berapa lama dari onset ke detect?

## Response
Apa yang dilakukan, oleh siapa, kapan?

## Root Cause
Analisis dengan 5-Why atau Causal Loop. Hindari single-blame.

## Contributing Factors
- Faktor desain yang membuat ini mungkin.
- Faktor proses yang tidak menangkapnya lebih awal.

## What Went Well
- ...
- ...

## What Went Poorly
- ...
- ...

## Action Items
| ID | Action | Owner | Due |
|----|--------|-------|-----|
| AI-1 | Tambah test webhook signature mismatch | @yogi | 2026-05-29 |
| AI-2 | Otomatisasi rotation env via secret manager | @platform | 2026-06-15 |
| AI-3 | Tambah alert deploy-mismatch (config vs deployed) | @platform | 2026-06-30 |

## Lessons Learned
Kesimpulan structural yang berlaku di luar incident ini.
```

Blameless culture: yang dianalisis adalah sistem, bukan individu.

---

## 12. Communications During Incident

### 12.1 Internal

Channel: `#inc-YYYYMMDD-<slug>`.

Status update format:

```
[UPDATE 10:15 UTC]
Impact: Checkout error rate 4% (down from 12%).
Mitigation: rollback deploy abc123, recheck Stripe webhook.
Next steps: monitor 30m, prepare hotfix.
Next update: 10:45 UTC.
```

### 12.2 Eksternal (Status Page & Email)

Templates:

**Investigating**

> We are investigating reports of elevated error rates on checkout. Some users may experience failed payments. We will provide updates every 30 minutes.

**Identified**

> We have identified the cause: a misconfiguration in our payment webhook. Mitigation is in progress.

**Monitoring**

> A fix has been deployed. We are monitoring service to ensure full recovery.

**Resolved**

> The incident has been resolved at 10:35 UTC. Total duration: 53 minutes. We will publish a postmortem within 5 business days.

---

## 13. Runbook Template

File: `docs/runbooks/<alert-or-procedure-slug>.md`.

```markdown
# Runbook — Checkout Error Rate High

Owner: payments-team
Last reviewed: 2026-05-22

## Symptoms
- Alert `CheckoutErrorRateHigh` firing.
- Grafana `payment-pipeline` shows 5xx spike.

## Likely Causes
1. Stripe webhook signature mismatch (env rotation).
2. Stripe API outage.
3. DB write contention on `orders` table.

## Diagnosis Steps
1. Cek dashboard `payment-pipeline` & alert.
2. Lihat last deploy: `gh release view`.
3. Cek Sentry tag `webhook.signature_mismatch`.
4. Cek Stripe status: `stripe.statuspage.io`.

## Mitigations
- Webhook mismatch: rollback secret rotation, paste secret terakhir dari secret manager.
- Stripe outage: enable degraded mode (queue webhooks, show banner).
- DB contention: scale up Neon pooler, check long-running queries.

## Verification
- Error rate <0.5% selama 10 menit.
- Sample order baru: payment success end-to-end.

## Escalation
- 15 menit no recovery → escalate ke engineering lead.
- 30 menit no recovery → declare SEV1.
```

Setiap alert WAJIB punya runbook link, di-test minimal 2× per quarter (table-top exercise).

---

## 14. Synthetic Monitoring

Synthetic check (Checkly / Datadog / self-hosted Playwright) menjalankan setiap 5 menit:

1. Homepage load: HTTP 200, hero text muncul.
2. Signup form: render valid, CSRF token ada.
3. Login form: render valid.
4. Storefront `/demo`: render, product card tampil.
5. Product detail: render, buy button aktif.
6. Checkout session: redirect ke Stripe (status 302 ke domain stripe).
7. License validate API: 100 ms median.
8. Public API status: `/v1/status` returns `ok`.

Failure berturut 2× → SEV3 alert; 4× → SEV2; 8× → SEV1.

Hasil di-track di `status.teskel.com` (public).

---

## 15. Backup & Recovery Drill

Lihat detail di `06_INFRA_SECURITY_DEVOPS.md` §6. Observability spesifik:

- Backup job menulis metric `backup_last_success_timestamp`.
- Alert `BackupStale` jika tidak ada backup sukses dalam 26 jam.
- Quarterly: restore drill ke env terisolasi, validasi via skrip otomatis. Hasil disimpan di `docs/runbooks/restore-drill-YYYY-Qx.md`.

---

## 16. Cost Observability

Setiap minggu:

- Dashboard `cost-tracking` direview oleh engineering lead.
- Anomali cost >25% week-over-week memicu ticket investigasi.
- Setiap org tier dipantau cost-to-serve: bila negatif margin >30 hari, eskalasi ke product/finance.

---

## 17. Observability Anti-Patterns

Dilarang:

- Log seluruh body request/response (PII risk + cost).
- Alert "high CPU" tanpa hubungan ke SLO.
- Dashboard yang tidak dibaca lebih dari 30 hari (hapus).
- Postmortem yang menyalahkan individu.
- "Mute alert biar tidur" tanpa ticket follow-up.

---

## 18. Maturity Roadmap

| Phase | Capability |
|-------|-----------|
| Phase 1 | Structured log + Sentry + uptime monitor uptime-only |
| Phase 2 | Dashboards utama + alert SEV2 untuk checkout/webhook/delivery |
| Phase 3 | OpenTelemetry tracing + RED metric + on-call rotation |
| Phase 4 | Synthetic monitoring 24/7 + chaos test mingguan + cost observability |
| Phase 5 | Pen-tested observability, anomaly detection ML-assisted |
| Phase 6 | Public statuspage, SLA-aware throttling, predictive scaling |

---

*Dokumen ini melengkapi `06_INFRA_SECURITY_DEVOPS.md` §5–§6. Bila conflict, dokumen ini menang untuk policy/operasional, sementara `06_INFRA_SECURITY_DEVOPS.md` menang untuk arsitektur cloud.*
