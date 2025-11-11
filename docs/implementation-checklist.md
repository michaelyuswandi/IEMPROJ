# Implementation Checklist — Small measurable tasks

Format: tiap baris adalah tugas singkat yang dapat dieksekusi oleh AI agent. ID mengikuti format area-xxx.

Server (area S)
- S-Server-001: UDP discovery listener
  - Est: 4h | Priority: High
  - Acceptance: client menemukan server dalam 3/3 percobaan di LAN.
- S-Server-002: Basic UDP sender (test packets)
  - Est: 2h | Priority: High
  - Acceptance: server dapat mengirim paket test ke client dummy.
- S-Server-003: Integrasi Opus encoder skeleton
  - Est: 1 day | Priority: High
  - Acceptance: encoder dapat menerima PCM buffer dan menghasilkan frames (verify frame headers logged).
- S-Server-004: MixEngine minimal (route channels -> stereo)
  - Est: 2 days | Priority: High
  - Acceptance: mix untuk 2 klien berbeda tersimpan ke buffer per-klien.
- S-Server-005: UDP per-client streaming with headers (timestamp)
  - Est: 1 day | Priority: High
  - Acceptance: client menerima paket berisi timestamp dan frame index, server logs stream start/stop.
- S-Server-006: TalkbackRouter receive path
  - Est: 1 day | Priority: Medium
  - Acceptance: server menerima talkback packets dari client dan menyalurkannya ke monitoring output.

Client (area C)
- C-Client-001: Discovery probe sender
  - Est: 2h | Priority: High
  - Acceptance: probe diterima dan server di-list.
- C-Client-002: UDP receiver & buffer plumbing
  - Est: 1 day | Priority: High
  - Acceptance: client menerima paket test dan menulis ke circular buffer.
- C-Client-003: Opus decoder integration skeleton
  - Est: 1 day | Priority: High
  - Acceptance: decoder dapat mengubah frames menjadi PCM buffer (logged).
- C-Client-004: PTT capture path (mono) and send
  - Est: 1 day | Priority: Medium
  - Acceptance: pressing PTT menghasilkan outgoing packets ke server.

Web Dashboard (area W)
- W-Web-001: WebSocket server stub
  - Est: 4h | Priority: High
  - Acceptance: dashboard dapat terhubung dan menerima heartbeat events.
- W-Web-002: `/api/status` endpoint
  - Est: 4h | Priority: High
  - Acceptance: returns JSON with server uptime and connected clients count.

Shared / Tests / CI (area T)
- T-Test-001: Unit test harness for Opus encode/decode (mock input)
  - Est: 1 day | Priority: High
  - Acceptance: round-trip test passes for 100 frames.
- T-Test-002: Network integration smoke test (local)
  - Est: 1 day | Priority: High
  - Acceptance: discovery -> stream -> playback pipeline validated in lab.
- T-CI-001: Add basic CI job that runs unit tests
  - Est: 4h | Priority: Medium
  - Acceptance: CI runs and reports test results on PR.

Documentation & Licensing
- D-Lic-001: Document source provenance for any code copied from SonoBus
  - Est: 2h | Priority: High
  - Acceptance: `docs/licensing-guidelines.md` diisi dan referensi dicatat untuk setiap file.

Notes:
- Semua tugas harus ditulis sebagai PR tunggal per tugas dengan ID tugas pada pesan commit.
- Jika hardware audio tidak tersedia, buat mock implementations dan test stubs agar tugas dapat diselesaikan di CI.
- Prioritas dapat disesuaikan sesuai kebutuhan tim.