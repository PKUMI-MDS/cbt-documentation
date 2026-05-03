# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 2 - Registration, Payment Approval, and Test Approval

### Tujuan fase

Membangun alur onboarding peserta: register, upload bukti pembayaran, menunggu verifikasi admin, lalu mendapatkan status akun aktif dan hak tes bila pembayaran disetujui. Fase ini juga harus mendukung upload bukti pembayaran ulang agar user dapat mengikuti tes lagi pada sesi yang berbeda.

### Scope pekerjaan

- Form/API registrasi peserta.
- Penyimpanan data profil peserta.
- Upload bukti pembayaran.
- Review bukti pembayaran oleh admin.
- Aktivasi akun oleh admin.
- Pembuatan approval tes dari pembayaran yang disetujui.
- Penolakan bukti pembayaran dengan alasan revisi.
- Pembatasan login untuk akun nonaktif.
- Riwayat bukti pembayaran dan approval tes per user.
- Dukungan retake dengan payment proof baru.

### Deliverables

- Endpoint registrasi.
- Endpoint upload bukti pembayaran.
- Endpoint admin review payment proof.
- Endpoint admin approve/reject payment proof.
- Endpoint list riwayat bukti pembayaran user.
- Endpoint list approval tes user.
- Middleware blok login untuk akun `pending`, `rejected`, atau `suspended`.
- Mekanisme generate `test_approval` otomatis saat payment proof approved.

### Task breakdown

#### 1. Registrasi peserta

- Buat endpoint register.
- Simpan user dengan status default `pending_verification`.
- Simpan profile pendukung seperti nomor identitas, institusi, nomor HP, alamat jika dibutuhkan.
- Kirim response status bahwa akun menunggu verifikasi.

#### 2. Upload bukti pembayaran

- Buat endpoint upload file bukti pembayaran.
- Validasi format file: jpg, jpeg, png, pdf.
- Simpan nominal dan tanggal pembayaran bila diperlukan.
- Simpan status bukti pembayaran: `pending_review`.
- Simpan semua riwayat payment proof tanpa menghapus data lama.

#### 3. Review dan approval admin

- Buat dashboard/API admin untuk melihat payment proof pending.
- Admin dapat membuka detail bukti pembayaran.
- Admin dapat approve atau reject.
- Saat approve:
  - status akun menjadi `active` jika sebelumnya belum aktif
  - field `approved_at` dan `approved_by` terisi
  - sistem membuat satu `test_approval`
- Saat reject:
  - status payment proof menjadi `rejected`
  - alasan penolakan disimpan
  - user diminta upload ulang bila masih ingin ikut tes

#### 4. Pembuatan approval tes

- Setiap payment proof yang di-approve menghasilkan satu approval tes.
- Approval tes memiliki quota satu kali tes.
- Approval tes berstatus `available` sampai dipakai ke sesi tes.
- Approval tes tidak boleh digunakan lebih dari satu kali.
- Approval tes dicatat dalam audit log.

#### 5. Restriksi login

- Login hanya diizinkan untuk akun `active`.
- Jika belum aktif, kembalikan error message yang jelas.
- Jika rejected, user diarahkan upload ulang bukti pembayaran.
- Akun aktif tidak otomatis berarti user bisa tes tanpa approval tes yang tersedia.

#### 6. Dukungan retake

- User yang ingin tes lagi dapat upload bukti pembayaran baru.
- Bukti pembayaran baru harus melewati proses review admin lagi.
- Jika approved, sistem membuat approval tes baru.
- Approval tes baru nantinya harus digunakan pada sesi yang berbeda dari sesi sebelumnya.

### Aturan bisnis

- Satu user dapat memiliki riwayat beberapa upload bukti pembayaran.
- Bukti pembayaran lama tidak dihapus, hanya diarsipkan untuk audit.
- Satu payment proof approved menghasilkan satu approval tes.
- Satu approval tes hanya berlaku untuk satu kali tes.
- Approval akun dan approval bukti pembayaran harus tercatat pada activity log.
- User dapat login jika akun active, tetapi hanya dapat mengikuti ujian jika memiliki approval tes yang masih tersedia dan sudah ditempatkan ke sesi.
- Retake hanya bisa dilakukan melalui payment proof baru dan approval baru.

### Acceptance criteria

- User bisa register dan upload bukti pembayaran.
- Admin bisa melihat daftar payment proof pending.
- Admin bisa approve/reject payment proof.
- User belum bisa login sebelum status akun active.
- Payment proof yang approved menghasilkan satu approval tes.
- Satu approval tes hanya berlaku untuk satu kali tes.
- User dapat upload bukti pembayaran lagi untuk tes berikutnya.
- Riwayat payment proof dan approval tes tersimpan dengan baik.

### Risiko teknis

- Jika status approval hanya disimpan di frontend, login bisa bocor; validasi status harus selalu dilakukan di backend.
- Upload file tanpa validasi ketat berisiko pada storage abuse dan file berbahaya.
- Jika approval tes tidak dipisah dari approval akun, sistem akan sulit membedakan hak login dan hak ikut tes.
- Jika payment proof lama ditimpa, audit trail untuk retake akan hilang.

---

## Prioritas implementasi

### MVP wajib

- Registrasi peserta
- Upload bukti pembayaran
- Review dan approval admin
- Login by status
- Generate approval tes satu kali per pembayaran
- Riwayat payment proof
- Riwayat approval tes
- CRUD bank soal
- Pembuatan sesi tes
- Assignment peserta ke sesi
- Start exam
- Snapshot randomisasi soal
- Simpan jawaban
- Timer 120 menit
- Submit ujian
- Hasil dasar

### Pasca-MVP

- Monitoring pelanggaran yang lebih detail
- Rule auto-submit berbasis threshold pelanggaran
- Dashboard statistik yang lebih kaya
- Notifikasi email
- Reset password flow yang lebih lengkap

## Catatan arsitektur penting

### Approval tes bukan sekadar approval akun

Akun aktif hanya berarti user bisa login. Hak tes harus tetap dipisah agar setiap pembayaran yang disetujui hanya menghasilkan satu kesempatan tes.

### Retake harus memakai payment proof baru

Jika user ingin tes lagi, sistem tidak boleh memakai approval lama. User harus upload bukti pembayaran baru, mendapat approval baru, lalu baru bisa ditempatkan ke sesi lain.

### Assignment ke sesi adalah langkah terpisah

Setelah payment proof approved dan approval tes terbentuk, user belum otomatis mulai ujian. User harus lebih dulu ditempatkan ke sesi yang memiliki tanggal, waktu, kuota, dan aturan tampil hasil yang jelas.

### Backend tetap menjadi sumber waktu utama

Walaupun fase ini belum membangun engine ujian penuh, desain approval dan assignment tetap harus mempertimbangkan bahwa durasi sesi di backend adalah 120 menit dan bukan hasil manipulasi frontend.

### Anti-cheat harus realistis

Sebagian kontrol browser tidak bisa dipaksa 100 persen. Karena itu, sejak fase approval dan assignment, semua aktivitas penting perlu dicatat untuk audit dan troubleshooting.