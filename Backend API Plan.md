\# Backend API Plan - Laravel CBT



\## Arsitektur Modul



Sistem dibagi menjadi beberapa domain module:



1\. Authentication \& Account Status

2\. Payment Proof Verification

3\. Test Approval Management

4\. Question Bank Management

5\. Exam Package Management

6\. Exam Session Management

7\. Session Participant Assignment

8\. Exam Runtime Engine

9\. Answer Processing \& Scoring

10\. Violation \& Proctoring Events

11\. Reporting \& Export

12\. Settings \& Feature Toggle



\## Rekomendasi Teknis



\- Framework: Laravel 13

\- Auth API: Laravel Sanctum

\- Queue: Queue Worker

\- Database: MySQL

\- Media Storage: local/public

\- Permission: Spatie Laravel Permission

\- Activity Log: Spatie Activitylog

\- Excel Export: Laravel Excel

\- RTE: Trix atau editor setara dengan dukungan RTL dan paste Arabic

\- Image processing: Intervention Image bila diperlukan



\## Prinsip Alur Bisnis



\- User register akun satu kali.

\- User upload bukti pembayaran untuk mendapatkan hak tes.

\- Admin review bukti pembayaran.

\- Jika pembayaran disetujui:

&#x20; - akun dapat diaktifkan bila belum aktif

&#x20; - sistem membuat satu `test\_approval`

\- Satu `test\_approval` hanya berlaku untuk satu kali tes.

\- User yang ingin tes lagi harus upload bukti pembayaran baru.

\- Setiap approval baru harus dipakai untuk sesi yang berbeda.

\- Sesi tes memiliki tanggal, waktu, durasi 120 menit, batas peserta, dan pengaturan tampil/tidaknya nilai.



\## Alur Utama



\### 1. Registrasi sampai Approval Tes



\- User register.

\- User upload payment proof.

\- System set status akun = `pending\_verification`.

\- Admin review payment proof.

\- Jika approved:

&#x20; - akun aktif

&#x20; - user boleh login

&#x20; - sistem membuat `test\_approval` dengan kuota 1

\- Jika rejected:

&#x20; - user diminta upload ulang



\### 2. Pembuatan Sesi Tes



\- Admin membuat sesi tes.

\- Admin menentukan:

&#x20; - exam package

&#x20; - tanggal tes

&#x20; - jam mulai

&#x20; - jam selesai

&#x20; - durasi 120 menit

&#x20; - batas peserta

&#x20; - apakah nilai dapat dilihat user atau tidak

&#x20; - apakah auto generate peserta diaktifkan atau tidak



\### 3. Assignment Peserta ke Sesi



\- Admin dapat assign user ke sesi secara manual.

\- Admin dapat menjalankan auto generate peserta.

\- Auto generate hanya mengambil user yang:

&#x20; - sudah membayar

&#x20; - payment proof sudah approved

&#x20; - memiliki `test\_approval` status `available`

&#x20; - belum dipakai pada sesi lain

\- Sistem menolak assignment baru jika kuota sesi sudah penuh.



\### 4. Start Exam



\- User hanya bisa start exam jika:

&#x20; - akun aktif

&#x20; - sudah terdaftar di sesi

&#x20; - sesi sedang aktif sesuai tanggal dan waktu

&#x20; - memiliki `test\_approval` yang valid untuk sesi tersebut

\- Saat start:

&#x20; - sistem membuat attempt

&#x20; - sistem membuat snapshot soal

&#x20; - urutan soal diacak per user

&#x20; - urutan opsi dapat diacak per user

&#x20; - timer backend dimulai dengan durasi 120 menit



\### 5. Auto Resume



\- Jawaban tersimpan berkala.

\- Attempt status tetap berjalan selama belum submitted dan belum expired.

\- Saat user login kembali:

&#x20; - cek attempt aktif

&#x20; - kembalikan posisi soal terakhir

&#x20; - hitung sisa waktu dari backend



\### 6. Hasil Ujian



\- Setelah submit, sistem menghitung hasil.

\- Admin selalu dapat melihat hasil.

\- User hanya dapat melihat nilai jika `show\_result\_to\_user = true` pada sesi tersebut.

\- Jika `show\_result\_to\_user = false`, user hanya melihat status selesai tanpa skor detail.



\## Standar Response API



Semua endpoint menggunakan helper response formatter yang konsisten dengan struktur:



```json

{

&#x20; "code": 200,

&#x20; "status": "success",

&#x20; "message": "Request berhasil diproses.",

&#x20; "data": {}

}

```



Untuk error:



```json

{

&#x20; "code": 422,

&#x20; "status": "error",

&#x20; "message": "Validasi gagal.",

&#x20; "data": {

&#x20;   "errors": {}

&#x20; }

}

```



\## Desain Endpoint



\## Auth \& Registration



\- POST `/api/register`

\- POST `/api/login`

\- POST `/api/forgot-password`

\- POST `/api/reset-password`

\- POST `/api/logout` *(auth required)*

\- GET `/api/me` *(auth required)*



\## User Dashboard



\- GET `/api/my/profile`

\- PATCH `/api/my/profile`

\- GET `/api/my/test-approvals`

\- GET `/api/my/exam-sessions`

\- GET `/api/my/active-attempt`

\- GET `/api/my/results`



\## Exam Runtime



\- POST `/api/exam-sessions/{session}/start`

\- GET `/api/exam-attempts/{attempt}`

\- GET `/api/exam-attempts/{attempt}/questions/{number}`

\- POST `/api/exam-attempts/{attempt}/answers`

\- POST `/api/exam-attempts/{attempt}/mark-doubt`

\- POST `/api/exam-attempts/{attempt}/navigate`

\- POST `/api/exam-attempts/{attempt}/heartbeat`

\- POST `/api/exam-attempts/{attempt}/violations`

\- POST `/api/exam-attempts/{attempt}/submit`



\## Admin Participant Management



\- GET `/api/admin/users`

\- GET `/api/admin/users/{id}`

\- PATCH `/api/admin/users/{id}/approve-account`

\- PATCH `/api/admin/users/{id}/reject-account`

\- PATCH `/api/admin/users/{id}/reset-password`



\## Admin Payment \& Test Approval



\- GET `/api/admin/payment-proofs`

\- GET `/api/admin/payment-proofs/{id}`

\- PATCH `/api/admin/payment-proofs/{id}/approve`

\- PATCH `/api/admin/payment-proofs/{id}/reject`

\- GET `/api/admin/test-approvals`

\- GET `/api/admin/test-approvals/{id}`



\## Admin Question Bank



\- GET `/api/admin/question-banks`

\- POST `/api/admin/question-banks`

\- PATCH `/api/admin/question-banks/{id}`

\- DELETE `/api/admin/question-banks/{id}`



\## Admin Questions



\- GET `/api/admin/questions`

\- POST `/api/admin/questions`

\- GET `/api/admin/questions/{id}`

\- PATCH `/api/admin/questions/{id}`

\- DELETE `/api/admin/questions/{id}`



\## Admin Exam Package



\- GET `/api/admin/exam-packages`

\- POST `/api/admin/exam-packages`

\- PATCH `/api/admin/exam-packages/{id}`

\- DELETE `/api/admin/exam-packages/{id}`



\## Admin Exam Session



\- GET `/api/admin/exam-sessions`

\- POST `/api/admin/exam-sessions`

\- GET `/api/admin/exam-sessions/{id}`

\- PATCH `/api/admin/exam-sessions/{id}`

\- PATCH `/api/admin/exam-sessions/{id}/publish`

\- PATCH `/api/admin/exam-sessions/{id}/close`



\## Admin Session Participant Assignment



\- GET `/api/admin/exam-sessions/{id}/participants`

\- POST `/api/admin/exam-sessions/{id}/participants/manual-assign`

\- POST `/api/admin/exam-sessions/{id}/participants/auto-generate`

\- DELETE `/api/admin/exam-sessions/{id}/participants/{registrationId}`



\## Admin Monitoring



\- GET `/api/admin/exam-attempts`

\- GET `/api/admin/exam-attempts/{id}`

\- GET `/api/admin/results`

\- GET `/api/admin/violations`

\- GET `/api/admin/export/results`



\## Settings



\- GET `/api/settings/exam` *(public, no auth)*

\- GET `/api/admin/settings/exam`

\- PATCH `/api/admin/settings/exam`



\## Aturan Bisnis Penting



\### Login dan Aktivasi Akun



\- User tidak boleh login jika status:

&#x20; - `pending\_verification`

&#x20; - `rejected`

&#x20; - `suspended`

\- Hanya status `active` yang boleh login.



\### Approval Tes



\- Satu payment proof yang di-approve menghasilkan satu `test\_approval`.

\- Satu `test\_approval` hanya boleh dipakai untuk satu kali tes.

\- Jika user ingin tes lagi, user harus upload bukti pembayaran baru.

\- Bukti pembayaran baru yang di-approve menghasilkan approval tes baru.



\### Sesi Tes



\- Satu sesi harus memiliki:

&#x20; - tanggal tes

&#x20; - jam mulai

&#x20; - jam selesai

&#x20; - durasi 120 menit

&#x20; - max participants

&#x20; - pengaturan `show\_result\_to\_user`

\- User hanya boleh mengikuti sesi jika sudah terdaftar pada sesi tersebut.



\### Auto Generate Peserta



\- Auto generate hanya mengambil user dengan payment proof approved dan test approval available.

\- Auto generate harus menghormati kuota `max\_participants`.

\- Auto generate tidak boleh menempatkan approval yang sama ke lebih dari satu sesi.



\### Snapshot Soal



\- Begitu ujian dimulai, sistem membuat snapshot.

\- Perubahan bank soal setelah ujian dimulai tidak mengubah attempt yang sedang berjalan.

\- Ini penting agar hasil konsisten dan adil.



\### Randomisasi



\- Randomisasi dilakukan saat create attempt, bukan setiap kali load soal.

\- Dengan begitu:

&#x20; - soal tetap konsisten untuk satu user

&#x20; - user A dan user B tetap berbeda urutan

&#x20; - auto-resume tidak merusak urutan



\### Timer Logic



\- Timer tidak lagi fleksibel on/off per attempt.

\- Semua sesi memiliki durasi tetap 120 menit.

\- Frontend hanya menampilkan countdown.

\- Backend tetap menjadi sumber kebenaran waktu habis.

\- Saat waktu habis, sistem wajib auto-submit.



\### Audio Rules



\- Audio memiliki field `audio\_max\_play\_count`.

\- API menyimpan jumlah play per question per attempt.

\- Frontend hanya boleh request play jika belum melebihi limit.

\- URL audio sebaiknya signed URL atau protected route.



\### Single Session Login



\- Simpan `current\_session\_token` di tabel `user\_sessions`.

\- Saat login dari device lain:

&#x20; - opsi 1: tolak login baru

&#x20; - opsi 2: tendang session lama

\- Untuk mode ujian, lebih aman tolak login baru saat attempt aktif.



\### Violation Logging



\- Event yang dicatat:

&#x20; - keluar fullscreen

&#x20; - tab hidden / switch tab

&#x20; - reconnect

&#x20; - multi-login attempt

&#x20; - copy/paste attempt

&#x20; - right click

\- Simpan `severity` dan `action\_taken`.



\## Validasi Teknis



\### Upload Bukti Pembayaran



\- Hanya jpg, jpeg, png, pdf.

\- Max size ditentukan admin.

\- Simpan path file, original filename, mime type, uploaded\_at.

\- User boleh upload lebih dari satu bukti pembayaran sebagai riwayat retake.



\### Test Approval



\- Approval tidak boleh dibuat manual tanpa payment proof approved.

\- Approval yang sudah consumed tidak boleh dipakai lagi.

\- Approval harus terhubung ke satu registrasi sesi.



\### Sesi Tes



\- `session\_date` wajib.

\- `start\_time` wajib.

\- `end\_time` wajib.

\- `duration\_minutes` harus 120.

\- `max\_participants` minimal 1.

\- `show\_result\_to\_user` wajib boolean.



\### Soal



\- Stem soal wajib.

\- Minimal 1 jawaban benar.

\- Jika `question\_type = audio`, audio file wajib ada.

\- Jika `question\_type = image`, image file opsional atau wajib sesuai subtype.



\### Bahasa Arab



\- Field soal dan opsi simpan HTML UTF-8.

\- Aktifkan dukungan RTL pada editor.

\- Sanitasi HTML agar aman, tapi jangan merusak karakter Arab.



\## Struktur Folder Laravel yang Disarankan



\- app/Actions

\- app/Enums

\- app/Http/Controllers/Api

\- app/Http/Requests

\- app/Models

\- app/Policies

\- app/Services

\- app/Jobs

\- app/Events

\- app/Listeners

\- app/Repositories

\- app/DataTransferObjects



\## Acceptance Criteria



\- User tidak bisa login sebelum approval admin.

\- Bukti pembayaran bisa diupload dan direview admin.

\- Setiap payment proof approved menghasilkan satu approval tes.

\- User bisa upload bukti pembayaran lagi untuk tes berikutnya.

\- Admin bisa membuat sesi tes dengan tanggal dan waktu.

\- Admin bisa mengatur apakah nilai dapat dilihat atau tidak per sesi.

\- Admin bisa mengatur batas peserta per sesi.

\- Admin bisa menjalankan auto generate peserta approved ke sesi.

\- Satu approval tes hanya berlaku untuk satu kali tes.

\- Satu sesi memiliki durasi pengerjaan 120 menit.

\- Satu ujian menghasilkan urutan soal berbeda antar user.

\- Audio, image, dan text bisa dipakai pada soal.

\- Attempt ujian bisa resume dari progres terakhir selama sesi masih aktif.

\- Semua pelanggaran terekam di sistem.

