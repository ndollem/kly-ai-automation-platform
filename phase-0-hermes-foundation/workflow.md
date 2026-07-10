# Workflow Runtime — Phase 0: Hermes Foundation

> Parent: [`../00-architecture-overview.md`](../00-architecture-overview.md) · [`architecture.md`](architecture.md) · [`erd.md`](erd.md)
> Cakupan: alur runtime fondasi. Belum ada MCP/agent bisnis (itu Phase 1–2) — di sini agent menerima & memproses request, siap memakai tools.

## 1. Sequence: WhatsApp → Hermes → Agent → Response

```text
┌────────┐     ┌────────────┐     ┌─────────────────────────┐     ┌─────────┐
│  User  │     │  WhatsApp    │     │         HERMES           │     │   LLM    │
│ (Tim   │     │  Gateway     │     │  (orchestration layer)   │     │ endpoint │
│  KLY)  │     │ (Hermes      │     └────────────┬───────────┘     └────┬─────┘
└───┬────┘     │  built-in)   │                  │                        │
    │          └─────┬───────┘                  │                        │
    │ 1. kirim pesan  │                          │                        │
    │──────────────>│                          │                        │
    │                 │ 2. inbound message        │                        │
    │                 │──────────────────>│                        │
    │                 │                          │                        │
    │                 │             3. AUTH CHECK │ (allowlist?)           │
    │                 │             ┌────────────┤                        │
    │                 │             │ ditolak →  │ 3a. abaikan / tolak     │
    │                 │             └────────────┤   (audit log)           │
    │                 │                          │                        │
    │                 │             4. resolve SESSION + load MEMORY        │
    │                 │                (FTS5 + summarization)              │
    │                 │                          │                        │
    │                 │             5. susun konteks: SOUL.md +            │
    │                 │                Context File + history + pesan      │
    │                 │                          │                        │
    │                 │             6. inference  │                        │
    │                 │                          │─────────────────>│
    │                 │                          │   7. respons / tool req│
    │                 │                          │<─────────────────│
    │                 │                          │                        │
    │                 │     8. (opsional) tool-calling loop:              │
    │                 │        execute tool → hasil → inference lagi      │
    │                 │        [Phase 0: tools minim; MCP siap Phase 1]   │
    │                 │                          │                        │
    │                 │             9. simpan ke MEMORY + tulis LOG        │
    │                 │                          │                        │
    │                 │ 10. outbound reply        │                        │
    │                 │<─────────────────│                        │
    │ 11. terima balasan                         │                        │
    │<──────────────│                          │                        │
    │                 │                          │                        │
```

**Catatan Phase 0:** langkah 8 (tool-calling) sudah tersedia di framework eksekusi Hermes, tapi belum ada MCP server/skill bisnis. Validasi Phase 0 hanya memastikan loop ini **siap** (lihat `T-VAL-02`).

## 2. Auth Flow

```text
pesan inbound
     │
     ▼
identitas pengirim (dari gateway WA) ──> ada di ALLOWLIST?
     │                                          │
     │                                   ┌──────┴──────┐
     │                                  YA            TIDAK
     │                                   │              │
     │                            kanal diizinkan?   tulis AuditLog
     │                            (DM / group ybs)   (akses ditolak)
     │                                   │              │
     │                            ┌──────┴──────┐    abaikan / balas tolak
     │                           YA            TIDAK      │
     │                            │              │        ▼
     │                       lanjut proses    abaikan   selesai
     │                            │
     ▼                            ▼
   (ke Session resolve)      (lihat §1 langkah 4+)
```

Prinsip: **default-deny**. Hanya identitas allowlist + kanal yang ditetapkan (lihat `T-AUTH-01`, `T-CHN-02`) yang dilayani. Setiap penolakan dicatat di AuditLog (lihat [`erd.md`](erd.md)).

## 3. Session Lifecycle

```text
       pesan pertama dari user X
              │
              ▼
      ┌─────────────┐   tidak ada sesi aktif → BUKA sesi baru
      │  SESSION OPEN │   (Hermes-managed)
      └───────┬───────┘
              │  pesan-pesan berikutnya (window aktif)
              ▼
      ┌─────────────┐   konteks dalam sesi terjaga;
      │ SESSION ACTIVE│   memory jangka panjang diakses bila relevan
      └───────┬───────┘
              │  idle / window berakhir
              ▼
      ┌─────────────┐   ringkasan disimpan ke MEMORY lintas sesi
      │ SESSION CLOSE │   (FTS5 + LLM summarization — bawaan Hermes)
      └───────┬───────┘
              │  user X kembali nanti
              ▼
      sesi baru, tapi MEMORY lama dapat dipanggil → kontinuitas
```

**Dikelola Hermes secara internal** (memory lintas sesi = fitur bawaan, ADR-001). KLY tidak membangun store ini — hanya memverifikasi berfungsi (`T-SES-02`) dan mencatat lokasi/retensi untuk backup (`T-SES-04`). Isolasi antar-user wajib (`T-SES-03`).

## 4. Failure & Recovery (ringkas)

```text
LLM endpoint error/timeout ─> Hermes retry/laporkan error ke user + LOG ─> operator dari alert
WA gateway putus            ─> health check turun ─> alert operator ─> re-pairing (runbook)
instance down               ─> monitoring alert ─> restart/rollback (runbook, ≤15 mnt)
```

Detail langkah pemulihan ada di [`runbook.md`](runbook.md).
