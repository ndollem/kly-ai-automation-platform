# Plan — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat [PRD](./prd.md), [Architecture](./architecture.md), [Roadmap](../00-roadmap.md), [Glossary](../00-glossary.md).

## 1. Prinsip Pelaksanaan

1. **On-demand, bukan big-bang.** MCP server dibangun saat use case yang akan segera dipakai menuntutnya. Tidak ada milestone "semua MCP selesai".
2. **Template dulu, lalu replikasi.** MCP pertama membangun *template/standar reusable*. MCP berikutnya mengisi template, bukan mulai dari nol.
3. **Reusable lintas use case.** Satu MCP melayani banyak use case (Analytics MCP → Use Case B + D).
4. **Kredensial terisolasi.** Tiap MCP menangani auth-nya sendiri via secret/vault ref. Tidak pernah ke agent/percakapan.
5. **Hermes-native.** MCP client, registrasi tool, dan filtering memakai kapabilitas bawaan Hermes — tidak dibangun ulang.

## 2. Urutan Pembangunan MCP yang Disarankan

Mengikuti [roadmap](../00-roadmap.md) — urutan ini **saran**, bukan kewajiban; sesuaikan prioritas bisnis.

| Urutan | MCP | Membuka Use Case | Alasan prioritas |
|--------|-----|------------------|------------------|
| 1 | **Google Analytics (GA4) MCP** | B — Analytics | Single-source, value tinggi, cepat dibangun. Validasi template di sumber yang well-documented. |
| 2 | **RSS MCP** | C — Competitor | Sumber publik, tanpa auth kompleks, integrasi paling ringan. Cepat menambah cakupan. |
| 3 | **Figma MCP + File/Storage MCP** | A — Story+ | Pasangan untuk produksi Story+. Figma butuh API key; Storage butuh akses GCS/file. |
| 4 | **BigQuery + CMS + Search Console MCP** | D — Revenue | Paling kompleks: multi-source, join data, CMS internal. Dibangun setelah template matang. |

**Alasan urutan:** mulai dari yang **cepat + value tinggi + single-source** (GA4) untuk memvalidasi template, lalu yang **paling ringan** (RSS), baru naik ke yang **multi-source & kompleks** (Use Case D) saat pola sudah terbukti.

## 3. Standar / Template MCP Server Reusable

Setiap MCP server dibangun dari template yang sama (detail di [architecture.md](./architecture.md)):

```text
mcp-server-<source>/
├── manifest          # nama server, daftar tools, schema input/output
├── auth/             # resolve secret dari vault ref → klien API ter-auth
├── tools/            # 1 file per tool: validasi input, call upstream, map output
├── cache/            # TTL policy + cache key strategy (read-heavy)
├── errors/           # mapping error upstream → error MCP terstruktur
├── observability/    # structured logging + metrik per request
└── tests/            # test harness: kontrak tool, auth, error, cache
```

**Komponen reusable lintas MCP** (dibangun sekali, dipakai semua):
- Auth/secret resolver pattern (vault ref → credential).
- Error mapper (klasifikasi auth/rate-limit/not-found/upstream/validation).
- Cache layer (TTL + keying).
- Logging/observability middleware.
- Test harness skeleton.

## 4. Fase Eksekusi

```text
Tahap 0  Bangun template + komponen reusable (auth, error, cache, logging, test harness)
Tahap 1  GA4 MCP        → validasi template end-to-end, register ke Hermes      [Use Case B]
Tahap 2  RSS MCP        → replikasi template (no-auth path)                      [Use Case C]
Tahap 3  Figma + Storage MCP → API key + file/storage path                       [Use Case A]
Tahap 4  BigQuery + CMS + GSC MCP → multi-source kompleks                         [Use Case D]
```

> Tahap 1–4 dipicu oleh kesiapan use case di Phase 2, **berjalan beriringan** dengan Phase 2 (lihat roadmap). Tidak harus sekuensial penuh.

## 5. Prasyarat per Sumber Data

| MCP | Prasyarat kredensial / akses |
|-----|------------------------------|
| GA4 | Service account dengan akses ke GA4 property; GA4 Data API enabled. |
| RSS | URL feed kompetitor; tidak butuh auth (publik). |
| Figma | Personal access token / Figma API key; file/team key. |
| File/Storage | Akses bucket GCS (atau storage KLY); service account / signed access. |
| BigQuery | Service account dengan akses dataset relevan; BigQuery API enabled. |
| Search Console (GSC) | Service account terverifikasi di property GSC; Search Console API enabled. |
| CMS | Kontrak akses dengan tim CMS internal (API/DB read); kredensial terbatas read. |

**Prasyarat lintas semua MCP:** secret store / vault (atau env terisolasi) tersedia untuk menyimpan kredensial di atas.

## 6. Definition of Done (Phase 1)

- Template MCP + komponen reusable jadi & terdokumentasi.
- Minimal GA4 MCP live, ter-register di Hermes, dipanggil agent.
- Tiap MCP yang dibangun lulus test harness (kontrak, auth, error, cache).
- Nol kredensial di config plaintext / log / percakapan.
- Pola registrasi & filtering tool ke Hermes terstandar.
