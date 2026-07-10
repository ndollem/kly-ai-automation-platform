# Rencana Eksekusi — Phase 0: Hermes Foundation

> Parent: [`../00-master-plan.md`](../00-master-plan.md) · [`../00-decision-log.md`](../00-decision-log.md) · PRD: [`prd.md`](prd.md)

## 1. Pendekatan

Phase 0 adalah **deploy + konfigurasi + hardening** instance Hermes existing — bukan pembangunan. Strategi: ambil keputusan terbuka dulu (deployment backend & LLM endpoint), provisioning, deploy minimal, sambungkan WhatsApp, kunci akses, lalu validasi end-to-end. Hindari menambah apa pun yang belum dituntut fondasi (no MCP server, no agent bisnis — itu fase lain).

Prinsip kerja (dari `00-master-plan.md`): **Hermes-native dulu** — jika kapabilitas sudah ada (memory, session, cron), pakai bawaan, jangan bangun ulang.

## 2. Keputusan yang Harus Diambil (blocking)

Dua ADR terbuka **wajib** diputuskan di awal Phase 0 — semua langkah deploy bergantung padanya.

### D1 — Deployment backend (rujuk [ADR-004](../00-decision-log.md))

| Opsi | Kelebihan | Kekurangan | Cocok bila |
|------|-----------|-----------|------------|
| **Docker (VPS)** | Kontrol penuh, uptime persisten, latency stabil, WA gateway selalu online | Butuh kelola VPS, biaya jalan terus, patching OS | Uptime WA jadi prioritas; ada budget VPS tetap |
| **Modal / Daytona (serverless)** | Hibernate saat idle (hemat), setup cepat, minim ops | Cold start; **risiko**: koneksi WA gateway bisa putus saat hibernate | Beban sporadis & sensitif biaya — **perlu validasi WA tetap online** |

> Rekomendasi default untuk fondasi WhatsApp: **Docker di VPS** demi koneksi gateway persisten. Catat keputusan final + alasan di ADR-004. Bila pilih serverless, **uji eksplisit** bahwa WA tidak drop saat instance hibernate (lihat task `T-DEC-01`).

### D2 — LLM endpoint (rujuk [ADR-005](../00-decision-log.md))

Kriteria pilih: **biaya**, **privasi data internal KLY**, **kualitas reasoning** (penting untuk Phase 2 analytics/revenue).

| Opsi | Catatan |
|------|---------|
| Nous Portal | Native ke Hermes; cek kuota & model tersedia |
| OpenRouter | Fleksibel multi-model, mudah bandingkan biaya/kualitas |
| OpenAI | Kualitas reasoning tinggi; pertimbangkan privasi data |
| Endpoint internal/privat | Terbaik untuk privasi; perlu infra sendiri |

> Phase 0 boleh mulai dengan endpoint murah/cepat untuk validasi fondasi, lalu finalisasi di ADR-005 sebelum Phase 2. **Jangan kirim data sensitif KLY ke LLM eksternal di Phase 0.**

## 3. Prasyarat (sebelum mulai)

- [ ] Akses ke akun infra (VPS provider **atau** Modal/Daytona).
- [ ] Nomor WhatsApp khusus untuk Hermes (bukan nomor personal).
- [ ] Kredensial LLM endpoint terpilih (API key + kuota).
- [ ] Secret manager / brankas rahasia tersedia (bukan plaintext di repo).
- [ ] Daftar awal allowlist identitas tim (nomor WA / identitas yang dipakai gateway).
- [ ] Akses dokumentasi Hermes: https://hermes-agent.nousresearch.com/docs/

## 4. Urutan Kerja (workstream)

```text
[0] Keputusan ADR-004 & ADR-005  ── blocking, lakukan duluan
        │
        ▼
[1] Provisioning + Secret mgmt
        │
        ▼
[2] Deploy Hermes (install, LLM, SOUL.md, Context File)
        │
        ▼
[3] WhatsApp Integration (pairing, inbound/outbound)
        │
        ├─────────────┬─────────────┐
        ▼              ▼               ▼
[4] Auth/Access   [5] Session     [6] Logging/
    (allowlist,       (verifikasi      Monitoring
     group/channel)   memory)          (log, health)
        └─────────────┴─────────────┘
                        │
                        ▼
[7] Validasi end-to-end (smoke test) + Runbook + MCP client check
```

Langkah [4]–[6] dapat berjalan paralel setelah WhatsApp tersambung.

## 5. Milestone & Timeline Kasar

Estimasi relatif (bukan komitmen tanggal). Asumsi 1 Platform Operator fokus.

| Milestone | Isi | Estimasi |
|-----------|-----|----------|
| **M0 — Decisions locked** | ADR-004 & ADR-005 diputuskan & dicatat | 0.5–1 hari |
| **M1 — Hermes hidup** | Provisioning + deploy + LLM endpoint merespons (`[1]`,`[2]`) | 0.5–1 hari (install ~60 dtk, sisanya konfigurasi/hardening) |
| **M2 — WA tersambung** | Pairing nomor; inbound/outbound terverifikasi (`[3]`) | 0.5–1 hari |
| **M3 — Aman & teramati** | Allowlist, group/channel, session, logging, monitoring (`[4]`–`[6]`) | 1–2 hari |
| **M4 — Tervalidasi** | Smoke test end-to-end, MCP client check, runbook teruji (`[7]`) | 0.5–1 hari + observasi uptime 7 hari |

**Total kerja aktif:** ±3–6 hari. **Exit Phase 0** menunggu window observasi uptime 7 hari (NFR-01) selesai.

## 6. Definition of Done (Phase 0)

Mengacu Success Metrics di [`prd.md`](prd.md) §7 dan exit criteria [`../00-roadmap.md`](../00-roadmap.md):

- Round-trip WA pilot user sukses; non-allowlisted ditolak.
- Memory lintas sesi terbukti.
- Log per request tersedia; monitoring aktif.
- MCP client menyambung ke 1 MCP test.
- Runbook deploy + rollback dijalankan oleh operator kedua.
- Uptime ≥ 99% selama 7 hari.
- ADR-004 & ADR-005 berstatus **Accepted** dengan alasan tercatat.

## 7. Strategi Risiko (ringkas)

- Mulai di **staging/instance uji** sebelum nomor WA "produksi" di-pairing → lindungi nomor dari banned (R1).
- Pin versi Hermes; uji upgrade terpisah (R2).
- Jika serverless dipertimbangkan → buktikan WA persisten dulu, kalau gagal fallback ke Docker (R3).
- Secret hanya via secret manager; review tidak ada key ter-commit (R4).
- Set batas rate & alert biaya LLM sejak hari pertama (R5).
