# PRD — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Roadmap](../00-roadmap.md) · [Glossary](../00-glossary.md)
> Dependensi: [MCP Catalog (Phase 1)](../phase-1-data-access/mcp-catalog.md) · [Use Cases (Phase 2)](../phase-2-business-use-cases/)

## 1. Latar Belakang & Masalah

Sampai Phase 2, pola interaksi platform bersifat **reactive**: user bertanya lewat WhatsApp, agent menjawab dengan menarik data via MCP. Pola ini bagus untuk eksplorasi, tapi punya satu kelemahan struktural — **insight hanya muncul ketika seseorang ingat untuk bertanya.**

Konsekuensinya:

- **Telat ketahuan.** Traffic drop, fill rate turun, atau kompetitor mengangkat isu besar bisa berlangsung berjam-jam/berhari-hari sebelum ada yang membuka dashboard atau bertanya ke agent.
- **Bergantung pada disiplin manusia.** Monitoring rutin menjadi beban kognitif; mudah terlewat saat sibuk.
- **Window aksi menyempit.** Makin lama anomaly tidak terdeteksi, makin kecil ruang untuk merespons (mis. mengejar topik viral, memperbaiki ad inventory).

Phase 3 membalik arah informasi: dari "User bertanya → Agent menjawab" menjadi **"Agent mendeteksi → Agent memberi tahu"**.

## 2. Tujuan

1. Mendeteksi **anomaly** (penyimpangan dari baseline) dan **opportunity** (momentum yang bisa dimanfaatkan) secara otomatis dan terjadwal.
2. Mengirim **alert berkualitas tinggi** ke WhatsApp — hanya hal yang layak ditindaklanjuti.
3. Memperpendek waktu dari "kejadian" ke "tim tahu" dari jam/hari menjadi menit.
4. Membangun **alert engine reusable** di atas Cron + Subagent + MCP yang sudah ada — bukan scheduler baru.

> **Non-tujuan:** Phase 3 bukan laporan rutin harian. Bukan dashboard. Bukan pengganti analisis mendalam (itu tetap reactive di Phase 2).

## 3. Scope

Tiga kategori alert, bersumber dari MCP Phase 1:

| Kategori | Sumber sinyal (MCP) | Contoh deteksi |
|----------|---------------------|----------------|
| **Competitor Alerts** | RSS MCP | Topik baru muncul di banyak media; topik sedang viral; kompetitor lebih cepat mengangkat isu |
| **Traffic Alerts** | GA4 MCP | Traffic turun signifikan; traffic naik tidak normal; artikel tertentu melonjak |
| **Revenue Alerts** | BigQuery MCP · CMS MCP · GSC MCP | Fill rate turun; CTR menurun; inventory bermasalah |

Detail per-alert: lihat [alert-catalog.md](./alert-catalog.md).

**Di luar scope Phase 3:** prediksi/forecasting ML, auto-remediation (alert memberi tahu, manusia memutuskan), kanal selain WhatsApp.

## 4. Functional Requirements

| ID | Requirement |
|----|-------------|
| FR-1 | **Anomaly detection** — tiap metric dievaluasi terhadap baseline (mis. rata-rata + deviasi window, atau perbandingan periode-ke-periode). |
| FR-2 | **Threshold configurable** — tiap alert punya threshold yang bisa dikalibrasi tanpa ubah kode (disimpan sebagai `AlertRule`). |
| FR-3 | **Scheduling via Cron Hermes** — evaluasi dijalankan terjadwal oleh Cron bawaan Hermes; KLY tidak membangun scheduler sendiri. |
| FR-4 | **Parallel evaluation via Subagent** — beberapa alert dievaluasi paralel lewat Subagent agar satu kategori lambat tidak memblok yang lain. |
| FR-5 | **Deduplication & suppression** — alert yang sama tidak dikirim berulang dalam window suppression (mis. cooldown per rule). |
| FR-6 | **Delivery via WhatsApp** — alert lolos dikirim ke subscriber/grup yang tepat melalui gateway Hermes, dalam format ringkas + actionable. |
| FR-7 | **Subscription/routing** — alert diarahkan ke orang/grup relevan (Traffic → tim editorial; Revenue → tim commercial). |
| FR-8 | **Feedback loop** — penerima bisa menandai alert "berguna / noise" untuk kalibrasi (lihat [plan.md](./plan.md)). |
| FR-9 | **Audit trail** — tiap evaluasi & keputusan kirim/suppress tercatat (`AlertEvent`). |

## 5. Non-Functional Requirements

| ID | Requirement | Target (contoh, kalibrasi) |
|----|-------------|----------------------------|
| NFR-1 | **Latency deteksi** — jeda kejadian → alert terkirim | Traffic/Revenue: ≤ 1 siklus cron (mis. ≤ 15–60 mnt); Competitor: ≤ 30–60 mnt |
| NFR-2 | **Signal-to-noise** — proporsi alert yang dinilai relevan | ≥ 80% alert ditandai berguna |
| NFR-3 | **Anti-spam** — batas volume alert per kanal | Rate limit per rule + global cap harian per subscriber |
| NFR-4 | **Idempotency** — evaluasi ulang tidak menghasilkan alert ganda untuk event sama | Dijamin oleh dedup key |
| NFR-5 | **Isolasi kredensial** — semua akses data via MCP, tidak ada kredensial di logika alert | Sesuai prinsip platform |
| NFR-6 | **Resilience** — kegagalan satu sumber data tidak menjatuhkan kategori lain | Subagent terisolasi; error di-log, tidak crash |

## 6. Success Metrics

| Metrik | Definisi | Target awal (kalibrasi) |
|--------|----------|-------------------------|
| **Relevansi alert** | % alert ditandai "berguna" oleh penerima | ≥ 80% |
| **False-positive rate** | % alert yang ternyata bukan anomaly nyata | ≤ 15% |
| **Adopsi** | % subscriber yang membaca/menindaklanjuti alert | ≥ 70% mingguan |
| **Time-to-awareness** | median jeda kejadian → tim tahu | turun ≥ 70% vs baseline manual |
| **Unsubscribe / mute rate** | proxy alert fatigue | mendekati 0 |

## 7. Asumsi

- MCP sumber sinyal (GA4, RSS, BigQuery, CMS, GSC) dari Phase 1 **sudah aktif & stabil**. Phase 3 tidak dimulai sebelum ini.
- Cron & Subagent Hermes berfungsi sesuai docs ([hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/)).
- Data historis cukup untuk membentuk baseline (mis. ≥ beberapa minggu untuk pola harian/mingguan).
- Penerima alert terdefinisi (nomor/grup WA) dan setuju menerima notifikasi.

## 8. Risiko

| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| **Alert fatigue** — terlalu banyak alert → diabaikan | Platform kehilangan kredibilitas | Anomaly-based only; suppression; rate limit; feedback loop; mulai dari 1 kategori |
| **False positive** — threshold terlalu sensitif | Noise, tim terganggu | Kalibrasi bertahap; baseline adaptif; warm-up shadow mode (deteksi tanpa kirim) |
| **False negative** — threshold terlalu longgar | Anomaly nyata terlewat | Review berkala metrik; tuning dua arah |
| **Baseline drift** — pola musiman/event tidak terakomodasi | Alert salah saat hari besar/libur | Baseline per hari-dalam-minggu; flag event kalender |
| **Ketergantungan MCP** — sumber data down | Kategori alert mati senyap | Health check + alert meta ("source X tidak terjangkau") |

## 9. Prinsip Pemandu

> **Setiap notifikasi harus berbasis anomaly atau opportunity — bukan laporan rutin yang berpotensi spam.**
>
> Signal > noise. User harus merasa **terbantu, bukan terganggu.** Bila ragu apakah suatu alert layak dikirim, default-nya: **jangan kirim.**
