# Workflow — Phase 3: Proactive Intelligence

> Parent: [Platform README](../README.md) · [Glossary](../00-glossary.md)
> Lihat juga: [architecture.md](./architecture.md) · [alert-catalog.md](./alert-catalog.md)

## 1. Reactive (Phase 2) vs Proactive (Phase 3)

Perbedaan inti = **siapa yang memulai** dan **kapan**.

```text
PHASE 2 — REACTIVE (pull)                  PHASE 3 — PROACTIVE (push)
─────────────────────                  ────────────────────────
User memulai. Kapan saja.                  Cron Hermes memulai. Terjadwal.

  User (WhatsApp)                             Cron Hermes (tick terjadwal)
       │ "berapa traffic hari ini?"                 │ trigger
       ▼                                            ▼
  Hermes → Agent                              Hermes → spawn Subagent(s)
       │                                            │
       ▼                                            ▼
  MCP fetch data                              MCP fetch data
       │                                            │
       ▼                                            ▼
  Jawab user                                  Evaluasi anomaly/opportunity
                                                    │
                                          ┌─────────┴─────────┐
                                     lolos & unik         tidak lolos
                                          │                  │ / duplikat
                                          ▼                  ▼
                                   Kirim ALERT ke WA     Diam (suppress/log)
```

> **Kunci:** Phase 2 menjawab pertanyaan; Phase 3 **memutuskan untuk diam atau bicara.** Default keputusan = **diam**. Hanya anomaly/opportunity yang lolos threshold & bukan duplikat yang menembus ke WhatsApp.

## 2. Alur Proactive End-to-End

```text
┌───────────────────────────────────────────────────────────────┐
│ 1. CRON TRIGGER (Hermes built-in)                                      │
│    Jadwal per kategori (mis. Traffic /15–30 mnt, Competitor /30–60 mnt)│
└───────────────────────────────┬──────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 2. SPAWN SUBAGENTS (paralel)                                           │
│    Tiap kategori / grup rule dievaluasi di Subagent terpisah.          │
│    Satu sumber lambat tidak memblok yang lain (isolasi & resilience).  │
└───────────────────────────────┬──────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 3. FETCH DATA via MCP (Phase 1)                                        │
│    GA4 MCP · RSS MCP · BigQuery/CMS/GSC MCP                            │
│    Kredensial terisolasi di MCP server — tidak terekspos ke logika.    │
└───────────────────────────────┬──────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 4. EVALUASI ANOMALY / OPPORTUNITY                                      │
│    Bandingkan metric vs MetricBaseline (per hari/jam).                 │
│    Terapkan AlertRule: relative threshold + absolute guard.            │
└───────────────────────────────┬──────────────────────────────┘
                  lolos threshold? ──── tidak ─► catat AlertEvent(detected,
                            │                       below-threshold) → STOP
                           ya
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 5. DEDUP / SUPPRESSION                                                  │
│    Sudah pernah dikirim dalam cooldown? (dedup key + window)           │
│    Kena rate-limit global subscriber?                                  │
└───────────────────────────────┬──────────────────────────────┘
              duplikat / ter-rate-limit ──► AlertEvent(suppressed) → STOP
                            │
                       unik & dalam batas
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 6. FORMAT & ROUTE                                                      │
│    Susun pesan ringkas + actionable. Tentukan audience (Subscription): │
│    Traffic→editorial · Competitor→redaksi · Revenue→commercial.        │
└───────────────────────────────┬──────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 7. DELIVER ke WhatsApp (Gateway Hermes)                                │
│    Catat AlertEvent(sent). Sertakan cara beri feedback (👍/👎).        │
└───────────────────────────────┬──────────────────────────────┘
                            ▼
┌──────────────────────────────────────────────────────────┐
│ 8. FEEDBACK LOOP (asinkron)                                            │
│    Reaksi penerima → FeedbackLog → review berkala → tuning AlertRule.  │
└───────────────────────────────────────────────────────┘
```

## 3. Titik Keputusan (Gate Anti-Spam)

Sebuah sinyal harus melewati **dua gerbang** sebelum jadi alert. Gagal di salah satu = diam.

| Gate | Pertanyaan | Gagal → |
|------|-----------|---------|
| **Gate 1 — Threshold** | Apakah ini benar-benar anomaly/opportunity vs baseline? | `detected` tapi tidak dikirim |
| **Gate 2 — Dedup/Rate** | Apakah ini baru (bukan duplikat) & masih dalam batas volume? | `suppressed` |

> Inilah implementasi prinsip **signal > noise**: lebih mudah bagi sinyal untuk **gagal** lolos daripada lolos.

## 4. Contoh Lintasan (Traffic)

```text
Cron 14:00 ─► Subagent "Traffic" ─► GA4 MCP: sessions 15 mnt terakhir
   ─► Baseline Rabu 14:00 = ~X; aktual = X − 45%
   ─► Gate 1: −45% > threshold −30%  ✔ lolos
   ─► Gate 2: belum ada alert drop dalam 60 mnt  ✔ unik
   ─► Format: "⚠️ Traffic turun 45% vs normal Rabu sore. Cek: …"
   ─► Kirim ke grup Editorial. AlertEvent(sent). Tawarkan feedback.
```

Bandingkan: bila aktual hanya −12% → Gate 1 gagal → **diam**, hanya tercatat.
