# Task List — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Glossary](../00-glossary.md)
> Lihat juga: [plan.md](./plan.md) · [alert-catalog.md](./alert-catalog.md) · [architecture.md](./architecture.md)

Estimasi: **S** (≤1 hari) · **M** (2–4 hari) · **L** (≥1 minggu). Checklist; centang saat selesai.

> Prasyarat global: MCP sumber sinyal dari [Phase 1](../phase-1-data-access/mcp-catalog.md) sudah aktif & stabil.

---

## A. Tugas Lintas (Alert Framework Reusable)

Dibangun sekali, dipakai semua kategori. Lihat [architecture.md](./architecture.md) & [erd.md](./erd.md).

- [ ] **(M)** Definisikan skema `AlertRule` (metric, threshold, schedule, audience, cooldown)
- [ ] **(M)** Definisikan skema `AlertEvent` (status: detected / sent / suppressed) + audit trail
- [ ] **(M)** Definisikan skema `MetricBaseline` (per metric, per hari-dalam-minggu / jam)
- [ ] **(S)** Definisikan skema `Subscription` (siapa/grup terima alert apa)
- [ ] **(S)** Definisikan skema `FeedbackLog` (👍/👎 per AlertEvent)
- [ ] **(L)** Bangun **anomaly detector** reusable (relative + absolute guard, baseline-aware)
- [ ] **(M)** Bangun **dedup/suppression** (dedup key + cooldown per rule)
- [ ] **(M)** Bangun **delivery formatter** → pesan WhatsApp ringkas + actionable
- [ ] **(M)** Integrasi **Cron Hermes** sebagai scheduler (bukan scheduler sendiri)
- [ ] **(M)** Pola **Subagent** untuk evaluasi paralel antar-rule/kategori
- [ ] **(S)** **Rate limit global** per subscriber (anti-spam cap harian)
- [ ] **(M)** **Shadow mode** switch (deteksi tanpa kirim) untuk kalibrasi
- [ ] **(S)** Health check sumber MCP + alert-meta bila source unreachable

## B. Feedback Mechanism

- [ ] **(M)** Tangkap reaksi/balasan WA ("berguna"/"noise") → `FeedbackLog`
- [ ] **(S)** Dashboard/laporan ringkas: noise-rate per rule, mute/unsubscribe rate
- [ ] **(S)** Prosedur review berkala → penyesuaian `AlertRule`
- [ ] **(S)** Mekanisme mute/unsubscribe per kategori

---

## C. Traffic Alerts — Stage 1 (pilot, prioritas)

Sumber: **GA4 MCP**. Detail: [alert-catalog.md](./alert-catalog.md).

- [ ] **(S)** Define metric (sessions/pageviews per window; per artikel)
- [ ] **(M)** Bangun baseline & threshold (per hari/jam; contoh kalibrasi)
- [ ] **(M)** Anomaly logic: turun signifikan / naik abnormal / lonjakan artikel
- [ ] **(S)** Cron schedule via Hermes (mis. tiap 15–30 mnt)
- [ ] **(S)** Dedup/suppression (cooldown per artikel/metric)
- [ ] **(S)** Delivery format + routing → tim editorial
- [ ] **(M)** Shadow run ≥ 1–2 minggu → tuning → go-live
- [ ] **(S)** Tuning pasca go-live dari FeedbackLog

## D. Competitor Alerts — Stage 2

Sumber: **RSS MCP**.

- [ ] **(S)** Define metric (frekuensi/penyebaran topik antar media)
- [ ] **(M)** Topic clustering / relevance scoring (teks, bukan angka murni)
- [ ] **(M)** Baseline & threshold (apa itu "viral"/"multi-media"; contoh kalibrasi)
- [ ] **(M)** Anomaly logic: topik multi-media / viral / kompetitor lebih cepat
- [ ] **(S)** Cron schedule via Hermes (mis. tiap 30–60 mnt)
- [ ] **(S)** Dedup/suppression (cooldown per topik/cluster)
- [ ] **(S)** Delivery format + routing → tim editorial/redaksi
- [ ] **(M)** Shadow run → tuning → go-live
- [ ] **(S)** Tuning pasca go-live

## E. Revenue Alerts — Stage 3

Sumber: **BigQuery MCP · CMS MCP · GSC MCP** (multi-source).

- [ ] **(S)** Define metric (fill rate, CTR, inventory)
- [ ] **(M)** Baseline & threshold per metric (contoh kalibrasi)
- [ ] **(M)** Anomaly logic: fill rate turun / CTR turun / inventory bermasalah
- [ ] **(M)** Korelasi multi-source (gabung BigQuery + CMS + GSC)
- [ ] **(S)** Cron schedule via Hermes (mis. tiap 30–60 mnt)
- [ ] **(S)** Dedup/suppression (cooldown per metric/unit inventory)
- [ ] **(S)** Delivery format + routing → tim commercial
- [ ] **(M)** Shadow run → tuning → go-live
- [ ] **(S)** Tuning pasca go-live

---

## F. Hardening & Exit (Stage 4)

- [ ] **(S)** Verifikasi success metrics tercapai (relevansi ≥80%, FP ≤15% — [prd.md](./prd.md))
- [ ] **(S)** Review anti-spam: rate limit & relevance scoring efektif
- [ ] **(S)** Dokumentasi runbook: cara menambah alert baru (= tambah `AlertRule`)
- [ ] **(S)** Sign-off Phase 3: signal > noise terbukti, user terbantu bukan terganggu
