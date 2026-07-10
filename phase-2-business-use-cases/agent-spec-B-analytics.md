# Use Case B — Analytics Insight Agent — Phase 2

> Parent: [`./prd.md`](./prd.md) · Workflow: [`./workflow.md`](./workflow.md) · MCP: [`../phase-1-data-access/mcp-catalog.md`](../phase-1-data-access/mcp-catalog.md) · Glossary: [`../00-glossary.md`](../00-glossary.md)

## Objective

Memberi akses cepat ke insight **Google Analytics (GA4)** lewat WhatsApp — mengurangi ketergantungan pada dashboard dan mempercepat analisa.

## Target User

Tim **Analytics** dan siapa pun yang butuh metrik traffic cepat (Editorial, Commercial). Persona: Analytics. Rilis **Wave 1** (single-MCP, value cepat — sekaligus tempat menempa template Agent & eval harness).

## MCP Dependency

- **Google Analytics MCP** — query GA4 (users, sessions, pageviews, channel, top pages), read-only.

## Contoh Percakapan (input → output)

```text
User:  Traffic hari ini berapa?
Agent: [resolve-timeframe → hari ini] [query-ga4]
       Hari ini (dari GA4): Users 48.2K · Sessions 61.7K. Naik ~6% vs kemarin.
```

```text
User:  Artikel teratas minggu ini?
Agent: [resolve-timeframe → minggu ini] [query-ga4: top pages]
       Top minggu ini (dari GA4): 1) "…" 312K · 2) "…" 188K · 3) "…" 154K
```

```text
User:  Kenapa traffic turun?
Agent: [compare-period] [explain-anomaly]
       Minggu ini −9% WoW. Penurun utama: Organic Search −14%. Direct stabil.
```

```text
User:  Channel mana yang bertumbuh?
Agent: [query-ga4: sessions by channel WoW]
       Tumbuh: Referral +18%, Organic Social +11%. Turun: Organic Search −14%.
```

## Skills yang Dibutuhkan

| Skill | Scope | Fungsi |
|-------|-------|--------|
| `query-ga4` | domain | Query metrik/dimensi GA4 via MCP |
| `summarize-traffic` | domain | Ringkas metrik jadi insight WA |
| `compare-period` | domain | Bandingkan periode (DoD/WoW/MoM) |
| `explain-anomaly` | domain | Pecah penurunan/kenaikan per channel/page |
| `resolve-timeframe` | common | "hari ini/minggu ini/kemarin" → rentang tanggal |
| `format-whatsapp-reply` | common | Format jawaban WA |
| `cite-source` | common | Tandai "dari GA4" |
| `clarify-missing-params` | common | Tanya properti/rentang bila ambigu |

## Context File Outline

```text
1. Persona & objective — analis cepat, jawaban ringkas actionable
2. Properti GA4 default — id properti, timezone, definisi "hari ini"
3. Definisi metrik — users vs sessions vs pageviews; channel grouping
4. Format jawaban — ≤5 baris, angka tebal, tawarkan drill-down
5. Aturan grounding — angka HANYA dari GA4; tidak menebak
6. Clarification policy — rentang/properti ambigu → tanya
7. Error handling — MCP timeout/empty → jelaskan, jangan halusinasi
```

## Success Metric

- **Utama:** jumlah query/minggu via WA; waktu ke-insight turun.
- **Pendukung:** % pertanyaan terjawab tanpa buka dashboard; akurasi jawaban vs GA4.

## Edge Case & Batasan

- **Data GA4 belum final** (hari berjalan) → beri disclaimer "data hari ini belum lengkap".
- **Properti banyak** → konfirmasi properti bila tak default.
- **Pertanyaan di luar GA4** (revenue, search) → arahkan ke Agent D, jangan jawab dari GA4.
- **Rentang ekstrem** (setahun) → konfirmasi sebelum query berat.
- **MCP kosong/error** → laporkan, tawarkan ulang.
