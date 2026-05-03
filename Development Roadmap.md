\# Development Roadmap - CBT Laravel API



\## Fase 1 - Foundation Project



\### Target



\- Setup Laravel project

\- Setup auth, role, permission

\- Setup storage dan media upload

\- Setup API response standard

\- Setup audit log

\- Setup struktur dasar untuk session-based exam



\### Deliverables



\- Boilerplate Laravel API

\- Role admin dan participant

\- Login/logout dasar

\- Seeder role \& permission

\- Standard response helper integration

\- Struktur folder service, request, repository, dan policy



\## Fase 2 - Registration, Payment, and Test Approval



\### Target



\- Endpoint registrasi

\- Upload bukti pembayaran

\- Workflow review pembayaran oleh admin

\- Restriksi login berdasarkan status akun

\- Approval tes per pembayaran yang disetujui

\- Dukungan upload bukti pembayaran ulang untuk retake



\### Deliverables



\- Register API

\- Payment proof API

\- Admin payment review API

\- Status account middleware

\- Test approval generator

\- Riwayat payment submission user

\- Riwayat approval tes user



\### Catatan Bisnis



\- Satu pembayaran yang di-approve menghasilkan satu approval tes

\- Satu approval tes hanya berlaku untuk satu kali tes

\- Untuk tes ulang, user wajib upload bukti pembayaran baru dan menunggu approval admin lagi



\## Fase 3 - Question Bank Management



\### Target



\- CRUD bank soal

\- CRUD soal

\- CRUD opsi jawaban

\- Upload audio/image

\- RTE support Arabic

\- Validasi soal campuran text, image, dan audio



\### Deliverables



\- Question bank module

\- Question CRUD module

\- Media handling

\- Validation rules untuk soal campuran

\- Support HTML content dengan paste bahasa Arab



\## Fase 4 - Exam Session Management



\### Target



\- Pembuatan sesi tes

\- Pemilihan paket ujian atau bank soal

\- Penentuan tanggal tes

\- Penentuan jam mulai dan jam selesai

\- Penentuan durasi pengerjaan tetap 2 jam

\- Pengaturan batas peserta per sesi

\- Pengaturan nilai dapat dilihat atau tidak per sesi

\- Auto generate user approved ke sesi

\- Manual assign peserta ke sesi



\### Deliverables



\- Exam session API

\- Session schedule API

\- Session participant assignment API

\- Session quota validation

\- Show result per session setting

\- Auto generate participant service



\### Catatan Bisnis



\- Satu sesi memiliki tanggal dan waktu yang jelas

\- Satu sesi memiliki batas jumlah peserta

\- Auto generate hanya mengambil user yang sudah membayar dan sudah di-approve admin

\- Auto generate berhenti saat kuota sesi penuh

\- User yang akan tes lagi harus memakai approval tes baru dan ditempatkan ke sesi yang berbeda



\## Fase 5 - Exam Engine



\### Target



\- Start exam session

\- Snapshot random soal per user

\- Navigasi soal

\- Simpan jawaban

\- Mark doubtful

\- Auto-save

\- Submit manual

\- Auto-submit saat waktu 2 jam habis

\- Resume flow untuk sesi aktif



\### Deliverables



\- Exam runtime API

\- Randomization service

\- Answer submission flow

\- Resume flow

\- Timer backend 120 menit

\- Attempt lock per session



\### Catatan Bisnis



\- Randomisasi soal dilakukan sekali saat sesi dimulai

\- Urutan soal per user harus tetap konsisten saat refresh atau resume

\- Waktu ujian dihitung dari backend, bukan frontend



\## Fase 6 - Security \& Anti-Cheat



\### Target



\- Single active login/session

\- Tab switch logging

\- Fullscreen violation logging

\- Copy/paste attempt logging

\- Audio play limit

\- Session access validation



\### Deliverables



\- Violation API

\- Session guard

\- Audio play tracking

\- Rule-based auto action

\- Activity log untuk pelanggaran dan akses sesi



\## Fase 7 - Scoring \& Reporting



\### Target



\- Hitung skor

\- Simpan hasil ujian

\- Monitoring real-time

\- Export Excel/CSV

\- Pengaturan publikasi hasil berdasarkan sesi



\### Deliverables



\- Scoring service

\- Result API

\- Admin monitoring API

\- Export module

\- Result visibility rule per session



\### Catatan Bisnis



\- Admin selalu dapat melihat hasil peserta

\- User hanya dapat melihat nilai jika sesi mengizinkan nilai ditampilkan

\- Jika nilai disembunyikan, hasil tetap tersimpan untuk admin dan kebutuhan export



\## Fase 8 - QA \& Hardening



\### Target



\- Test API

\- Test race condition

\- Test randomisasi

\- Test resume session

\- Test keamanan upload

\- Test concurrent users

\- Test quota sesi

\- Test auto generate peserta

\- Test retake dengan pembayaran ulang



\### Deliverables



\- Feature test

\- Load test baseline

\- Bug fixing

\- Release candidate

\- UAT checklist admin dan user



\## Prioritas MVP



\### Wajib untuk MVP



\- Registrasi

\- Upload bukti pembayaran

\- Review dan approval admin

\- Login berdasarkan status akun

\- Approval tes 1 kali per pembayaran

\- CRUD bank soal

\- Pembuatan sesi tes

\- Tanggal dan waktu sesi

\- Batas peserta per sesi

\- Auto generate peserta approved ke sesi

\- Randomisasi soal

\- Timer ujian 2 jam

\- Simpan jawaban

\- Submit ujian

\- Hasil dasar

\- Pengaturan tampil/tidaknya nilai per sesi



\### Bisa menyusul setelah MVP



\- Pelanggaran ujian detail

\- Force fullscreen enforcement yang lebih ketat

\- Advanced analytics

\- Export yang lebih kompleks

\- Pembahasan soal

\- Notifikasi email/WhatsApp

\- Integrasi pengingat sesi otomatis



\## Estimasi Sprint



\### Sprint 1



\- Foundation

\- Auth

\- Role

\- Payment approval

\- Response standardization



\### Sprint 2



\- Question bank

\- Media upload

\- Arabic editor support



\### Sprint 3



\- Exam session management

\- Session schedule

\- Session quota

\- Show result setting

\- Auto generate peserta



\### Sprint 4



\- Exam engine

\- Randomization

\- Timer 120 menit

\- Resume

\- Submit



\### Sprint 5



\- Scoring

\- Monitoring

\- Export

\- Security hardening

\- QA utama



\## Risiko Teknis



\- Randomisasi soal bisa kacau jika tidak pakai snapshot session

\- Resume ujian bisa bermasalah jika waktu hanya dihitung di frontend

\- Auto generate peserta bisa bentrok jika tidak dikunci dengan validasi kuota dan approval availability

\- Approval tes bisa terpakai ganda jika tidak ada constraint satu approval untuk satu sesi

\- Audio protection tidak bisa 100% aman jika file benar-benar sampai ke browser

\- Shortcut seperti Alt+Tab tidak bisa diblok total di level browser

\- Paste bahasa Arab bisa rusak jika editor/sanitizer tidak diuji sejak awal



\## Definition of Done



Sebuah fitur dianggap selesai jika:



\- Endpoint selesai dan terdokumentasi

\- Validasi request lengkap

\- Permission dicek

\- Test minimal happy path dan edge case tersedia

\- Error response konsisten

\- Audit/activity log tersedia untuk aksi penting

\- Sudah diuji dari sisi admin dan user

\- Sudah diuji untuk skenario approval tes, assignment sesi, kuota sesi, dan retake

