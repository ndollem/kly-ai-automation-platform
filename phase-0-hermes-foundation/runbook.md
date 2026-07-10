# Runbook — Phase 0: Hermes Foundation

> Parent: [`plan.md`](plan.md) · [`task-list.md`](task-list.md) · [`architecture.md`](architecture.md)
> Dokumen operasional. Dijalankan oleh **Platform Operator**. Target: deploy dari nol ≤ 1 jam (NFR-06); rollback ≤ 15 menit (NFR-05).

> ⚠️ **Sebelum mulai:** pastikan keputusan ADR-004 (deployment backend) & ADR-005 (LLM endpoint) sudah dibuat (task `T-DEC-01`, `T-DEC-02`). Runbook ini mengasumsikan **Docker/VPS** sebagai jalur default; bila memakai Modal/Daytona, sesuaikan langkah deploy ke prosedur backend tersebut (referensi: docs Hermes).

---

## 1. Prasyarat

- [ ] Akses SSH ke VPS (atau akun Modal/Daytona siap).
- [ ] Nomor WhatsApp khusus tersedia (bukan personal).
- [ ] API key LLM endpoint terpilih + kuota.
- [ ] Secret store/vault siap (untuk menyimpan key — **bukan** plaintext di repo).
- [ ] Daftar allowlist awal (identitas tim) & kanal yang dilayani.
- [ ] Docs Hermes: https://hermes-agent.nousresearch.com/docs/

---

## 2. Deploy Step-by-Step

### Langkah 1 — Install Hermes

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Install ±60 detik. Verifikasi versi terpasang dan **pin** versi tersebut untuk reproducibility (catat di konfigurasi).

### Langkah 2 — Konfigurasi LLM endpoint

Set endpoint + API key sesuai ADR-005 lewat env/secret (jangan hardcode). Contoh pola env (nilai dari vault):

```bash
# contoh — nama variabel ikuti konvensi konfigurasi Hermes (lihat docs)
export LLM_ENDPOINT="<nous-portal | openrouter | openai | endpoint-internal>"
export LLM_API_KEY="<dari secret store>"
```

Uji satu prompt sederhana → pastikan ada respons (validasi `T-DEP-02`).

### Langkah 3 — Konfigurasi personality & konteks

- Tulis `SOUL.md` (personality global, nada, batasan dasar).
- Tulis Context File fondasi (konteks platform KLY, scope Phase 0, do/don't).

Simpan keduanya di konfigurasi versi (reproducible).

### Langkah 4 — Jalankan Hermes (Docker)

Jalankan via backend terpilih dengan **restart policy: always** (auto-recover). Mount volume untuk **state (session/memory)** dan **logs**, serta inject secret via env/mount (bukan di image).

```text
container: hermes
  - restart: always
  - volume: ./data  (session/memory store)  ──► persisten
  - volume: ./logs  (request/audit logs)    ──► persisten
  - env: LLM_*, gateway creds               ──► dari secret store
```

Konfirmasi proses **up & stabil** (`T-DEP-05`).

### Langkah 5 — Pairing WhatsApp Gateway

Lakukan pairing nomor WA ke gateway bawaan Hermes mengikuti **prosedur gateway Hermes** (lihat docs — detail wire-protocol ditangani Hermes internal, tidak diatur manual di sini).

Verifikasi:
- Inbound: pesan pilot user diterima Hermes (`T-WA-02`).
- Outbound: Hermes membalas ke pengirim yang sama (`T-WA-03`).

### Langkah 6 — Auth, Channel, Logging

- Set **allowlist** identitas (default-deny) → uji user non-allowlist ditolak (`T-AUTH-02`).
- Tetapkan **kanal yang dilayani** (DM/group) (`T-CHN-02`).
- Aktifkan **log per request** + retensi ≥14 hari (`T-LOG-01`, `T-LOG-02`).
- Pasang **health check** + alert down/error/biaya (`T-LOG-03`–`T-LOG-05`).

---

## 3. Smoke Test (validasi end-to-end)

Jalankan berurutan; semua harus lulus sebelum Phase 0 dianggap selesai.

| # | Uji | Harapan |
|---|-----|---------|
| 1 | Pilot user kirim "halo" via WA | Hermes membalas (round-trip OK) |
| 2 | Multi-turn: tanya lanjutan tanpa ulang konteks | Konteks dalam sesi terjaga |
| 3 | Tutup sesi, mulai sesi baru, rujuk fakta sebelumnya | Memory lintas sesi mengingat (`T-SES-02`) |
| 4 | Kirim dari nomor non-allowlist | Tidak dilayani; AuditLog mencatat `denied` |
| 5 | Dua user berbeda | Tidak ada kebocoran konteks antar-user (`T-SES-03`) |
| 6 | Cek log | Tiap request punya entri (who/when/status/latency) |
| 7 | Sambungkan 1 MCP test | Tool MCP terlihat oleh agent (siap Phase 1, `T-VAL-02`) |

---

## 4. Troubleshooting Umum

| Gejala | Kemungkinan penyebab | Tindakan |
|--------|----------------------|----------|
| Tidak ada balasan WA sama sekali | Gateway WA putus / belum pairing | Cek health WA; **re-pairing** (lihat §5) |
| Balasan lambat (> NFR p95) | LLM endpoint lambat / model berat | Cek latency endpoint; pertimbangkan model lebih ringan (ADR-005) |
| Error "no response" dari agent | LLM key salah/kuota habis | Cek key di vault; cek kuota/billing endpoint |
| User sah ikut ditolak | Salah konfigurasi allowlist | Periksa entri allowlist & format identitas |
| Bot tidak merespons di group | Kanal belum di-serve / butuh trigger | Cek konfigurasi channel (`T-CHN-02`), aturan mention |
| Konteks hilang antar pesan | Store session/memory tidak persisten | Pastikan volume `./data` ter-mount & tidak ephemeral |
| Instance mati saat idle (serverless) | Hibernate memutus WA | **Evaluasi ulang ADR-004** → pindah ke Docker bila WA drop |
| Secret muncul di log/percakapan | Kebocoran kredensial | Hentikan, rotasi key, audit, perbaiki konfigurasi (NFR-03) |

---

## 5. Re-pairing WhatsApp

Bila koneksi WA putus:

1. Cek status gateway lewat health check / log.
2. Ikuti prosedur re-pairing gateway Hermes (docs).
3. Verifikasi ulang inbound + outbound (smoke test #1).
4. Catat kejadian di AuditLog (`event_type=error`/restart).

---

## 6. Rollback / Recovery (target ≤ 15 menit)

> Lakukan berurutan. Berhenti di langkah pertama yang memulihkan layanan.

1. **Restart container** (paling cepat):
   ```bash
   docker restart hermes   # atau ekuivalen backend serverless
   ```
   Tunggu health check hijau → smoke test #1.

2. **Rollback ke versi Hermes sehat terakhir** (bila restart tak menyelesaikan):
   - Deploy ulang versi yang sudah di-pin sebelumnya (lihat konfigurasi versi).
   - Pastikan volume `./data` & `./logs` **tetap** (jangan hapus state).
   - Smoke test #1–#3.

3. **Restore konfigurasi** (bila perubahan config penyebabnya):
   - Kembalikan `SOUL.md` / Context File / env ke versi sebelumnya dari konfigurasi versi.
   - Restart, smoke test.

4. **Eskalasi**: bila masih gagal > 15 menit, beri tahu lead; pertimbangkan re-provision instance bersih dari runbook §2 (state dari volume di-restore).

> ⚠️ **Jangan** menghapus volume data saat rollback kecuali sengaja me-reset state — itu menghilangkan session & memory lintas sesi.

---

## 7. Checklist Exit Phase 0

- [ ] Semua smoke test (§3) lulus.
- [ ] Rollback (§6) teruji ≤ 15 menit.
- [ ] Operator kedua menjalankan runbook ini dari nol (`T-VAL-04`).
- [ ] Uptime ≥ 99% selama observasi 7 hari (`T-VAL-05`).
- [ ] ADR-004 & ADR-005 di-update ke **Accepted**.
