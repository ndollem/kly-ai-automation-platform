# Product Requirements Document — Phase 0: Hermes Foundation

> Dokumen parent: [`../00-master-plan.md`](../00-master-plan.md) · [`../00-architecture-overview.md`](../00-architecture-overview.md) · [`../00-decision-log.md`](../00-decision-log.md) · [`../00-glossary.md`](../00-glossary.md)

## 1. Latar Belakang

Platform KLY AI Automation memakai **Hermes Agent** (Nous Research, MIT) sebagai orchestration layer — bukan orchestrator buatan sendiri (lihat ADR-001). Hermes adalah produk existing yang **sudah menyediakan** gateway WhatsApp, memory lintas sesi, skills, MCP client, cron, dan subagents secara built-in.

Karena itu Phase 0 **bukan** pekerjaan membangun software. Phase 0 = **deploy, konfigurasi, dan hardening** instance Hermes milik KLY sampai ia stabil dan siap menerima MCP server (Phase 1) serta agent use case (Phase 2). Tanpa fondasi yang stabil, semua fase berikutnya menumpuk di atas tanah yang labil.

Sumber kebenaran scope: catatan asli Phase 0 — *"Menyiapkan fondasi platform yang stabil sebelum use case mulai dibangun."*

## 2. Tujuan

| # | Tujuan | Terukur |
|---|--------|---------|
| G1 | Hermes ter-deploy & berjalan stabil di infrastruktur KLY | Uptime instance ≥ 99% selama window observasi 7 hari |
| G2 | Tim KLY dapat berinteraksi dengan Hermes via WhatsApp | Pesan WA terkirim → balasan agent diterima end-to-end |
| G3 | Agent menerima & memproses request, siap pakai tools | Request percakapan diproses; LLM endpoint merespons |
| G4 | Akses terkontrol (hanya anggota tim yang berwenang) | Non-allowlisted user ditolak / tidak dilayani |
| G5 | Operasi teramati & dapat di-recover | Log per request tersedia; runbook rollback teruji |
| G6 | Infrastruktur siap menerima MCP & use case berikutnya | MCP client aktif; satu MCP test tersambung sukses |

## 3. Scope

### In Scope (Phase 0)

- **Provisioning** infrastruktur target deploy (server/akun serverless) + secret management.
- **Deploy Hermes** via metode terpilih (keputusan di ADR-004) — install, konfigurasi LLM endpoint (ADR-005), `SOUL.md`, Context File dasar.
- **WhatsApp integration** lewat gateway bawaan Hermes — pairing nomor, verifikasi inbound/outbound.
- **Group & channel management** — penataan kanal mana yang dilayani (DM personal, group tim).
- **Authentication & access control** — allowlist identitas, kebijakan siapa boleh memakai Hermes.
- **Session management** — verifikasi lifecycle sesi & memory lintas sesi Hermes berfungsi.
- **Logging & monitoring** — log per request, health check, alert dasar bila instance down.
- **Prompt orchestration dasar** — `SOUL.md` (personality global) + Context File fondasi.
- **Agent execution framework** — memastikan loop tool-calling Hermes jalan (siap menampung skills/MCP, walau belum ada use case).
- **Validasi** — smoke test end-to-end + runbook.

### Out of Scope (diserahkan ke fase lain)

- Pembangunan MCP server sumber data apa pun (GA4, BigQuery, RSS, dll) → **Phase 1**.
- Penulisan agent/skill bisnis (Story+, Analytics, Competitor, Revenue) → **Phase 2**.
- Cron / proactive alert / anomaly logic → **Phase 3**.
- Optimasi biaya LLM lanjutan, multi-region HA, dan fine-tuning model.
- Integrasi gateway selain WhatsApp (Telegram, Slack, dll — didukung Hermes tapi tidak diaktifkan sekarang).

## 4. User Persona (Tim KLY)

| Persona | Peran di Phase 0 | Kebutuhan |
|---------|------------------|-----------|
| **Platform Operator** (eng/infra) | Deploy, harden, jaga uptime, on-call | Runbook jelas, monitoring, rollback aman |
| **Pilot User** (anggota tim editorial/analytics) | Penguji awal interaksi WA | Bisa chat Hermes, respons cepat & relevan |
| **Admin / Lead** | Tentukan allowlist & kebijakan akses | Kontrol siapa yang boleh akses, audit trail |

> Catatan: di Phase 0, **value bisnis langsung masih minim** (belum ada agent use case). Pengguna utamanya internal/teknis. End-user value penuh datang di Phase 2.

## 5. Functional Requirements

| ID | Requirement | Prioritas |
|----|-------------|-----------|
| FR-01 | Instance Hermes dapat di-deploy ulang dari konfigurasi tersimpan (reproducible) | Must |
| FR-02 | Hermes terhubung ke satu LLM endpoint terpilih dan mengembalikan respons | Must |
| FR-03 | Nomor WhatsApp KLY ter-pairing ke gateway Hermes; inbound message diterima | Must |
| FR-04 | Hermes mengirim balasan ke WhatsApp (outbound) ke pengirim yang sama | Must |
| FR-05 | Hanya identitas dalam allowlist yang dilayani; lainnya ditolak/diabaikan | Must |
| FR-06 | Session & memory lintas sesi berfungsi (konteks percakapan terjaga) | Must |
| FR-07 | Setiap request menghasilkan entri log (siapa, kapan, ringkasan, status) | Must |
| FR-08 | Health check menandai instance up/down; operator dapat diberi tahu saat down | Should |
| FR-09 | `SOUL.md` & Context File fondasi termuat dan memengaruhi perilaku agent | Must |
| FR-10 | MCP client Hermes aktif; minimal satu MCP test dapat tersambung | Should |
| FR-11 | Konfigurasi group/channel: tentukan kanal mana yang dilayani Hermes | Should |
| FR-12 | Secret (API key LLM, kredensial gateway) tidak tersimpan plaintext di repo | Must |

## 6. Non-Functional Requirements

| ID | Kategori | Target |
|----|----------|--------|
| NFR-01 | Availability | Uptime instance ≥ 99% pada window observasi 7 hari |
| NFR-02 | Latency | Median respons WA (pesan ringan, tanpa tool berat) ≤ 5 dtk; p95 ≤ 15 dtk |
| NFR-03 | Security | Secret terenkripsi/terisolasi; akses via allowlist; kredensial tidak bocor ke percakapan |
| NFR-04 | Observability | Log terstruktur per request; retensi ≥ 14 hari |
| NFR-05 | Recoverability | Restart/rollback ke versi terakhir yang sehat ≤ 15 menit (runbook) |
| NFR-06 | Reproducibility | Deploy dari nol mengikuti runbook ≤ 1 jam |
| NFR-07 | Cost | Biaya infra+LLM Phase 0 dalam batas yang disepakati lead (idle hibernate bila Modal/Daytona) |

## 7. Success Metrics (Exit Criteria Phase 0)

Selaras dengan `00-roadmap.md` exit criteria: *"User bisa chat Hermes via WA; agent menerima & memproses request."*

- ✅ Pilot user mengirim pesan WA → menerima balasan agent (round-trip sukses).
- ✅ Non-allowlisted user **tidak** dilayani.
- ✅ Konteks percakapan terjaga lintas pesan dalam satu sesi, dan lintas sesi (memory).
- ✅ Log per request dapat ditelusuri operator.
- ✅ Instance bertahan ≥ 99% uptime selama observasi 7 hari.
- ✅ MCP client terbukti bisa menyambung (1 MCP test) → siap Phase 1.
- ✅ Runbook deploy + rollback teruji (dijalankan minimal sekali oleh operator kedua).

## 8. Asumsi

- A1 — Hermes versi stabil terbaru dipakai; install via `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash` berfungsi (~60 dtk).
- A2 — KLY memiliki nomor WhatsApp yang boleh dipakai untuk gateway (idealnya nomor khusus, bukan personal).
- A3 — Tersedia LLM endpoint dengan kuota memadai (Nous Portal / OpenRouter / OpenAI / endpoint internal).
- A4 — Tersedia anggaran infra (VPS untuk Docker, atau akun Modal/Daytona).
- A5 — Gateway WhatsApp Hermes menangani detail wire-protocol secara internal; KLY hanya melakukan pairing/konfigurasi (detail protokol **tidak** diasumsikan/dikarang).

## 9. Risiko & Mitigasi

| ID | Risiko | Dampak | Mitigasi |
|----|--------|--------|----------|
| R1 | Nomor WA kena banned/limit (penggunaan tak resmi) | WA mati total | Pakai nomor khusus; volume wajar; pantau status; siapkan nomor cadangan |
| R2 | Ketergantungan pada roadmap/API Hermes (ADR-001) | Breaking change | Pin versi; uji upgrade di staging; ikuti changelog Hermes |
| R3 | Salah pilih deployment backend (uptime vs biaya) | Re-platform | Putuskan via ADR-004 dengan kriteria uptime WA + budget eksplisit |
| R4 | Kebocoran secret / kredensial LLM | Keamanan & biaya | Secret manager, bukan plaintext; rotasi kunci; least-privilege |
| R5 | Biaya LLM membengkak saat idle/looping | Cost overrun | Batas rate; backend hibernate (Modal/Daytona); alert biaya |
| R6 | Privasi data internal KLY ke LLM eksternal | Compliance | Pertimbangkan endpoint privat di ADR-005; jangan kirim data sensitif di Phase 0 |
| R7 | Latency tinggi membuat pilot frustrasi | Adopsi rendah | Ukur p95; pilih model/endpoint sesuai NFR-02 |

## 10. Dependensi & Hand-off ke Fase Berikut

- **Output Phase 0** yang dikonsumsi Phase 1: MCP client aktif & teruji menyambung → Phase 1 tinggal menulis MCP server.
- **Output Phase 0** yang dikonsumsi Phase 2: agent execution framework + `SOUL.md`/Context File fondasi → Phase 2 tinggal menambah skills & Context File per use case.
- **Output Phase 0** yang dikonsumsi Phase 3: instance stabil + logging → prasyarat cron & alert.
