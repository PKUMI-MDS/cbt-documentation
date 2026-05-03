# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 4 - Exam Package & Session Management

### Tujuan fase

Menyusun bank soal menjadi paket ujian yang siap dijalankan melalui sesi tes resmi. Fase ini menjawab kebutuhan pembuatan sesi tes dengan tanggal dan waktu, pengaturan nilai tampil atau tidak per sesi, batas peserta per sesi, auto generate user approved ke sesi, serta durasi pengerjaan tetap 2 jam.

### Scope pekerjaan

- CRUD exam package.
- Relasi package ke satu atau lebih bank soal.
- Pengaturan jumlah soal per kategori.
- CRUD exam session.
- Pengaturan tanggal tes dan waktu tes.
- Pengaturan durasi sesi tetap 120 menit.
- Pengaturan batas peserta per sesi.
- Pengaturan hasil dapat dilihat atau tidak per sesi.
- Manual assign peserta ke sesi.
- Auto generate user approved ke sesi.
- Validasi bahwa hanya user dengan approval tes tersedia yang bisa masuk ke sesi.
- Opsi aturan pelanggaran seperti batas tab switch dan fullscreen exit.

### Deliverables

- Modul exam package.
- Modul exam session.
- Modul session participant assignment.
- Modul settings ujian per sesi.
- Validasi ketersediaan soal dalam package.
- Preview struktur paket ujian sebelum publish.
- Auto generate participant service.
- Validasi kuota sesi.
- Validasi show result per session.

### Task breakdown

#### 1. Pembuatan package

- Admin membuat paket ujian.
- Tentukan:
  - nama paket
  - kode
  - deskripsi
  - status aktif
  - shuffle questions
  - shuffle options
  - aturan dasar pelanggaran
- Package menjadi blueprint soal, bukan runtime ujian langsung.
- Package akan dipakai oleh satu atau beberapa sesi tes.

#### 2. Mapping bank soal ke package

- Satu package bisa mengambil soal dari satu atau beberapa bank.
- Tentukan jumlah soal per section.
- Validasi bahwa bank soal memiliki stok soal cukup.
- Package tidak boleh dipublish jika komposisi soal belum valid.

#### 3. Pembuatan sesi tes

- Admin membuat sesi tes.
- Setiap sesi harus memiliki:
  - nama sesi
  - tanggal tes
  - jam mulai
  - jam selesai
  - exam package
  - durasi pengerjaan 120 menit
  - batas peserta
  - status publish/draft
  - show result to user
- Satu sesi dapat dikaitkan dengan satu package tertentu.
- Sesi harus dapat dibedakan jelas dari sesi lain meskipun package yang dipakai sama.

#### 4. Pengaturan hasil per sesi

- Admin dapat mengatur apakah nilai user dapat dilihat atau tidak untuk sesi tertentu.
- Jika `show_result_to_user = true`, user dapat melihat nilai setelah ujian selesai.
- Jika `show_result_to_user = false`, user hanya melihat status selesai.
- Admin tetap bisa melihat hasil semua peserta untuk kebutuhan monitoring dan export.

#### 5. Pengaturan kuota peserta

- Admin menentukan `max_participants` untuk setiap sesi.
- Sistem harus menghitung jumlah peserta yang sudah terdaftar di sesi.
- Sistem menolak assignment baru jika kuota sesi penuh.
- Kuota harus berlaku baik untuk assignment manual maupun auto generate.

#### 6. Manual assign peserta

- Admin dapat memilih user tertentu untuk dimasukkan ke sesi.
- User yang dapat di-assign hanya user yang:
  - akun aktif
  - memiliki payment proof approved
  - memiliki approval tes status available
- Satu approval tes hanya boleh dipakai di satu sesi.

#### 7. Auto generate peserta

- Admin dapat menjalankan fitur auto generate peserta ke sesi.
- Auto generate hanya mengambil user yang:
  - sudah upload bukti pembayaran
  - payment proof sudah di-approve admin
  - memiliki approval tes tersedia
  - belum terdaftar di sesi lain dengan approval yang sama
- Auto generate berhenti saat kuota sesi penuh.
- Sistem harus menandai assignment sebagai `manual` atau `auto`.

#### 8. Validasi retake dan sesi berbeda

- Jika user ingin tes lagi, user harus upload bukti pembayaran baru.
- Setelah payment proof baru di-approve, sistem membuat approval tes baru.
- Approval baru hanya boleh dipakai untuk sesi lain.
- Approval lama yang sudah dipakai tidak boleh dipakai ulang.

#### 9. Rules ujian per sesi

- Set max tab switch.
- Set max fullscreen exit.
- Set auto submit on timeout.
- Set audio max play bila ingin override default soal.
- Semua sesi memiliki durasi tetap 120 menit dan auto-submit saat waktu habis.

### Aturan bisnis

- Jadwal atau sesi yang overlap harus divalidasi bila menyasar peserta yang sama.
- Package tidak boleh dipublish jika jumlah soal kurang.
- Semua sesi memiliki durasi tetap 120 menit.
- Nilai tampil atau tidak ditentukan per sesi, bukan global.
- User hanya bisa mengikuti sesi jika punya approval tes yang masih tersedia.
- Satu approval tes hanya berlaku untuk satu kali tes.
- Retake hanya bisa dilakukan dengan payment proof baru dan approval baru.
- Auto generate peserta harus menghormati kuota sesi dan ketersediaan approval.

### Acceptance criteria

- Admin bisa membuat package.
- Admin bisa membuat sesi tes dengan tanggal dan waktu.
- Admin bisa mengatur apakah nilai dapat dilihat atau tidak per sesi.
- Admin bisa mengatur batas peserta per sesi.
- Admin bisa menjalankan auto generate peserta approved ke sesi.
- Sistem hanya memasukkan peserta yang memiliki approval tes valid.
- Satu approval tes hanya dipakai untuk satu kali tes.
- Satu sesi memiliki durasi pengerjaan 120 menit.
- User yang ingin tes lagi harus memakai payment proof baru dan sesi yang berbeda.

### Risiko teknis

- Jika stok soal tidak divalidasi sebelum publish, attempt ujian bisa gagal saat generate snapshot.
- Jika package dan sesi tidak dipisah, penggunaan bank soal harian akan sulit dikelola.
- Jika approval tes tidak dicek saat assignment, satu user bisa salah masuk sesi tanpa hak tes valid.
- Jika auto generate tidak memakai locking atau transaksi, kuota sesi bisa terlewati saat banyak admin melakukan aksi bersamaan.
- Jika show result tidak ditempel di level sesi, hasil user bisa bocor atau salah tampil.

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
- Tanggal dan waktu sesi
- Pengaturan hasil per sesi
- Batas peserta per sesi
- Auto generate peserta approved ke sesi
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

### Session adalah entitas utama pelaksanaan ujian

Package hanya blueprint. Pelaksanaan nyata selalu terjadi di level sesi, karena di sanalah tanggal, waktu, kuota, dan visibilitas hasil ditentukan.

### Approval tes harus dicek saat assignment, bukan hanya saat login

Akun aktif tidak otomatis berarti user boleh ikut ujian. Sistem harus selalu memeriksa ketersediaan approval tes sebelum user dimasukkan ke sesi.

### Auto generate harus deterministik dan aman

Auto generate peserta sebaiknya memakai urutan yang jelas, misalnya berdasarkan approval tertua atau payment approval time, agar hasilnya konsisten dan mudah diaudit.

### Backend adalah sumber waktu utama

Waktu sesi dan durasi 120 menit harus dihitung di backend. Frontend hanya menampilkan countdown dan status.

### Snapshot tetap menjadi kunci randomisasi

Walaupun session dibuat lebih dulu, snapshot soal tetap baru dibentuk saat user benar-benar memulai attempt agar randomisasi tetap personal, konsisten, dan aman untuk resume.