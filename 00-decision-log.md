# Decision Log (ADR) — KLY AI Automation Platform

Keputusan arsitektur penting + alasannya. Format ringkas ADR. Tambah entri baru di atas.

---

## ADR-001 — Hermes Agent sebagai orchestration layer

**Status:** Accepted
**Konteks:** Butuh orchestration layer yang menghubungkan WhatsApp, AI agents, MCP, dan data internal.
**Keputusan:** Pakai Hermes Agent (Nous Research, MIT) alih-alih membangun orchestrator sendiri.
**Alasan:** Hermes sudah menyediakan gateway WhatsApp, memory lintas sesi, skills, MCP client, cron, dan subagents secara built-in. Membangun ulang = buang waktu.
**Konsekuensi:** Tergantung pada roadmap & API Hermes. Fokus tim bergeser ke MCP server + skills + konfigurasi, bukan infrastruktur orchestration.

---

## ADR-002 — MCP sebagai satu-satunya jalur akses data

**Status:** Accepted
**Konteks:** Agent butuh akses GA4, BigQuery, CMS, dll.
**Keputusan:** Semua akses data lewat MCP server. Agent tidak query sumber data langsung.
**Alasan:** Standarisasi, reusability lintas use case, isolasi kredensial dari percakapan.
**Konsekuensi:** Ada overhead membangun MCP server per sumber, tapi sekali dibangun dipakai banyak agent.

---

## ADR-003 — Pembangunan MCP on-demand (bukan front-load)

**Status:** Accepted
**Konteks:** Ada banyak sumber data potensial.
**Keputusan:** MCP hanya dibangun saat use case yang akan segera dipakai membutuhkannya.
**Alasan:** Prinsip Use Case Driven & MCP as Infrastructure. Hindari membangun connector yang belum jelas dipakai.
**Konsekuensi:** Phase 1 & Phase 2 berjalan beriringan, bukan sekuensial.

---

## ADR-004 — (Open) Pilihan deployment backend Hermes

**Status:** Proposed — diputuskan di Phase 0
**Konteks:** Hermes mendukung local / Docker / SSH / Daytona / Singularity / Modal.
**Opsi:** Docker (kontrol penuh, butuh VPS) vs Modal/Daytona (serverless, hibernate idle).
**Catatan:** Putuskan berdasarkan kebutuhan uptime WhatsApp & budget infra.

---

## ADR-005 — (Open) Pilihan LLM endpoint

**Status:** Proposed — diputuskan di Phase 0
**Konteks:** Hermes works dengan Nous Portal / OpenRouter / OpenAI / endpoint mana pun.
**Catatan:** Pertimbangkan biaya, privasi data internal KLY, dan kualitas reasoning untuk use case analytics/revenue.
