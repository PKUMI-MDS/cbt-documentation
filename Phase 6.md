# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 6 - Security & Anti-Cheat

### Tujuan fase

Menambahkan kontrol keamanan dan pencatatan pelanggaran selama ujian berlangsung, dengan pendekatan yang sesuai untuk model ujian berbasis sesi. Fase ini harus memastikan bahwa hanya user yang terdaftar pada sesi tes, memiliki approval tes valid, dan sedang menjalankan attempt resmi yang dapat melanjutkan ujian, sambil tetap mencatat pelanggaran seperti fullscreen exit, tab switch, copy/paste attempt, dan multi-login.

### Scope pekerjaan

- Single active login.
- Single active attempt per user.
- Validasi sesi tes aktif.
- Validasi approval tes yang dipakai pada attempt.
- Logging tab switch.
- Logging fullscreen exit.
- Logging copy/paste/right click attempt.
- Automatic session lock.
- Audio play limit.
- Warning threshold dan auto action.
- Audit log keamanan per sesi dan per attempt.

### Deliverables

- Middleware atau session guard multi-login.
- Attempt guard untuk satu attempt aktif.
- Violation log API.
- Rule engine sederhana untuk pelanggaran.
- Audio play tracking.
- Monitoring daftar pelanggaran oleh admin.
- Validasi keamanan berbasis `exam_session_id` dan `exam_attempt_id`.

### Task breakdown

#### 1. Single active login

- Simpan token login aktif per user.
- Saat login baru terjadi, sistem:
  - menolak login kedua, atau
  - memutus session lama
- Untuk mode ujian aktif, rekomendasi utama adalah menolak login baru agar attempt awal tetap aman.
- Kebijakan ini harus konsisten dengan assignment user ke sesi tes yang sedang berjalan.

#### 2. Single active attempt

- Satu user hanya boleh memiliki satu attempt aktif pada satu waktu.
- Jika user sudah sedang mengerjakan attempt pada sesi aktif, sistem tidak boleh membuat attempt baru.
- Attempt hanya boleh dibuat bila:
  - user aktif
  - user terdaftar di sesi
  - approval tes valid
  - approval tersebut belum pernah dipakai
- Ini penting untuk menjamin satu approval tes hanya dipakai untuk satu kali tes.

#### 3. Validasi sesi dan approval

- Setiap request penting saat ujian harus memverifikasi:
  - `exam_session_id`
  - `exam_attempt_id`
  - `user_id`
  - `test_approval_id`
- Sistem harus menolak akses jika approval tes tidak cocok dengan attempt.
- Sistem harus menolak akses jika sesi yang diakses bukan sesi assignment peserta.
- Sistem harus menolak attempt lanjutan bila approval sudah berstatus used dan attempt sudah selesai.

#### 4. Fullscreen tracking

- Frontend mengirim event saat keluar fullscreen.
- Backend mencatat jumlah pelanggaran.
- Pelanggaran dikaitkan ke attempt dan sesi.
- Jika melebihi threshold, backend dapat menandai warning atau auto-submit sesuai aturan sesi.

#### 5. Tab switch detection

- Frontend menggunakan Visibility API.
- Event `hidden` dicatat sebagai pelanggaran.
- Simpan severity, timestamp, attempt, dan sesi.
- Jika sesi mengaktifkan threshold tertentu, pelanggaran dapat memicu warning atau auto action.

#### 6. Copy/paste dan right click logging

- Frontend blok interaksi tertentu sebisa mungkin.
- Backend menerima event attempt untuk dicatat.
- Event tidak harus selalu dianggap curang, tetapi wajib tersedia untuk monitoring admin.
- Event harus tersimpan per attempt agar audit lebih jelas.

#### 7. Audio play control

- Tiap soal audio punya max play count.
- Setiap play dicatat pada level attempt-question.
- Audio tidak boleh di-seek pada player sisi klien sesuai requirement smart audio player.
- Audio play tracking harus tetap konsisten walaupun peserta refresh dan resume.

#### 8. Automatic session lock

- Jika akun yang sama mencoba akses dari device atau browser lain saat ujian berlangsung, sistem dapat:
  - menolak device kedua, atau
  - mengunci request baru
- Pendekatan yang direkomendasikan adalah mempertahankan attempt pertama dan menolak akses baru.
- Session lock harus mempertimbangkan bahwa satu approval tes hanya berlaku untuk satu attempt.

#### 9. Integrasi dengan hasil per sesi

- Walaupun modul ini berfokus pada keamanan, response akhir setelah submit tetap harus tunduk pada pengaturan sesi.
- Jika sesi tidak mengizinkan hasil tampil ke user, maka response submit tidak boleh membocorkan skor.
- Admin tetap dapat melihat hasil dan log pelanggaran untuk semua sesi.

#### 10. Integrasi dengan retake

- User yang sudah menyelesaikan satu attempt tidak boleh membuka lagi attempt lama untuk dianggap sebagai tes baru.
- Jika user ingin tes lagi, sistem harus menunggu payment proof baru.
- Payment proof baru harus di-approve admin.
- Approval tes baru lalu dipakai pada sesi yang berbeda.
- Security layer harus menganggap retake sebagai attempt baru yang terpisah penuh dari attempt sebelumnya.

### Aturan bisnis

- Beberapa shortcut OS tidak bisa diblok total di browser, jadi pendekatan realistis adalah deteksi, logging, warning, dan action threshold.
- Admin harus bisa melihat riwayat pelanggaran per sesi dan per attempt.
- Pelanggaran tidak otomatis menggugurkan ujian kecuali rule sesi atau package mengaturnya demikian.
- Satu approval tes hanya berlaku untuk satu kali tes.
- Retake hanya bisa dilakukan dengan payment proof baru, approval baru, dan sesi yang berbeda.
- Semua validasi keamanan harus tetap menghormati assignment sesi dan status approval.

### Acceptance criteria

- Satu akun tidak bisa bebas dipakai di dua device saat ujian.
- User tidak bisa membuat attempt baru jika approval tes yang sama sudah terpakai.
- Event fullscreen dan tab switch tercatat.
- Audio play count dibatasi dan tercatat.
- Admin bisa melihat log pelanggaran per sesi dan per user.
- User yang ingin tes lagi harus melalui payment proof baru dan approval baru.
- Response akhir tetap mengikuti setting tampil/tidaknya hasil pada sesi.

### Risiko teknis

- Browser tidak memberi kontrol penuh atas shortcut OS seperti Alt+Tab, jadi fitur ini tidak bisa dijanjikan 100 persen blok total.
- Audio protection di web tidak pernah sepenuhnya aman jika file sudah diterima browser; yang realistis adalah signed access, UI restriction, dan play tracking.
- Jika validasi approval tes tidak dilakukan di setiap request penting, user bisa mencoba menyalahgunakan attempt lama atau sesi yang bukan miliknya.
- Jika session lock tidak dirancang rapi, user yang reconnect bisa salah dianggap login ganda.

---

## Prioritas implementasi

### MVP wajib

- Registrasi peserta
- Upload bukti pembayaran
- Approval admin
- Approval tes satu kali per pembayaran
- Login by status
- CRUD bank soal
- Package ujian
- Pembuatan sesi tes
- Assignment peserta ke sesi
- Start exam
- Snapshot randomisasi soal
- Simpan jawaban
- Timer 120 menit
- Submit ujian
- Hasil dasar
- Pengaturan tampil/tidaknya hasil per sesi

### Pasca-MVP

- Monitoring pelanggaran yang lebih detail
- Rule auto-submit berbasis threshold pelanggaran
- Dashboard statistik yang lebih kaya
- Notifikasi email
- Reset password flow yang lebih lengkap

## Catatan arsitektur penting

### Security harus berbasis attempt, bukan sekadar login

Masalah keamanan ujian tidak cukup diselesaikan di level akun login. Semua rule penting harus dikaitkan ke `exam_attempt`, `exam_session`, dan `test_approval` agar lebih presisi.

### Satu approval harus terkunci ke satu alur tes

Begitu approval tes dipakai untuk membentuk attempt resmi, sistem harus mencegah approval yang sama dipakai lagi baik di sesi yang sama maupun sesi berbeda.

### Resume dan anti-double login harus seimbang

Sistem harus bisa membedakan antara reconnect yang sah dan login kedua yang mencurigakan. Karena itu, guard keamanan perlu membaca token, heartbeat, dan status attempt secara bersama-sama.

### Backend adalah sumber waktu utama

Walaupun modul ini fokus pada keamanan, semua rule timeout dan auto-submit tetap harus mengikuti waktu backend pada sesi berdurasi 120 menit.

### Anti-cheat harus realistis

Di browser, beberapa shortcut dan perlindungan media tidak bisa dibuat absolut. Implementasi yang sehat adalah kombinasi pembatasan UI, logging, threshold, dan audit admin, bukan janji blok total 100 persen.