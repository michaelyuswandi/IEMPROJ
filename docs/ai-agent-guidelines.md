# AI Agent Guidelines — Task format & conventions

Tujuan: Menyediakan format tugas yang konsisten, terukur, dan dapat dieksekusi oleh AI agent untuk mengimplementasikan fungsionalitas di proyek ini.

Prinsip umum:
- Tugas harus kecil dan deterministik. Lebih baik 15–120 menit pengerjaan per tugas bila memungkinkan.
- Sertakan acceptance criteria yang dapat diuji secara otomatis atau dengan langkah pengujian singkat.
- Jangan sertakan kode di dokumen ini; berikan instruksi, inputs/outputs, dan pointers ke file/area kode.

Template tugas (yang harus dipatuhi oleh setiap tugas baru):

- ID: unik, format prefiks area + urutan (mis. S-Server-001, C-Client-002, W-Web-003)
- Judul: ringkas dan jelas
- Prioritas: High / Medium / Low
- Estimasi waktu: dalam jam atau menit
- Deskripsi: apa yang harus diubah/ditambahkan
- Inputs: file, config, atau data yang dibutuhkan
- Outputs: file yang akan dihasilkan atau modified behavior
- Acceptance criteria: langkah singkat untuk verifikasi (lihat contoh di bawah)
- Dependencies: tugas lain atau resource eksternal
- Rollback plan: langkah untuk membatalkan jika memperkenalkan bug (mis. revert commit atau feature flag)

Acceptance criteria — contoh sederhana (format yang mudah di-parse):
- GIVEN: server berjalan pada port X dan client berada pada subnet yang sama
- WHEN: client mengirim discovery probe
- THEN: server harus merespon dengan JSON yang berisi keys: id, ip, port, capabilities
- VERIFICATION: jalankan `discovery_test` (automated) atau cek log server "DISCOVERY_RESPONSE_SENT"

Konvensi komunikasi antara agent dan repo:
- Agent membuat commit per tugas selesai, sertakan ID tugas di pesan commit.
- Jika tugas membutuhkan perubahan linting/format, pastikan mengikut konfigurasi repo (jika ada).
- Agent harus membuat atau memperbarui file doc/notes ketika mengambil code atau konfigurasi dari sumber eksternal (sumber, URL, tanggal).

Error handling & retries:
- Untuk operasi jaringan, pakai retry dengan backoff (3 percobaan, eksponensial) dan log kejadian.
- Jika integrasi native (audio), agent harus membuat test stub/mocks jika hardware tidak tersedia.

Reporting progress:
- Setelah menyelesaikan tugas, agent menulis update singkat di ISSUE/TASK tracker atau file `docs/agent-run-log.md` (ID tugas, start time, end time, hasil, catatan)

Privasi & keamanan:
- Jangan mengunggah secrets atau credential ke repo.
- Jika tugas memerlukan koneksi eksternal, agent harus meminta token via secure channel dan menyimpan tidak di repo.

Catatan akhir: file tugas yang dihasilkan oleh agent harus mematuhi template di atas sehingga pekerjaan dapat diotomasi atau di-review manusia dengan cepat.