# Task List — Phase 2: Business Use Cases

> Parent: [`../README.md`](../README.md) · Plan: [`./plan.md`](./plan.md)
> Estimasi: **S** (≤1 hari) · **M** (beberapa hari) · **L** (≥1 minggu). Checkbox `- [ ]`.

## 0. Lintas-Agent (kerjakan saat Wave 1)

### Template & fondasi reusable

- [ ] (M) Susun **Template Agent** — kerangka Context File standar (objective, persona, batasan, format jawaban, error handling, clarification policy)
- [ ] (S) Skill `format-whatsapp-reply` — format jawaban ringkas untuk WA
- [ ] (S) Skill `resolve-timeframe` — normalisasi "hari ini/minggu ini/kemarin" → rentang tanggal
- [ ] (S) Skill `clarify-missing-params` — tanya balik bila parameter wajib kurang
- [ ] (S) Skill `cite-source` — sisipkan asal data ke jawaban
- [ ] (M) **Intent routing config** — peta keyword/pola → Agent A/B/C/D
- [ ] (M) **Eval harness** — set pertanyaan + expected behavior per Agent; cek akurasi, grounding, format
- [ ] (S) Konvensi versioning Context File & Skills (changelog per Agent)

## A. Story+ Production Agent — Wave 2

- [ ] (M) **Context File** — domain Story+, struktur microsite, quality gate, batasan publish
- [ ] (M) **Skills**: `parse-brief`, `collect-assets`, `pull-figma-design`, `generate-microsite`, `quality-check`, `request-approval`
- [ ] (M) **MCP wiring**: Figma MCP, File/Storage MCP, Story+ Generation Services
- [ ] (S) **Prompt/conversation design** — alur brief → asset → generate → approval
- [ ] (M) **Testing** — generate microsite dari brief sampel; verifikasi konsistensi quality gate
- [ ] (S) **Human-in-the-loop** — approval gate sebelum publish
- [ ] (M) **UAT** dengan tim Producer; ukur waktu produksi vs baseline
- [ ] (S) **Rollout** + kartu contoh brief

## B. Analytics Insight Agent — Wave 1

- [ ] (S) **Context File** — domain GA4, definisi metrik, properti default, format insight
- [ ] (S) **Skills**: `query-ga4`, `summarize-traffic`, `compare-period`, `explain-anomaly`
- [ ] (S) **MCP wiring**: Google Analytics MCP
- [ ] (S) **Prompt/conversation design** — Example Questions (traffic hari ini, artikel teratas, kenapa turun, channel bertumbuh)
- [ ] (S) **Testing** — akurasi jawaban vs GA4 dashboard
- [ ] (S) **UAT** dengan tim Analytics; ukur waktu ke-insight
- [ ] (S) **Rollout** + kartu contoh pertanyaan

## C. Competitor Intelligence Agent — Wave 1

- [ ] (S) **Context File** — daftar media kompetitor, topik fokus KLY, definisi "tren"
- [ ] (M) **Skills**: `fetch-feeds`, `cluster-topics`, `cross-media-overlap`, `gap-vs-kly`
- [ ] (S) **MCP wiring**: RSS MCP (+ opsional CMS/Search untuk gap-vs-KLY)
- [ ] (S) **Prompt/conversation design** — Example Questions (topik ramai, belum ditulis KLY, muncul lintas media)
- [ ] (S) **Testing** — verifikasi clustering & overlap dengan feed nyata
- [ ] (S) **UAT** dengan tim Editorial; ukur lead time deteksi tren
- [ ] (S) **Rollout** + kartu contoh pertanyaan

## D. Revenue & Inventory Intelligence Agent — Wave 3

- [ ] (M) **Context File** — domain monetization (OMP), definisi inventory/revenue, kunci join (URL/slug)
- [ ] (L) **Skills**: `query-revenue-bq`, `map-page-to-inventory`, `join-ga4-gsc`, `explain-impression-drop`, `rank-revenue-pages`
- [ ] (M) **MCP wiring**: BigQuery MCP, CMS MCP, Analytics MCP, Search Console MCP
- [ ] (M) **Join key prototype** — validasi konsistensi URL/slug lintas-sumber
- [ ] (S) **Prompt/conversation design** — Example Questions (inventory perform, revenue tertinggi, kenapa impression turun)
- [ ] (M) **Testing** — akurasi join vs query manual BigQuery
- [ ] (M) **UAT** dengan tim Commercial; ukur waktu investigasi
- [ ] (S) **Rollout** + kartu contoh pertanyaan

## Z. Penutup Fase

- [ ] (S) Catat baseline & hasil success metric per Agent (PRD §7)
- [ ] (S) Update [`../00-decision-log.md`](../00-decision-log.md) bila ada keputusan arsitektur baru
- [ ] (S) Serahkan sinyal/Skill reusable ke Phase 3 (proactive alerts)
