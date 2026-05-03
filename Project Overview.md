\# Project Overview - CBT Arabic/TOEFL Style



\## Ringkasan Proyek



Sistem ini adalah platform CBT berbasis web menggunakan Laravel dengan fokus utama pada backend API untuk manajemen user, verifikasi pembayaran manual, sesi tes, bank soal, pelaksanaan ujian, penilaian, dan monitoring admin.



Target sistem:



\- Registrasi mandiri user

\- Upload bukti pembayaran

\- Approval admin sebelum user mendapatkan hak tes

\- Pembuatan sesi tes dengan tanggal dan waktu pelaksanaan

\- Pengaturan nilai dapat dilihat atau tidak per sesi

\- Pengaturan batas peserta per sesi

\- Auto generate user yang sudah membayar dan di-approve admin ke dalam sesi

\- Randomisasi soal per user

\- Penggunaan bank soal berdasarkan sesi atau jadwal

\- Dukungan soal text, image, dan audio

\- Rich text editor yang mendukung copy-paste bahasa Arab

\- Auto-save, auto-submit, dan auto-resume saat koneksi terganggu

\- Durasi pengerjaan tes per sesi adalah 2 jam



\## Tujuan Bisnis



\- Mempermudah pelaksanaan ujian CBT berbasis sesi.

\- Memudahkan admin mengelola peserta, pembayaran, sesi tes, bank soal, dan hasil.

\- Memungkinkan satu user mengikuti tes lebih dari satu kali dengan mekanisme pembayaran ulang dan approval ulang.

\- Memungkinkan penggunaan bank soal berbeda pada sesi yang berbeda.



\## Aturan Bisnis Inti



\- User dapat register akun satu kali.

\- User wajib upload bukti pembayaran untuk mendapatkan hak ikut tes.

\- Satu kali approval tes setelah upload bukti pembayaran hanya berlaku untuk satu kali tes.

\- User yang ingin mengikuti tes lagi wajib upload bukti pembayaran baru.

\- Setiap bukti pembayaran baru tetap harus direview dan di-approve admin.

\- User yang sudah mendapat approval tes dapat dimasukkan ke sesi tertentu.

\- Satu sesi memiliki tanggal tes, jam mulai, jam selesai, batas peserta, dan durasi pengerjaan 2 jam.

\- Nilai dapat ditampilkan atau disembunyikan per sesi sesuai pengaturan admin.

\- User yang akan tes lagi harus ditempatkan ke sesi yang berbeda dengan approval tes yang baru.



\## User Roles



\### 1. User



\- Register akun

\- Upload bukti pembayaran

\- Login jika akun sudah aktif

\- Melihat sesi tes yang sudah di-assign

\- Mengikuti tes sesuai sesi yang ditentukan

\- Menjawab soal dan menyimpan progres

\- Melihat hasil hanya jika sesi mengizinkan nilai ditampilkan

\- Upload bukti pembayaran lagi untuk tes berikutnya



\### 2. Admin



\- Verifikasi pembayaran user

\- Aktivasi akun user

\- Memberikan approval tes

\- Membuat sesi tes

\- Menentukan tanggal dan waktu tes

\- Mengatur apakah nilai dapat dilihat atau tidak per sesi

\- Mengatur batas peserta tiap sesi

\- Menambahkan user ke sesi secara manual

\- Menjalankan auto generate user approved ke sesi

\- Mengelola bank soal

\- Mengelola media soal

\- Mengelola paket ujian

\- Monitoring hasil dan pelanggaran

\- Export laporan



\## Fitur Utama



\### A. Registrasi, Pembayaran, dan Approval Tes



\- User daftar akun

\- User upload bukti pembayaran

\- Status awal akun adalah `pending\_verification`

\- Admin review bukti pembayaran

\- Jika valid:

&#x20; - akun user diaktifkan bila belum aktif

&#x20; - sistem memberikan 1 approval tes

\- Satu approval tes hanya dapat digunakan untuk satu kali tes

\- Jika user ingin tes lagi, user harus upload bukti pembayaran baru

\- Approval tes baru hanya berlaku untuk sesi yang berbeda dari tes sebelumnya



\### B. Manajemen Sesi Tes



\- Admin dapat membuat banyak sesi tes

\- Setiap sesi memiliki:

&#x20; - nama sesi

&#x20; - tanggal tes

&#x20; - jam mulai

&#x20; - jam selesai

&#x20; - durasi pengerjaan 120 menit

&#x20; - batas peserta

&#x20; - status publish

&#x20; - pengaturan tampilkan nilai atau tidak

\- Setiap sesi dapat dihubungkan ke paket ujian atau bank soal tertentu



\### C. Penempatan Peserta ke Sesi



\- User hanya dapat mengikuti sesi jika memiliki approval tes yang masih tersedia

\- Admin dapat menempatkan user ke sesi secara manual

\- Admin dapat menjalankan auto generate user yang:

&#x20; - sudah upload bukti pembayaran

&#x20; - pembayaran sudah di-approve

&#x20; - memiliki approval tes yang belum dipakai

\- Auto generate berhenti saat kuota sesi penuh

\- Satu approval tes tidak boleh dipakai untuk lebih dari satu sesi



\### D. Engine Ujian



\- Soal ditampilkan satu per satu

\- Nomor soal setiap user berbeda urutan

\- Opsi jawaban juga bisa diacak

\- Ada tombol next, previous, dan ragu-ragu

\- Auto-save jawaban

\- Auto-submit saat waktu 2 jam habis

\- Auto-resume jika browser tertutup atau koneksi putus selama sesi masih aktif



\### E. Hasil Ujian



\- Hasil ujian dihitung setelah submit

\- Admin dapat mengatur apakah nilai dapat dilihat oleh user atau tidak pada setiap sesi

\- Jika nilai diizinkan tampil, user dapat melihat skor setelah ujian selesai

\- Jika nilai disembunyikan, user hanya melihat status bahwa ujian telah selesai

\- Admin tetap dapat melihat hasil semua peserta untuk kebutuhan monitoring dan laporan



\### F. Bank Soal



\- Admin bisa membuat banyak bank soal

\- Contoh:

&#x20; - Bank Soal 1

&#x20; - Bank Soal 2

&#x20; - Bank Soal 3

\- Paket ujian dapat memilih bank soal tertentu

\- Sesi hari ini bisa memakai Bank Soal 3

\- Sesi berikutnya bisa memakai Bank Soal 2



\### G. Tipe Soal



\- Text only

\- Image based

\- Audio based

\- Kombinasi text + image

\- Kombinasi text + audio



\### H. Editor Soal



\- Gunakan rich text editor yang support paste bahasa Arab

\- Konten soal, opsi, dan pembahasan harus disimpan dalam format HTML yang aman

\- Media upload harus mendukung gambar dan audio



\## Kebutuhan Non-Fungsional



\- API-first architecture

\- Role-based access control

\- Audit log untuk approval dan aktivitas penting

\- Keamanan upload file

\- Auto-save berkala

\- Session control untuk single active session

\- Pelanggaran ujian tercatat

\- Skalabel untuk banyak user bersamaan

\- Backend menjadi sumber kebenaran waktu ujian

\- Semua response API mengikuti helper response formatter yang konsisten



\## Asumsi Scope



\### In Scope



\- Backend API Laravel

\- Admin panel internal

\- Manajemen user

\- Upload bukti pembayaran manual

\- Approval tes berbasis pembayaran

\- Manajemen sesi tes

\- Batas peserta per sesi

\- Auto generate peserta ke sesi

\- Manajemen bank soal

\- Exam engine

\- Hasil ujian

\- Monitoring pelanggaran



\### Out of Scope



\- Payment gateway otomatis

\- Aplikasi mobile native

\- Sertifikat PDF otomatis

\- Penyediaan konten soal oleh developer

\- Hosting dan domain

