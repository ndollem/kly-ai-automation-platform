# Glossary — KLY AI Automation Platform

Istilah dipakai konsisten di seluruh dokumen fase.

| Istilah | Definisi |
|---------|----------|
| **Hermes / Hermes Agent** | Autonomous AI agent open-source dari Nous Research (MIT). Orchestration layer platform ini. Menyediakan gateway messaging, memory, skills, MCP client, cron, subagents. Docs: https://hermes-agent.nousresearch.com/docs/ |
| **Gateway** | Kanal masuk/keluar pesan Hermes. KLY memakai **WhatsApp**. Hermes mendukung 20+ (Telegram, Slack, Discord, dll). |
| **Agent** | Persona/kapabilitas AI untuk satu domain (mis. Analytics Agent). Di Hermes diwujudkan sebagai kombinasi Context Files + Skills + MCP tools. |
| **Skill** | Procedural memory Hermes — prosedur yang dibuat/dipakai-ulang agent. Mengikuti standar agentskills.io, portable via Skills Hub. |
| **MCP (Model Context Protocol)** | Protokol standar agar agent mengakses data & sistem eksternal. Hermes adalah MCP client; KLY membangun **MCP server** untuk tiap sumber data. |
| **MCP Connector / MCP Server** | Implementasi MCP untuk satu sumber data (GA4 MCP, BigQuery MCP, dll). Lapisan infrastruktur, dibangun on-demand. |
| **Subagent** | Workstream paralel terisolasi yang di-spawn Hermes untuk tugas konkuren. |
| **Cron** | Penjadwal bawaan Hermes untuk tugas terjadwal + delivery ke gateway mana pun. Dipakai Phase 3 (proactive alerts). |
| **SOUL.md** | File personality global Hermes. |
| **Context File** | File konteks per-project di Hermes (instruksi domain, batasan). |
| **Tool Gateway** | 4 tool bawaan Hermes via satu OAuth: web search, image gen, TTS, browser. |
| **Programmatic Tool Calling** | Eksekusi multi-step pipeline lewat `execute_code` dalam satu inference call. |
| **Use Case** | Kebutuhan bisnis konkret yang men-drive pembangunan capability (A/B/C/D di Phase 2). |
| **Proactive Alert** | Notifikasi berbasis anomaly/opportunity (Phase 3), bukan laporan rutin. |
| **Story+** | Produk microsite KLY; Use Case A mengotomasi produksinya. |
| **OMP** | Online Media Platform / monetization context KLY (sumber Use Case D). |
| **GA4 / GSC** | Google Analytics 4 / Google Search Console. |
| **CMS** | Content Management System internal KLY. |
