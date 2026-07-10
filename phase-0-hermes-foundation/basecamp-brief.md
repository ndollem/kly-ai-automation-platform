# Basecamp Brief — Phase 0: Hermes Foundation

> Format Shape Up pitch. Sumber: graph `graphify-out` (community 4 "Phase 0 Foundation & Deployment") + `phase-0-hermes-foundation/*.md`, `00-decision-log.md` (ADR-004/005).

## 🔴 Problem Statement

Tim Commercial mau tahu "traffic GIIAS microsite hari ini berapa" — harus buka GA4 dashboard, login, pilih property, set date range, baca chart. Tim Editorial mau cek topik kompetitor — buka tool lain lagi. Tiap insight tersebar di dashboard berbeda, tiap orang harus belajar tool-nya masing-masing. Padahal semua tim sudah hidup di WhatsApp sepanjang hari.

Sebelum satu pun agent bisa dibangun, **belum ada lapisan yang menyambungkan WhatsApp ke sistem internal KLY**. Tidak ada pintu masuk. Status quo: nol automation, semua manual lewat dashboard.

## ⏳ Budget

- **Resources:** 1 engineer (DevOps/backend) + akses infra
- **Time Box:** **±1 minggu** kerja deploy (3–6 hari, milestone M0–M4) **+ 7 hari observasi** stabilitas sebelum lanjut Phase 1
- Hermes install ~60 detik — beban sebenarnya di konfigurasi, hardening, & integrasi WhatsApp, bukan coding

## 💡 Potential Solution

**Deploy & konfigurasi Hermes Agent** (Nous Research, open-source) sebagai orchestration layer — **bukan** membangun orchestrator dari nol. Hermes sudah menyediakan gateway WhatsApp, memory lintas sesi, skills, MCP client, cron, dan subagents secara built-in (ADR-001).

Lingkup kerja konkret:
- Provisioning + deploy Hermes (Docker/VPS sebagai default)
- Sambungkan **WhatsApp Gateway** → user bisa chat Hermes
- **Auth default-deny allowlist** + session management
- **Logging & monitoring** aktif
- Siapkan `SOUL.md` + Context File dasar
- Infra siap menerima MCP (Phase 1) & agent (Phase 2)

> Cukup konkret agar tim tahu harus apa, cukup abstrak agar detail deploy mereka putuskan sendiri.

## 📈 Estimation Impact

- ✅ User bisa berinteraksi dengan Hermes via WhatsApp (kirim pesan → dapat respons)
- ✅ Agent dapat menerima & memproses request
- ✅ Infrastruktur terbukti siap menerima MCP & use case berikutnya
- ✅ Stabil melewati 7 hari observasi tanpa intervensi manual

## 🕳️ Rabbit Holes

- **WhatsApp pairing/session drop** — koneksi WA putus & perlu re-pairing berulang (runbook wajib punya prosedur re-pairing)
- **Pilihan deployment backend** (ADR-004, masih _open_) — Docker/VPS vs Modal/Daytona serverless. Salah pilih → uptime WA bermasalah atau biaya bengkak
- **Pilihan LLM endpoint** (ADR-005, masih _open_) — Nous Portal / OpenRouter / OpenAI. Pertimbangan biaya & privasi data internal KLY
- **Auth model terlalu ketat/longgar** — default-deny salah konfigurasi bisa kunci tim sendiri

## 🚫 No-Gos

- ❌ Membangun MCP server apa pun → **Phase 1**
- ❌ Membangun agent bisnis (Story+/Analytics/Competitor/Revenue) → **Phase 2**
- ❌ Proactive alert / cron logic → **Phase 3**
- ❌ Akses data sumber mana pun (GA4, BigQuery, CMS) — Phase 0 hanya fondasi
- ❌ Membangun ulang kapabilitas yang sudah ada di Hermes (memory, cron, subagents)

---

**Keputusan tertunda:** ADR-004 (deployment backend) & ADR-005 (LLM endpoint) diputuskan di fase ini. Detail operasional: `runbook.md`.
