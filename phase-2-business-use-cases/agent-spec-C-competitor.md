# Use Case C — Competitor Intelligence Agent — Phase 2

> Parent: [`./prd.md`](./prd.md) · Workflow: [`./workflow.md`](./workflow.md) · MCP: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

## Objective

Memantau topik yang berkembang di media kompetitor — meningkatkan awareness tren dan mengurangi keterlambatan editorial.

## Target User

Tim **Editorial** / redaksi. Persona: Editorial. Rilis **Wave 1** (single-MCP, integrasi ringan, sumber publik).

## MCP Dependency

- **RSS MCP** — menarik artikel dari feed media kompetitor.
- *(Opsional)* CMS/Search MCP untuk skill `gap-vs-kly` (cek apa yang sudah/belum ditulis KLY). Bila belum siap, `gap-vs-kly` jalan dengan heuristik judul.

## Contoh Percakapan (input → output)

```text
User:  Topik apa yang sedang ramai?
Agent: [fetch-feeds] [cluster-topics]
       Tren 24 jam (dari RSS): Topik X — 14 artikel/6 media · Topik Y — 9/4 · Topik Z — 7/3.
```

```text
User:  Apa yang belum ditulis KLY?
Agent: [cluster-topics] [gap-vs-kly]
       Ramai di kompetitor, belum di KLY: Topik Y (9 artikel/4 media) — peluang editorial.
       Mau draft 3 angle untuk Topik Y?
```

```text
User:  Topik apa yang muncul di banyak media sekaligus?
Agent: [cross-media-overlap]
       Muncul di ≥5 media: Topik X (6 media), Topik W (5 media). Sinyal kuat.
```

## Skills yang Dibutuhkan

| Skill | Scope | Fungsi |
|-------|-------|--------|
| `fetch-feeds` | domain | Tarik artikel dari RSS MCP (window waktu) |
| `cluster-topics` | domain | Kelompokkan artikel jadi topik |
| `cross-media-overlap` | domain | Hitung berapa media membahas tiap topik |
| `gap-vs-kly` | domain | Topik ramai yang belum ditulis KLY |
| `resolve-timeframe` | common | "24 jam/hari ini/minggu ini" → window |
| `format-whatsapp-reply` | common | Format jawaban WA |
| `cite-source` | common | Tandai "dari RSS" |

> `cluster-topics` & `cross-media-overlap` adalah tugas memproses banyak feed — kandidat dijalankan via **subagents** Hermes (paralel).

## Context File Outline

```text
1. Persona & objective — radar tren editorial
2. Daftar media kompetitor — sumber feed yang dipantau
3. Topik fokus KLY — kategori/vertikal relevan
4. Definisi "tren/ramai" — ambang jumlah artikel & jumlah media
5. Definisi "gap" — ramai di kompetitor, absen di KLY
6. Format jawaban — topik + jumlah artikel + jumlah media + saran aksi
7. Aturan grounding — hanya dari feed; tidak mengarang topik
8. Error handling — feed down/kosong → laporkan cakupan parsial
```

## Success Metric

- **Utama:** lead time deteksi tren editorial turun.
- **Pendukung:** jumlah ide artikel dari Agent yang dipakai; cakupan media kompetitor termonitor.

## Edge Case & Batasan

- **Feed sebagian down** → jawab dengan disclaimer cakupan parsial, sebut media yang gagal.
- **Topik duplikat/judul mirip** → dedup di `cluster-topics`.
- **Clickbait/noise** → ambang minimal artikel+media untuk disebut "tren".
- **Bahasa campuran** → clustering robust lintas bahasa.
- **`gap-vs-kly` tanpa CMS MCP** → tandai hasil sebagai estimasi (heuristik judul), bukan pasti.
- **Window terlalu lebar** → konfirmasi sebelum tarik banyak feed.
