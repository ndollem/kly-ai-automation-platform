# ERD (Ringan) — Phase 0: Hermes Foundation

> Parent: [`architecture.md`](architecture.md) · [`workflow.md`](workflow.md) · [`../00-glossary.md`](../00-glossary.md)

Phase 0 **minim data model bisnis** — belum ada entitas domain (artikel, revenue, dll; itu Phase 1–2). Yang relevan hanya entitas operasional fondasi: **User/Identity, Session, Group/Channel, AuditLog**.

> **Penting:** sebagian besar entitas ini **dikelola Hermes secara internal** (memory, session). KLY tidak membangun database untuk itu — hanya **memverifikasi** & mencatat lokasi/retensi. Tabel di bawah adalah **model konseptual** untuk pemahaman, bukan skema yang wajib kita implementasikan sendiri.

## 1. Diagram Entitas

```text
┌──────────────────────┐         ┌──────────────────────┐
│   USER / IDENTITY     │         │   GROUP / CHANNEL     │
│──────────────────────│         │──────────────────────│
│ identity_id (PK)      │         │ channel_id (PK)       │
│ wa_handle             │         │ type (DM | GROUP)     │
│ display_name          │         │ wa_chat_ref           │
│ role                  │         │ served (bool)         │
│ allowlisted (bool)    │         │ label                 │
│ status (active|revoked)│        └──────────┬───────────┘
└──────────┬───────────┘                    │
           │ 1                              │ 1
           │                                │
           │ N                              │ N
┌──────────▼────────────────────────────────▼─────────┐
│                     SESSION                          │
│──────────────────────────────────────────────────────│
│ session_id (PK)                                       │
│ identity_id (FK ──► USER)                             │
│ channel_id  (FK ──► GROUP/CHANNEL)                    │
│ started_at / last_active_at / closed_at               │
│ state (open | active | closed)                        │
│ memory_ref  ──► (Hermes internal: FTS5 + summary)     │
└──────────────────────┬───────────────────────────────┘
                       │ 1
                       │
                       │ N
            ┌──────────▼───────────┐
            │      AUDIT LOG        │
            │──────────────────────│
            │ log_id (PK)           │
            │ ts                    │
            │ identity_id (FK, nullable for anon-denied)│
            │ session_id  (FK, nullable)                │
            │ channel_id  (FK, nullable)                │
            │ event_type (msg | denied | error |        │
            │             restart | acl_change)         │
            │ summary / status / latency_ms             │
            └──────────────────────┘
```

## 2. Entitas & Atribut

### USER / IDENTITY
Siapa yang berinteraksi dengan Hermes (anggota tim KLY).

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| identity_id | PK | id internal |
| wa_handle | string | identitas dari gateway WA |
| display_name | string | nama tampil |
| role | enum | mis. operator / pilot / admin |
| allowlisted | bool | inti auth (default-deny) |
| status | enum | active / revoked |

### GROUP / CHANNEL
Kanal tempat Hermes melayani (DM personal atau group tim).

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| channel_id | PK | id internal |
| type | enum | DM / GROUP |
| wa_chat_ref | string | referensi chat dari gateway |
| served | bool | apakah kanal ini dilayani |
| label | string | nama/keterangan |

### SESSION
Satu rangkaian percakapan aktif. Lifecycle dikelola Hermes (lihat [`workflow.md`](workflow.md) §3).

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| session_id | PK | id sesi |
| identity_id | FK → USER | pemilik sesi |
| channel_id | FK → CHANNEL | kanal sesi |
| started_at / last_active_at / closed_at | timestamp | lifecycle |
| state | enum | open / active / closed |
| memory_ref | ref | tautan ke memory lintas sesi (Hermes internal) |

### AUDIT LOG
Jejak operasional & keamanan. Sumber observability Phase 0 (lihat [`architecture.md`](architecture.md) §5).

| Atribut | Tipe | Keterangan |
|---------|------|-----------|
| log_id | PK | id entri |
| ts | timestamp | waktu |
| identity_id | FK (nullable) | null bila pengirim ditolak/anon |
| session_id | FK (nullable) | bila ada |
| channel_id | FK (nullable) | bila ada |
| event_type | enum | msg / denied / error / restart / acl_change |
| summary / status / latency_ms | mixed | ringkasan, hasil, latency |

## 3. Relasi

- **USER 1 — N SESSION** — satu identitas bisa punya banyak sesi (lintas waktu).
- **CHANNEL 1 — N SESSION** — satu kanal menampung banyak sesi.
- **SESSION 1 — N AUDIT LOG** — tiap sesi menghasilkan banyak entri log.
- **USER 1 — N AUDIT LOG** — termasuk event di luar sesi (mis. akses ditolak, acl_change).
- **MEMORY** lintas sesi dirujuk via `memory_ref` namun **bukan tabel milik kita** — internal Hermes (FTS5 + LLM summarization).

## 4. Dikelola Hermes vs Disimpan KLY

| Entitas / data | Dikelola Hermes (internal) | Perlu disimpan/dikelola KLY |
|----------------|:--------------------------:|:---------------------------:|
| Session state & lifecycle | ✅ | verifikasi + catat lokasi/retensi (backup) |
| Memory lintas sesi (FTS5 + summary) | ✅ | verifikasi isolasi antar-user; backup ref |
| User/Identity (handle, history) | ✅ (sebagian, via gateway) | **allowlist & role** = kebijakan KLY |
| Group/Channel | ✅ (referensi) | **served / label / kebijakan** = konfigurasi KLY |
| Audit / request log | ✅ (logging) | retensi ≥14 hari, alerting, audit kebijakan |

**Implikasi praktis Phase 0:** KLY **tidak** mendirikan database bisnis. Pekerjaan data = (a) kelola **allowlist & kebijakan kanal** sebagai konfigurasi, (b) **verifikasi** session/memory Hermes berfungsi & terisolasi, (c) pastikan **audit/log** tercatat & ter-retensi. Data model bisnis sesungguhnya baru muncul saat MCP & use case (Phase 1–2).
