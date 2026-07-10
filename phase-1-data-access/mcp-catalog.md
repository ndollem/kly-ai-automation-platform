# MCP Catalog — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Dokumen pendukung [Plan](./plan.md), [Architecture](./architecture.md), [Task List](./task-list.md).
> Katalog tiap MCP server yang dibangun KLY. **Status & prioritas mengikuti [roadmap](../00-roadmap.md) — on-demand, bukan semua sekaligus.**

## Ringkasan

| MCP | Sumber data | Auth | Use Case | Prioritas | Status |
|-----|-------------|------|----------|-----------|--------|
| GA4 | Google Analytics 4 | service account | B, D | 1 | Planned |
| RSS | Feed kompetitor (publik) | none | C | 2 | Planned |
| Figma | Figma API | api key | A | 3 | Planned |
| File/Storage | GCS / storage KLY | service account | A | 3 | Planned |
| BigQuery | BigQuery | service account | D | 4 | Planned |
| Search Console | Google Search Console | service account | D | 4 | Planned |
| CMS | CMS internal KLY | api/db read-only | D | 4 | Planned |

---

## 1. Google Analytics (GA4) MCP

- **Sumber data:** GA4 property (GA4 Data API).
- **Tools (contoh):** `query_report`, `list_metrics`, `realtime`.
- **Auth:** Service account; akses GA4 property; via vault ref.
- **Use case:** B (Analytics Insight), juga dipakai D (Revenue).
- **Prioritas:** 1 — single-source, value tinggi, validasi template.
- **Dependensi:** GA4 Data API enabled; service account ter-grant ke property.
- **Caching:** TTL ~15 menit (metrics).

## 2. RSS MCP

- **Sumber data:** Feed RSS kompetitor (publik).
- **Tools (contoh):** `fetch_feed`, `list_items`, `since`.
- **Auth:** None (sumber publik).
- **Use case:** C (Competitor Intelligence).
- **Prioritas:** 2 — integrasi paling ringan, tanpa auth.
- **Dependensi:** Daftar URL feed kompetitor.
- **Caching:** TTL ~5 menit (item feed).

## 3. Figma MCP

- **Sumber data:** Figma API (file & nodes).
- **Tools (contoh):** `get_file`, `get_node`, `export_image`.
- **Auth:** Figma API key / personal access token; via vault ref.
- **Use case:** A (Story+ Production).
- **Prioritas:** 3 — pasangan dengan File/Storage MCP.
- **Dependensi:** API key + file/team key; hormati rate limit Figma.
- **Caching:** Node/file sesuai kebutuhan produksi.

## 4. File/Storage MCP

- **Sumber data:** Google Cloud Storage / storage internal KLY.
- **Tools (contoh):** `list_objects`, `read_object`, `write_object`.
- **Auth:** Service account / signed access; via vault ref.
- **Use case:** A (Story+ Production) — aset & file output.
- **Prioritas:** 3 — bersama Figma MCP.
- **Dependensi:** Akses bucket/storage; scope read/write sesuai kebutuhan.
- **Caching:** Opsional (metadata listing).

## 5. BigQuery MCP

- **Sumber data:** BigQuery dataset KLY.
- **Tools (contoh):** `run_query`, `list_tables`, `get_schema`.
- **Auth:** Service account; akses dataset; via vault ref.
- **Use case:** D (Revenue & Inventory).
- **Prioritas:** 4 — multi-source kompleks; dibangun setelah template matang.
- **Dependensi:** BigQuery API enabled; service account ter-grant; cost guard.
- **Caching:** Hasil query (TTL sesuai cost/freshness) untuk hindari rerun mahal.

## 6. Search Console (GSC) MCP

- **Sumber data:** Google Search Console property.
- **Tools (contoh):** `query_search_analytics`, `list_sites`, `inspect_url`.
- **Auth:** Service account terverifikasi di property; via vault ref.
- **Use case:** D (Revenue & Inventory).
- **Prioritas:** 4.
- **Dependensi:** Search Console API enabled; property terverifikasi.
- **Caching:** TTL sesuai latensi data GSC.

## 7. CMS MCP

- **Sumber data:** CMS internal KLY (konten/artikel).
- **Tools (contoh):** `list_content`, `get_content`, `search`.
- **Auth:** API/DB read-only; via vault ref.
- **Use case:** D (Revenue & Inventory).
- **Prioritas:** 4.
- **Dependensi:** Kontrak akses dengan tim CMS internal (read-only); kredensial terbatas.
- **Caching:** Sesuai frekuensi update konten.

---

## Catatan Lintas Katalog

- **Reusable:** Analytics MCP (GA4) melayani Use Case B **dan** D — dibangun sekali, dipakai lintas use case.
- **On-demand:** Status "Planned" diaktifkan hanya saat use case terkait di [Phase 2](../phase-2-business-use-cases/) siap.
- **Kredensial terisolasi:** Semua "via vault ref" — kredensial tidak pernah ke agent/percakapan (lihat [architecture.md §4](./architecture.md)).
