# Basecamp Brief — Phase 2: Business Use Cases

> Format Shape Up pitch. Sumber: graph `graphify-out` (community 3 Story+, 6 Revenue, 8 Competitor, 0 Hermes framework) + `phase-2-business-use-cases/*.md`.

## 🔴 Problem Statement

Tim Editorial baru sadar kompetitor sudah menerbitkan artikel soal isu yang sedang viral — tiga jam setelah ramai. Tim Commercial baru tahu revenue satu inventory anjlok saat tutup buku bulanan. Insight selalu datang terlambat dan lewat dashboard yang harus dibuka manual. Story+ microsite diproduksi tangan demi tangan dari brief, asset, sampai Figma — makan waktu, kualitas tidak konsisten.

Fondasi (Phase 0) dan akses data (Phase 1) sudah ada, tapi **belum ada satu pun agent** yang mengubahnya jadi nilai bisnis. Platform masih sunyi.

## ⏳ Budget

- **Resources:** 1–2 engineer + 1 domain champion per agent (Editorial/Analytics/Commercial)
- **Time Box:** **per-wave, bukan sekaligus.** Wave 1 (B + C, single-MCP) lebih cepat; Wave 2 (A, produksi + approval); Wave 3 (D, multi-MCP join) paling berat.
- Tiap agent mewarisi template agent yang sama → agent berikutnya lebih cepat.

## 💡 Potential Solution

Bangun **4 agent** sebagai kombinasi **Context File + Skills + MCP tools** di atas Hermes (memory, subagents, programmatic tool calling sudah bawaan):

| Agent | Objective | MCP |
|-------|-----------|-----|
| **B — Analytics Insight** | Insight GA4 via chat | GA4 |
| **C — Competitor Intelligence** | Radar tren media kompetitor | RSS |
| **A — Story+ Production** | Otomasi produksi microsite | Figma, File/Storage, Generation |
| **D — Revenue & Inventory** | Insight monetization conversational | BigQuery, CMS, GA4, GSC |

Rilis bertahap: **B & C dulu** (cepat, value langsung), lalu A, lalu D. Pola reusable antar-agent: template + skill umum + eval harness.

## 📈 Estimation Impact

Outcome terukur per agent (baseline diambil saat UAT, bukan dikarang):
- **B:** waktu cari insight turun (dari buka dashboard → satu pesan WA)
- **C:** keterlambatan editorial terhadap tren menurun
- **A:** waktu produksi Story+ turun, konsistensi kualitas naik
- **D:** investigasi performa inventory & revenue lebih cepat
- ✅ Agent dipakai tim nyata, bukan demo

## 🕳️ Rabbit Holes

- **Halusinasi angka** — agent analytics/revenue WAJIB grounding ke MCP (Tool Grounding, no hallucinated numbers); jangan biarkan LLM mengarang metrik
- **Join multi-MCP (Use Case D)** — menggabungkan revenue (BigQuery) + page (CMS) + traffic (GA4) lewat URL/slug; mismatch key = jawaban salah diam-diam
- **Intent routing** — menentukan pesan user masuk ke agent mana tanpa salah sambung
- **Human-in-the-loop Story+** — approval gate produksi A jangan jadi bottleneck atau malah dilewati

## 🚫 No-Gos

- ❌ Membangun MCP baru di sini — itu Phase 1 (Phase 2 hanya **memakai** MCP)
- ❌ Proactive/scheduled alert → **Phase 3** (Phase 2 murni **reactive**: user tanya → agent jawab)
- ❌ Agent ke-5/6 (Search, Editorial, dst) sebelum 4 agent inti terbukti dipakai
- ❌ Akses data langsung tanpa MCP
- ❌ Mengganti kapabilitas Hermes (memory/subagents) dengan buatan sendiri

---

**Dependensi:** tiap agent butuh MCP dari Phase 1 sudah aktif (graph: RSS MCP → enables → Use Case C; dst). Spec detail: `agent-spec-A/B/C/D-*.md`.
