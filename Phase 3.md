# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 3 - Question Bank Management

### Tujuan fase

Membangun modul bank soal yang fleksibel agar admin dapat menyusun banyak bank soal, mengelola soal per kategori, dan mendukung media text, image, serta audio. Modul ini harus siap dipakai oleh sesi tes yang berbeda, termasuk skenario user retake pada sesi lain dengan approval tes yang baru.

### Scope pekerjaan

- CRUD bank soal.
- CRUD soal.
- CRUD opsi jawaban.
- Upload media image/audio.
- Kategori section soal.
- RTE yang mendukung copy-paste bahasa Arab.
- Validasi struktur soal sebelum bisa dipakai di paket ujian.
- Penandaan bank soal agar dapat dipakai oleh sesi tes tertentu.
- Menjamin perubahan bank soal tidak merusak sesi atau attempt yang sudah berjalan.

### Deliverables

- Modul question bank.
- Modul question CRUD.
- Media upload image/audio.
- API list/filter/search soal.
- Preview soal untuk admin.
- Dukungan editor HTML yang aman.
- Dukungan relasi bank soal ke exam package dan exam session.

### Task breakdown

#### 1. CRUD bank soal

- Admin dapat membuat bank soal seperti `Bank Soal 1`, `Bank Soal 2`, `Bank Soal 3`.
- Setiap bank memiliki:
  - nama
  - kode
  - deskripsi
  - status aktif/nonaktif
- Tambahkan informasi pembuat dan tanggal update terakhir.
- Setiap bank soal harus bisa dipilih saat pembuatan package dan sesi tes.

#### 2. CRUD soal

- Field minimal:
  - `section_type`
  - `question_type`
  - `stem_html`
  - `explanation_html` opsional
  - `image_path` opsional
  - `audio_path` opsional
  - `audio_max_play_count`
- Soal dapat dikelompokkan ke kategori seperti:
  - listening
  - structure
  - reading
  - arabic custom section
- Soal harus siap dipakai dalam randomisasi per user saat sesi dimulai.

#### 3. CRUD opsi jawaban

- Setiap soal punya beberapa opsi.
- Minimal satu jawaban benar.
- Opsi disimpan sebagai HTML agar mendukung teks Arab dan formatting.
- Opsi harus kompatibel dengan pengacakan urutan jawaban saat attempt dibuat.

#### 4. Upload media

- Upload image untuk soal atau opsi jika diperlukan.
- Upload audio khusus listening.
- Siapkan metadata durasi audio bila memungkinkan.
- Simpan file dengan naming aman.
- Gunakan protected path atau signed access bila diperlukan agar media tidak mudah diekspos.

#### 5. Editor bahasa Arab

- Integrasi CKEditor 5 atau TinyMCE.
- Pastikan support:
  - UTF-8
  - paste Arabic
  - RTL/LTR switch
  - heading/list/basic formatting
- Sanitasi HTML harus aman tetapi tidak merusak karakter Arab.

#### 6. Validasi kualitas soal

- Soal audio wajib punya file audio.
- Soal image harus punya image bila tipe itu diwajibkan.
- Tidak boleh publish soal tanpa opsi valid.
- Tidak boleh ada dua jawaban benar jika tipe soal single answer.
- Soal yang akan dipakai di sesi harus lolos validasi minimum struktur.

#### 7. Integrasi ke package dan session

- Bank soal harus dapat dipetakan ke exam package.
- Satu exam package dapat mengambil soal dari satu atau beberapa bank.
- Session tes tertentu dapat menggunakan package yang mengarah ke bank soal tertentu.
- Sistem harus mendukung penggunaan bank soal berbeda pada sesi yang berbeda.

### Aturan bisnis

- Soal yang sudah dipakai di sesi aktif tidak boleh dihapus keras; gunakan soft delete atau nonaktif.
- Perubahan soal setelah dipakai ujian tidak boleh mengubah snapshot attempt yang sudah berjalan.
- Audio dan image harus dianggap sebagai aset sensitif yang URL-nya tidak diekspos sembarangan.
- Bank soal harus bisa dipakai ulang untuk banyak sesi tanpa mengubah hasil attempt lama.
- Retake pada sesi lain boleh memakai bank soal yang sama atau berbeda, tetapi snapshot tetap dibuat terpisah per attempt.

### Acceptance criteria

- Admin bisa membuat beberapa bank soal.
- Admin bisa membuat soal text, image, dan audio.
- Teks Arab dapat dipaste dan tersimpan benar.
- Soal dapat difilter berdasarkan bank dan kategori.
- Bank soal dapat dipakai oleh package dan sesi yang berbeda.
- Perubahan soal baru tidak merusak attempt yang sudah berjalan.

### Risiko teknis

- Paste bahasa Arab sering rusak jika editor dan sanitizer tidak diuji pada data nyata.
- Upload audio besar bisa memengaruhi storage dan performa bila tidak dibatasi.
- Jika soal diubah langsung tanpa snapshot, hasil ujian lama bisa ikut berubah.
- Jika media diakses terbuka, file audio listening bisa terlalu mudah diambil user.

---

## Prioritas implementasi

### MVP wajib

- Registrasi peserta
- Upload bukti pembayaran
- Approval admin
- Approval tes satu kali per pembayaran
- Login by status
- CRUD bank soal
- Pembuatan sesi tes
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

### Snapshot adalah kunci randomisasi

Agar nomor soal setiap peserta benar-benar berbeda namun tetap konsisten saat refresh atau resume, randomisasi harus dilakukan sekali saat `exam_attempt` dibuat lalu disimpan ke tabel snapshot attempt-question. Pendekatan ini paling aman untuk model CBT dengan auto-resume.

### Bank soal adalah sumber, bukan runtime

Bank soal hanya menjadi sumber data. Saat ujian dimulai, sistem harus membuat snapshot sendiri agar perubahan bank soal di kemudian hari tidak memengaruhi ujian yang sedang atau sudah berlangsung.

### Package dan session harus dipisah

Bank soal tidak langsung ditembak ke user. Admin lebih dulu menyusun package, lalu package dipakai oleh sesi tes tertentu yang memiliki tanggal, waktu, kuota, dan aturan tampil hasil masing-masing.

### Backend adalah sumber waktu utama

Walaupun fokus fase ini adalah bank soal, desain data tetap harus mempertimbangkan bahwa attempt berjalan dalam sesi berdurasi 120 menit yang dikontrol backend.

### Anti-cheat harus realistis

Di browser, beberapa shortcut dan perlindungan media tidak bisa dibuat absolut. Implementasi yang sehat adalah kombinasi pembatasan UI, logging, threshold, dan audit admin, bukan janji blok total 100 persen.