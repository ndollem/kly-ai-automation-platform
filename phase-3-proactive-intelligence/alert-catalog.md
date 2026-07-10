# Alert Catalog — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Glossary](../00-glossary.md)
> Lihat juga: [prd.md](./prd.md) · [erd.md](./erd.md) · [MCP Catalog (Phase 1)](../phase-1-data-access/mcp-catalog.md)

Katalog alert konkret per kategori. Setiap alert = satu `AlertRule` ([erd.md](./erd.md)).

> **Catatan kalibrasi:** semua angka threshold/window/cooldown di bawah adalah **contoh awal yang HARUS dikalibrasi** lewat shadow mode + feedback loop ([plan.md](./plan.md)). Bukan nilai final.

Legenda prioritas: **High** (aksi cepat, dampak besar) · **Medium** · **Low** (informatif).

---

## A. Competitor Alerts — sumber: RSS MCP

Berbasis teks → butuh topic clustering / relevance scoring, bukan sekadar angka vs threshold.

### A1. Topik Muncul di Banyak Media
| Aspek | Detail |
|-------|--------|
| **Metric** | jumlah media berbeda yang mengangkat satu topik/cluster dalam window |
| **Kondisi pemicu (contoh)** | ≥ N media (mis. ≥ 5) menerbitkan topik sama dalam ≤ 2 jam, dan KLY belum mengangkatnya |
| **Tipe** | Opportunity |
| **Prioritas** | High |
| **Contoh pesan** | "📈 Topik *\<judul cluster\>* sudah diangkat 6 media dalam 2 jam terakhir. KLY belum. Pertimbangkan angle: …" |

### A2. Topik Sedang Viral
| Aspek | Detail |
|-------|--------|
| **Metric** | laju penyebaran topik (akselerasi jumlah artikel/mention dari waktu ke waktu) |
| **Kondisi pemicu (contoh)** | laju penerbitan topik > X× baseline dalam window pendek (lonjakan cepat) |
| **Tipe** | Opportunity |
| **Prioritas** | High |
| **Contoh pesan** | "🔥 *\<topik\>* viral — penyebaran 3× lebih cepat dari biasanya. Window aksi sempit." |

### A3. Kompetitor Lebih Cepat Mengangkat Isu
| Aspek | Detail |
|-------|--------|
| **Metric** | selisih waktu antara terbit kompetitor vs status KLY pada topik bernilai |
| **Kondisi pemicu (contoh)** | kompetitor utama menerbitkan topik relevan-tinggi & KLY belum dalam ≥ T menit |
| **Tipe** | Opportunity / gap |
| **Prioritas** | Medium |
| **Contoh pesan** | "⏱️ \<Kompetitor\> sudah terbit soal *\<topik\>* 40 mnt lalu; KLY belum ada. Skor relevansi: tinggi." |

---

## B. Traffic Alerts — sumber: GA4 MCP

Kategori **pilot** (Stage 1). Sinyal numerik, baseline per hari/jam.

### B1. Traffic Turun Signifikan
| Aspek | Detail |
|-------|--------|
| **Metric** | sessions/pageviews per window vs `MetricBaseline` (per hari-dalam-minggu/jam) |
| **Kondisi pemicu (contoh)** | aktual < baseline − 30% DAN melewati absolute guard (volume cukup besar) |
| **Tipe** | Anomaly |
| **Prioritas** | High |
| **Contoh pesan** | "⚠️ Traffic turun 45% vs normal Rabu sore. Cek kemungkinan: outage, perubahan distribusi, atau isu indeks." |

### B2. Traffic Naik Tidak Normal
| Aspek | Detail |
|-------|--------|
| **Metric** | sessions/pageviews per window vs baseline |
| **Kondisi pemicu (contoh)** | aktual > baseline + X% (mis. +50%) di luar pola musiman |
| **Tipe** | Opportunity |
| **Prioritas** | Medium |
| **Contoh pesan** | "📊 Traffic naik 60% di atas normal. Identifikasi sumber & pertimbangkan amplifikasi konten terkait." |

### B3. Lonjakan Artikel Tertentu
| Aspek | Detail |
|-------|--------|
| **Metric** | pageviews per artikel vs baseline artikel (atau laju akselerasi) |
| **Kondisi pemicu (contoh)** | satu artikel melonjak > Y× rata-rata jam-jam sebelumnya, dengan volume minimum |
| **Tipe** | Opportunity |
| **Prioritas** | High |
| **Contoh pesan** | "🚀 Artikel *\<judul\>* melonjak 5× dalam 30 mnt. Pertimbangkan: push homepage, follow-up, atau monetisasi." |

---

## C. Revenue Alerts — sumber: BigQuery MCP · CMS MCP · GSC MCP

Multi-source, sering korelatif. Routing → tim commercial.

### C1. Fill Rate Turun
| Aspek | Detail |
|-------|--------|
| **Sumber** | BigQuery MCP (data monetisasi/OMP) |
| **Metric** | fill rate (impresi terisi / tersedia) vs baseline |
| **Kondisi pemicu (contoh)** | fill rate < baseline − 20% pada inventory bernilai |
| **Tipe** | Anomaly |
| **Prioritas** | High |
| **Contoh pesan** | "⚠️ Fill rate turun ke 62% (normal ~85%) di \<unit/section\>. Potensi revenue loss. Cek demand/ad source." |

### C2. CTR Menurun
| Aspek | Detail |
|-------|--------|
| **Sumber** | BigQuery MCP (ad CTR) dan/atau GSC MCP (search CTR) |
| **Metric** | CTR vs baseline |
| **Kondisi pemicu (contoh)** | CTR < baseline − 25% berkelanjutan di atas satu window |
| **Tipe** | Anomaly |
| **Prioritas** | Medium |
| **Contoh pesan** | "📉 CTR \<unit/query\> turun 30% vs normal. Cek penempatan, kreatif, atau perubahan SERP." |

### C3. Inventory Bermasalah
| Aspek | Detail |
|-------|--------|
| **Sumber** | CMS MCP + BigQuery MCP (status slot/unit) |
| **Metric** | unit aktif/error/kosong vs konfigurasi normal |
| **Kondisi pemicu (contoh)** | unit bernilai blank/error, atau jumlah unit aktif < ekspektasi |
| **Tipe** | Anomaly |
| **Prioritas** | High |
| **Contoh pesan** | "🛑 Ad unit *\<id\>* blank/error sejak 14:10 di \<section\>. Inventory bernilai tidak ter-monetisasi." |

---

## D. Ringkasan & Pola Bersama

| ID | Kategori | Sumber MCP | Tipe | Prioritas |
|----|----------|-----------|------|-----------|
| A1 | Competitor | RSS | Opportunity | High |
| A2 | Competitor | RSS | Opportunity | High |
| A3 | Competitor | RSS | Opportunity | Medium |
| B1 | Traffic | GA4 | Anomaly | High |
| B2 | Traffic | GA4 | Opportunity | Medium |
| B3 | Traffic | GA4 | Opportunity | High |
| C1 | Revenue | BigQuery | Anomaly | High |
| C2 | Revenue | BigQuery / GSC | Anomaly | Medium |
| C3 | Revenue | CMS / BigQuery | Anomaly | High |

**Pola format pesan (semua alert):** [emoji + tingkat] → [apa yang terjadi + angka vs normal] → [konteks/lokasi] → [saran aksi singkat] → [opsi feedback 👍/👎].

**Aturan bersama:**
- Tiap alert wajib punya `cooldown` (anti-duplikat) dan tunduk pada rate-limit global.
- Threshold = **relative + absolute guard** (hindari alert dari angka kecil).
- Default = **diam**; hanya yang lolos threshold & dedup yang dikirim ([workflow.md](./workflow.md)).
