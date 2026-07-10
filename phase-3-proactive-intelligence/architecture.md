# Architecture — Phase 3: Proactive Intelligence

> Parent: [Platform Architecture Overview](../00-architecture-overview.md) · [Glossary](../00-glossary.md)
> Lihat juga: [workflow.md](./workflow.md) · [erd.md](./erd.md) · [MCP Catalog (Phase 1)](../phase-1-data-access/mcp-catalog.md)

## 1. Prinsip

Phase 3 **tidak menambah layer baru** ke arsitektur platform. Ia berdiri di atas yang sudah ada:

- **Scheduler = Cron Hermes** (built-in). Kita tidak membangun scheduler.
- **Paralelisme = Subagent Hermes** (built-in).
- **Akses data = MCP Phase 1** (built-in client + MCP server KLY).
- **Delivery = Gateway WhatsApp Hermes** (built-in).

Yang **dibangun KLY di Phase 3** = "Alert Engine": kumpulan logika + state untuk **mendeteksi** dan **memutuskan** apakah suatu sinyal layak dikirim.

## 2. Alert Engine — Komponen

```text
┌──────────────────────────────────────────────────────────────────┐
│                    HERMES (Orchestration Layer)                       │
│                                                                       │
│   ┌────────────┐         ┌───────────────────────────────┐  │
│   │  CRON         │ trigger │  ALERT ENGINE (dibangun KLY)         │  │
│   │ (built-in     │────────▶│                                      │  │
│   │  scheduler)   │         │  ┌──────────────────────────┐  │  │
│   └────────────┘         │  │ A. Orchestrator                 │  │  │
│                            │  │   spawn Subagent per kategori   │  │  │
│   ┌────────────┐         │  └─────────────┬────────────┘  │  │
│   │ SUBAGENTS     │◄────────┤                  ▼                   │  │
│   │ (built-in,    │ spawn   │  ┌──────────────────────────┐  │  │
│   │  paralel)     │         │  │ B. Data Fetcher (via MCP)       │  │  │
│   └──────┬─────┘         │  └─────────────┬────────────┘  │  │
│          │                 │                  ▼                   │  │
│          │                 │  ┌──────────────────────────┐  │  │
│          │                 │  │ C. Anomaly Detector             │  │  │
│          │                 │  │   metric vs MetricBaseline       │  │  │
│          │                 │  │   + AlertRule (threshold)        │  │  │
│          │                 │  └─────────────┬────────────┘  │  │
│          │                 │                  ▼                   │  │
│          │                 │  ┌──────────────────────────┐  │  │
│          │                 │  │ D. Suppression / Dedup          │  │  │
│          │                 │  │   cooldown + dedup key + rate   │  │  │
│          │                 │  └─────────────┬────────────┘  │  │
│          │                 │                  ▼                   │  │
│          │                 │  ┌──────────────────────────┐  │  │
│          │                 │  │ E. Delivery (format + route)    │  │  │
│          │                 │  └─────────────┬────────────┘  │  │
│          │                 │                  │                   │  │
│          │                 │  ┌─────────────▼─────────────┐  │  │
│          │                 │  │ STATE STORE                     │  │  │
│          │                 │  │ AlertRule · MetricBaseline ·    │  │  │
│          │                 │  │ AlertEvent · Subscription ·     │  │  │
│          │                 │  │ FeedbackLog                      │  │  │
│          │                 │  └──────────────────────────┘  │  │
│          │                 └─────────────────┤─────────────────┘  │
│          ▼                                    ▼                      │
│   ┌────────────┐                   ┌──────────────┐           │
│   │ MCP CLIENT   │                   │ GATEWAY (WhatsApp)│           │
│   └─────┬─────┘                   └──────────────┘           │
└────────────────────┴──────────────────────────────────────────────────────┘
           ▼
   ┌──────────────────────────────────────┐
   │ MCP SERVERS (Phase 1): GA4 · RSS · BigQuery ·  │
   │ CMS · GSC                                       │
   └────────────────────────────────────────────────┘
```

### Tanggung jawab komponen

| Komp. | Nama | Tanggung jawab | Disediakan / Dibangun |
|-------|------|----------------|------------------------|
| — | **Cron** | Memicu evaluasi terjadwal per kategori | Hermes (built-in) |
| A | **Orchestrator** | Saat di-trigger, spawn Subagent per kategori/grup rule | KLY (logika) |
| — | **Subagent** | Eksekusi evaluasi terisolasi & paralel | Hermes (built-in) |
| B | **Data Fetcher** | Ambil metric via MCP sesuai `AlertRule` | KLY (logika) di atas MCP |
| C | **Anomaly Detector** | Bandingkan metric vs baseline; terapkan threshold | KLY |
| D | **Suppression/Dedup** | Cegah duplikat & spam (cooldown, dedup key, rate limit) | KLY |
| E | **Delivery** | Format pesan + tentukan audience + kirim ke gateway | KLY (format) di atas Gateway |
| — | **State Store** | Simpan rule, baseline, event, subscription, feedback | KLY |

## 3. Bagaimana Subagent Dipakai (Paralel)

- Cron tick → Orchestrator membaca rule aktif → **spawn satu Subagent per kategori** (atau per grup rule berat).
- Tiap Subagent menjalankan langkah B→E secara independen.
- **Manfaat:**
  - **Isolasi kegagalan** — RSS MCP down tidak menjatuhkan evaluasi Traffic (NFR-6).
  - **Latensi rendah** — kategori dievaluasi serempak, bukan antre.
  - **Skalabilitas** — menambah kategori = menambah Subagent, bukan memperlambat yang ada.

## 4. Di Mana State Disimpan

Lihat skema lengkap di [erd.md](./erd.md).

| State | Isi | Sifat |
|-------|-----|-------|
| **AlertRule** | metric, threshold, schedule, audience, cooldown | Konfigurasi — diubah saat tuning, bukan deploy |
| **MetricBaseline** | nilai normal per metric, per hari/jam | Diperbarui berkala dari data historis (via MCP) |
| **AlertEvent** | tiap deteksi: detected/sent/suppressed + alasan | Append-only, audit trail |
| **Subscription** | siapa/grup terima alert apa | Konfigurasi routing |
| **FeedbackLog** | 👍/👎 per AlertEvent | Input kalibrasi |

> Konsisten dengan prinsip platform "**stateless agent, stateful platform**": logika alert tidak menyimpan state sendiri; semua di State Store + memory Hermes. Baseline di-derive dari sumber kebenaran (MCP), bukan disalin sembarangan.

## 5. Strategi Anti-Spam

Berlapis — sinyal harus lolos semuanya:

1. **Threshold (relative + absolute guard).** Relative menangkap penyimpangan; absolute mencegah alert dari fluktuasi angka kecil.
2. **Baseline kontekstual.** Per hari-dalam-minggu/jam → mengurangi false positive musiman.
3. **Cooldown per rule (dedup).** Satu jenis anomaly tidak dikirim berulang dalam window (mis. 60 mnt).
4. **Rate limit global per subscriber.** Cap jumlah alert/hari per penerima — pelindung terakhir terhadap badai alert.
5. **Relevance scoring.** Terutama Competitor (topik): hanya yang skor relevansinya tinggi yang lolos.
6. **Shadow mode & feedback loop.** Kalibrasi sebelum & sesudah go-live (lihat [plan.md](./plan.md)).

> Default sistem = **diam**. Anti-spam bukan fitur tambahan — ia adalah perilaku inti engine.
