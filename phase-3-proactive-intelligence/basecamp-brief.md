# Basecamp Brief — Phase 3: Proactive Intelligence

> Format Shape Up pitch. Sumber: graph `graphify-out` (community 5 "Alert Engine Components", 7 "Traffic Alerts & Anti-Spam", 6 Revenue, 8 Competitor) + `phase-3-proactive-intelligence/*.md`.

## 🔴 Problem Statement

Traffic satu artikel andalan turun 40% sejak subuh. Tidak ada yang tahu sampai siang, saat seseorang kebetulan tanya ke Analytics Agent. Pola interaksi platform masih sepenuhnya **reactive** — agent hanya menjawab kalau ditanya. Masalahnya: orang tidak tahu apa yang harus ditanyakan, dan kapan. Anomali revenue, lonjakan kompetitor, dan drop traffic lewat begitu saja karena tidak ada yang sedang melihat dashboard pada menit yang tepat.

Phase 2 sudah punya agent yang bisa menjawab. Yang belum ada: agent yang **mendeteksi lebih dulu dan memberi tahu**.

## ⏳ Budget

- **Resources:** 1 engineer + tuning bareng domain owner (Editorial/Commercial)
- **Time Box:** **bertahap per kategori.** Mulai 1 kategori paling bernilai (Traffic) sebagai pilot + framework, lalu Competitor, lalu Revenue. Tiap kategori butuh waktu **kalibrasi threshold** + observasi, bukan sekadar coding.
- Stage 0 (framework + shadow mode) → Stage 1 Traffic → Stage 2 Competitor → Stage 3 Revenue → Stage 4 hardening.

## 💡 Potential Solution

Bangun **Alert Engine** di atas kapabilitas Hermes-native: **Cron** (scheduler bawaan) + **Subagents** (evaluasi paralel) + **MCP** (sumber sinyal dari Phase 1). Komponen: Orchestrator → Data Fetcher → Anomaly Detector → Suppression/Dedup → Delivery (ke WhatsApp).

Tiga kategori, 9 alert konkret:
- **Traffic** (GA4): drop signifikan, naik abnormal, lonjakan artikel
- **Competitor** (RSS): topik di banyak media, topik viral, kompetitor lebih cepat
- **Revenue** (BigQuery/CMS/GSC): fill rate turun, CTR turun, inventory bermasalah

**Guiding Principle:** alert hanya berbasis **anomaly atau opportunity** — bukan laporan rutin. Mulai dengan **shadow mode** (deteksi tanpa kirim) untuk kalibrasi sebelum benar-benar mengganggu user.

## 📈 Estimation Impact

- ✅ Signal > noise — alert relevan, false-positive rendah
- ✅ User merasa **terbantu, bukan terganggu** (tidak ada spam)
- ✅ Anomali ketahuan dalam menit, bukan jam/hari
- ✅ Adopsi: user membiarkan alert aktif, bukan mute

## 🕳️ Rabbit Holes

- **Alert fatigue** — terlalu banyak/terlalu cerewet → user mute semua → seluruh fase mati. Ini risiko nomor satu.
- **Kalibrasi threshold** — apa itu "turun signifikan"? Butuh baseline (MetricBaseline) + tuning per metric; angka di katalog cuma contoh
- **Dedup/suppression** — anomali yang sama jangan diteriakkan berulang tiap cron tick
- **Baseline musiman** — traffic Ramadan vs hari biasa beda; threshold statis bikin false positive

## 🚫 No-Gos

- ❌ **Laporan rutin terjadwal** (daily/weekly digest) — melanggar prinsip anomaly-based, jadi spam
- ❌ Membangun scheduler sendiri — pakai **Cron bawaan Hermes**
- ❌ Sumber data baru / MCP baru — pakai yang sudah ada dari Phase 1
- ❌ Alert untuk metric yang belum punya baseline jelas
- ❌ Mengirim ke produksi sebelum lulus **shadow mode** & feedback loop

---

**Dependensi:** butuh MCP (Phase 1) + agent reactive (Phase 2) sudah aktif sebagai sumber sinyal (graph: Cron → enables → Phase 3; Phase 3 → depends_on → Phase 2). Detail tiap alert + contoh pesan: `alert-catalog.md`.
