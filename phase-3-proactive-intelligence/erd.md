# ERD — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Glossary](../00-glossary.md)
> Lihat juga: [architecture.md](./architecture.md) · [alert-catalog.md](./alert-catalog.md)

Model data **Alert Engine**. Bukan data bisnis (itu hidup di sumber, diakses via MCP) — ini state untuk **mendeteksi, memutuskan, dan menelusuri** alert.

## 1. Diagram Relasi

```text
                       ┌───────────────┐
                       │   AlertRule     │
                       │  (konfigurasi)  │
                       └───┬─────────┬───┘
                           │1       1│
            menghasilkan   │         │   menargetkan
                           │N        │N
                  ┌────────▼───┐ ┌───▼─────────────┐
                  │ AlertEvent │ │  Subscription    │
                  │ (deteksi/  │ │ (routing penerima)│
                  │  kirim)    │ └──────────────────┘
                  └─────┬──────┘
                        │1
              dinilai   │
                        │N
                  ┌─────▼──────┐
                  │ FeedbackLog │
                  └────────────┘

   ┌──────────────┐        dirujuk saat evaluasi
   │ MetricBaseline   │◄───────── AlertRule (via metric)
   │ (nilai normal)   │
   └──────────────┘
```

Ringkasan kardinalitas:

| Relasi | Tipe | Arti |
|--------|------|------|
| AlertRule → AlertEvent | 1 : N | satu rule memicu banyak event dari waktu ke waktu |
| AlertRule → Subscription | 1 : N | satu rule dialamatkan ke banyak penerima/grup |
| AlertEvent → FeedbackLog | 1 : N | satu alert terkirim bisa dinilai beberapa penerima |
| AlertRule → MetricBaseline | N : 1 (via `metric`) | banyak rule bisa merujuk baseline metric yang sama |

## 2. Entitas & Atribut

### AlertRule — definisi aturan deteksi
Konfigurasi; diubah saat tuning, bukan saat deploy.

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| `rule_id` (PK) | id | unik |
| `name` | text | mis. "Traffic Drop Signifikan" |
| `category` | enum | competitor / traffic / revenue |
| `metric` | text | metric dipantau (mis. `sessions_15m`, `fill_rate`) |
| `mcp_source` | text | MCP penyedia data (GA4 / RSS / BigQuery / CMS / GSC) |
| `condition` | text | logika pemicu (mis. `delta < -30% AND abs > guard`) |
| `threshold_relative` | number | ambang relatif (contoh, **kalibrasi**) |
| `threshold_absolute` | number | absolute guard (anti fluktuasi angka kecil) |
| `schedule` | cron-expr | jadwal eval (dieksekusi oleh **Cron Hermes**) |
| `cooldown` | duration | window suppression (mis. 60m) |
| `priority` | enum | high / medium / low |
| `enabled` | bool | aktif / shadow / mati |
| `updated_at` | ts | jejak tuning |

### MetricBaseline — nilai "normal" untuk perbandingan

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| `baseline_id` (PK) | id | |
| `metric` | text | metric yang dibaselinekan |
| `context_key` | text | konteks (mis. `dow=Wed,hour=14`) — baseline per hari/jam |
| `value` | number | nilai normal |
| `spread` | number | deviasi/variabilitas (untuk relative threshold) |
| `window` | text | rentang historis (mis. 4 minggu) |
| `computed_at` | ts | kapan dihitung dari data (via MCP) |

### AlertEvent — catatan tiap evaluasi yang relevan (audit trail)
Append-only.

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| `event_id` (PK) | id | |
| `rule_id` (FK) | id | → AlertRule |
| `detected_at` | ts | waktu evaluasi |
| `observed_value` | number | nilai aktual |
| `baseline_value` | number | nilai baseline saat itu |
| `deviation` | number | selisih / % |
| `status` | enum | **detected** (tak lolos threshold) / **suppressed** (duplikat/rate) / **sent** |
| `suppress_reason` | text | bila suppressed (cooldown / rate-limit) |
| `dedup_key` | text | kunci dedup (mis. `rule_id+artikel+window`) |
| `delivered_to` | text[] | subscriber/grup penerima (bila sent) |
| `message` | text | isi alert terkirim |

### Subscription — siapa menerima alert apa

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| `sub_id` (PK) | id | |
| `rule_id` (FK) | id | → AlertRule (atau `category` untuk langganan kategori) |
| `recipient` | text | nomor WA / id grup |
| `recipient_type` | enum | individu / grup |
| `team` | text | editorial / commercial / redaksi |
| `muted_until` | ts | dukungan mute sementara (anti-fatigue) |
| `active` | bool | |

### FeedbackLog — input kalibrasi noise

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| `feedback_id` (PK) | id | |
| `event_id` (FK) | id | → AlertEvent |
| `verdict` | enum | useful / noise |
| `note` | text | opsional |
| `submitted_by` | text | penerima |
| `submitted_at` | ts | |

## 3. Bagaimana Skema Mendukung Prinsip

| Prinsip ([prd.md](./prd.md)) | Didukung oleh |
|---|---|
| Anomaly/opportunity-based | `MetricBaseline` + `condition`/threshold di `AlertRule` |
| Anti-spam / dedup | `cooldown`, `dedup_key`, `status=suppressed` di `AlertEvent` |
| Signal > noise (tuning) | `FeedbackLog` → review → ubah `AlertRule` |
| Routing tepat sasaran | `Subscription` (team, recipient) |
| Auditability | `AlertEvent` append-only dengan alasan keputusan |
| Tambah alert tanpa deploy | tambah baris `AlertRule`, bukan ubah kode |
