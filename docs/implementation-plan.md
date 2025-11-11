# Implementation Plan — Local IEM + Talkback (AI-agent friendly)

Tujuan: Berikan rencana implementasi terstruktur untuk memungkinkan AI agent (atau tim) menerjemahkan tiap tujuan menjadi perubahan kode yang terukur, dapat dieksekusi, dan diverifikasi.

Panduan singkat:
- Dokumen ini tidak mengandung kode sumber, hanya tujuan, metrik penerimaan, dan dependensi.
- Setiap item kecil (task) ditulis sebagai satu tugas terukur yang dapat dieksekusi oleh AI agent.

1 — Scope dan Deliverables

- Deliverable utama:
  - Headless JUCE C++ server yang melakukan capture, mix per-klien, enkode Opus, dan stream UDP.
  - Flutter Web dashboard yang berkomunikasi lewat WebSocket/HTTP untuk control dan monitoring.
  - Flutter mobile client yang menerima stream, decode Opus, dan mengirim talkback melalui UDP.
  - Dokumen lisensi & compliance untuk integrasi kode pihak ketiga (mis. SonoBus).

2 — Milestones (tingkat tinggi)

- M1: Infrastruktur jaringan & discovery — UDP discovery, client/server handshake.
  - Ukuran: discovery broadcast berhasil mendeteksi client dalam LAN pada 3 percobaan berturut-turut.
- M2: Basic audio path (one-way) — server menangkap audio, encode Opus, kirim UDP; client terima, decode, dan playback.
  - Ukuran: end-to-end audio loop bekerja pada jaringan lokal dengan buffer default, latency < 30 ms pada pengukuran awal.
- M3: Per-client mixing & per-user routing.
  - Ukuran: server menghasilkan mix stereo berbeda untuk 2 klien simultan pada mesin pengujian.
- M4: Talkback (PTT) satu-arah dan full-duplex latensi rendah.
  - Ukuran: PTT peserta terdengar pada monitor engineer dengan total round-trip < 40 ms pada jaringan lab.
- M5: Dashboard control & monitoring.
  - Ukuran: Web dashboard menampilkan list clients, level meter, dan tombol talkback.
- M6: Tests, CI, dan licensing compliance.
  - Ukuran: Unit tests untuk encoder/decoder & network mocks; dokumen compliance tersedia.

3 — Strategi pendekatan bertahap (iterasi kecil)

- Iterasi A (Proof-of-concept):
  - Tugas kecil: implementasi minimal UDP server yang menerima dan mengirim paket dummy; client menerima.
  - Durasi estimasi: 1–2 hari.
  - Acceptance: paket test loop terdeteksi dan dilaporkan.

- Iterasi B (Codec & I/O):
  - Tugas kecil: integrasi Opus encoder/decoder untuk frame 2.5 ms.
  - Durasi estimasi: 2–4 hari.
  - Acceptance: audio sederhana dikirim dan diputar di client tanpa crash.

- Iterasi C (Mixing & routing):
  - Tugas kecil: MixEngine melakukan routing per-client.
  - Estimasi: 3–5 hari.

- Iterasi D (PTT & talkback):
  - Tugas kecil: implementasi PTT path, test round-trip latency.
  - Estimasi: 2–4 hari.

4 — Kriteria penerimaan (per tugas kecil)

- Setiap tugas harus menyertakan:
  - Objective yang jelas (apa yang berubah).
  - Acceptance test singkat (bagaimana membuktikan tugas selesai).
  - Skenario pengujian dengan data deterministik atau mock.
  - Dampak pada API/public surface (jika ada).

Contoh format tugas (dipakai di dokumen lain):
- ID: S1-a
- Judul: Setup UDP discovery server
- Deskripsi: Implementasi UDP broadcast listener di server yang menanggapi discovery probe dengan payload JSON minimal (id, ip, port, capabilities).
- Input: discovery probe dari client
- Output: response JSON dan entri di registry server
- Acceptance: client menemukan server pada 3/3 percobaan di jaringan lab
- Estimasi: 4 jam

5 — Metode pengukuran & metrik

- Latency: gunakan timestamping pada packet header dan ukur one-way/round-trip di lingkungan terkontrol.
- Packet loss/jitter: catat distribusi jitter dan paket hilang per 10 detik selama pengujian.
- Resource: penggunaan CPU per proses under load (2–8 klien).

6 — Dependencies

- JUCE (server & possibly forks referenced), libOpus, boost::asio (network), WebSocket++ / custom HTTP API, Flutter (web + mobile), CMake.
- Hardware: audio interface multichannel untuk integrasi akhir.

7 — Next actions (immediate, AI-agent friendly)

- Buat tugas terukur di `docs/implementation-checklist.md` untuk S1..S12 (server, client, web).
- Buat panduan AI-agent (format tugas yang bisa dieksekusi otomatis).
- Buat dokumen lisensi singkat (lihat `docs/licensing-guidelines.md`).

---

Catatan: semua langkah di atas disusun tanpa menyertakan potongan kode di dokumentasi (hanya deskripsi tugas dan acceptance criteria).