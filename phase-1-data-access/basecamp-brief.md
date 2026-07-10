# Basecamp Brief — Phase 1: Data Access Foundation

> Format Shape Up pitch. Sumber: graph `graphify-out` (community 1 "MCP Servers & Reusable Template", community 2 "MCP Infrastructure & Credential Isolation") + `phase-1-data-access/*.md`.

## 🔴 Problem Statement

Analytics Insight Agent sudah siap ditulis, tapi ia tidak punya mata. Saat user tanya "traffic hari ini berapa?", agent harus menarik angka dari Google Analytics — dan saat ini **tidak ada jalur standar** bagi Hermes untuk menyentuh GA4, BigQuery, CMS, atau RSS. Tiap engineer yang butuh data akhirnya menulis integrasi ad-hoc sendiri, kredensial tersebar di mana-mana, dan tidak ada satu pun yang bisa dipakai ulang oleh agent berikutnya.

Phase 0 sudah memberi Hermes (sudah jadi **MCP client** bawaan), tapi belum ada satu pun **MCP server** untuk data KLY. Pintu ada, di baliknya kosong.

## ⏳ Budget

- **Resources:** 1–2 engineer (backend + akses data/infra)
- **Time Box:** **on-demand, bukan blok tunggal.** Tiap MCP server ≈ beberapa hari. Yang pertama (GA4) lebih lama karena sekaligus membangun **Reusable MCP Server Template**; MCP berikutnya mewarisi template → jauh lebih cepat.
- Berjalan **beriringan dengan Phase 2** — tidak menunggu semua MCP selesai.

## 💡 Potential Solution

Bangun **MCP server per sumber data**, di-instantiate dari satu **Reusable MCP Server Template** (auth, rate-limit, caching, error handling, observability sebagai komponen bersama). Hermes tinggal di-`register` ke server ini sebagai tools.

Prinsip kunci dari graph:
- **MCP as Infrastructure** — MCP bukan produk, hanya lapisan akses standar
- **On-demand Expansion** — MCP hanya dibangun saat use case Phase 2 segera memakainya
- **Credential Isolation** — kredensial hidup di MCP server, **tidak pernah** terekspos ke percakapan/agent

Urutan disarankan: **GA4 → RSS → Figma/Storage → BigQuery/CMS/GSC** (membuka Use Case B → C → A → D).

## 📈 Estimation Impact

- ✅ Hermes dapat mengakses sumber data via MCP
- ✅ Agent dapat memakai MCP sebagai tools
- ✅ Pola integrasi MCP **terstandarisasi** — MCP ke-2 dst dibangun jauh lebih cepat dari template
- ✅ Kredensial terisolasi & teraudit (RequestLog), nol kebocoran ke percakapan

## 🕳️ Rabbit Holes

- **Rate limit & kuota API pihak ke-3** (GA4, GSC, BigQuery) — desain caching + backoff sejak template, jangan ditambal belakangan
- **Heterogenitas auth** — OAuth (Google), service account (BigQuery), API key (Figma), akses internal (CMS). Template harus menampung semua tanpa jadi bocor abstraksi
- **Join key antar-sumber** (URL/slug) untuk Use Case D — kalau CMS & BigQuery pakai identifier beda, mapping bisa jadi lubang
- **Over-engineering template** — menambah fitur template yang belum dibutuhkan MCP mana pun

## 🚫 No-Gos

- ❌ Membangun **semua** MCP sekaligus / front-load connector yang belum ada use case-nya
- ❌ Logika agent atau percakapan → **Phase 2**
- ❌ Proactive alert → **Phase 3**
- ❌ Agent query sumber data langsung tanpa lewat MCP (melanggar ADR-002)
- ❌ Menyimpan kredensial di Context File / prompt / memory percakapan

---

**Catatan:** semua MCP di-instantiate dari `Reusable MCP Server Template` (graph: template → instantiates → GA4/RSS/Figma/Storage/BigQuery/CMS/GSC). Detail per connector: `mcp-catalog.md`.
