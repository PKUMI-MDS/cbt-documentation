# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 5 - Exam Engine

### Tujuan fase

Membangun inti sistem CBT yang berjalan berdasarkan sesi tes resmi: memulai ujian, menghasilkan urutan soal acak per peserta, navigasi soal, simpan jawaban, auto-save, auto-submit, dan auto-resume. Fase ini harus mengikuti aturan bahwa peserta hanya bisa mengerjakan ujian jika sudah terdaftar pada sesi tertentu, memiliki approval tes yang valid, dan durasi pengerjaan sesi adalah tetap 120 menit.

### Scope pekerjaan

- Start exam berdasarkan sesi tes.
- Validasi peserta terhadap sesi, approval tes, dan status assignment.
- Generate snapshot soal per user.
- Randomisasi urutan soal.
- Randomisasi urutan opsi jawaban.
- Navigasi soal next/prev.
- Mark ragu-ragu.
- Auto-save jawaban.
- Submit ujian manual dan otomatis.
- Resume session aktif.
- Auto-submit saat durasi 120 menit habis.
- Kontrol visibilitas hasil berdasarkan setting sesi.

### Deliverables

- API start exam.
- API get session detail.
- API get question by display number.
- API save answer.
- API mark doubtful.
- API heartbeat/resume.
- API submit exam.
- API finish result payload sesuai setting `show_result_to_user` pada sesi.

### Task breakdown

#### 1. Start exam session

- Saat peserta klik mulai ujian, backend harus cek:
  - akun aktif
  - peserta terdaftar pada sesi tes yang valid
  - sesi sedang aktif sesuai tanggal dan waktu
  - approval tes tersedia dan belum pernah dipakai
  - belum ada attempt submitted untuk approval yang sama
  - tidak ada konflik attempt aktif lain
- Backend kemudian membuat `exam_attempt`.
- `exam_attempt` harus terhubung ke:
  - `user_id`
  - `exam_session_id`
  - `test_approval_id`
- Saat attempt berhasil dibuat, approval tes ditandai sebagai `used` atau `locked_for_attempt` agar tidak dipakai ulang.

#### 2. Validasi sesi tes

- Peserta hanya boleh memulai ujian pada sesi yang:
  - sudah publish
  - tanggal tes valid
  - jam mulai sudah tercapai
  - jam selesai belum terlewati
- Sistem harus menolak start jika user mencoba masuk ke sesi lain yang bukan assignment-nya.
- Sistem harus menolak start jika kuota sesi sudah penuh dan user belum terdaftar resmi.
- Satu approval tes hanya bisa dipakai untuk satu sesi tes.

#### 3. Snapshot randomisasi

- Backend mengambil soal dari package yang dipakai oleh sesi aktif.
- Soal diacak sekali saat attempt dibuat.
- Opsi jawaban juga diacak bila setting aktif.
- Snapshot disimpan ke tabel attempt question agar stabil sampai ujian selesai.
- Randomisasi tidak boleh berubah saat refresh, reconnect, atau resume.

#### 4. Navigasi dan jawaban

- Peserta dapat membuka soal berdasarkan nomor tampil.
- Jawaban disimpan incremental, tidak menunggu submit akhir.
- Ada status `ragu-ragu` per soal.
- API mengembalikan progress dan daftar nomor yang sudah dijawab.
- Semua jawaban terikat ke `exam_attempt`, bukan langsung ke bank soal.

#### 5. Timer dan timeout

- Setiap sesi memiliki durasi tetap 120 menit.
- Saat attempt dimulai, backend menghitung:
  - `started_at`
  - `ends_at = started_at + 120 menit`
- Frontend hanya menampilkan countdown.
- Backend tetap menjadi sumber waktu utama.
- Jika waktu habis, attempt auto-submit.
- User tidak boleh mendapat durasi baru hanya karena refresh atau login ulang.

#### 6. Auto-save dan heartbeat

- Frontend mengirim heartbeat berkala.
- Backend update `last_activity_at`.
- Jawaban tersimpan per perubahan atau interval tertentu.
- Heartbeat juga dipakai untuk mendukung resume dan mendeteksi sesi yang masih aktif.

#### 7. Resume attempt

- Jika browser tertutup atau koneksi putus, peserta login lagi.
- Backend mengembalikan attempt aktif, nomor soal terakhir, sisa waktu, dan progress jawaban.
- Resume hanya boleh dilakukan selama waktu sesi masih tersisa.
- Resume harus memanggil snapshot yang sama dengan saat attempt pertama dibuat.

#### 8. Submit ujian

- Peserta dapat submit manual sebelum waktu habis.
- Sistem juga bisa auto-submit saat timeout atau rule pelanggaran tertentu tercapai.
- Setelah submit:
  - attempt berstatus `submitted`
  - waktu submit dicatat
  - jawaban dikunci
  - scoring bisa dijalankan
- Approval tes dianggap habis terpakai setelah attempt submitted.

#### 9. Kontrol hasil per sesi

- Setelah submit, sistem melihat setting sesi `show_result_to_user`.
- Jika `true`, user dapat menerima hasil dan skor sesuai rule publikasi sesi.
- Jika `false`, user hanya menerima status bahwa ujian selesai.
- Admin tetap dapat melihat hasil semua peserta untuk monitoring internal.

#### 10. Dukungan retake

- Jika user ingin mengikuti tes lagi, sistem tidak boleh membuka ulang approval yang sudah terpakai.
- User harus upload bukti pembayaran baru.
- Payment proof baru harus di-approve admin.
- Approval tes baru hanya bisa dipakai pada sesi yang berbeda dari sesi sebelumnya.

### Aturan bisnis

- Randomisasi harus dilakukan saat create attempt, bukan saat load question, agar urutan tidak berubah-ubah.
- Attempt hanya boleh satu untuk satu user pada satu sesi aktif.
- Satu approval tes hanya berlaku untuk satu kali tes.
- Approval yang sudah dipakai tidak boleh dipakai ulang untuk sesi lain.
- User hanya dapat mengikuti tes lagi jika mengirim payment proof baru dan memperoleh approval baru.
- Perubahan bank soal setelah attempt dibuat tidak memengaruhi attempt yang sedang berjalan.
- Hasil dapat dilihat atau tidak harus mengikuti setting sesi, bukan setting global.

### Acceptance criteria

- User A dan User B mendapat urutan soal berbeda pada sesi yang sama.
- Attempt yang direfresh tetap menampilkan urutan yang sama untuk user yang sama.
- User tidak bisa memulai ujian jika tidak terdaftar pada sesi tes.
- User tidak bisa memulai ujian jika approval tes sudah pernah dipakai.
- Jawaban tersimpan tanpa menunggu submit akhir.
- Attempt bisa resume dari progress terakhir selama waktu masih ada.
- Durasi attempt selalu 120 menit.
- Hasil tampil atau disembunyikan sesuai setting per sesi.

### Risiko teknis

- Randomisasi tanpa snapshot menyebabkan soal berubah saat refresh.
- Mengandalkan timer frontend saja akan rawan manipulasi.
- Jika relasi attempt ke approval tes tidak disimpan dengan benar, satu approval bisa salah dipakai lebih dari sekali.
- Jika setting tampil hasil tidak diletakkan di level sesi, hasil bisa bocor ke peserta yang seharusnya belum boleh melihat nilai.

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

### Snapshot adalah kunci randomisasi

Agar nomor soal setiap peserta benar-benar berbeda namun tetap konsisten saat refresh atau resume, randomisasi harus dilakukan sekali saat `exam_attempt` dibuat lalu disimpan ke tabel snapshot attempt-question. Pendekatan ini paling aman untuk model CBT dengan auto-resume.

### Attempt harus terkait ke sesi dan approval tes

Exam engine tidak boleh hanya tahu siapa user-nya. Engine juga harus tahu sesi mana yang diikuti dan approval tes mana yang dipakai agar rule satu approval untuk satu tes bisa dipastikan di backend.

### Backend adalah sumber waktu utama

Timer harus dihitung dari `started_at` dan `ends_at` di backend. Frontend hanya menampilkan countdown agar manipulasi waktu di browser tidak memengaruhi hasil ujian.

### Visibilitas hasil harus mengikuti sesi

Keputusan apakah nilai tampil atau tidak harus dibaca dari pengaturan sesi, karena tiap sesi dapat memiliki aturan publikasi hasil yang berbeda.

### Retake harus melalui approval baru

Jika user ingin tes lagi, sistem harus memaksa alur pembayaran ulang, approval baru, dan assignment ke sesi yang berbeda agar audit trail tetap rapi dan adil.