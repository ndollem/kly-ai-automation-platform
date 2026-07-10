# Storyboard & Plan Presentasi KLY Automation

Dokumen ini berisi draf narasi, visual storyboard, spesifikasi infrastruktur, dan panduan teknis untuk file presentasi interaktif `index.html`. Dokumen ini diperbarui untuk mencerminkan pendekatan minimal effort (automation atas proses/data yang sudah ada), dukungan lintas platform (WhatsApp, Google Chat, Slack, Discord, Telegram), dan menyediakan tautan langsung ke file dokumentasi teknis di setiap fasenya.

---

## 🎨 Tema Desain & Visual (Anthropic Warm-Calm Tone)
*   **Warna Latar Belakang:** Warm Paper/Cream (`#FBF9F6`) — nyaman dipandang lama.
*   **Warna Teks Utama:** Charcoal/Off-Black (`#191919`) — tingkat kontras tinggi dan elegan.
*   **Warna Aksen:** Sunset Orange (`#E05A36` / `#E57A44`) — memberikan aksen hangat, modern, dan presisi.
*   **Warna Sukses/Sinyal:** Soft Sage Green (`#E8F0EC` background, `#2A6F4E` text).
*   **Tipografi:**
    *   *Headings:* Georgia atau Serif system (premium, editorial, intellectual feel).
    *   *Body/UI:* Inter atau Sans-Serif system (modern, clean, readable).

---

## 🗺️ Struktur & Alur Slide Deck (8 Slide Desktop-First)

### Slide 1: Visi Utama — KLY Automation & Assistant
*   **Target Audiens:** Semua Divisi
*   **Pesan Kunci:** Mengintegrasikan data dan alur kerja internal KLY langsung ke aplikasi chat harian Anda. Cepat, instan, dan *minimal effort*.
*   **Teks Narasi (Kiri):**
    > "Kita tidak sedang membuat sistem baru yang rumit atau mengubah cara kerja tim. **KLY Automation & Assistant** adalah asisten pintar minimalis yang menghubungkan data analitik, monitoring kompetitor, dan status sistem langsung ke sarana komunikasi yang sudah Anda gunakan setiap hari. 
    >
    > Ini adalah otomatisasi praktis untuk melipatgandakan kecepatan kerja tanpa menambah beban overhead teknologi."
*   **Tautan Dokumen Pendukung:**
    *   [`00-master-plan.md`](00-master-plan.md) — Rencana Induk & Visi Platform
    *   [`README.md`](README.md) — Pengenalan Umum Project
*   **Interaktivitas Visual (Kanan):** Mockup handphone menampilkan chat WhatsApp asisten pintar yang masuk satu per satu dengan ketukan delay alami (bila tombol ditekankan).

---

### Slide 2: Akses Lintas Platform (Aktifkan di Mana Saja)
*   **Target Audiens:** Editorial & Product
*   **Pesan Kunci:** Fleksibilitas penuh. Asisten ini tidak hanya berjalan di WhatsApp, tetapi juga siap diaktifkan di **Google Chat** (standar harian Google Workspace KLY), **Slack**, **Discord**, maupun **Telegram**.
*   **Teks Narasi (Kiri):**
    > "Setiap divisi memiliki ruang kerja digital favoritnya sendiri. Oleh karena itu, KLY Assistant dirancang modular:
    > *   **Google Chat:** Ideal untuk kolaborasi langsung di ekosistem Gmail kantor.
    > *   **WhatsApp:** Terbaik untuk kecepatan respons tinggi di HP saat berada di lapangan.
    > *   **Slack & Discord:** Sempurna untuk tim Tech dan operasional yang butuh integrasi channel khusus."
*   **Tautan Dokumen Pendukung:**
    *   [`00-master-plan.md`](00-master-plan.md) — Bagian Kapabilitas Bawaan & Integrasi Gateway
*   **Interaktivitas Visual (Kanan):** Selektor tab platform (WhatsApp, Google Chat, Slack, Telegram). Klik tab akan mengubah visual mockup obrolan secara instan dengan efek transisi GSAP yang memukau.

---

### Slide 3: Phase 0 — Pondasi Sistem (KLY Assistant Core)
*   **Target Audiens:** Tech & Operations
*   **Pesan Kunci:** Memastikan server dasar stabil, sistem whitelist nomor telepon aktif, dan pencatatan audit logging berjalan aman sebelum menyentuh data sensitif.
*   **Teks Narasi (Kiri):**
    > "Kami memulai dengan langkah aman dan teruji. Phase 0 bertujuan untuk:
    > *   **Deploy Core Engine:** Menjalankan instance dasar KLY Assistant yang andal.
    > *   **Membuka Webhook Secure:** Menghubungkan gateway chat terenkripsi.
    > *   **Whitelist Authorization:** Memastikan hanya staf terdaftar yang bisa berkirim pesan (Default-Deny).
    > *   **Audit Logging:** Mencatat setiap kueri demi kepatuhan dan keamanan data."
*   **Tautan Dokumen Pendukung:**
    *   [`phase-0-hermes-foundation/prd.md`](phase-0-hermes-foundation/prd.md) — Persyaratan Fondasi
    *   [`phase-0-hermes-foundation/plan.md`](phase-0-hermes-foundation/plan.md) — Rencana Eksekusi & Deployment
    *   [`phase-0-hermes-foundation/architecture.md`](phase-0-hermes-foundation/architecture.md) — Arsitektur Teknis Server
*   **Interaktivitas Visual (Kanan):** Diagram alur audit log interaktif yang menampilkan bagaimana data pesan diproses secara aman.

---

### Slide 4: Phase 1 — Standardisasi Akses Data (Secure MCP)
*   **Target Audiens:** Tech & Security Team
*   **Pesan Kunci:** Mengamankan kredensial data internal. Asisten AI tidak pernah mengakses database langsung, melainkan melalui standar universal *Model Context Protocol* (MCP).
*   **Teks Narasi (Kiri):**
    > "Bagi tim Tech, keamanan data adalah prioritas utama. Dengan arsitektur MCP:
    > *   **Zero Credentials Leak:** Kredensial BigQuery, GA4, dan CMS terisolasi di layer MCP server — tidak ada API keys dalam file obrolan AI.
    > *   **Reusable Connector:** Sekali konektor GA4 dibangun, ia dapat digunakan kembali oleh agen atau asisten mana pun di masa mendatang."
*   **Tautan Dokumen Pendukung:**
    *   [`phase-1-data-access/prd.md`](phase-1-data-access/prd.md) — PRD Konektor Data
    *   [`phase-1-data-access/plan.md`](phase-1-data-access/plan.md) — Rencana Build MCP
    *   [`phase-1-data-access/architecture.md`](phase-1-data-access/architecture.md) — Arsitektur Keamanan Data
    *   [`phase-1-data-access/erd.md`](phase-1-data-access/erd.md) — Skema & Relasi Kredensial MCP
*   **Interaktivitas Visual (Kanan):** Diagram arsitektur interaktif. Sorot kursor ke tiap layer untuk melihat detail keamanan isolasi data.

---

### Slide 5: Phase 2 — Use Cases: Editorial & Content (Story+ & Radar Kompetitor)
*   **Target Audiens:** Redaksi & Tim Konten
*   **Pesan Kunci:** Memproduksi draf berita SEO-friendly instan dari rilis pers, serta memantau pergerakan viral kompetitor secara real-time.
*   **Teks Narasi (Kiri):**
    > "Asisten ini bertindak sebagai asisten redaksi pribadi:
    > *   **Story+ (Agent A):** Mengonversi fakta kunci, rilis pers, atau ringkasan wawancara menjadi draf berita terstruktur SEO dalam hitungan detik.
    > *   **Radar Kompetitor (Agent C):** Memindai RSS kompetitor 24/7. Begitu ada berita kompetitor yang mendadak viral, notifikasi instan langsung dikirim ke chat tim Redaksi."
*   **Tautan Dokumen Pendukung:**
    *   [`phase-2-business-use-cases/prd.md`](phase-2-business-use-cases/prd.md) — Persyaratan Use Case Bisnis
    *   [`phase-2-business-use-cases/plan.md`](phase-2-business-use-cases/plan.md) — Rencana Rollout Use Case
    *   [`phase-2-business-use-cases/workflow.md`](phase-2-business-use-cases/workflow.md) — Alur Kerja Otomasi Redaksional
*   **Interaktivitas Visual (Kanan):** Demo interaktif notifikasi alert kompetitor viral di WhatsApp diikuti simulasi otomatisasi penyusunan draf berita.

---

### Slide 6: Phase 2 — Use Cases: Product & Revenue (Yield Protektor)
*   **Target Audiens:** Product Managers & Revenue Team
*   **Pesan Kunci:** Deteksi dini penurunan fill rate iklan dan anomali traffic secara otomatis.
*   **Teks Narasi (Kiri):**
    > "Untuk menjaga performa monetisasi situs, KLY Assistant memantau sinyal-sinyal kritis:
    > *   **Revenue & Inventory (Agent D):** Menghubungkan data inventori halaman CMS dengan BigQuery untuk mendeteksi anomali CPM atau fill rate iklan secara langsung.
    > *   **Proactive Alerting:** Jika ada iklan premium di halaman depan gagal ter-load, sistem segera mendeteksi dan memberi tahu tim Ad-Ops sebelum kerugian membengkak."
*   **Tautan Dokumen Pendukung:**
    *   [`phase-2-business-use-cases/prd.md`](phase-2-business-use-cases/prd.md) — Persyaratan Produk & Revenue
    *   [`phase-2-business-use-cases/erd.md`](phase-2-business-use-cases/erd.md) — Model Data Log & Metrik Yield
*   **Interaktivitas Visual (Kanan):** Mini-dashboard interaktif dengan bar chart. Tombol "Simulasi Drop" akan menurunkan bar chart pendapatan iklan dan memicu push alert anomali otomatis.

---

### Slide 7: Setup Infrastruktur & Persyaratan Teknis
*   **Target Audiens:** Tech Team & IT Infra
*   **Pesan Kunci:** Kebutuhan sistem yang minimal dan mudah di-deploy di infrastruktur yang sudah dimiliki KLY saat ini.
*   **Teks Narasi (Kiri):**
    > "Kami memastikan setup awal ini sangat ramah sumber daya (resource-efficient) dan dapat di-host di server cloud/on-premise manapun dengan biaya minimal:
    > *   **Dockerized Deployment:** Terbungkus rapi dalam container Docker untuk portabilitas penuh.
    > *   **Low Resource Footprint:** Cukup dengan spesifikasi server virtual standar.
    > *   **Secure Keys Management:** Integrasi aman untuk penyimpanan kunci API pihak ketiga."
*   **Tautan Dokumen Pendukung:**
    *   [`00-decision-log.md`](00-decision-log.md) — Catatan Keputusan Arsitektur
    *   [`phase-0-hermes-foundation/plan.md`](phase-0-hermes-foundation/plan.md) — Bagian Prasyarat Deployment
*   **Interaktivitas Visual (Kanan):** Bento grid interaktif yang menampilkan rincian:
    1.  *Server Host:* Virtual Machine / VPS Hemat / Mini (1 vCPU, 1GB RAM)
    2.  *Software:* Python 3.10+, Node.js 18+, Docker Engine
    3.  *Koneksi Gateway:* Webhook HTTPS (SSL) + 1 Nomor WhatsApp Biasa (tanpa pihak ke-3 / Twilio / WABA) + Google Chat API
    4.  *Model API:* OpenAI / OpenRouter / Gemini API Key / Model Lokal Privat

---

### Slide 8: Roadmap Eksekusi & Phased Rollout
*   **Target Audiens:** Management & Stakeholders
*   **Pesan Kunci:** Implementasi bertahap berisiko rendah yang memberikan hasil nyata sejak minggu pertama.
*   **Teks Narasi (Kiri):**
    > "Kami mengadopsi prinsip rilis bertahap (agile rollout) agar tim tidak perlu menunggu berbulan-bulan untuk merasakan manfaatnya:
    > *   **Phase 0 (W1-2):** Setup Fondasi Server & Gateways.
    > *   **Phase 1 & 2 (W3-4):** Integrasi Konektor Data & Aktivasi Agen Redaksi/Analitik.
    > *   **Phase 3 (W5-6):** Aktivasi Otomasi Proaktif & Sistem Alarm Anomali."
*   **Tautan Dokumen Pendukung:**
    *   [`00-roadmap.md`](00-roadmap.md) — Detil Rencana & Dependensi Fase
*   **Interaktivitas Visual (Kanan):** Garis waktu (timeline) interaktif horizontal. Klik tiap checkpoint fase untuk memunculkan syarat kelolosan (*Exit Criteria*) dan deliverable konkretnya secara instan.
