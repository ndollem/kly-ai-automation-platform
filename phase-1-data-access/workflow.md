# Workflow — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat [Architecture](./architecture.md), [PRD](./prd.md), [Glossary](../00-glossary.md).

## 1. Alur Utama (happy path)

Agent memanggil tool → **Hermes (MCP client)** → **MCP server** → sumber data → response balik.

```text
┌──────────┐   percakapan    ┌─────────────────────┐
│  USER    │────────────▶│  HERMES (Orchestration) │
│ (WA)     │◀────────────│  + MCP CLIENT            │
└──────────┘   jawaban       └───────────┬───────────┘
                                          │ (1) tool call (MCP)
                                          │     name + input args
                                          ▼
                              ┌───────────────────┐
                              │  MCP SERVER <source>    │
                              │  (dibangun KLY)         │
                              │  ┌───────────────────┐  │
                              │  │ validate input    │  │
                              │  │ resolve secret ───▼──│──▶ VAULT (kredensial)
                              │  │ check cache       │  │
                              │  │ call upstream ────▼──│──▶ DATA SOURCE
                              │  │ map error/output  │  │     (GA4/GSC/BQ/RSS/
                              │  │ log request       │  │      Figma/Storage/CMS)
                              │  └───────────────────┘  │
                              └───────────┬─────────┘
                                          │ (2) structured response
                                          ▼
                              ┌───────────────────┐
                              │  HERMES  → agent reasoning│
                              │  → jawaban natural ke user│
                              └───────────────────┘
```

- **(1)** Agent (di Hermes) memutuskan butuh data → memanggil tool MCP dengan `name` + `input args`. Hermes sebagai MCP client mengirim request ke MCP server yang tepat. Agent hanya melihat tools yang difilter untuknya.
- **(2)** MCP server memproses dan mengembalikan response terstruktur. Hermes memberi response ke reasoning agent → jawaban ke user.

> Kredensial **tidak pernah** keluar dari MCP server. Agent & percakapan hanya melihat hasil, bukan secret.

## 2. Pola Standar Request / Response

```text
REQUEST  (Hermes MCP client → MCP server)
  {
    tool:  "<server>.<tool_name>"        # mis. ga4.query_report
    input: { ...args sesuai schema... }
  }

RESPONSE (MCP server → Hermes)
  success:
    { status: "ok", data: {...}, meta: { cache: "hit|miss", latency_ms } }
  error:
    { status: "error", error: { code, message, retryable } }
```

Setiap tool punya **input schema** terdefinisi (kontrak MCP). Output dipetakan ke bentuk yang konsisten lintas MCP agar agent reasoning mudah.

## 3. Auth Flow

```text
tool call masuk
   │
   ▼
MCP server ambil VAULT REF dari config (bukan secret mentah)
   │
   ▼
resolve ref ──▶ VAULT ──▶ kembalikan credential (in-memory, MCP server saja)
   │
   ▼
bangun klien API ter-auth (service account / API key / OAuth)
   │
   ▼
call upstream  ───────────────────────────────────────────────▶ DATA SOURCE
```

- Config MCP server hanya menyimpan **referensi** ke secret, bukan secret.
- Credential di-resolve runtime, hidup di memori MCP server, **tidak** di-log dan **tidak** dikirim ke Hermes/percakapan.
- OAuth/SA expired → MCP server refresh otomatis; gagal → error `auth` (retryable=false) + alert.

## 4. Error Flow

```text
upstream error / timeout / rate-limit
   │
   ▼
ERROR MAPPER (reusable)
   ├─ auth        → { code:"auth",       retryable:false }
   ├─ rate_limit  → { code:"rate_limit", retryable:true  } → backoff + retry
   ├─ not_found   → { code:"not_found",  retryable:false }
   ├─ validation  → { code:"validation", retryable:false }
   └─ upstream    → { code:"upstream",   retryable:true  } → retry terbatas
   │
   ▼
response error terstruktur → Hermes → agent menjelaskan ke user (bukan stack trace)
```

- **retryable** menentukan apakah MCP server retry (dengan backoff) sebelum menyerah.
- Rate-limit pihak ke-3 (GA4/GSC/Figma) ditangani di sini: backoff eksponensial + throttle.

## 5. Caching Flow

```text
tool call
   │
   ▼
cache key = hash(server + tool + input args)
   │
   ├─ HIT  & belum kedaluwarsa ──▶ return cached  (meta.cache="hit")
   │
   └─ MISS / expired
         │
         ▼
      call upstream ──▶ simpan ke cache (TTL per source) ──▶ return (meta.cache="miss")
```

- TTL per sumber: mis. GA4 ~15 mnt, RSS ~5 mnt, BigQuery sesuai cost/freshness.
- Cache key memuat parameter request agar tidak salah serve.
- Bypass/invalidate didukung untuk data yang harus fresh.

## 6. Observability

Setiap request mencatat: `server`, `tool`, `latency_ms`, `status`, `cache hit/miss`, `error.code` (bila ada). Kredensial & isi sensitif di-redact. Log dipakai untuk audit trail & debugging lintas MCP.
