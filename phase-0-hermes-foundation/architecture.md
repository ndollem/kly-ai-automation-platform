# Arsitektur Teknis — Phase 0: Hermes Foundation

> Parent: [`../00-architecture-overview.md`](../00-architecture-overview.md) · [`../00-decision-log.md`](../00-decision-log.md) · [`workflow.md`](workflow.md) · [`erd.md`](erd.md)

Fase ini hanya memakai **Gateway Layer + Orchestration Layer** dari arsitektur platform. Agent Layer ada tapi kosong-bisnis (siap diisi Phase 2); MCP Connector & Data Layer belum aktif (Phase 1).

## 1. Komponen Hermes yang Dipakai di Phase 0

| Komponen Hermes | Status Phase 0 | Catatan |
|-----------------|----------------|---------|
| **WhatsApp Gateway** (built-in) | ✅ Aktif | Satu kanal masuk/keluar utama |
| **Memory lintas sesi** (FTS5 + summarization) | ✅ Aktif | Diverifikasi, bukan dibangun |
| **Session management** | ✅ Aktif | Lifecycle dikelola Hermes |
| **Auth / access control** | ✅ Aktif | Allowlist + kanal yang dilayani |
| **Agent execution / tool-calling loop** | ✅ Aktif (idle) | Siap, belum ada tools bisnis |
| **MCP client** | 🟡 Aktif, diuji 1 MCP test | Siap Phase 1 |
| **`SOUL.md`** (personality global) | ✅ Diisi fondasi | — |
| **Context File** | ✅ Diisi fondasi | Konteks platform & scope |
| **Logging** | ✅ Aktif | Per request + audit |
| **Skills** | ⛔ Belum | Phase 2 |
| **Cron** | ⛔ Belum | Phase 3 |
| **Subagents** | ⛔ Belum | sesuai kebutuhan use case |
| **Tool Gateway** (web/img/TTS/browser) | ⚙️ Opsional | aktifkan bila perlu untuk smoke test |

## 2. Deployment Topology

Dua opsi (keputusan final di [ADR-004](../00-decision-log.md), task `T-DEC-01`).

### Opsi A — Docker di VPS (rekomendasi default fondasi WA)

```text
┌──────────────────────── VPS (KLY) ────────────────────────┐
│                                                            │
│   ┌──────────── Docker container: Hermes ───────────────┐  │
│   │  Hermes process (always-on)                          │  │
│   │   • WhatsApp gateway (persistent connection)         │  │
│   │   • Session + Memory store (FTS5) ──► volume mount   │  │
│   │   • Logs ──► volume mount / log shipper              │  │
│   │  restart policy: always                              │  │
│   └──────────────────┬───────────────────────────────────┘ │
│                      │ env / secrets (mounted, not in image)│
│   ┌──────────────────┴──────┐   ┌────────────────────────┐  │
│   │ Secret store / .env vault│   │ Health check + monitor │  │
│   └─────────────────────────┘   └────────────────────────┘  │
└───────────────┬──────────────────────────┬─────────────────┘
                │ HTTPS                       │ HTTPS
                ▼                             ▼
        WhatsApp (gateway)              LLM endpoint
```

**Trade-off:** koneksi WA persisten (tidak drop), latency stabil, kontrol penuh — **tapi** biaya VPS jalan terus + tanggung jawab patching OS.

### Opsi B — Modal / Daytona (serverless, hibernate idle)

```text
          WhatsApp ──► ??? ──► Hermes (Modal/Daytona)
                                  • hibernate saat idle
                                  • cold start saat ada beban
                                  • state persisten via storage backend
                                  ▲
                                  │  RISIKO: koneksi WA gateway saat hibernate
                                  └─ WAJIB diuji (T-DEC-01) sebelum dipakai produksi
```

**Trade-off:** hemat saat idle, setup cepat, ops minim — **tapi** cold start + risiko koneksi WA putus saat hibernate. Gateway WhatsApp idealnya butuh koneksi persisten; jika serverless dipilih, buktikan dulu WA tetap online. Install tetap ~60 dtk.

> **Putusan default:** mulai dengan **Docker/VPS** demi keandalan gateway. Pertimbangkan serverless hanya bila uji hibernate-WA lolos & biaya jadi pendorong utama. Catat di ADR-004.

## 3. Integrasi WhatsApp

- Lewat **gateway bawaan Hermes** (built-in, 20+ platform). KLY hanya melakukan pairing nomor + konfigurasi kanal.
- Detail wire-protocol WhatsApp **ditangani internal oleh gateway Hermes** — tidak dimodelkan/dikarang di sini (lihat asumsi A5 di `prd.md`).
- Konfigurasi: nomor khusus, kanal yang dilayani (DM/group), prosedur re-pairing (runbook).

## 4. Auth Model

```text
Identitas pengirim (dari gateway WA)
        │
        ▼
   ALLOWLIST check ──(default-deny)──► tolak + AuditLog
        │ lolos
        ▼
   KANAL check (DM / group yang ditetapkan) ──► tolak + AuditLog
        │ lolos
        ▼
   layani request
```

- **Default-deny**: hanya allowlist + kanal sah yang dilayani.
- Kredensial sumber data & LLM **terisolasi** di layer Hermes/secret store — tidak pernah masuk isi percakapan (NFR-03, `T-AUTH-03`).
- Prinsip ini lanjut ke fase berikut: saat MCP hadir (Phase 1), kredensial sumber tetap terisolasi di MCP server, tak terekspos ke agent/percakapan (sesuai `00-architecture-overview.md`).

## 5. Logging & Monitoring Stack

| Lapisan | Isi | Sumber |
|---------|-----|--------|
| **Request log** | who, when, kanal, ringkasan, status, latency | Hermes logging |
| **Audit log** | akses ditolak, error, restart, perubahan allowlist | Hermes + operasional (lihat ERD) |
| **Health check** | up/down instance + koneksi WA & LLM | monitor eksternal/uptime probe |
| **Alerting** | instance down, error spike, biaya LLM melewati ambang | channel operator (mis. WA/email) |
| **Retensi** | log ≥ 14 hari (NFR-04) | volume / log store |

Implementasi minimal cukup untuk Phase 0 — bukan stack observability penuh. Tujuannya: operator bisa menelusuri request & tahu saat instance bermasalah.

## 6. Environment & Secret Management

- Semua rahasia (API key LLM, kredensial gateway) di **secret store / vault** atau env terenkripsi — **bukan** plaintext di repo (FR-12, NFR-03).
- Konfigurasi versi (`SOUL.md`, Context File, env template tanpa nilai rahasia) disimpan untuk **reproducible deploy** (FR-01, NFR-06).
- Rotasi kunci & least-privilege diterapkan; review pra-commit memastikan tidak ada secret bocor.

## 7. Apa yang TIDAK dibangun di Phase 0

Sesuai prinsip **Hermes-native dulu** & scope fondasi:

- ❌ Orchestrator/router custom — sudah disediakan Hermes (ADR-001).
- ❌ Store memory/session custom — bawaan Hermes.
- ❌ MCP server sumber data — Phase 1, on-demand (ADR-003).
- ❌ Skill/agent bisnis — Phase 2.
- ❌ Cron/alert proaktif — Phase 3.
