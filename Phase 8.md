# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 8 - Testing, UAT, Deployment, and Handover

### Tujuan fase

Memastikan seluruh modul CBT siap digunakan secara operasional melalui testing, UAT, deployment, dan handover. Fase ini harus memverifikasi bahwa semua business rule penting berjalan benar, terutama alur pembayaran, approval tes, assignment ke sesi, durasi ujian 120 menit, visibilitas hasil per sesi, dan retake dengan payment proof baru.

### Scope pekerjaan

- Internal testing backend API.
- Integration testing antar modul.
- UAT dengan skenario bisnis utama.
- Bug fixing final.
- Deployment ke environment target.
- Handover dokumentasi teknis dan operasional.
- Smoke test pasca deployment.
- Validasi akhir format response API.

### Deliverables

- Checklist UAT.
- Daftar test case utama.
- Laporan bug fixing final.
- API collection final.
- Environment deployment siap pakai.
- Dokumen handover admin dan teknis.
- Checklist operasional sesi tes.

### Task breakdown

#### 1. Internal testing

- Uji endpoint utama:
  - register
  - upload payment proof
  - approve/reject payment proof
  - generate approval tes
  - create exam package
  - create exam session
  - assign participant ke sesi
  - auto generate participant
  - start exam
  - save answer
  - submit exam
  - get result
- Pastikan semua endpoint mengembalikan response yang konsisten.

#### 2. UAT skenario bisnis utama

- Skenario 1:
  - user register
  - user upload bukti pembayaran
  - admin approve
  - sistem membuat approval tes
  - admin memasukkan user ke sesi
  - user ikut tes
- Skenario 2:
  - admin membuat sesi dengan tanggal dan waktu
  - admin set batas peserta
  - admin set show result true atau false
  - admin jalankan auto generate peserta approved ke sesi
- Skenario 3:
  - user submit ujian
  - sistem hitung hasil
  - user hanya melihat nilai jika sesi mengizinkan
- Skenario 4:
  - user ingin tes lagi
  - user upload bukti pembayaran baru
  - admin approve lagi
  - sistem membuat approval tes baru
  - user ditempatkan ke sesi berbeda
  - user ikut tes kedua tanpa merusak hasil tes pertama

#### 3. Validasi kuota dan assignment sesi

- Uji bahwa satu sesi hanya menerima peserta sampai `max_participants`.
- Uji assignment manual saat kuota hampir penuh.
- Uji auto generate peserta saat kuota penuh.
- Pastikan sistem tidak memasukkan user melebihi kapasitas sesi.
- Pastikan hanya user dengan approval tes tersedia yang dapat dimasukkan ke sesi.

#### 4. Validasi durasi 120 menit

- Uji bahwa setiap attempt mendapatkan durasi tepat 120 menit.
- Uji countdown, timeout, dan auto-submit.
- Uji resume agar tidak menambah durasi baru.
- Uji refresh browser agar waktu tetap mengikuti backend.

#### 5. Validasi hasil per sesi

- Uji sesi dengan `show_result_to_user = true`.
- Uji sesi dengan `show_result_to_user = false`.
- Pastikan admin tetap dapat melihat seluruh hasil.
- Pastikan user hanya melihat hak akses hasil sesuai sesi yang diikutinya.

#### 6. Validasi retake

- Uji bahwa satu approval tes hanya dapat dipakai satu kali.
- Uji bahwa approval yang sudah dipakai tidak bisa dipakai ulang.
- Uji bahwa user harus upload payment proof baru untuk tes berikutnya.
- Uji bahwa approval baru hanya dipakai pada sesi yang berbeda.
- Pastikan histori hasil lama dan baru tetap terpisah.

#### 7. Deployment

- Siapkan environment production atau staging.
- Konfigurasi:
  - database
  - queue
  - storage
  - cache
  - mail bila digunakan
- Jalankan migration dan seed awal jika diperlukan.
- Lakukan smoke test pasca deploy.

#### 8. Handover

- Serahkan dokumentasi:
  - struktur modul
  - endpoint API
  - aturan approval tes
  - aturan sesi tes
  - aturan tampil hasil
  - prosedur operasional admin
- Serahkan akun admin awal bila diperlukan.
- Serahkan panduan membuat sesi dan menjalankan auto generate peserta.

### Aturan bisnis

- Semua business rule utama harus lulus UAT sebelum dinyatakan selesai.
- Satu approval tes hanya boleh dipakai untuk satu kali tes.
- Retake harus melalui payment proof baru dan approval baru.
- Approval baru harus dipakai pada sesi yang berbeda.
- Nilai tampil atau tidak harus mengikuti setting sesi.
- Durasi ujian per sesi selalu 120 menit.
- Auto generate peserta harus menghormati kuota sesi dan status approval.

### Acceptance criteria

- Semua endpoint utama lolos pengujian dasar dan integrasi.
- Admin bisa membuat sesi dengan tanggal dan waktu.
- Admin bisa mengatur batas peserta dan visibilitas hasil per sesi.
- Auto generate peserta approved ke sesi berjalan benar.
- User hanya bisa tes jika punya approval valid.
- Satu approval tes hanya dipakai satu kali.
- User bisa retake hanya setelah upload payment proof baru dan approve ulang.
- Attempt selalu berdurasi 120 menit.
- Hasil tampil sesuai aturan sesi.

### Risiko teknis

- Jika UAT tidak menguji skenario retake, bug pada approval ganda bisa lolos ke production.
- Jika pengujian kuota sesi tidak dilakukan dengan data realistis, auto generate bisa melewati batas peserta.
- Jika testing hasil per sesi diabaikan, nilai bisa salah tampil ke user yang tidak berhak.

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
- Auto generate peserta approved ke sesi
- Start exam
- Snapshot randomisasi soal
- Simpan jawaban
- Timer 120 menit
- Submit ujian
- Hasil dasar
- Pengaturan tampil/tidaknya hasil per sesi

### Pasca-MVP

- UAT lanjutan dengan volume lebih besar
- Monitoring aplikasi pasca go-live
- Optimasi performa export dan reporting
- Pelatihan admin tambahan
- Improvement berdasarkan temuan operasional awal

## Catatan arsitektur penting

### UAT harus mengikuti business rule nyata

Pengujian tidak cukup hanya memeriksa endpoint hidup. Yang harus dibuktikan adalah alur nyata dari payment proof sampai user menyelesaikan tes dan melihat atau tidak melihat hasil sesuai sesi.

### Handover harus fokus ke operasional sesi

Karena sistem ini berbasis sesi, dokumen handover harus menjelaskan cara membuat sesi, menetapkan kuota, mengatur visibilitas hasil, dan menjalankan auto generate peserta approved.

### Retake adalah skenario wajib uji

Salah satu risiko terbesar ada pada retake. Karena itu, skenario payment proof baru, approval baru, sesi berbeda, dan hasil terpisah harus selalu masuk checklist UAT.

### Konsistensi response API harus dijaga sampai akhir

Semua endpoint final sebaiknya tetap menggunakan formatter response yang sama agar integrasi frontend dan debugging lebih rapi.