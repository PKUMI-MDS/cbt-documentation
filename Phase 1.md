# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi user, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, manajemen sesi tes, randomisasi soal per user, manajemen bank soal, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 1 - Foundation Project

### Tujuan fase

Membangun fondasi proyek Laravel agar semua modul berikutnya dapat dikembangkan secara konsisten, aman, dan cepat. Karena sistem akan menangani autentikasi, approval pembayaran, approval tes, assignment sesi, exam engine, monitoring hasil, dan aturan sesi ujian, maka struktur awal harus sudah siap untuk API, role management, media storage, logging, serta standar respons yang konsisten.

### Scope pekerjaan

- Inisialisasi project Laravel dan environment `.env`.
- Setup koneksi MySQL.
- Setup folder struktur API (`Controllers/Api`, `Requests`, `Services`, `Repositories`, `Enums`).
- Setup Laravel Sanctum untuk autentikasi API.
- Setup role dan permission admin/user.
- Setup media storage untuk upload image, audio, dan bukti pembayaran.
- Setup activity log untuk approval, perubahan soal, assignment sesi, dan aktivitas penting admin.
- Menyiapkan fondasi struktur database untuk:
  - payment proof
  - test approval
  - exam session
  - exam session registration
  - exam attempt
- Menyiapkan fondasi setting agar hasil dapat ditampilkan atau disembunyikan per sesi.
- Menyiapkan fondasi aturan bahwa durasi sesi adalah 120 menit.

### Deliverables

- Project Laravel siap jalan di local/dev server.
- Base migration users dan roles.
- Base migration payment proofs, test approvals, exam sessions, dan session registrations.
- Authentication guard untuk API.
- Seeder role: `admin`, `participant`.
- Standard JSON response helper.
- Global exception handler untuk error API.
- Dokumentasi struktur folder dan coding convention.
- Enum dasar untuk status akun, status approval, status sesi, dan status attempt.

### Task breakdown

#### 1. Inisialisasi aplikasi

- Install Laravel.
- Konfigurasi `.env` untuk database, app key, queue, mail fallback, dan filesystem.
- Setup timezone dan locale.
- Aktifkan UTF-8 penuh untuk kebutuhan teks Arab.

#### 2. Setup autentikasi API

- Install dan konfigurasi Laravel Sanctum.
- Siapkan endpoint login/logout dasar.
- Buat middleware auth untuk API user dan admin.
- Siapkan struktur token/session tracking.
- Pastikan hanya akun `active` yang bisa login.

#### 3. Setup role dan permission

- Integrasi package permission.
- Definisikan permission dasar seperti:
  - manage-users
  - manage-question-banks
  - manage-exams
  - manage-exam-sessions
  - assign-session-participants
  - monitor-results
  - review-payments
- Mapping permission per role.

#### 4. Setup storage media

- Siapkan disk storage untuk:
  - payment proofs
  - question images
  - question audio
- Definisikan naming file dan struktur direktori upload.
- Pastikan validasi MIME type dan file size bisa disiapkan sejak awal.

#### 5. Setup activity dan audit log

- Log aktivitas penting:
  - approve user
  - approve payment proof
  - reject payment proof
  - generate test approval
  - create/update exam session
  - assign participant ke sesi
  - auto generate participant ke sesi
  - create/update/delete question
  - publish session
  - force submit exam
- Siapkan format metadata log JSON.

#### 6. Setup struktur domain session-based exam

- Siapkan enum dan konstanta dasar untuk:
  - `account_status`
  - `payment_proof_status`
  - `test_approval_status`
  - `exam_session_status`
  - `exam_session_registration_status`
  - `exam_attempt_status`
- Siapkan struktur model awal:
  - `User`
  - `PaymentProof`
  - `TestApproval`
  - `ExamSession`
  - `ExamSessionRegistration`
  - `ExamAttempt`
- Pastikan fondasi ini mendukung:
  - satu approval tes untuk satu kali tes
  - assignment user ke sesi tertentu
  - retake dengan payment proof baru
  - visibilitas hasil per sesi
  - kuota peserta per sesi

#### 7. Setup standar response API

- Gunakan helper response formatter secara konsisten.
- Format response sukses:
  - `code`
  - `status`
  - `message`
  - `data`
- Format error juga menggunakan struktur yang sama.
- Siapkan base trait/helper bila diperlukan agar controller konsisten.

### Acceptance criteria

- Project dapat dijalankan tanpa error di environment dev.
- Login API dasar berjalan.
- Role admin dan user sudah bisa dibedakan.
- Storage upload siap dipakai oleh modul lain.
- Struktur dasar database siap untuk payment proof, test approval, sesi tes, dan assignment peserta.
- Semua response API mengikuti format yang konsisten.
- Fondasi sistem sudah mendukung pola sesi tes bertanggal dan berjam.
- Fondasi sistem sudah mendukung batas peserta dan pengaturan tampil/tidaknya hasil per sesi.
- Fondasi sistem sudah mendukung approval tes satu kali per pembayaran.

### Risiko teknis

- Struktur proyek yang terlalu sederhana akan menyulitkan saat exam engine mulai kompleks.
- Jika support UTF-8/Arab tidak diuji dari awal, data soal bisa rusak saat input atau render.
- Jika entitas approval tes dan sesi tes tidak dipisah dari awal, implementasi retake dan kuota sesi akan sulit dirapikan.
- Jika format response API tidak distandarkan sejak awal, integrasi frontend akan mudah kacau.

---

## Prioritas implementasi

### MVP wajib

- Registrasi user
- Upload bukti pembayaran
- Approval admin
- Approval tes satu kali per pembayaran
- Login by status
- CRUD bank soal
- Pembuatan sesi tes
- Penentuan tanggal dan waktu sesi
- Penentuan batas peserta per sesi
- Auto generate peserta approved ke sesi
- Start exam
- Snapshot randomisasi soal
- Simpan jawaban
- Timer 120 menit
- Submit ujian
- Hasil dasar
- Pengaturan tampil/tidaknya nilai per sesi

### Pasca-MVP

- Monitoring pelanggaran yang lebih detail
- Rule auto-submit berbasis threshold pelanggaran
- Dashboard statistik yang lebih kaya
- Notifikasi email
- Reset password flow yang lebih lengkap

## Catatan arsitektur penting

### Snapshot adalah kunci randomisasi

Agar nomor soal setiap user benar-benar berbeda namun tetap konsisten saat refresh atau resume, randomisasi harus dilakukan sekali saat `exam_attempt` dibuat lalu disimpan ke tabel snapshot attempt-question. Pendekatan ini paling aman untuk model CBT dengan auto-resume.

### Backend adalah sumber waktu utama

Waktu ujian harus dihitung dari `started_at` dan `ends_at` di backend. Frontend hanya menampilkan countdown agar manipulasi waktu di browser tidak memengaruhi hasil ujian.

### Approval akun dan approval tes harus dipisah

Akun aktif hanya berarti user boleh login. Hak ikut tes harus tetap dihitung terpisah melalui `test_approval` agar satu pembayaran hanya berlaku untuk satu kali tes dan retake bisa dikelola dengan payment proof baru.

### Sesi tes adalah master pelaksanaan ujian

Sistem harus membedakan antara master sesi tes dan runtime ujian user. Dengan begitu, admin bisa mengatur tanggal, waktu, kuota, dan visibilitas hasil per sesi tanpa mencampur logika runtime user.

### Anti-cheat harus realistis

Di browser, beberapa shortcut dan perlindungan media tidak bisa dibuat absolut. Implementasi yang sehat adalah kombinasi pembatasan UI, logging, threshold, dan audit admin, bukan janji blok total 100 persen.