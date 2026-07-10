# Master Plan — KLY AI Automation Platform

## Visi

Membangun platform AI Automation berbasis WhatsApp yang memungkinkan seluruh tim KLY berinteraksi dengan data, sistem internal, dan workflow bisnis melalui percakapan natural — tanpa perlu membuka banyak dashboard atau tools berbeda.

Hermes Agent menjadi orchestration layer yang menghubungkan user, AI agents, MCP connectors, dan berbagai data source internal maupun eksternal. Pada tahap akhir, Hermes menjadi **AI Operating System** untuk berbagai unit bisnis KLY: content, analytics, revenue, monitoring, research, automation.

## Core Principles

### 1. WhatsApp First
WhatsApp jadi entry point utama — sudah dipakai sehari-hari oleh seluruh tim. User tidak perlu belajar dashboard atau aplikasi baru. (Hermes mendukung WhatsApp sebagai salah satu dari 20+ gateway bawaan.)

### 2. Use Case Driven
Pengembangan tidak dimulai dengan membangun seluruh connector & agent sekaligus. Setiap capability dibangun berdasarkan use case yang jelas dan punya nilai bisnis terukur.

### 3. MCP as Infrastructure
MCP bukan produk akhir — ia lapisan infrastruktur yang memungkinkan agent mengakses data & sistem secara standar. MCP hanya dibangun saat dibutuhkan oleh use case tertentu.

### 4. Reusable Platform
Setiap komponen harus dapat dipakai ulang oleh use case lain. Tujuan akhir bukan satu agent, tapi platform yang melahirkan banyak agent.

## Strategi Eksekusi

1. **Pondasi dulu, baru use case.** Phase 0 menstabilkan Hermes sebelum agent bisnis dibangun.
2. **MCP on-demand.** Tidak ada target menyelesaikan semua MCP lebih dulu (Phase 1 jalan beriringan dengan Phase 2).
3. **Nilai bisnis terukur.** Tiap use case Phase 2 punya outcome yang bisa diukur (waktu produksi, kecepatan analisa, dll).
4. **Reactive → Proactive.** Phase 3 mengubah pola dari "user bertanya" menjadi "agent mendeteksi & memberi tahu".

## Memanfaatkan Kapabilitas Bawaan Hermes

Hermes Agent sudah menyediakan banyak hal yang tidak perlu kita bangun ulang:

| Kebutuhan | Disediakan Hermes |
|-----------|-------------------|
| Gateway WhatsApp | ✅ Built-in (20+ platform) |
| Memory lintas sesi | ✅ FTS5 + LLM summarization |
| Eksekusi agent / skill | ✅ Skills (agentskills.io compatible) |
| Akses tools eksternal | ✅ MCP client ("connect any MCP server") |
| Notifikasi terjadwal | ✅ Built-in cron, delivery ke platform mana saja |
| Paralel workstream | ✅ Spawn isolated subagents |
| Deploy | ✅ local / Docker / SSH / Modal / Daytona |

**Implikasi:** fokus kerja kita = (a) deploy & harden Hermes, (b) membangun MCP server untuk data KLY, (c) menulis skills/agent per use case, (d) merancang cron + logika anomaly untuk Phase 3.

## End State Vision

```text
WhatsApp
    ↓
Hermes
    ├── Story+ Agent
    ├── Analytics Agent
    ├── Competitor Agent
    ├── Revenue Agent
    ├── Search Agent
    ├── Editorial Agent
    └── Future Agents
            ↓
        MCP Layer
    ├── GA4   ├── GSC   ├── BigQuery  ├── RSS
    ├── CMS   ├── Figma ├── Storage   └── Future Systems
```

Hermes menjadi lapisan interaksi tunggal antara user dan seluruh knowledge, data, workflow, serta sistem yang dimiliki KLY.
