# Architecture — Phase 1: Data Access Foundation

> Bagian dari [KLY AI Automation Platform](../README.md). Lihat [Architecture Overview](../00-architecture-overview.md), [Workflow](./workflow.md), [ERD](./erd.md), [Glossary](../00-glossary.md).

Phase 1 membangun **MCP Connector Layer** dari [layered architecture platform](../00-architecture-overview.md). Hermes sudah menyediakan MCP **client**; kita membangun MCP **server** per sumber data.

## 1. Posisi dalam Platform

```text
HERMES (MCP client)  ──register──▶  MCP SERVER REGISTRY
       │                                  │
       │ tool call                        │ tiap entry = 1 MCP server
       ▼                                  ▼
┌────────────── MCP CONNECTOR LAYER (Phase 1) ─────────────┐
│  GA4 MCP · RSS MCP · Figma MCP · Storage MCP ·               │
│  BigQuery MCP · GSC MCP · CMS MCP                             │
└────────────────────────┬────────────────────────┘
                           ▼
        DATA / SYSTEM LAYER (sumber existing KLY + pihak ke-3)
```

## 2. Anatomi Sebuah MCP Server

Setiap MCP server (apa pun sumbernya) punya struktur sama:

```text
MCP SERVER <source>
├── Manifest          # identitas server + daftar tools yang diekspos
├── Tools             # kapabilitas yang dipanggil agent (verb + input schema + output)
│     └─ tiap tool: validate → (auth) → (cache) → call upstream → map output
├── Resources         # (opsional) data yang bisa dibaca agent sbg konteks read-only
├── Auth module       # resolve secret/vault ref → klien API ter-auth (isolated)
├── Cache             # TTL + cache key (read-heavy)
├── Error mapper      # upstream error → error MCP terstruktur
└── Observability     # structured logging + metrik per request
```

- **Tools** = aksi (mis. `ga4.query_report`, `bigquery.run_query`). Punya input schema → kontrak.
- **Resources** = data read-only opsional sebagai konteks (mis. daftar metrik tersedia).
- **Auth** = satu-satunya tempat kredensial hidup. Lihat §4.

## 3. Pola Standar Reusable

Dibangun **sekali**, dipakai semua MCP (lihat [plan.md](./plan.md)):

| Komponen reusable | Fungsi |
|-------------------|--------|
| Auth/secret resolver | Vault ref → credential, dengan redaction. |
| Error mapper | Klasifikasi & mapping error upstream → error MCP. |
| Cache layer | TTL + cache key strategy. |
| Observability middleware | Logging + metrik per request. |
| Test harness | Verifikasi kontrak tool, auth, error, cache. |
| Manifest/registration helper | Daftarkan server + tools ke Hermes, filtering per agent. |

Konsekuensi: MCP baru = isi template + adapter upstream spesifik, bukan tulis ulang infrastruktur.

## 4. Isolasi Kredensial (prinsip inti)

```text
Config MCP server   →  hanya VAULT REF  (mis. "vault://ga4/service-account")
Runtime             →  MCP server resolve ref → VAULT → credential (in-memory)
Boundary             →  credential TIDAK pernah keluar dari MCP server:
                        ✗ tidak ke Hermes   ✗ tidak ke agent   ✗ tidak ke percakapan
                        ✗ tidak ke log (redacted)
```

- Agent/percakapan hanya melihat **hasil** tool, bukan secret.
- Ini memperkuat prinsip [Architecture Overview](../00-architecture-overview.md): "kredensial sumber data terisolasi di MCP server".

## 5. Rate-Limit & Caching Strategy

| Aspek | Strategi |
|-------|----------|
| Rate limit | Hormati kuota API pihak ke-3. Error `rate_limit` → backoff eksponensial + retry; throttle bila perlu. |
| Caching | Cache read-heavy dengan TTL per source (GA4 ~15 mnt, RSS ~5 mnt, BigQuery sesuai cost/freshness). |
| Cache key | `hash(server + tool + input args)` agar tidak salah serve. |
| Invalidation | Bypass/invalidate untuk data yang wajib fresh. |
| Cost guard (BigQuery) | Batas estimasi cost query + cache hasil untuk hindari rerun mahal. |

## 6. Registrasi ke Hermes

```text
MCP SERVER  ──manifest (tools + schema)──▶  HERMES MCP CLIENT
HERMES      ──register──▶  tools tersedia ke agent
            ──filter──▶    tiap agent hanya lihat tools relevan (Hermes mendukung filtering)
```

- Hermes "connect to any MCP server for extended tool capabilities" + filter tools — kita pakai itu, tidak membangun client.
- Pola registrasi distandarkan agar konsisten lintas MCP.

## 7. Deployment MCP Server

Dua pola, dipilih per kebutuhan (final deployment dibahas selaras Phase 0):

```text
SIDECAR                          STANDALONE
┌─────────────┐                  ┌─────────────┐     ┌──────────────┐
│ Hermes      │                  │ Hermes      │◀───▶│ MCP server   │
│  └ MCP srv  │ (co-located)     │             │ net │ (service     │
└─────────────┘                  └─────────────┘     │  terpisah)   │
                                                     └──────────────┘
```

- **Sidecar:** MCP server co-located dengan Hermes — sederhana, latensi rendah, cocok MCP ringan (RSS).
- **Standalone:** MCP server sebagai service terpisah — skalabel & terisolasi, cocok MCP berat/multi-source (BigQuery, CMS).
- Keduanya memakai template & komponen reusable yang sama; deployment tidak mengubah kontrak tool.

## 8. Prinsip yang Ditegakkan

1. **MCP sebagai kontrak** — agent tidak akses data langsung, selalu via MCP.
2. **Reusable** — satu MCP melayani banyak use case; komponen dibangun sekali.
3. **On-demand** — MCP dibangun saat use case menuntut.
4. **Kredensial terisolasi** — hidup hanya di MCP server.
