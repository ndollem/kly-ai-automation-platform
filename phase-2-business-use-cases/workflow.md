# Workflow — Phase 2: Business Use Cases

> Parent: [`../README.md`](../README.md) · Architecture: [`./architecture.md`](./architecture.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

Alur conversational umum tiap Agent, lalu contoh percakapan nyata per Agent memakai Example Questions.

## 1. Alur Umum

```text
┌────────┐   pesan    ┌────────────────────────────┐
│  User    │ ─────────▶ │  HERMES (Orchestration)             │
│ WhatsApp │            │  • session + memory                  │
└────────┘ ◀───────── │  • intent routing → pilih Agent      │
     ▲      jawaban     └───────────────┬─────────────┘
     │                                  │ aktifkan Agent
     │                  ┌─────────────▼─────────────┐
     │                  │  AGENT (Context File + Skills)       │
     │                  │  • interpretasi maksud               │
     │                  │  • pilih Skill                       │
     │                  │  • (clarify bila parameter kurang)   │
     │                  └───────────────┬─────────────┘
     │                                  │ panggil tool
     │                  ┌─────────────▼─────────────┐
     │                  │  MCP TOOLS (GA4 / RSS / BQ / Figma…) │
     │                  └───────────────┬─────────────┘
     │                                  │ data
     │                  ┌─────────────▼─────────────┐
     └───────────── │  Agent susun jawaban + cite source   │
                        │  format-whatsapp-reply               │
                        └─────────────────────┘
```

Catatan: follow-up ("bandingkan dengan kemarin") memanfaatkan **Hermes memory** — user tak perlu mengulang konteks. Tugas berat/paralel (mis. Agent A menarik banyak asset, Agent C memproses banyak feed) dapat di-spawn sebagai **subagents**.

## 2. Agent B — Analytics Insight

```text
User ─► Hermes ─► Analytics Agent ─► Google Analytics MCP ─► jawaban
```

**Contoh 1 — Traffic hari ini**

```text
User:  Traffic hari ini berapa?
Agent: [resolve-timeframe → 2026-06-25] [query-ga4: activeUsers, sessions]
       Hari ini (25 Jun, s/d sekarang) — dari GA4:
       • Users: 48.2K  • Sessions: 61.7K
       Naik ~6% vs kemarin jam yang sama. Mau breakdown per channel?
```

**Contoh 2 — Artikel teratas + investigasi (follow-up pakai memory)**

```text
User:  Artikel teratas minggu ini?
Agent: [resolve-timeframe → 19–25 Jun] [query-ga4: top pages by views]
       Top minggu ini (dari GA4):
       1. "…" — 312K views
       2. "…" — 188K
       3. "…" — 154K
User:  Kenapa traffic turun?
Agent: [compare-period] [explain-anomaly]
       Total minggu ini −9% vs minggu lalu. Penurun utama: channel Organic
       Search (−14%). Direct & Referral stabil. Indikasi dari sisi search —
       mau aku cek Search Console juga (lewat Revenue Agent)?
```

**Contoh 3 — Channel bertumbuh**

```text
User:  Channel mana yang bertumbuh?
Agent: [query-ga4: sessions by channel, WoW] 
       Tumbuh WoW (dari GA4): Referral +18%, Organic Social +11%.
       Turun: Organic Search −14%. Email datar.
```

## 3. Agent C — Competitor Intelligence

```text
User ─► Hermes ─► Competitor Agent ─► RSS MCP ─► (clustering) ─► jawaban
```

**Contoh 1 — Topik ramai**

```text
User:  Topik apa yang sedang ramai?
Agent: [fetch-feeds] [cluster-topics]
       Tren 24 jam terakhir di media kompetitor (dari RSS):
       • Topik X — 14 artikel, 6 media
       • Topik Y — 9 artikel, 4 media
       • Topik Z — 7 artikel, 3 media
```

**Contoh 2 — Gap vs KLY**

```text
User:  Apa yang belum ditulis KLY?
Agent: [cluster-topics] [gap-vs-kly]
       Ramai di kompetitor tapi belum ada di KLY:
       • Topik Y (9 artikel, 4 media) — peluang editorial
       • Sub-tema dari Topik X (angle "…") 
       Mau aku draft 3 angle artikel untuk Topik Y?
```

**Contoh 3 — Overlap lintas media**

```text
User:  Topik apa yang muncul di banyak media sekaligus?
Agent: [cross-media-overlap]
       Muncul di ≥5 media bersamaan (dari RSS):
       • Topik X — 6 media  • Topik W — 5 media
       Sinyal kuat; layak diprioritaskan.
```

## 4. Agent D — Revenue & Inventory Intelligence

```text
User ─► Hermes ─► Revenue Agent ─┬─► BigQuery MCP
                                 ├─► CMS MCP
                                 ├─► Analytics MCP
                                 └─► Search Console MCP
                          (join via URL/slug) ─► jawaban
```

**Contoh 1 — Inventory perform**

```text
User:  Inventory mana yang perform?
Agent: [query-revenue-bq] [map-page-to-inventory via CMS]
       Top inventory (revenue 7 hari, dari BigQuery):
       1. Inventory A — Rp …  (eCPM tinggi)
       2. Inventory B — Rp …
       Inventory C underperform (fill rate rendah). Mau detail per halaman?
```

**Contoh 2 — Revenue per halaman**

```text
User:  Halaman mana yang menghasilkan revenue tertinggi?
Agent: [rank-revenue-pages] [join: BQ revenue × CMS title]
       Revenue tertinggi 7 hari (BigQuery × CMS):
       1. "…" — Rp …  
       2. "…" — Rp …
       3. "…" — Rp …
```

**Contoh 3 — Investigasi impression turun (multi-MCP)**

```text
User:  Kenapa impression turun?
Agent: [explain-impression-drop] [join-ga4-gsc]
       Impression −12% WoW. Pemecahan (BigQuery + GSC + GA4):
       • GSC: impression search −15% (beberapa query kehilangan posisi)
       • GA4: pageviews artikel terkait −10%
       • Inventory di halaman tsb ikut turun → revenue −8%
       Akar utama di sisi search visibility. Mau daftar query yang turun?
```

## 5. Agent A — Story+ Production (workflow produksi, bukan tanya-jawab)

```text
User ─► Hermes ─► Story+ Agent ─┬─► Figma MCP (desain)
                                ├─► File/Storage MCP (asset)
                                └─► Story+ Generation Services
                         ─► draft microsite ─► APPROVAL ─► publish
```

```text
User:  Buat Story+ dari brief campaign "Ramadan Sale" ini: [brief + link asset]
Agent: [parse-brief] [collect-assets via Storage] [pull-figma-design]
       Brief terbaca. Asset: 12 gambar, 1 video. Desain dari Figma "Ramadan v3".
       [generate-microsite] [quality-check]
       Draft microsite siap (quality gate: ✅ struktur, ✅ asset, ⚠ 1 alt-text kosong).
       Preview: [link]. Setujui untuk publish? (ya / revisi)
User:  Perbaiki alt-text lalu publish
Agent: [quality-check ulang] [request-approval ✅]
       Alt-text diperbaiki, quality gate hijau. Microsite dipublish: [link live].
```

> Angka pada contoh adalah ilustrasi format jawaban, bukan data nyata. Agent selalu mengambil angka dari MCP dan menyebut sumbernya (FR-2/FR-7 di [`prd.md`](./prd.md)).
