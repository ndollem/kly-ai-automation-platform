# Use Case A — Story+ Production Agent — Phase 2

> Parent: [`./prd.md`](./prd.md) · Workflow: [`./workflow.md`](./workflow.md) · MCP: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

## Objective

Mengotomasi pembuatan **Story+** microsite dari brief, content, asset, dan desain — mengurangi waktu produksi, meningkatkan konsistensi kualitas, dan mempercepat delivery campaign.

## Target User

Tim **Producer** Story+ (dan Editorial yang menyiapkan brief). Persona: Production. Rilis **Wave 2**.

## MCP Dependency

- **Figma MCP** — menarik desain/layout sumber.
- **File/Storage MCP** — mengambil & menyimpan asset (gambar, video, copy).
- **Story+ Generation Services** — service generate microsite (API internal).

## Contoh Percakapan (input → output)

```text
User:  Buat Story+ dari brief campaign "Ramadan Sale": [brief + link asset folder]
Agent: [parse-brief] [collect-assets] [pull-figma-design "Ramadan v3"]
       Brief terbaca. Asset: 12 gambar + 1 video. Desain: Figma "Ramadan v3".
       [generate-microsite] [quality-check]
       Draft siap. Quality gate: ✅ struktur ✅ asset ⚠ 1 alt-text kosong.
       Preview: [link]. Setujui publish? (ya / revisi)

User:  Perbaiki alt-text lalu publish
Agent: [quality-check ulang] [request-approval ✅]
       Alt-text diperbaiki, gate hijau. Dipublish: [link live].
```

```text
User:  Status produksi Ramadan Sale?
Agent: [lookup job via memory] Job #… — generated, menunggu approval (1 warning).
```

## Skills yang Dibutuhkan

| Skill | Scope | Fungsi |
|-------|-------|--------|
| `parse-brief` | domain | Ekstrak struktur, tujuan, CTA dari brief |
| `collect-assets` | domain | Ambil & validasi asset via Storage MCP |
| `pull-figma-design` | domain | Tarik desain/layout dari Figma MCP |
| `generate-microsite` | domain | Panggil Story+ Generation Services |
| `quality-check` | domain | Quality gate: struktur, asset lengkap, alt-text, link |
| `request-approval` | domain | Human-in-the-loop sebelum publish |
| `format-whatsapp-reply` | common | Format jawaban WA |
| `clarify-missing-params` | common | Tanya brief/asset bila kurang |

## Context File Outline

```text
1. Persona & objective — produser Story+, output microsite konsisten
2. Definisi Story+ — struktur microsite, komponen wajib
3. Quality gate — checklist lulus/gagal (struktur, asset, alt-text, link, mobile)
4. Sumber asset & desain — kapan pakai Figma vs Storage
5. Batasan publish — WAJIB approval; tidak auto-publish
6. Format jawaban — preview link + status gate + pertanyaan approval
7. Error handling — asset hilang, generation gagal, Figma tak terjangkau
8. Clarification policy — brief tak lengkap → tanya balik
```

## Success Metric

- **Utama:** waktu produksi per microsite turun vs baseline.
- **Pendukung:** jumlah revisi manual; konsistensi quality gate (rate lolos gate pertama kali); lead time delivery campaign.

> Baseline diukur saat UAT; angka tidak dikarang.

## Edge Case & Batasan

- **Asset tidak lengkap/korup** → Agent berhenti, minta asset benar (jangan generate sebagian).
- **Generation service down** → laporkan, simpan brief, tawarkan retry.
- **Quality gate fail** → tidak boleh masuk approval; jelaskan item gagal.
- **Tidak ada auto-publish** → publish hanya setelah approval eksplisit.
- **Desain Figma berubah** → tarik versi terbaru; konfirmasi versi ke user bila ambigu.
- **Brief multibahasa/aneh** → clarify, jangan asumsi.
