# Task List — Phase 0: Hermes Foundation

> Parent: [`plan.md`](plan.md) · [`prd.md`](prd.md) · [`../00-decision-log.md`](../00-decision-log.md)
> Estimasi effort relatif: **S** (jam), **M** (≈1 hari), **L** (>1 hari / butuh observasi). Centang `- [ ]` saat selesai.

## 0. Decisions (blocking — kerjakan duluan)

- [ ] `T-DEC-01` Putuskan deployment backend & catat di ADR-004 (Docker vs Modal/Daytona). Bila serverless dipertimbangkan, **uji WA gateway tetap online saat hibernate**. **(M)**
- [ ] `T-DEC-02` Putuskan LLM endpoint & catat di ADR-005 (biaya, privasi, kualitas reasoning). **(S)**
- [ ] `T-DEC-03` Tetapkan kebijakan akses: siapa di allowlist, kanal mana yang dilayani. **(S)**

## 1. Provisioning

- [ ] `T-PRV-01` Siapkan target deploy (VPS dengan akses SSH **atau** akun Modal/Daytona). **(M)**
- [ ] `T-PRV-02` Siapkan nomor WhatsApp khusus untuk Hermes (bukan personal). **(S)**
- [ ] `T-PRV-03` Siapkan secret manager / brankas; simpan API key LLM & kredensial gateway (no plaintext di repo). **(S)**
- [ ] `T-PRV-04` Catat baseline biaya target (infra + LLM) & set alert biaya. **(S)**
- [ ] `T-PRV-05` Siapkan repo/folder konfigurasi versi (reproducible deploy) — `SOUL.md`, Context File, env template. **(S)**

## 2. Deploy Hermes

- [ ] `T-DEP-01` Install Hermes: `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash` (verifikasi versi, pin versi). **(S)**
- [ ] `T-DEP-02` Konfigurasi LLM endpoint terpilih (dari T-DEC-02); test 1 prompt → respons. **(S)**
- [ ] `T-DEP-03` Tulis `SOUL.md` fondasi (personality global, nada, batasan dasar). **(S)**
- [ ] `T-DEP-04` Tulis Context File fondasi (konteks platform KLY, scope Phase 0, do/don't). **(S)**
- [ ] `T-DEP-05` Jalankan Hermes via backend terpilih; konfirmasi proses up & stabil. **(M)**
- [ ] `T-DEP-06` Aktifkan auto-restart / supervisor (Docker restart policy atau ekuivalen serverless). **(S)**

## 3. WhatsApp Integration

- [ ] `T-WA-01` Pairing nomor WA ke gateway bawaan Hermes (via prosedur gateway Hermes). **(M)**
- [ ] `T-WA-02` Verifikasi inbound: pesan dari pilot user diterima Hermes. **(S)**
- [ ] `T-WA-03` Verifikasi outbound: Hermes membalas ke pengirim yang sama. **(S)**
- [ ] `T-WA-04` Uji round-trip percakapan multi-turn (konteks terjaga dalam sesi). **(S)**
- [ ] `T-WA-05` Dokumentasikan langkah re-pairing bila koneksi WA putus (untuk runbook). **(S)**

## 4. Group & Channel Management

- [ ] `T-CHN-01` Tentukan kanal yang dilayani: DM personal vs group tim. **(S)**
- [ ] `T-CHN-02` Konfigurasi Hermes agar hanya merespons di kanal yang ditetapkan. **(S)**
- [ ] `T-CHN-03` Uji perilaku di group (mention/trigger) vs DM. **(S)**

## 5. Authentication & Access Control

- [ ] `T-AUTH-01` Implement allowlist identitas (hanya tim berwenang dilayani). **(M)**
- [ ] `T-AUTH-02` Uji penolakan: user di luar allowlist tidak dilayani / diabaikan. **(S)**
- [ ] `T-AUTH-03` Pastikan kredensial sumber/LLM tidak pernah bocor ke isi percakapan. **(S)**
- [ ] `T-AUTH-04` Dokumentasikan prosedur tambah/cabut akses (onboarding/offboarding). **(S)**

## 6. Session Management

- [ ] `T-SES-01` Verifikasi session lifecycle Hermes (mulai, lanjut, konteks dalam sesi). **(S)**
- [ ] `T-SES-02` Verifikasi memory lintas sesi (FTS5 + summarization) — fakta sesi lama teringat. **(M)**
- [ ] `T-SES-03` Uji isolasi: sesi user A tidak bocor ke user B. **(S)**
- [ ] `T-SES-04` Catat lokasi & retensi store session/memory (untuk backup & ERD). **(S)**

## 7. Logging & Monitoring

- [ ] `T-LOG-01` Aktifkan log per request (who, when, ringkasan, status, latency). **(M)**
- [ ] `T-LOG-02` Set retensi log ≥ 14 hari (NFR-04). **(S)**
- [ ] `T-LOG-03` Pasang health check instance (up/down). **(M)**
- [ ] `T-LOG-04` Pasang alert ke operator saat instance down / error spike. **(M)**
- [ ] `T-LOG-05` Pasang alert biaya LLM (ambang harian). **(S)**
- [ ] `T-LOG-06` Definisikan AuditLog: event apa yang dicatat (akses ditolak, error, restart). **(S)**

## 8. Validasi & Hand-off

- [ ] `T-VAL-01` Smoke test end-to-end: WA → Hermes → agent → respons (lihat runbook). **(S)**
- [ ] `T-VAL-02` Uji MCP client: sambungkan 1 MCP test → konfirmasi tool terlihat agent (siap Phase 1). **(M)**
- [ ] `T-VAL-03` Uji rollback/restart: kembalikan ke versi sehat ≤ 15 menit. **(M)**
- [ ] `T-VAL-04` Operator kedua menjalankan [`runbook.md`](runbook.md) dari nol (reproducibility). **(M)**
- [ ] `T-VAL-05` Window observasi uptime 7 hari → konfirmasi ≥ 99% (NFR-01). **(L)**
- [ ] `T-VAL-06` Review exit criteria Phase 0; update ADR-004 & ADR-005 ke **Accepted**. **(S)**

---

### Ringkasan estimasi

| Grup | Task | Beban dominan |
|------|------|---------------|
| Decisions | 3 | M, S, S |
| Provisioning | 5 | mostly S |
| Deploy Hermes | 6 | S–M |
| WhatsApp | 5 | S–M |
| Group/Channel | 3 | S |
| Auth | 4 | S–M |
| Session | 4 | S–M |
| Logging/Monitoring | 6 | S–M |
| Validasi | 6 | termasuk 1× **L** (observasi 7 hari) |

**Jalur kritis:** Decisions → Provisioning → Deploy → WhatsApp → Validasi. Auth/Session/Logging paralel setelah WA tersambung. Exit menunggu `T-VAL-05` (7 hari).
