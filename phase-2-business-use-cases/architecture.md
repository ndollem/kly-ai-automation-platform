# Architecture — Phase 2: Business Use Cases

> Parent: [`../00-architecture-overview.md`](../00-architecture-overview.md) · Glossary: [`../00-glossary.md`](../00-glossary.md) · MCP layer: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md)

Fokus: **Agent Layer**. Layer lain (gateway, orchestration, MCP, data) dibahas di [`../00-architecture-overview.md`](../00-architecture-overview.md). Phase 2 tidak membangun runtime — ia mengonfigurasi Hermes per use case.

## 1. Anatomi Sebuah Agent

Satu Agent = tiga komponen di atas Hermes:

```text
┌──────────────────── AGENT ─────────────────────┐
│                                                        │
│  ┌────────────┐  ┌────────────┐  ┌───────────┐  │
│  │ Context File │  │   Skills     │  │  MCP tools  │  │
│  │ instruksi    │  │ procedural   │  │ akses data  │  │
│  │ domain,      │  │ memory       │  │ (Phase 1)   │  │
│  │ batasan,     │  │ (reusable    │  │             │  │
│  │ format       │  │ procedures)  │  │             │  │
│  └────────────┘  └────────────┘  └───────────┘  │
│         WHAT it knows   HOW it acts    WHERE data is   │
└──────────────────────────────────────────┘
            ▲                    ▲                ▲
            └──── dipakai oleh Hermes runtime ────┘
       (memory, subagents, execution disediakan Hermes)
```

| Komponen | Isi | Contoh |
|----------|-----|--------|
| **Context File** | Persona, domain, definisi metrik, batasan, format jawaban, clarification policy | "Analytics Agent: properti GA4 default = …; jangan tebak angka; jawab ≤5 baris" |
| **Skills** | Prosedur berulang (agentskills.io), portable | `query-ga4`, `resolve-timeframe`, `cluster-topics` |
| **MCP tools** | Koneksi ke MCP server (Phase 1) | GA4 MCP, RSS MCP, BigQuery MCP |

## 2. Kapabilitas Hermes yang Dimanfaatkan (tidak dibangun ulang)

| Kapabilitas Hermes | Dipakai Agent untuk |
|--------------------|---------------------|
| **Memory lintas sesi** (FTS5 + summarization) | Follow-up tanpa ulang konteks ("bandingkan dengan kemarin"); ingat preferensi user/properti default |
| **Subagents** (paralel terisolasi) | Tugas konkuren: Agent C memproses banyak feed; Agent A menarik banyak asset; Agent D query 4 MCP paralel |
| **Skills engine** | Eksekusi & reuse prosedur lintas Agent |
| **MCP client** | Memanggil MCP tools tanpa kode integrasi per-Agent |
| **Programmatic Tool Calling** (`execute_code`) | Pipeline multi-step Agent D (join BQ × CMS × GA4 × GSC) dalam satu inference |
| **Cron** | Tidak dipakai Phase 2 (milik Phase 3 — proactive) |

**Implikasi:** Agent = data (Context File) + prosedur (Skills) + koneksi (MCP). State & eksekusi milik Hermes → Agent tetap **stateless**, platform tetap **stateful**.

## 3. Pola Reusable Lintas Agent

```text
                 ┌───────────── TEMPLATE AGENT ─────────────┐
                 │ Context File skeleton + skill umum            │
                 └───────────────────┬─────────────────────┘
        ┌──────────────┬───────────┬───────────┬──────────────┐
        ▼                ▼           ▼           ▼                ▼
   Analytics(B)    Competitor(C)  Story+(A)  Revenue(D)     Future Agents
        │                │           │           │
        └──── skill umum: format-whatsapp-reply, resolve-timeframe, ──┘
                         clarify-missing-params, cite-source
```

- **Template Agent** — kerangka Context File standar; tiap Agent menyalin lalu mengisi domain.
- **Skill umum** — dipasang ke semua Agent (format, timeframe, clarify, cite).
- **Skill domain** — spesifik per Agent (`query-ga4`, `cluster-topics`, `query-revenue-bq`).
- **Eval harness** — kontrak uji seragam (akurasi, grounding, format).

Skill domain pun bisa naik-pangkat jadi reusable bila relevan (mis. `resolve-timeframe` lahir di B, dipakai D).

## 4. Routing Intent → Agent

Hermes menerima satu stream pesan WhatsApp; routing memilih Agent yang tepat.

```text
        pesan user
            │
            ▼
   ┌───────────────┐
   │  Intent Router       │  (config: keyword/pola → Agent)
   └───────┬───────────┘
   ┌─────────┬───────────┬───────────┐
   ▼         ▼           ▼           ▼
 "traffic" "topik     "buat       "revenue/
 "channel"  ramai"     story+"     inventory/
 "artikel"  "kompetitor"           impression"
   │         │           │           │
   ▼         ▼           ▼           ▼
  B         C           A           D
```

Mekanisme:

- **Keyword/pola** sebagai sinyal awal (cepat, deterministik).
- **LLM intent classification** untuk pesan ambigu (Hermes orchestration).
- **Fallback clarify** — bila tak yakin, tanya balik ("Mau analytics traffic atau revenue inventory?").
- **Context carry-over** — Hermes memory menjaga Agent aktif untuk follow-up sampai topik berganti.

Konfigurasi routing adalah artefak lintas-agent (lihat [`task-list.md`](./task-list.md) §0 dan [`plan.md`](./plan.md) §4).

## 5. Aliran Data Multi-MCP (kasus Agent D)

```text
            ┌── BigQuery MCP ──┐  revenue
 Revenue ───┼── CMS MCP ──────┤  judul/slug/inventory map
 Agent      ├── Analytics MCP ─┤  pageviews
            └── Search Console ┘  impression/position
                    │
                    ▼
            JOIN via URL/slug  ← kunci konsistensi (prototipe lebih dulu)
                    │
                    ▼
              insight terpadu
```

Risiko utama: **kunci join tidak konsisten** antar-sumber. Mitigasi arsitektural: normalisasi URL/slug di Skill `map-page-to-inventory`, validasi silang sebelum menyimpulkan (lihat [`agent-spec-D-revenue.md`](./agent-spec-D-revenue.md)).

## 6. Prinsip Arsitektur Phase 2

1. **Hermes-native dulu** — memory/subagent/execution dari Hermes, bukan kode Agent.
2. **MCP sebagai kontrak** — Agent tak akses data langsung; selalu via MCP.
3. **Stateless agent** — Context File & Skills deklaratif; state di Hermes/MCP.
4. **Isolasi Agent** — menambah Agent tak mengubah Agent lain (Context File terpisah).
5. **Reusable-by-default** — apa pun yang dipakai >1 Agent jadi skill/template bersama.
