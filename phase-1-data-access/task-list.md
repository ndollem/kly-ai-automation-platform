# Task List — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat [Plan](./plan.md), [PRD](./prd.md).
> Estimasi: **S** (≤1 hari) · **M** (2–4 hari) · **L** (≥1 minggu). Checkbox `- [ ]`.

## A. Standar & Komponen Reusable (lintas MCP — dibangun sekali)

- [ ] Define **MCP contract template** (manifest, tool schema, input/output convention) — **M**
- [ ] Bangun **auth/secret pattern**: vault ref → credential resolver, redaction di log — **M**
- [ ] Bangun **error mapper** (auth / rate-limit / not-found / upstream / validation → error MCP) — **S**
- [ ] Bangun **cache layer** reusable (TTL policy + cache key strategy) — **M**
- [ ] Bangun **observability middleware** (structured logging, metrik per request) — **S**
- [ ] Bangun **test harness** skeleton (kontrak tool, auth, error, cache) — **M**
- [ ] Definisikan **pola registrasi ke Hermes** (register MCP server + tool filtering per agent) — **S**
- [ ] Dokumentasikan **template MCP server** (cara replikasi untuk MCP baru) — **S**

## B. GA4 MCP — Use Case B *(Tahap 1, prioritas)*

- [ ] Siapkan service account GA4 + enable GA4 Data API — **S**
- [ ] Simpan kredensial sebagai vault ref — **S**
- [ ] Implement auth (service account → klien GA4) — **S**
- [ ] Implement tools (mis. `query_report`, `list_metrics`, `realtime`) — **M**
- [ ] Caching metrics (TTL ~15 menit) — **S**
- [ ] Error + rate-limit handling (backoff/retry) — **S**
- [ ] Test harness GA4 — **S**
- [ ] Register ke Hermes + verifikasi agent bisa call — **S**

## C. RSS MCP — Use Case C *(Tahap 2)*

- [ ] Kumpulkan daftar feed kompetitor — **S**
- [ ] Implement tools (mis. `fetch_feed`, `list_items`, `since`) — **S**
- [ ] Caching feed (TTL ~5 menit) — **S**
- [ ] Error handling (feed down / malformed) — **S**
- [ ] Test harness RSS — **S**
- [ ] Register ke Hermes — **S**

## D. Figma MCP — Use Case A *(Tahap 3)*

- [ ] Siapkan Figma API key + file/team key → vault ref — **S**
- [ ] Implement auth (API key) — **S**
- [ ] Implement tools (mis. `get_file`, `get_node`, `export_image`) — **M**
- [ ] Rate-limit handling (Figma API) — **S**
- [ ] Caching node/file (TTL sesuai kebutuhan) — **S**
- [ ] Test harness Figma — **S**
- [ ] Register ke Hermes — **S**

## E. File/Storage MCP — Use Case A *(Tahap 3)*

- [ ] Siapkan akses bucket GCS / storage KLY → vault ref — **S**
- [ ] Implement auth (service account / signed access) — **S**
- [ ] Implement tools (mis. `list_objects`, `read_object`, `write_object`) — **M**
- [ ] Error handling (not-found / permission) — **S**
- [ ] Test harness Storage — **S**
- [ ] Register ke Hermes — **S**

## F. BigQuery MCP — Use Case D *(Tahap 4)*

- [ ] Siapkan service account + akses dataset → vault ref — **S**
- [ ] Implement auth (service account) — **S**
- [ ] Implement tools (mis. `run_query`, `list_tables`, `get_schema`) — **M**
- [ ] Query cost guard + caching hasil query — **M**
- [ ] Error + rate-limit handling — **S**
- [ ] Test harness BigQuery — **S**
- [ ] Register ke Hermes — **S**

## G. CMS MCP — Use Case D *(Tahap 4)*

- [ ] Sepakati kontrak akses dengan tim CMS (API/DB read-only) — **M**
- [ ] Siapkan kredensial read-only → vault ref — **S**
- [ ] Implement auth — **S**
- [ ] Implement tools (mis. `list_content`, `get_content`, `search`) — **M**
- [ ] Error handling — **S**
- [ ] Test harness CMS — **S**
- [ ] Register ke Hermes — **S**

## H. Search Console (GSC) MCP — Use Case D *(Tahap 4)*

- [ ] Siapkan service account terverifikasi di property GSC + enable API → vault ref — **S**
- [ ] Implement auth (service account) — **S**
- [ ] Implement tools (mis. `query_search_analytics`, `list_sites`, `inspect_url`) — **M**
- [ ] Caching (TTL sesuai latensi data GSC) — **S**
- [ ] Error + rate-limit handling — **S**
- [ ] Test harness GSC — **S**
- [ ] Register ke Hermes — **S**

## I. Verifikasi Phase 1 (exit)

- [ ] Hermes mengakses ≥1 sumber via MCP (GA4) — **S**
- [ ] Agent memanggil MCP sebagai tools di percakapan nyata — **S**
- [ ] Review keamanan: nol kredensial di log/percakapan — **S**
- [ ] Konfirmasi pola integrasi terstandar (MCP ke-2 dibangun dari template) — **S**
