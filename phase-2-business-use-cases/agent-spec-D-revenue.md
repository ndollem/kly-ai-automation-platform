# Use Case D — Revenue & Inventory Intelligence Agent — Phase 2

> Parent: [`./prd.md`](./prd.md) · Workflow: [`./workflow.md`](./workflow.md) · Architecture: [`./architecture.md`](./architecture.md) · MCP: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

## Objective

Memberi insight **monetization & inventory** (OMP) secara conversational — membantu optimasi revenue dan mempercepat investigasi performa inventory.

## Target User

Tim **Commercial** / monetization. Persona: Commercial. Rilis **Wave 3** (paling kompleks — empat MCP dengan join lintas-sumber; dirilis setelah template & pola join matang).

## MCP Dependency

- **BigQuery MCP** — data revenue/inventory (eCPM, fill rate, impression iklan).
- **CMS MCP** — metadata halaman (judul, slug, kategori) untuk memetakan page → inventory.
- **Analytics MCP (GA4)** — pageviews/traffic per halaman.
- **Search Console MCP (GSC)** — impression/position search per query/halaman.

**Kunci join:** `URL/slug` konsisten lintas BigQuery × CMS × GA4 × GSC. Normalisasi & validasi join adalah risiko utama (lihat [`architecture.md`](./architecture.md) §5).

## Contoh Percakapan (input → output)

```text
User:  Inventory mana yang perform?
Agent: [query-revenue-bq] [map-page-to-inventory via CMS]
       Top inventory (revenue 7 hari, dari BigQuery): A — Rp… (eCPM tinggi), B — Rp…
       Underperform: C (fill rate rendah).
```

```text
User:  Halaman mana yang menghasilkan revenue tertinggi?
Agent: [rank-revenue-pages] [join BQ × CMS]
       Revenue tertinggi 7 hari (BigQuery × CMS): 1) "…" Rp… 2) "…" Rp… 3) "…" Rp…
```

```text
User:  Kenapa impression turun?
Agent: [explain-impression-drop] [join-ga4-gsc]
       Impression −12% WoW. GSC: search impression −15% (posisi turun).
       GA4: pageviews −10%. Dampak: revenue inventory −8%. Akar: search visibility.
```

## Skills yang Dibutuhkan

| Skill | Scope | Fungsi |
|-------|-------|--------|
| `query-revenue-bq` | domain | Query revenue/inventory dari BigQuery |
| `map-page-to-inventory` | domain | Map halaman ↔ inventory via CMS; normalisasi slug |
| `join-ga4-gsc` | domain | Gabungkan traffic (GA4) + search (GSC) per halaman |
| `rank-revenue-pages` | domain | Ranking halaman by revenue |
| `explain-impression-drop` | domain | Pecah penurunan impression (ad vs search) lintas sumber |
| `resolve-timeframe` | common | Normalisasi rentang |
| `format-whatsapp-reply` | common | Format jawaban WA |
| `cite-source` | common | Tandai sumber tiap angka (BQ/CMS/GA4/GSC) |
| `clarify-missing-params` | common | Tanya rentang/segmen bila ambigu |

> Pipeline multi-MCP cocok dijalankan via **Programmatic Tool Calling** (`execute_code`) Hermes — join dalam satu inference; sumber paralel via **subagents**.

## Context File Outline

```text
1. Persona & objective — analis revenue, jawaban actionable
2. Domain OMP — definisi inventory, revenue, eCPM, fill rate, impression (ad vs search)
3. Kunci join — aturan normalisasi URL/slug lintas sumber
4. Prioritas sumber — angka revenue dari BQ; traffic dari GA4; search dari GSC; metadata dari CMS
5. Disambiguasi "impression" — selalu perjelas ad-impression vs search-impression
6. Format jawaban — angka + sumber + akar masalah + saran
7. Aturan grounding — tidak menyimpulkan tanpa data; validasi silang sebelum kausalitas
8. Error handling — salah satu MCP gagal → jawab parsial + sebut yang hilang
9. Clarification policy — rentang/segmen/properti ambigu → tanya
```

## Success Metric

- **Utama:** waktu investigasi performa inventory turun.
- **Pendukung:** jumlah keputusan optimasi revenue yang didukung Agent; akurasi jawaban vs query manual BigQuery.

## Edge Case & Batasan

- **Join key tidak match** (slug beda antar sumber) → jangan paksa join; laporkan ketidakcocokan, jangan mengarang kausalitas.
- **Salah satu MCP down** → jawab parsial, sebut sumber yang hilang (jangan diam).
- **"Impression" ambigu** → selalu perjelas ad vs search.
- **Kausalitas vs korelasi** → "kenapa turun" dijawab sebagai indikasi berbukti, bukan klaim pasti.
- **Query BigQuery berat/mahal** → batasi rentang default; konfirmasi sebelum query besar.
- **Pertanyaan murni traffic** → boleh delegasi pola ke Agent B; revenue tetap di D.
