# Licensing Guidelines & Sonobus (high-level, non-legal)

Tujuan: Panduan praktis untuk menangani kode atau inspirasi yang berasal dari SonoBus (atau proyek GPL lainnya) tanpa memasukkan teks lisensi lengkap di dokumentasi teknis.

Poin penting tentang SonoBus:
- SonoBus dilisensikan di bawah GNU GPLv3.
- GPLv3 adalah copyleft: menggabungkan kode GPL ke dalam produk terdistribusi biasanya mewajibkan seluruh karya yang digabung dilisensikan di bawah GPLv3 juga, serta distribusi source.

Praktik yang disarankan (ketika mengacu atau mengambil komponen dari SonoBus):

1. Hindari menyalin kode jika memungkinkan.
   - Prefer mengadaptasi ide/arsitektur dan menulis implementasi sendiri.
   - Jika hanya menggunakan parameter (mis. nilai bitrate Opus, frame size) dan dokumentasi konsep, ini tidak otomatis mengikat lisensi.

2. Jika perlu menggunakan kode secara langsung:
   - Tandai file yang diambil dengan header yang menyatakan asal (URL, commit hash, tanggal).
   - Simpan salinan LICENSE ke folder `third_party/sonobus/` dan sertakan note di `docs/` yang menyebut asal dan lisensi.
   - Pastikan bahwa distribusi binari mengikuti ketentuan GPL (sediakan source atau akses ke source sesuai persyaratan).

3. Dokumentasi internal wajib:
   - Buat file `third_party/README.md` yang mencatat semua komponen eksternal, lisensi, dan alasan penggunaannya.
   - Untuk setiap file yang diturunkan dari SonoBus, sertakan referensi dalam daftar provinsi sumber.

4. Tindakan operational sebelum merge:
   - Lakukan audit lisensi terhadap setiap file baru yang ditambahkan dari luar.
   - Jika tim ingin karya tetap berada di bawah lisensi permissive, pertimbangkan ulang integrasi kode GPL atau minta izin eksplisit dari pemilik hak cipta.

5. Template catatan provenance (gunakan di commit message atau file dokumentasi):
   - Source: https://github.com/sonosaurus/sonobus
   - Commit/tag: <isi>
   - Files copied: list of paths
   - Rationale: why copied
   - Compliance steps taken: (LICENSE included, source link added, notified stakeholders)

Catatan hukum: dokumen ini bersifat panduan teknis, bukan nasihat hukum. Untuk keputusan pasti tentang distribusi dan kepatuhan lisensi, konsultasikan penasihat legal.

---

Acceptable outputs for AI agent:
- Jika agent menemukan file yang diduga berasal dari SonoBus, agent harus membuat PR draft yang menambahkan metadata provenance dan membuka issue untuk review lisensi oleh manusia.
- Agent harus tidak melakukan publikasi binari yang menyertakan kode GPL tanpa dokumentasi source yang sesuai.

