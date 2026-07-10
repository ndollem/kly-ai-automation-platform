# Roadmap — KLY AI Automation Platform

## Urutan Fase & Dependensi

```text
Phase 0 ──> Phase 1 ──> Phase 2 ──> Phase 3
Foundation   Data        Use Cases   Proactive
             Access       (A/B/C/D)   Intelligence
```

- **Phase 0** wajib selesai sebelum apa pun (Hermes harus jalan & stabil).
- **Phase 1 & Phase 2 berjalan beriringan** — MCP (Phase 1) dibangun on-demand mengikuti use case (Phase 2) yang sedang dikerjakan. Bukan sekuensial penuh.
- **Phase 3** butuh data & MCP dari Phase 1–2 sudah aktif (alert butuh sumber sinyal).

## Milestone per Fase

| Fase | Milestone utama | Exit criteria |
|------|-----------------|---------------|
| **Phase 0** | Hermes deployed, WhatsApp tersambung, auth & logging aktif | User bisa chat Hermes via WA; agent menerima & memproses request |
| **Phase 1** | Pola integrasi MCP terstandar; MCP prioritas aktif | Hermes akses sumber data via MCP; agent pakai MCP sebagai tools |
| **Phase 2** | Agent A/B/C/D live & dipakai tim | Tiap agent menghasilkan outcome bisnis terukur |
| **Phase 3** | Alert engine berbasis anomaly/opportunity | Signal > noise; user terbantu, bukan terganggu |

## Pemetaan Use Case → MCP (dependensi Phase 1)

MCP dibangun mengikuti use case yang akan segera dipakai:

| Use Case (Phase 2) | MCP yang dibutuhkan (Phase 1) |
|--------------------|-------------------------------|
| A — Story+ Production | Figma MCP, File/Storage MCP, Story+ Generation Services |
| B — Analytics Insight | Google Analytics MCP |
| C — Competitor Intelligence | RSS MCP |
| D — Revenue & Inventory | BigQuery MCP, CMS MCP, Analytics MCP, Search Console MCP |

## Urutan Pembangunan MCP yang Disarankan

Prioritaskan MCP yang mengaktifkan use case bernilai tinggi & paling cepat dibangun:

1. Google Analytics MCP → membuka Use Case B (cepat, single-source, value tinggi)
2. RSS MCP → membuka Use Case C (sumber publik, integrasi ringan)
3. Figma + File/Storage MCP → membuka Use Case A
4. BigQuery + CMS + Search Console MCP → membuka Use Case D (paling kompleks)

> Urutan ini saran, bukan kewajiban — sesuaikan dengan prioritas bisnis berjalan.

## Prinsip Roadmap

- Tidak ada "big bang". Tiap fase menghasilkan kapabilitas yang langsung dipakai.
- Setiap MCP & agent harus reusable lintas use case.
- Phase 3 bukan fitur baru, tapi **pola interaksi baru** di atas data yang sudah ada.
