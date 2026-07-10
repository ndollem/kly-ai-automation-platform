# Plan — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Roadmap](../00-roadmap.md) · [Glossary](../00-glossary.md)
> Lihat juga: [prd.md](./prd.md) · [architecture.md](./architecture.md) · [task-list.md](./task-list.md)

## 1. Filosofi Pendekatan

Phase 3 **bukan fitur baru, tapi pola interaksi baru** di atas data Phase 1–2. Maka prinsip utamanya:

- **Hermes-native dulu.** Penjadwalan = Cron Hermes. Paralelisme = Subagent. Akses data = MCP. Kita hanya membangun **logika deteksi + aturan alert**.
- **Mulai sempit, perdalam dulu, baru melebar.** Satu kategori alert yang benar-benar berguna lebih baik daripada tiga kategori yang spammy.
- **Kalibrasi di lapangan.** Threshold awal pasti salah. Yang penting adalah loop untuk memperbaikinya cepat.

## 2. Ketergantungan ke MCP Phase 1

Phase 3 **tidak bisa dimulai** sebelum MCP sumber sinyal aktif. Pemetaan:

| Kategori alert | MCP wajib (Phase 1) | Status prasyarat |
|----------------|---------------------|------------------|
| Traffic | GA4 MCP | Harus aktif & stabil |
| Competitor | RSS MCP | Harus aktif & stabil |
| Revenue | BigQuery MCP · CMS MCP · GSC MCP | Paling kompleks; terakhir |

> Urutan ini sejalan dengan urutan kematangan MCP di [Roadmap](../00-roadmap.md): GA4 dulu (cepat, value tinggi), RSS, lalu BigQuery/CMS/GSC.

## 3. Pendekatan Bertahap

```text
Stage 0 ── Stage 1 ────── Stage 2 ─────── Stage 3 ──────── Stage 4
Framework  Traffic         Competitor       Revenue          Hardening
+ Shadow   (pilot, 1 kat)  (perluasan)      (kompleks)        + Feedback
```

### Stage 0 — Alert Framework + Shadow Mode
- Bangun komponen reusable: data-fetch (via MCP), anomaly detector, dedup/suppression, delivery. Lihat [architecture.md](./architecture.md).
- Definisikan skema `AlertRule`, `AlertEvent`, `MetricBaseline` ([erd.md](./erd.md)).
- **Shadow mode:** Cron berjalan, deteksi berjalan, alert **dicatat tapi tidak dikirim**. Tujuan: kumpulkan data untuk kalibrasi threshold tanpa mengganggu siapa pun.

### Stage 1 — Traffic Alerts (pilot, kategori paling bernilai)
- **Kenapa Traffic dulu:** single-source (GA4 MCP), sinyal jelas & cepat berubah, dampak langsung dirasakan editorial, baseline relatif mudah dibentuk.
- Aktifkan 3 alert: traffic turun signifikan, naik abnormal, lonjakan artikel ([alert-catalog.md](./alert-catalog.md)).
- Jalankan shadow ≥ 1–2 minggu → kalibrasi → baru "go-live" (mulai kirim).

### Stage 2 — Competitor Alerts
- Tambah RSS MCP sebagai sumber. Tantangan: topik = teks, butuh clustering/relevance scoring, bukan sekadar angka vs threshold.
- Re-use framework Stage 0; tambahkan komponen relevance/topic logic.

### Stage 3 — Revenue Alerts
- Multi-source (BigQuery + CMS + GSC). Paling kompleks, korelasi antar metric (fill rate × CTR × inventory).
- Routing ke tim commercial, bukan editorial.

### Stage 4 — Hardening + Feedback Loop matang
- Aktifkan feedback mechanism penuh, rate-limit global, relevance scoring lintas kategori, review berkala.

> Tiap stage punya **exit criteria** sebelum lanjut: signal>noise terbukti pada kategori berjalan (lihat success metrics di [prd.md](./prd.md)).

## 4. Strategi Kalibrasi Threshold

Threshold awal adalah **tebakan terdidik**, bukan kebenaran. Strategi:

1. **Shadow first.** Jalankan deteksi tanpa kirim; bandingkan "alert yang akan terkirim" dengan penilaian manual analis.
2. **Baseline kontekstual.** Jangan satu angka global. Baseline per **hari-dalam-minggu** dan idealnya per **jam** (traffic Senin pagi ≠ Sabtu malam). Simpan di `MetricBaseline`.
3. **Relative + absolute guard.** Pakai deviasi relatif (mis. > X% dari baseline) DAN ambang absolut minimum (hindari alert dari fluktuasi pada angka kecil).
4. **Mulai longgar di sisi yang aman.** Untuk pilot, lebih baik sedikit lebih konservatif (kirim hanya yang sangat jelas) agar kepercayaan terbangun, lalu perlonggar.
5. **Kalibrasi dua arah & berkala.** Review false-positive DAN false-negative tiap minggu di fase awal, lalu bulanan.

> Angka spesifik (X%, window, cooldown) di [alert-catalog.md](./alert-catalog.md) adalah **contoh yang HARUS dikalibrasi**, bukan nilai final.

## 5. Feedback Loop untuk Menekan Noise

```text
Alert terkirim ─► Penerima menilai (👍 berguna / 👎 noise)
       │                         │
       │                         ▼
       │                 FeedbackLog (per rule)
       │                         │
       ▼                         ▼
  AlertEvent ──────────►  Review berkala ──► Sesuaikan AlertRule
                            (threshold, cooldown, routing)
```

- **Mekanisme ringan:** reaksi/balasan singkat di WhatsApp ("noise" / "berguna") ditangkap Hermes → `FeedbackLog`.
- **Sinyal yang dipantau:** rasio noise per rule, mute/unsubscribe rate, alert yang tak pernah ditindaklanjuti.
- **Aksi:** rule dengan noise tinggi → naikkan threshold / perpanjang cooldown / persempit audience, atau dimatikan.
- **Prinsip:** lebih baik mematikan satu alert yang spammy daripada kehilangan kepercayaan pada seluruh sistem.

## 6. Definition of Done Phase 3

- Minimal kategori Traffic live dengan signal>noise terbukti (≥ target di [prd.md](./prd.md)).
- Framework reusable: menambah alert baru = menambah `AlertRule`, bukan menulis ulang pipeline.
- Feedback loop berjalan; ada bukti threshold pernah dikalibrasi dari data nyata.
- Tidak ada laporan rutin/spam — semua alert anomaly/opportunity-based.
