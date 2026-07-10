# Architecture Overview — KLY AI Automation Platform

Arsitektur tingkat platform (lintas fase). Detail per fase ada di `architecture.md` masing-masing folder.

## Layered Architecture

```text
┌───────────────────────────────────────────────┐
│  USER LAYER                                   │
│  Tim KLY (Editorial, Commercial, Analytics…)  │
└─────────────────┬─────────────────────
                    │ percakapan natural
┌─────────────────▼─────────────────────┐
│  GATEWAY LAYER                                  │
│  WhatsApp (via Hermes built-in gateway)         │
└─────────────────┬─────────────────────
┌─────────────────▼─────────────────────┐
│  ORCHESTRATION LAYER — HERMES AGENT             │
│  • Routing pesan & session                      │
│  • Memory lintas sesi (FTS5 + summarization)    │
│  • Auth & access control                        │
│  • Skills (procedural memory)                   │
│  • Subagents (paralel)                           │
│  • Cron (scheduled / proactive)                 │
│  • MCP client                                   │
└─────────────────┬─────────────────────
┌─────────────────▼─────────────────────┐
│  AGENT LAYER                                    │
│  Story+ · Analytics · Competitor · Revenue · …  │
│  (Context Files + Skills + MCP tools)           │
└─────────────────┬─────────────────────
┌─────────────────▼─────────────────────┐
│  MCP CONNECTOR LAYER (dibangun KLY, on-demand)  │
│  GA4 · GSC · BigQuery · RSS · CMS · Figma · …    │
└─────────────────┬─────────────────────
┌─────────────────▼─────────────────────┐
│  DATA / SYSTEM LAYER                            │
│  Google Analytics · Search Console · BigQuery   │
│  CMS Internal · RSS Feed · Figma · GCS · APIs   │
└───────────────────────────────────────────────┘
```

## Tanggung Jawab Tiap Layer

| Layer | Disediakan oleh | Dibangun KLY |
|-------|-----------------|--------------|
| Gateway | Hermes (built-in) | Konfigurasi nomor WA, group/channel |
| Orchestration | Hermes (built-in) | Deploy, hardening, auth policy, SOUL.md |
| Agent | — | Context Files + Skills per use case |
| MCP Connector | — | MCP server per sumber data (on-demand) |
| Data/System | Sistem existing KLY + pihak ke-3 | Kredensial, service account, akses |

## Prinsip Arsitektur Lintas Fase

1. **Hermes-native dulu.** Jika kapabilitas sudah ada di Hermes (cron, memory, subagent), pakai itu — jangan bangun ulang.
2. **MCP sebagai kontrak.** Agent tidak akses data langsung; selalu lewat MCP server. Ini menjaga reusability & isolasi kredensial.
3. **Stateless agent, stateful platform.** State (memory, session) dikelola Hermes & MCP backend, bukan di logika agent.
4. **On-demand expansion.** MCP & agent baru ditambah saat use case menuntut, bukan di-_front-load_.

## Pertimbangan Cross-Cutting

- **Keamanan & akses:** auth di layer Hermes; kredensial sumber data terisolasi di MCP server (tidak terekspos ke agent/percakapan).
- **Observability:** logging & monitoring Hermes; audit trail per request.
- **Deployment:** Hermes mendukung Docker/SSH/Modal/Daytona — pilihan final dibahas di Phase 0.
- **Model LLM:** Hermes works dengan Nous Portal / OpenRouter / OpenAI / endpoint mana pun — pilihan dibahas di Phase 0.

## Decision Log

Keputusan arsitektur penting dicatat di `00-decision-log.md` (format ADR).
