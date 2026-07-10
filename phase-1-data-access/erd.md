# ERD — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat [Architecture](./architecture.md), [Workflow](./workflow.md), [Glossary](../00-glossary.md).

MCP Connector Layer punya data model sendiri: registry konfigurasi MCP, referensi kredensial, definisi tool, cache, dan log request. Dokumen ini memetakan entitas, atribut, relasi, dan mana yang **persisten** vs **ephemeral**.

## 1. Diagram Relasi

```text
                 ┌──────────────────┐
                 │   DataSource     │
                 │ (GA4, RSS, BQ…)  │
                 └────────┬─────────┘
                          │ 1
                          │ disajikan oleh
                          │ 1
                 ┌────────▼─────────┐        1     ┌────────────────┐
                 │   MCPServer      │──────────────│  CredentialRef   │
                 │  (registry)      │  pakai  1    │  (vault ref)     │
                 └────────┬─────────┘              └────────────────┘
                          │ 1
                          │ mengekspos
                          │ N
                 ┌────────▼─────────┐
                 │  ToolDefinition  │
                 └────────┬─────────┘
                          │ 1
              ┌───────────┴────────────┐
              │ N                      │ N
     ┌────────▼────────┐      ┌────────▼────────┐
     │   CacheEntry    │      │   RequestLog    │
     │  (ephemeral)    │      │  (append-only)  │
     └─────────────────┘      └─────────────────┘
```

## 2. Entitas & Atribut

### DataSource
Sumber data eksternal/internal yang diakses (GA4, GSC, BigQuery, RSS, Figma, Storage, CMS).

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `id` | string | PK |
| `name` | string | mis. "Google Analytics 4" |
| `kind` | enum | analytics / search / warehouse / rss / design / storage / cms |
| `auth_method` | enum | service_account / api_key / oauth / none |
| `priority` | enum | mengikuti urutan roadmap (1–4) |

### MCPServer (registry/config)
Satu entry per MCP server yang dibangun KLY.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `id` | string | PK |
| `name` | string | mis. "ga4-mcp" |
| `data_source_id` | FK → DataSource | sumber yang dilayani |
| `credential_ref_id` | FK → CredentialRef | kredensial yang dipakai |
| `deployment` | enum | sidecar / standalone |
| `status` | enum | planned / active / deprecated |
| `cache_ttl_default` | int (sec) | TTL default cache |
| `registered_to_hermes` | bool | sudah ter-register sbg MCP client? |

### CredentialRef (secret/vault reference)
**Referensi** ke kredensial, **bukan** kredensial mentah.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `id` | string | PK |
| `vault_ref` | string | mis. `vault://ga4/service-account` |
| `auth_method` | enum | service_account / api_key / oauth |
| `scope` | string | hak akses (mis. read-only) |
| `expires_at` | datetime? | untuk OAuth/SA yang perlu refresh |

> Secret **tidak pernah** disimpan di sini — hanya pointer ke vault. Resolusi terjadi runtime di MCP server.

### ToolDefinition
Kontrak satu tool yang diekspos sebuah MCP server.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `id` | string | PK |
| `mcp_server_id` | FK → MCPServer | server pemilik |
| `name` | string | mis. "query_report" (dipanggil `ga4.query_report`) |
| `description` | string | untuk reasoning agent |
| `input_schema` | json | kontrak input |
| `output_shape` | json | bentuk output standar |
| `cacheable` | bool | apakah hasil boleh di-cache |

### CacheEntry (ephemeral)
Hasil tool yang di-cache.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `cache_key` | string | PK — `hash(server+tool+input)` |
| `tool_definition_id` | FK → ToolDefinition | |
| `value` | blob/json | hasil ter-cache |
| `created_at` | datetime | |
| `expires_at` | datetime | TTL per source |

### RequestLog (append-only)
Audit trail per tool call.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `id` | string | PK |
| `mcp_server_id` | FK → MCPServer | |
| `tool_definition_id` | FK → ToolDefinition | |
| `status` | enum | ok / error |
| `error_code` | string? | auth / rate_limit / not_found / validation / upstream |
| `latency_ms` | int | |
| `cache` | enum | hit / miss |
| `created_at` | datetime | |

> Tidak pernah memuat kredensial atau payload sensitif (redacted).

## 3. Relasi (ringkas)

| Relasi | Kardinalitas |
|--------|--------------|
| DataSource → MCPServer | 1 : 1 |
| MCPServer → CredentialRef | N : 1 (server pakai 1 credential ref) |
| MCPServer → ToolDefinition | 1 : N |
| ToolDefinition → CacheEntry | 1 : N |
| ToolDefinition → RequestLog | 1 : N |

## 4. Persisten vs Ephemeral

| Entitas | Sifat | Alasan |
|---------|-------|--------|
| DataSource | **Persisten** | Konfigurasi dasar. |
| MCPServer | **Persisten** | Registry MCP layer. |
| CredentialRef | **Persisten** (ref saja) | Pointer ke vault; secret aktual di vault. |
| ToolDefinition | **Persisten** | Kontrak tool. |
| CacheEntry | **Ephemeral** | Expired by TTL; dapat dibuang kapan saja. |
| RequestLog | **Persisten, append-only** | Audit trail; retensi sesuai kebijakan. |

> Secret aktual hidup di **vault** (di luar model ini) + sementara di memori MCP server saat runtime — tidak dipersist di entitas mana pun.
