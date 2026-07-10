# ERD — Phase 2: Business Use Cases

> Parent: [`../README.md`](../README.md) · Architecture: [`./architecture.md`](./architecture.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

Data model **Agent Layer**. Sebagian state milik Hermes (memory, session) — di sini dirujuk, bukan dimiliki KLY. Entitas yang KLY definisikan/simpan: definisi Agent, Skill, konfigurasi use case, dan output (ProductionJob untuk Agent A).

## 1. Diagram Relasi

```text
┌──────────────────┐        ┌──────────────────┐
│  UseCaseConfig   │ 1    1 │ AgentDefinition  │
│  (A/B/C/D)       │───────▶│                  │
└──────────────────┘        └───┬───────┬──────┘
                                │ 1   N │ 1   N
                                ▼       ▼
                       ┌────────────┐ ┌────────────┐
                       │ AgentSkill   │ │ AgentMCPBind │
                       │ (M:N → Skill)│ │ (M:N → MCP) │
                       └──────┬─────┘ └──────┬─────┘
                              │ N             │ N
                              ▼               ▼
                       ┌──────────┐ ┌──────────┐
                       │   Skill      │ │  MCPServer   │
                       │ (reusable)   │ │ (Phase 1)    │
                       └──────────┘ └──────────┘

┌─────────────────┐        ┌──────────────────┐
│ ConversationSession│ N    1 │ AgentDefinition  │
│ (rujuk Hermes mem) │───────▶│                  │
└─────────┬──────────┘        └──────────────────┘
          │ 1
          │ N (hanya Agent A)
          ▼
┌─────────────────┐
│  ProductionJob     │  output Story+ (Use Case A)
│                    │
└─────────┬──────────┘
          │ 1   N
          ▼
┌─────────────────┐
│  ProductionAsset   │
└─────────────────┘
```

Catatan: `AgentSkill` & `AgentMCPBind` adalah tabel jembatan M:N (satu Skill/MCP dipakai banyak Agent — pola reusable).

## 2. Entitas

### AgentDefinition
Definisi satu Agent (kombinasi Context File + Skills + MCP).

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `agent_id` | PK | mis. `A`, `B`, `C`, `D` |
| `name` | string | "Analytics Insight Agent" |
| `domain` | enum | Editorial / Analytics / Commercial / Production |
| `context_file_ref` | string | path/versi Context File |
| `routing_keywords` | list | sinyal intent router |
| `status` | enum | draft / uat / live |
| `version` | string | versi definisi |

### UseCaseConfig
Konfigurasi bisnis use case yang men-drive sebuah Agent.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `use_case_id` | PK | A / B / C / D |
| `agent_id` | FK → AgentDefinition | 1:1 |
| `objective` | string | tujuan bisnis |
| `success_metric` | string | indikator terukur (PRD §7) |
| `wave` | int | urutan rilis (1–3) |
| `required_mcps` | list | MCP yang dibutuhkan |

### Skill
Procedural memory (agentskills.io), reusable lintas Agent.

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `skill_id` | PK | mis. `resolve-timeframe` |
| `name` | string | nama skill |
| `scope` | enum | common / domain |
| `description` | string | apa yang dilakukan |
| `version` | string | versi skill |

### AgentSkill (jembatan M:N)
| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `agent_id` | FK → AgentDefinition | |
| `skill_id` | FK → Skill | |
| PK | (`agent_id`,`skill_id`) | komposit |

### MCPServer (rujuk Phase 1)
| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `mcp_id` | PK | mis. `ga4`, `rss`, `bigquery` |
| `name` | string | "Google Analytics MCP" |
| `status` | enum | planned / live (Phase 1) |

### AgentMCPBind (jembatan M:N)
| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `agent_id` | FK → AgentDefinition | |
| `mcp_id` | FK → MCPServer | |
| PK | (`agent_id`,`mcp_id`) | komposit |

### ConversationSession (rujuk Hermes memory — dimiliki Hermes)
Tidak disimpan KLY; dirujuk untuk konteks. Hermes mengelola memory (FTS5 + summarization).

| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `session_id` | PK (Hermes) | id sesi Hermes |
| `user_ref` | string | identitas WA (via gateway) |
| `active_agent_id` | FK → AgentDefinition | Agent yang sedang menangani |
| `memory_ref` | string | handle ke memory Hermes |
| `last_intent` | string | untuk context carry-over |

### ProductionJob (output Use Case A)
| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `job_id` | PK | id job produksi |
| `session_id` | FK → ConversationSession | asal request |
| `brief_ref` | string | brief campaign |
| `figma_ref` | string | desain sumber (Figma MCP) |
| `quality_gate` | enum | pass / warn / fail |
| `approval_status` | enum | pending / approved / rejected |
| `published_url` | string | link microsite live (bila approved) |
| `status` | enum | draft / generated / published |

### ProductionAsset (Use Case A)
| Atribut | Tipe | Keterangan |
|---------|------|------------|
| `asset_id` | PK | id asset |
| `job_id` | FK → ProductionJob | |
| `type` | enum | image / video / copy |
| `storage_ref` | string | lokasi (File/Storage MCP) |
| `alt_text` | string | untuk quality gate |

## 3. Relasi (ringkas)

| Relasi | Kardinalitas |
|--------|--------------|
| UseCaseConfig — AgentDefinition | 1 : 1 |
| AgentDefinition — Skill | M : N (via AgentSkill) |
| AgentDefinition — MCPServer | M : N (via AgentMCPBind) |
| AgentDefinition — ConversationSession | 1 : N |
| ConversationSession — ProductionJob | 1 : N (hanya Agent A) |
| ProductionJob — ProductionAsset | 1 : N |

> Entitas output hanya ada untuk Agent A (produksi microsite). Agent B/C/D bersifat read/insight — tak menyimpan output domain; jawaban hidup di ConversationSession (memory Hermes).
