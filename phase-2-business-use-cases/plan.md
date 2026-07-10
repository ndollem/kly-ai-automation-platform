# Plan — Phase 2: Business Use Cases

> Parent: [`../README.md`](../README.md) · Roadmap: [`../00-roadmap.md`](../00-roadmap.md) · MCP layer: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md)

## 1. Prinsip Eksekusi

- **Single-MCP dulu, multi-MCP belakangan.** Agent yang bergantung pada satu MCP lebih cepat live dan memberi nilai duluan.
- **Reusable sejak awal.** Skill & pola prompt pertama jadi template; Agent berikutnya menyalin, bukan mulai dari nol.
- **MCP on-demand.** Tidak menunggu semua MCP selesai. Tiap Agent dirilis saat MCP-nya siap (Phase 1 berjalan beriringan).
- **Human-in-the-loop di output berisiko.** Agent A (publish microsite) wajib approval sebelum go-live.

## 2. Urutan Rilis Agent (disarankan)

```text
Wave 1 (single-MCP, cepat)   Wave 2 (asset/produksi)   Wave 3 (multi-MCP, kompleks)
┌─────────────┐            ┌─────────────┐          ┌─────────────────────┐
│ B Analytics   │            │ A Story+      │          │ D Revenue & Inventory │
│ C Competitor  │            │ Production    │          │ (BQ+CMS+GA4+GSC)      │
└───┬───────┘            └───┬───────┘          └─────────┬───────────┘
        │ GA4 MCP / RSS MCP          │ Figma+Storage MCP            │ 4 MCP join
        ▼                            ▼                              ▼
   value tercepat              produksi terotomasi           insight terdalam
```

**Alasan urutan:**

1. **B (Analytics) + C (Competitor)** — masing-masing single-MCP (GA4, RSS), integrasi ringan, value tinggi & cepat terbukti. Jadi tempat menempa **template Agent** dan **eval harness**.
2. **A (Story+ Production)** — butuh Figma + File/Storage MCP + Story+ Generation Services; lebih banyak moving parts (asset, generation, approval) tapi masih terbatas pada satu workflow.
3. **D (Revenue & Inventory)** — paling kompleks: empat MCP dengan join lintas-sumber (URL/slug sebagai kunci). Dirilis terakhir agar pola join sudah matang dan template stabil.

> Urutan adalah saran; sesuaikan dengan prioritas bisnis berjalan. Bila Commercial mendesak, D bisa didahulukan dengan menerima kompleksitas join lebih awal.

## 3. Dependensi ke Phase 1 (MCP)

| Wave | Agent | MCP yang harus siap | Catatan |
|------|-------|---------------------|---------|
| 1 | B | Google Analytics MCP | Single-source, query read-only |
| 1 | C | RSS MCP | Sumber publik, integrasi ringan |
| 2 | A | Figma MCP, File/Storage MCP, Story+ Generation Services | Generation service bisa internal API |
| 3 | D | BigQuery MCP, CMS MCP, Analytics MCP, Search Console MCP | Butuh kunci join konsisten antar-sumber |

Bila MCP belum siap, Agent dikembangkan dengan **mock/stub MCP** agar Context File & Skills tetap maju, lalu di-wire saat MCP live.

## 4. Pola Reusable Antar-Agent

Dibangun saat Wave 1, dipakai semua Agent berikutnya:

- **Template Agent** — kerangka Context File standar (objective, persona, batasan, format jawaban, error handling, clarification policy).
- **Skill umum:**
  - `format-whatsapp-reply` — ringkas jawaban agar enak dibaca di WA (bullet, angka tebal, link).
  - `resolve-timeframe` — normalisasi "hari ini / minggu ini / kemarin" → rentang tanggal eksplisit.
  - `clarify-missing-params` — tanya balik bila parameter wajib kurang.
  - `cite-source` — sisipkan asal data ke jawaban.
- **Eval harness** — set pertanyaan + expected behavior per Agent; uji akurasi (vs sumber), grounding (tidak halusinasi), dan format.
- **Intent routing config** — peta keyword/pola → Agent (lihat [`architecture.md`](./architecture.md)).

## 5. Strategi Rollout & Adopsi

```text
Build ─▶ Internal test ─▶ UAT (tim domain) ─▶ Pilot (champion) ─▶ Rollout penuh ─▶ Iterate
```

- **Champion per Agent.** Satu PIC tiap tim (Editorial/Analytics/Commercial) jadi early adopter & feedback loop.
- **Onboarding ringan.** Kartu "contoh pertanyaan" siap pakai (ambil dari Example Questions tiap spec) dibagikan via WA.
- **UAT terukur.** Bandingkan output Agent vs cara lama; catat baseline metric (waktu, akurasi) untuk success metric PRD.
- **Iterasi mingguan.** Kumpulkan pertanyaan yang gagal/ambigu → perbaiki Context File/Skill, bukan tunggu rilis besar.

## 6. Milestone Phase 2

| Milestone | Exit criteria |
|-----------|---------------|
| M1 — Wave 1 live | B & C dipakai tim; template Agent + eval harness ada |
| M2 — Wave 2 live | A menghasilkan microsite dengan approval gate |
| M3 — Wave 3 live | D menjawab pertanyaan multi-MCP dengan join akurat |
| M4 — Adopsi | Tiap Agent punya active users & baseline metric terukur |

## 7. Risiko Eksekusi

- **MCP slip** → Agent terblokir. Mitigasi: stub MCP, urutan fleksibel.
- **Template terlambat** → tiap Agent reinvent. Mitigasi: kunci template di akhir Wave 1 sebelum mulai A.
- **Join Agent D rumit** → insight salah. Mitigasi: prototipe join key di awal Wave 3, validasi silang.
