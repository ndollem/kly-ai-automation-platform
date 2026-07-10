# PRD — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat juga [Architecture Overview](../00-architecture-overview.md), [Roadmap](../00-roadmap.md), [Glossary](../00-glossary.md).

## 1. Masalah

Data KLY yang dibutuhkan agent tersebar di banyak sistem yang berbeda (Google Analytics, Search Console, BigQuery, CMS internal, RSS publik, Figma, file storage). Setiap sumber punya cara auth, format response, dan rate limit sendiri. Akibatnya:

- Tidak ada cara terstandar bagi agent untuk mengakses data — tiap integrasi jadi one-off.
- Kredensial sumber data berisiko bocor jika dimasukkan langsung ke logika agent atau percakapan.
- Integrasi tidak reusable — kerja akses GA4 untuk Use Case B tidak otomatis dipakai ulang oleh Use Case D.

**Hermes Agent (Nous Research)** sudah berperan sebagai **MCP client** — ia bisa "connect to any MCP server for extended tool capabilities" dan memfilter tools-nya. Yang belum ada adalah **MCP server** untuk tiap sumber data KLY. Phase 1 mengisi celah itu.

## 2. Tujuan

Membangun **MCP Connector Layer** yang reusable lintas use case: satu MCP server per sumber data, dengan pola integrasi yang terstandar, sehingga Hermes dapat mengakses sumber data sebagai tools secara konsisten, aman, dan dapat diobservasi.

Kita **tidak** membangun MCP client — itu bawaan Hermes.

## 3. Scope

### In scope
- MCP server per sumber data prioritas: Figma, File/Storage, Google Analytics (GA4), RSS, BigQuery, Search Console (GSC), CMS.
- Template & standar reusable untuk membangun MCP server (auth, error, caching, observability).
- Pola registrasi MCP server ke Hermes (MCP client).
- Isolasi kredensial (vault/secret reference) — kredensial **tidak** terekspos ke agent maupun percakapan.

### Out of scope
- Logika agent / use case (Phase 2).
- Proactive alert engine (Phase 3).
- Deployment & hardening Hermes itu sendiri (Phase 0).
- Membangun MCP client (sudah ada di Hermes).

### Prinsip scope
- **On-demand.** MCP dibangun hanya saat mendukung use case yang akan segera dipakai. Tidak ada target "selesaikan semua MCP dulu".
- **Reusable.** Satu MCP server melayani banyak use case (mis. Analytics MCP dipakai Use Case B dan D).

## 4. Functional Requirements

| ID | Requirement |
|----|-------------|
| FR-1 | Tiap MCP server mengekspos tools sesuai kontrak MCP standar (name, description, input schema, output). |
| FR-2 | Hermes dapat me-register MCP server dan agent dapat memanggil tools-nya sebagai tools. |
| FR-3 | Tiap MCP server menangani auth ke sumber data secara internal (service account / API key / OAuth), tanpa mengekspos kredensial ke caller. |
| FR-4 | Tiap MCP server mengembalikan error terstruktur (kode, pesan, retryable?) — bukan stack trace mentah. |
| FR-5 | MCP server read-heavy menyediakan caching untuk response yang mahal/sering diminta. |
| FR-6 | Tiap request ke MCP server tercatat (request log) untuk audit & observability. |
| FR-7 | Tools dapat difilter per agent (Hermes mendukung filtering) agar agent hanya melihat tools relevan. |
| FR-8 | Pola integrasi (template MCP server) terdokumentasi sehingga MCP berikutnya dibangun konsisten. |

## 5. Non-Functional Requirements

| Kategori | Requirement |
|----------|-------------|
| **Auth** | Kredensial sumber data disimpan sebagai secret/vault reference, di-resolve saat runtime di dalam MCP server. Tidak pernah ada di config plaintext, log, atau percakapan. |
| **Rate limit** | MCP server menghormati rate limit API pihak ke-3 (GA4, GSC, Figma): backoff + retry pada error retryable; antrian/throttle bila perlu. |
| **Error handling** | Error diklasifikasi (auth / rate-limit / not-found / upstream / validation) dan dipetakan ke response MCP yang bisa dipahami agent. |
| **Caching** | TTL per data source (mis. GA4 metrics 15 menit, RSS 5 menit). Cache key memuat parameter request. Bypass/invalidate didukung. |
| **Observability** | Structured logging per request (server, tool, latency, status, cache hit/miss). Metrik dasar diekspos. |
| **Performance** | Latency p95 wajar per tool; caching menutup beban berulang. |
| **Portability** | MCP server deploy sebagai sidecar atau standalone; tidak terkunci ke satu sumber data tunggal. |

## 6. Success Metrics

Selaras [exit criteria roadmap](../00-roadmap.md):

- Hermes dapat mengakses ≥1 sumber data via MCP (target awal: GA4 MCP).
- Agent dapat memanggil MCP sebagai tools dalam percakapan nyata.
- Pola integrasi MCP terstandar: MCP ke-2 dst dibangun dari template, bukan dari nol.
- Nol kebocoran kredensial ke log/percakapan (verifikasi review).
- Cache hit rate terukur pada MCP read-heavy (mis. GA4) > 0 dan menurunkan call ke upstream.

## 7. Asumsi

- Phase 0 selesai: Hermes jalan, stabil, MCP client aktif.
- KLY dapat menyediakan kredensial tiap sumber (service account GA4/GSC/BigQuery, API key Figma, akses CMS).
- Sumber data punya API/feed yang dapat diakses programatis.
- Ada secret store / vault (atau env terisolasi) untuk menyimpan kredensial.

## 8. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|--------|--------|----------|
| Kebocoran kredensial ke percakapan/log | Tinggi | Isolasi di MCP server; secret via vault ref; redaction di logging; review. |
| Rate limit API pihak ke-3 (GA4/GSC/Figma) | Sedang | Caching + backoff/retry + throttle; pantau kuota. |
| Sumber data tidak punya API stabil (CMS internal) | Sedang | Adapter khusus; sepakati kontrak akses dengan tim CMS. |
| Over-engineering (bangun semua MCP di depan) | Sedang | Tegakkan prinsip on-demand; MCP hanya saat use case menuntut. |
| Skema response upstream berubah | Rendah-Sedang | Versioning kontrak tool; test harness deteksi drift. |
| Kredensial expired (OAuth/SA) | Sedang | Refresh otomatis; alert saat gagal auth. |

## 9. Dependensi

- **Hulu:** Phase 0 (Hermes + MCP client).
- **Hilir:** Phase 2 (use case A/B/C/D mengonsumsi MCP). Lihat pemetaan use case → MCP di [roadmap](../00-roadmap.md).
- **Eksternal:** kredensial & akses tiap sumber data dari pemilik sistem KLY.
