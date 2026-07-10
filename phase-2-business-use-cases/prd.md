# PRD — Phase 2: Business Use Cases

> Parent: [`../README.md`](../README.md) · Glossary: [`../00-glossary.md`](../00-glossary.md) · MCP layer: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md) (Phase 1)

## 1. Ringkasan

Phase 2 menghasilkan **nilai bisnis nyata** dengan mengimplementasikan empat **Agent** prioritas di atas Hermes. Setiap Agent diwujudkan sebagai kombinasi **Context File** (instruksi domain) + **Skills** (procedural memory) + **MCP tools** (akses data) — bukan runtime baru. Hermes menyediakan gateway WhatsApp, memory lintas sesi, subagents, dan execution framework secara built-in.

Empat Agent:

| Kode | Agent | Domain | MCP Dependency |
|------|-------|--------|----------------|
| A | Story+ Production Agent | Editorial/Production | Figma MCP, File/Storage MCP, Story+ Generation Services |
| B | Analytics Insight Agent | Analytics | Google Analytics MCP |
| C | Competitor Intelligence Agent | Editorial | RSS MCP |
| D | Revenue & Inventory Intelligence Agent | Commercial | BigQuery MCP, CMS MCP, Analytics MCP, Search Console MCP |

## 2. Tujuan Bisnis

- **Mengurangi waktu produksi & analisa.** Tugas yang kini butuh banyak dashboard/tools diselesaikan lewat satu percakapan WhatsApp.
- **Mempercepat keputusan.** Insight (traffic, tren kompetitor, revenue inventory) tersedia on-demand, bukan menunggu laporan.
- **Meningkatkan konsistensi kualitas.** Produksi Story+ mengikuti template & quality gate yang sama.
- **Membangun platform reusable.** Skills & MCP yang dibuat di satu Agent dipakai ulang Agent lain dan Phase 3.

## 3. Persona Pengguna

| Persona | Tim | Agent utama | Kebutuhan inti |
|---------|-----|-------------|----------------|
| **Editorial** | Redaksi / content | C, A | Tahu tren kompetitor lebih cepat; mempercepat produksi microsite |
| **Analytics** | Data / audience | B, D | Akses metrik tanpa buka GA4; investigasi anomali |
| **Commercial** | Sales / monetization | D | Tahu inventory & halaman penghasil revenue tertinggi; investigasi performa |
| **Producer** | Tim Story+ | A | Generate microsite dari brief + asset tanpa langkah manual berulang |

Catatan: satu orang bisa memakai beberapa Agent. Routing intent (lihat [`architecture.md`](./architecture.md)) menentukan Agent mana yang menangani pesan.

## 4. Lingkup (Scope)

**In scope**

- Context File + Skills + MCP wiring untuk Agent A/B/C/D.
- Prompt/conversation design, testing, UAT, rollout ke tim terkait.
- Template Agent reusable + eval harness (lintas-agent).

**Out of scope (di fase lain)**

- Deploy/hardening Hermes, gateway WhatsApp, auth (Phase 0).
- Pembangunan MCP server itu sendiri (Phase 1, on-demand).
- Proactive alert / cron (Phase 3).

## 5. Functional Requirements (umum semua Agent)

- **FR-1 Conversational I/O.** Agent menerima pertanyaan natural via WhatsApp dan menjawab ringkas, actionable.
- **FR-2 Tool grounding.** Setiap klaim data berasal dari MCP tool; tidak ada angka karangan.
- **FR-3 Skill invocation.** Tugas berulang dijalankan lewat Skill, bukan prompt ad-hoc setiap kali.
- **FR-4 Context awareness.** Agent memakai Hermes memory untuk follow-up ("bandingkan dengan kemarin") tanpa user mengulang konteks.
- **FR-5 Clarification.** Bila pertanyaan ambigu/parameter kurang (rentang tanggal, properti GA4), Agent bertanya balik, bukan menebak.
- **FR-6 Graceful failure.** Bila MCP error/timeout, Agent menjelaskan keterbatasan + sarankan langkah, tidak diam atau halusinasi.
- **FR-7 Source transparency.** Jawaban menyebut sumber (mis. "dari GA4", "dari BigQuery") agar bisa diverifikasi.

## 6. Non-Functional Requirements

| Kategori | Requirement |
|----------|-------------|
| **Latensi** | Pertanyaan single-MCP: jawaban dalam hitungan detik–puluhan detik (target, bergantung MCP). Multi-MCP (D): boleh lebih lama, beri status bila perlu. |
| **Akurasi** | Angka harus konsisten dengan sumber. Diskrepansi = bug prioritas tinggi. |
| **Keamanan** | Kredensial sumber data terisolasi di MCP server, tidak terekspos ke percakapan/Context File. Auth di layer Hermes. |
| **Reusability** | Skill & pola prompt dapat dipakai ulang antar-agent (lihat template Agent). |
| **Observability** | Tiap request ter-log (audit trail Hermes); error MCP tercatat untuk debugging. |
| **Skalabilitas** | Penambahan Agent baru tidak memodifikasi Agent lain (isolasi Context File). |
| **Maintainability** | Context File & Skills versioned; perubahan domain tidak butuh redeploy Hermes. |

## 7. Success Metrics (indikator, ditetapkan baseline saat UAT)

> Angka konkret ditetapkan setelah baseline diukur; di bawah ini **indikator terukur**, bukan target yang dikarang.

| Agent | Metric utama | Indikator pendukung |
|-------|--------------|---------------------|
| **A — Story+** | Waktu produksi per microsite (turun vs baseline) | Jumlah revisi manual; konsistensi quality gate; lead time campaign |
| **B — Analytics** | Jumlah query/minggu via WA; waktu ke-insight (turun) | % pertanyaan terjawab tanpa buka dashboard; akurasi vs GA4 |
| **C — Competitor** | Lead time deteksi tren editorial (turun) | Jumlah ide artikel dari Agent yang dipakai; cakupan media kompetitor |
| **D — Revenue** | Waktu investigasi performa inventory (turun) | Jumlah keputusan optimasi yang didukung Agent; akurasi vs BigQuery |

Metric platform lintas-agent: **adopsi** (active users/Agent), **retensi** (repeat usage), **deflection** (pertanyaan terselesaikan tanpa eskalasi manual).

## 8. Asumsi

- Phase 0 selesai: Hermes live, WhatsApp tersambung, auth & logging aktif.
- MCP yang dibutuhkan tiap Agent sudah/akan tersedia (Phase 1 on-demand) — lihat urutan rilis di [`plan.md`](./plan.md).
- Tim terkait bersedia memakai WhatsApp sebagai entry point dan ikut UAT.
- Sumber data (GA4, GSC, BigQuery, CMS, RSS, Figma) dapat diakses via service account/kredensial yang disediakan tim infra.

## 9. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| MCP belum siap saat Agent dikembangkan | Agent terblokir | Urutkan rilis Agent mengikuti kesiapan MCP (B/C dulu, single-MCP) |
| Halusinasi angka | Hilang kepercayaan | FR-2/FR-7 ketat; eval harness uji akurasi terhadap sumber |
| Multi-MCP join salah (Agent D) | Insight menyesatkan | Skill eksplisit untuk join key (URL/slug); validasi silang antar-MCP |
| Adopsi rendah | Nilai bisnis tak terwujud | Onboarding per tim, contoh pertanyaan siap pakai, champion per Agent |
| Story+ generation tidak deterministik | Output tak konsisten | Quality gate skill + human-in-the-loop approval sebelum publish |
| Ambiguitas pertanyaan | Jawaban salah konteks | FR-5 clarification; default parameter eksplisit di Context File |

## 10. Dependensi

- **Phase 0:** Hermes + WhatsApp + auth.
- **Phase 1:** MCP per Agent ([roadmap mapping](../00-roadmap.md)).
- **Lintas-fase:** Skills & Context File yang dibuat di Phase 2 menjadi fondasi sinyal Phase 3.
