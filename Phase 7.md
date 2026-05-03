# Development Detail per Fase - CBT Laravel API

Dokumen ini merinci tiap fase pengembangan sistem CBT berbasis Laravel dengan fokus backend API, mengikuti kebutuhan proyek: registrasi peserta, upload bukti pembayaran, approval admin, approval tes satu kali per pembayaran, randomisasi soal per user, manajemen sesi tes, dukungan soal audio/image/text, hasil ujian per sesi, dan kontrol keamanan dasar seperti single session, auto-resume, fullscreen warning, serta tab switch detection.

## Fase 7 - Result, Scoring, and Reporting

### Tujuan fase

Membangun modul penilaian, publikasi hasil, dan pelaporan ujian berbasis sesi. Fase ini harus memastikan bahwa nilai peserta dihitung dari attempt yang sah, terhubung ke sesi tes tertentu, dan hanya ditampilkan ke peserta jika pengaturan sesi mengizinkannya.

### Scope pekerjaan

- Hitung skor per attempt.
- Hitung skor per section.
- Simpan hasil ujian per peserta.
- Tampilkan atau sembunyikan hasil berdasarkan setting sesi.
- Monitoring hasil peserta oleh admin.
- Riwayat hasil user.
- Export hasil ke CSV/Excel.
- Rekap hasil per sesi.
- Audit relasi hasil dengan approval tes yang dipakai.

### Deliverables

- Service scoring ujian.
- API result summary untuk admin.
- API user result history.
- API detail hasil per sesi.
- API export hasil.
- Modul publish result per session.
- Rekap peserta hadir, submit, dan belum submit per sesi.

### Task breakdown

#### 1. Perhitungan skor

- Sistem menghitung nilai berdasarkan jawaban pada `exam_attempt`.
- Skor dapat dihitung per section seperti:
  - listening
  - structure
  - reading
- Sistem menyimpan:
  - total_score
  - section_scores
  - correct_count
  - wrong_count
  - unanswered_count
- Scoring harus mengambil data dari snapshot attempt, bukan dari bank soal live.

#### 2. Penyimpanan hasil

- Setelah attempt submitted, sistem membuat atau mengupdate `exam_result`.
- Hasil harus terhubung ke:
  - `user_id`
  - `exam_session_id`
  - `exam_attempt_id`
  - `test_approval_id`
- Relasi ini penting agar satu hasil jelas berasal dari satu sesi dan satu approval tes.

#### 3. Pengaturan visibilitas hasil per sesi

- Setiap sesi memiliki field seperti `show_result_to_user`.
- Jika `show_result_to_user = true`, user dapat melihat skor setelah submit atau setelah dipublish.
- Jika `show_result_to_user = false`, user hanya melihat bahwa ujian telah selesai.
- Admin tetap bisa melihat hasil seluruh sesi tanpa terpengaruh setting ini.

#### 4. Dashboard hasil admin

- Admin dapat melihat daftar hasil peserta per sesi.
- Admin dapat memfilter berdasarkan:
  - sesi tes
  - tanggal
  - status submit
  - peserta
- Admin dapat membuka detail hasil peserta, termasuk:
  - skor
  - section score
  - waktu mulai
  - waktu submit
  - log pelanggaran bila diperlukan

#### 5. Riwayat hasil user

- User dapat melihat riwayat hasil miliknya sendiri.
- Riwayat hanya menampilkan attempt dari approval tes yang sudah selesai.
- Jika sesi tidak mengizinkan tampil hasil, detail nilai tidak ditampilkan ke user.
- Riwayat tetap dapat menampilkan metadata dasar seperti nama sesi dan tanggal tes.

#### 6. Export laporan

- Admin dapat export hasil per sesi ke CSV/Excel.
- File export minimal berisi:
  - nama peserta
  - email
  - nama sesi
  - tanggal tes
  - status submit
  - total score
  - section score
- Export harus dibatasi berdasarkan sesi atau filter tertentu agar mudah dioperasikan.

#### 7. Rekap sesi

- Sistem menyiapkan ringkasan per sesi:
  - total peserta terdaftar
  - total peserta mulai ujian
  - total submit
  - total belum submit
  - rata-rata nilai
- Rekap sesi membantu admin melakukan monitoring operasional harian.

#### 8. Validasi retake pada hasil

- Jika user mengikuti tes lagi dengan payment proof baru dan approval baru, hasil attempt lama dan baru harus tetap terpisah.
- Satu hasil hanya boleh terkait dengan satu approval tes.
- Riwayat user dapat memuat banyak hasil, tetapi masing-masing harus jelas sesi dan approval-nya.

### Aturan bisnis

- Hasil harus dihitung dari snapshot attempt agar stabil dan tidak berubah saat bank soal diperbarui.
- Hasil yang tampil ke user mengikuti setting sesi, bukan setting global.
- Admin selalu dapat melihat semua hasil untuk keperluan monitoring.
- Satu approval tes hanya boleh menghasilkan satu jalur hasil ujian.
- Retake hanya bisa menghasilkan hasil baru jika user memiliki payment proof baru, approval baru, dan sesi yang berbeda.

### Acceptance criteria

- Sistem dapat menghitung skor attempt yang selesai.
- Hasil tersimpan per peserta dan per sesi.
- Admin dapat melihat dan export hasil per sesi.
- User hanya bisa melihat nilai jika sesi mengizinkan.
- Riwayat hasil user tetap rapi meskipun user pernah retake di sesi lain.
- Hasil dari approval lama dan approval baru tidak tercampur.

### Risiko teknis

- Jika scoring mengambil data dari bank soal live, hasil lama bisa berubah setelah soal diedit.
- Jika visibilitas hasil tidak ditempel di level sesi, nilai bisa salah tampil ke peserta.
- Jika relasi hasil ke approval tes tidak disimpan, audit retake akan sulit ditelusuri.

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

- Analitik hasil yang lebih detail
- Dashboard statistik yang lebih kaya
- Export dengan template lanjutan
- Notifikasi publikasi hasil
- Ranking atau grouping hasil bila dibutuhkan

## Catatan arsitektur penting

### Result harus berbasis session

Nilai ujian tidak boleh berdiri sendiri. Setiap hasil harus jelas berasal dari sesi mana, attempt mana, dan approval tes mana.

### Visibilitas hasil harus dibaca dari sesi

Keputusan apakah user boleh melihat nilai harus dibaca dari pengaturan sesi agar admin bisa punya kebijakan berbeda untuk tiap jadwal tes.

### Retake harus tetap terpisah penuh

Jika user tes lagi, hasil lama tidak boleh ditimpa. Sistem harus menyimpan hasil baru sebagai attempt dan approval yang berbeda agar audit trail tetap rapi.

### Admin dan user memiliki hak lihat hasil yang berbeda

Admin selalu perlu akses monitoring penuh, sedangkan user hanya melihat hasil sesuai aturan sesi masing-masing.