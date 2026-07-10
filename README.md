# KLY AI Automation Platform

Platform AI Automation berbasis WhatsApp untuk seluruh tim KLY. Interaksi data, sistem internal, dan workflow bisnis lewat percakapan natural — tanpa membuka banyak dashboard.

**Orchestration layer:** [Hermes Agent](https://hermes-agent.nousresearch.com/docs/) (Nous Research, open-source / MIT). Hermes menyediakan gateway WhatsApp, memory lintas sesi, skills, MCP client, cron, dan subagents secara built-in. Platform ini **mengkonfigurasi dan memperluas** Hermes, bukan membangun orchestrator dari nol.

## Struktur Dokumen

| Folder | Isi |
|--------|-----|
| `00-master-plan.md` | Visi, prinsip, strategi keseluruhan |
| `00-roadmap.md` | Urutan fase, milestone, dependensi antar-fase |
| `00-architecture-overview.md` | Arsitektur tingkat platform (semua fase) |
| `00-glossary.md` | Definisi istilah (Hermes, MCP, agent, skill, dll) |
| `00-decision-log.md` | Keputusan arsitektur penting + alasannya (ADR) |
| `phase-0-hermes-foundation/` | Deploy & konfigurasi Hermes |
| `phase-1-data-access/` | MCP connector layer (akses data reusable) |
| `phase-2-business-use-cases/` | Agent A/B/C/D (Story+, Analytics, Competitor, Revenue) |
| `phase-3-proactive-intelligence/` | Alert proaktif berbasis anomaly/opportunity |

## Isi tiap folder fase

Setiap fase berisi (di-_tailor_ sesuai relevansi):

- `prd.md` — Product Requirements Document
- `plan.md` — Rencana eksekusi & strategi fase
- `task-list.md` — Breakdown task aktionable + estimasi
- `workflow.md` — Alur kerja / sequence runtime
- `architecture.md` — Arsitektur teknis fase
- `erd.md` — Data model (hanya fase yang punya state/data tersimpan)
- dokumen pendukung spesifik fase (runbook, MCP catalog, agent spec, dll)

## Status

Dokumen ideation — belum implementasi. Sumber: `worklog/20260625.md`.
