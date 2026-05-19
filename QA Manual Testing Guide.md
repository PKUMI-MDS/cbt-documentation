# QA Manual Testing Guide — CBT TOAFL

> **Versi:** 1.0  
> **Tanggal:** 14 Mei 2026  
> **Target Audience:** QA Engineer Pemula (first-time web QA)  
> **Scope:** Manual functional testing untuk `fe-cbt` (Peserta) dan `cbt-admin` (Admin Panel)  
> **Environment:** Production (live URLs)

---

## Daftar Isi

1. [Pengenalan QA untuk Pemula](#1-pengenalan-qa-untuk-pemula)
2. [Persiapan Sebelum Testing](#2-persiapan-sebelum-testing)
3. [Fitur Aplikasi yang Perlu Diuji](#3-fitur-aplikasi-yang-perlu-diuji)
4. [FE-CBT — Test Cases Detail (Peserta)](#4-fe-cbt--test-cases-detail-peserta)
5. [CBT-Admin — Test Cases Detail](#5-cbt-admin--test-cases-detail)
6. [End-to-End Flow Testing](#6-end-to-end-flow-testing)
7. [Cross-Browser & Device Testing](#7-cross-browser--device-testing)
8. [Cara Melaporkan Bug (Bug Report)](#8-cara-melaporkan-bug-bug-report)
9. [Checklist Harian QA](#9-checklist-harian-qa)
10. [FAQ & Troubleshooting](#10-faq--troubleshooting)

---

## 1. Pengenalan QA untuk Pemula

### 1.1 Apa Itu QA?

QA (Quality Assurance) adalah proses memastikan aplikasi/software berjalan sesuai yang diharapkan. Tugas QA adalah **mencari bug/cacat** sebelum aplikasi digunakan oleh user nyata.

> **Bug** = kesalahan/fitur yang tidak berjalan sesuai desain.  
> **Defect** = kekurangan yang membuat pengalaman user tidak nyaman.

### 1.2 Jenis Testing Web yang Akan Dilakukan

| Jenis Testing | Apa yang Diuji? | Contoh |
|---|---|---|
| **Functional** | Fitur berjalan sesuai requirement | Tombol login bisa diklik dan redirect ke dashboard |
| **UI/UX** | Tampilan dan pengalaman user | Warna tombol konsisten, teks tidak terpotong |
| **Compatibility** | Berjalan di berbagai browser & device | Chrome vs Edge, Desktop vs Mobile |
| **Security (Basic)** | Akses yang tidak seharusnya | User non-admin tidak bisa akses halaman admin |
| **Performance (Basic)** | Kecepatan loading | Halaman dashboard load dalam < 3 detik |
| **Edge Case** | Skenario aneh/tidak biasa | Upload file 100 MB, klik tombol 10x berturut-turut |

### 1.3 Prinsip Dasar QA

1. **"Break the app" mindset** — Coba cari cara agar aplikasi error.
2. **Test both happy path & negative path** — Uji skenario sukses DAN skenario gagal.
3. **Document everything** — Screenshot setiap bug, catat langkah reproduksi dengan detail.
4. **Be objective** — Jangan berasumsi "user pasti tidak akan begini". Kalau bisa dilakukan, harus ditangani.

---

## 2. Persiapan Sebelum Testing

### 2.1 Tools yang Wajib Dipasang

| Tool | Fungsi | Cara Install |
|---|---|---|
| **Google Chrome** atau **Microsoft Edge** | Browser utama untuk testing | Download dari google.com/chrome atau microsoft.com/edge |
| **Developer Tools (F12)** | Inspeksi elemen, lihat error console, network request | Tekan `F12` di browser — sudah built-in |
| **Screenshots & Screen Recording** | Dokumentasi bug | Windows: `Win + Shift + S`, Mac: `Cmd + Shift + 5` |
| **Excel / Google Sheets** | Menyimpan test case & hasil testing | Google Sheets (gratis) atau Microsoft Excel |
| **WhatsApp Web / Telegram Desktop** | Komunikasi dengan tim | web.whatsapp.com |

### 2.2 Cara Buka Developer Tools (Penting!)

1. Buka browser (Chrome/Edge).
2. Tekan tombol `F12` di keyboard.
3. Akan muncul panel di bawah/halaman. Ada beberapa tab:
   - **Elements** → Lihat struktur HTML & CSS
   - **Console** → Lihat error JavaScript (merah = error)
   - **Network** → Lihat request API yang gagal (status merah = error)
   - **Application → Storage → Local Storage** → Lihat data tersimpan di browser

> **Tips:** Saat menemukan bug, selalu cek tab **Console** dan **Network** untuk melihat apakah ada error teknis di baliknya.

### 2.3 Akun Test yang Dibutuhkan

Mintalah ke tim developer/admin akun berikut:

| Aplikasi | Peran | Kegunaan |
|---|---|---|
| FE-CBT | Peserta (belum terverifikasi) | Test register, waiting-approval |
| FE-CBT | Peserta (aktif, belum bayar) | Test payment proof upload |
| FE-CBT | Peserta (aktif, sudah bayar, ada jadwal) | Test exam runtime |
| FE-CBT | Peserta (aktif, sudah ujian) | Test lihat hasil |
| CBT-Admin | Admin | Test semua fitur admin |

> **Catatan:** Untuk test register, gunakan email baru yang belum pernah didaftarkan. Format: `qa.test.001@mailinator.com` (bisa pakai mailinator.com untuk email throwaway).

### 2.4 File Test Case Template

Buatlah spreadsheet dengan kolom berikut:

| Test Case ID | Module | Test Description | Pre-condition | Test Steps | Expected Result | Actual Result | Status | Priority | Screenshot | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| FE-001 | Login | Login dengan email & password benar | Akun aktif tersedia | 1. Buka halaman login 2. Isi email & password 3. Klik Login | Redirect ke dashboard | | | High | | |

> **Status:** `Pass` (berhasil), `Fail` (gagal), `Blocked` (tidak bisa diuji karena fitur lain rusak), `N/A` (tidak applicable)

---

## 3. Fitur Aplikasi yang Perlu Diuji

### 3.1 FE-CBT (Frontend Peserta)

**URL:** https://cat.pkumionline.cloud/

| Modul | Deskripsi Singkat |
|---|---|
| Landing Page | Halaman depan aplikasi |
| Panduan | Halaman panduan penggunaan |
| Register | Pendaftaran akun baru + upload bukti pembayaran |
| Login | Masuk ke akun yang sudah terdaftar |
| AuthGuard | Proteksi halaman yang memerlukan login |
| Waiting Approval | Halaman tunggu verifikasi admin |
| Dashboard | Halaman utama setelah login, lihat jadwal ujian |
| Payment Proof | Upload & history bukti pembayaran |
| Profile | Lihat data diri |
| Exam Instruction | Halaman instruksi sebelum ujian |
| Exam Runtime | Halaman ujian berlangsung (soal, timer, navigasi) |
| Anti-Cheat | Fullscreen, tab switch detection, copy-paste block |
| Exam Result | Halaman hasil ujian |
| Logout | Keluar dari akun |

### 3.2 CBT-Admin (Admin Panel)

**URL:** https://admin-cat.pkumionline.cloud/

| Modul | Deskripsi Singkat |
|---|---|
| Login Admin | Masuk ke panel admin |
| Dashboard | Statistik, chart, notifikasi pending |
| User Management | Kelola akun peserta (lihat, approve, reject, reset password) |
| Payment Proof Review | Review & approve/reject bukti pembayaran |
| Question Bank | Kelola bank soal |
| Question CRUD | Kelola soal individual (create, read, update, delete) |
| Exam Package | Kelola paket ujian |
| Exam Session | Kelola sesi ujian (publish, close, finish, cancel) |
| Participant Management | Assign peserta ke sesi |
| Live Monitoring | Pantau peserta yang sedang ujian real-time |
| Export Results | Export hasil ujian ke Excel/CSV |
| Analytics | Statistik skor per sesi/paket |
| Activity Logs | Riwayat aktivitas admin |
| Global Settings | Pengaturan global ujian |

---

## 4. FE-CBT — Test Cases Detail (Peserta)

> **Format Test Case:**
> - **Pre-condition** = Apa yang harus sudah siap sebelum test?
> - **Steps** = Langkah 1, 2, 3...
> - **Expected Result** = Apa yang HARUS terjadi?
> - **Priority** = `P0` (Critical/Must fix), `P1` (High), `P2` (Medium), `P3` (Low)

---

### 4.1 Landing Page & Panduan

#### FE-001 — Landing Page Tampil dengan Benar
- **Pre-condition:** Tidak perlu login.
- **Steps:**
  1. Buka https://cat.pkumionline.cloud/
  2. Tunggu halaman load sepenuhnya.
  3. Periksa semua elemen: logo, judul, deskripsi, tombol "Masuk", tombol "Daftar".
  4. Scroll ke bawah hingga footer.
- **Expected Result:** Semua elemen tampil tanpa error. Tidak ada teks "undefined" atau "null". Tidak ada error di Console (F12).
- **Priority:** P1

#### FE-002 — Navigasi ke Halaman Login
- **Steps:**
  1. Di landing page, klik tombol "Masuk" / "Login".
- **Expected Result:** Browser pindah ke halaman `/login`. Form login tampil.
- **Priority:** P0

#### FE-003 — Navigasi ke Halaman Register
- **Steps:**
  1. Di landing page, klik tombol "Daftar" / "Register".
- **Expected Result:** Browser pindah ke halaman `/register`. Form registrasi tampil.
- **Priority:** P0

#### FE-004 — Navigasi ke Halaman Panduan
- **Steps:**
  1. Di landing page, klik link "Panduan".
- **Expected Result:** Browser pindah ke halaman `/panduan`. Konten panduan tampil lengkap.
- **Priority:** P1

#### FE-005 — Responsive Design — Landing Page di Mobile
- **Pre-condition:** Buka di HP atau resize browser ke ukuran layar HP (±375px lebar).
- **Steps:**
  1. Buka landing page di HP / dev tools mobile view.
  2. Periksa tata letak, teks tidak terpotong, tombol bisa diklik.
- **Expected Result:** Layout menyesuaikan layar kecil. Tidak ada horizontal scroll.
- **Priority:** P1

---

### 4.2 Register (Pendaftaran Akun Baru)

> **Catatan Penting:** Register SEKARANG sudah termasuk upload bukti pembayaran. Admin akan review payment proof + akun dalam satu langkah.

#### FE-006 — Register dengan Data Valid (Happy Path)
- **Pre-condition:** Gunakan email yang BELUM pernah didaftarkan.
- **Steps:**
  1. Buka `/register`.
  2. Isi:
     - Nama: `QA Tester`
     - Email: `qa.test.001@mailinator.com`
     - Password: `Password123!`
     - Konfirmasi Password: `Password123!`
     - No. HP: `081234567890` (11–13 digit)
     - Institusi: `Universitas QA`
  3. Upload file bukti pembayaran (JPG/PNG/PDF, < 5MB).
  4. Klik tombol "Daftar".
- **Expected Result:**
  - Tidak ada error.
  - Redirect ke `/waiting-approval`.
  - Tampil pesan sukses: "Registrasi berhasil. Akun menunggu verifikasi admin."
  - Di Console/Network, POST `/api/register` response 201.
- **Priority:** P0

#### FE-007 — Register tanpa Upload Payment Proof
- **Steps:**
  1. Isi semua field register.
  2. JANGAN upload file payment proof.
  3. Klik "Daftar".
- **Expected Result:** Muncul pesan error validasi: "Bukti pembayaran wajib diupload." atau form tidak bisa disubmit.
- **Priority:** P0

#### FE-008 — Register dengan Email yang Sudah Terdaftar
- **Pre-condition:** Email sudah pernah didaftarkan sebelumnya.
- **Steps:**
  1. Isi form register dengan email yang sudah ada.
  2. Klik "Daftar".
- **Expected Result:** Muncul error: "Email sudah terdaftar." atau "The email has already been taken."
- **Priority:** P1

#### FE-009 — Register dengan Password Kurang dari 8 Karakter
- **Steps:**
  1. Isi password: `12345`
  2. Konfirmasi password: `12345`
  3. Klik "Daftar".
- **Expected Result:** Error validasi: "Password minimal 8 karakter."
- **Priority:** P1

#### FE-010 — Register dengan Password & Konfirmasi Tidak Sama
- **Steps:**
  1. Password: `Password123!`
  2. Konfirmasi: `Password456!`
  3. Klik "Daftar".
- **Expected Result:** Error validasi: "Password tidak cocok."
- **Priority:** P1

#### FE-011 — Register dengan Nomor HP Tidak Valid
- **Steps:**
  1. Isi No. HP: `abc123` atau `0812` (kurang dari 11 digit).
  2. Klik "Daftar".
- **Expected Result:** Error validasi: "Nomor HP harus 11–13 digit angka."
- **Priority:** P1

#### FE-012 — Register dengan File Payment Proof Terlalu Besar (>5MB)
- **Steps:**
  1. Cari/buat file JPG/PNG/PDF dengan ukuran > 5MB (misal: 6MB).
  2. Upload file tersebut.
  3. Klik "Daftar".
- **Expected Result:** Error validasi: "Ukuran file maksimal 5MB." atau form tidak bisa disubmit.
- **Priority:** P1

#### FE-013 — Register dengan File Payment Proof Format Tidak Didukung
- **Steps:**
  1. Upload file `.exe`, `.zip`, atau `.txt`.
  2. Klik "Daftar".
- **Expected Result:** Error validasi: "Format file harus JPG, PNG, atau PDF."
- **Priority:** P1

#### FE-014 — Preview File setelah Upload di Register Form
- **Steps:**
  1. Pilih file JPG/PNG untuk diupload.
  2. Perhatikan area bawah form upload.
- **Expected Result:** Muncul preview/thumbnail file, nama file, ukuran file, dan badge "Siap diupload". Ada tombol "Hapus" untuk menghapus file.
- **Priority:** P2

#### FE-015 — Payment Info Muncul di Halaman Register
- **Steps:**
  1. Buka `/register`.
  2. Scroll ke bawah form atau perhatikan area di sekitar form.
- **Expected Result:** Muncul informasi pembayaran: bank (Mega Syariah), nominal (Rp. 150.000,-), nomor rekening, dan kontak WhatsApp.
- **Priority:** P2

---

### 4.3 Login

#### FE-016 — Login dengan Kredensial Valid
- **Pre-condition:** Akun sudah terdaftar dan aktif.
- **Steps:**
  1. Buka `/login`.
  2. Isi email dan password yang benar.
  3. Klik "Masuk".
- **Expected Result:**
  - Redirect ke `/dashboard`.
  - Token tersimpan di Local Storage (cek F12 → Application → Local Storage).
  - Tidak ada error di Console.
- **Priority:** P0

#### FE-017 — Login dengan Password Salah
- **Steps:**
  1. Isi email yang benar.
  2. Isi password yang salah.
  3. Klik "Masuk".
- **Expected Result:** Pesan error: "Email atau password salah." atau "Invalid credentials." Tidak redirect ke dashboard.
- **Priority:** P0

#### FE-018 — Login dengan Email yang Tidak Terdaftar
- **Steps:**
  1. Isi email yang belum pernah didaftarkan.
  2. Isi password bebas.
  3. Klik "Masuk".
- **Expected Result:** Pesan error: "Email atau password salah."
- **Priority:** P0

#### FE-019 — Login dengan Field Kosong
- **Steps:**
  1. Biarkan email & password kosong.
  2. Klik "Masuk".
- **Expected Result:** Error validasi: "Email wajib diisi" dan "Password wajib diisi."
- **Priority:** P1

#### FE-020 — Login dengan Format Email Salah
- **Steps:**
  1. Isi email: `bukan-email`.
  2. Isi password.
  3. Klik "Masuk".
- **Expected Result:** Error validasi: "Format email tidak valid."
- **Priority:** P1

#### FE-021 — Redirect ke Login saat Akses Halaman Protected Tanpa Login
- **Steps:**
  1. Logout (jika sedang login).
  2. Akses langsung `/dashboard` atau `/profile`.
- **Expected Result:** Redirect ke `/login`. Setelah login berhasil, redirect kembali ke halaman yang dituju.
- **Priority:** P0

#### FE-022 — User dengan Role Non-Participant Tidak Bisa Login di FE-CBT
- **Pre-condition:** Punya akun admin.
- **Steps:**
  1. Coba login dengan akun admin di fe-cbt.
- **Expected Result:** Error: "Akses ditolak. Halaman ini hanya untuk peserta." atau semacamnya. Tidak bisa masuk.
- **Priority:** P1

---

### 4.4 AuthGuard & Route Protection

#### FE-023 — User Pending Verification Tidak Bisa Akses Dashboard
- **Pre-condition:** Login dengan akun yang statusnya `pending_verification`.
- **Steps:**
  1. Login dengan akun pending.
  2. Coba akses `/dashboard`, `/profile`, `/exam/instruction`.
- **Expected Result:** Semua redirect ke `/waiting-approval`. Hanya `/waiting-approval` dan `/payment-proof` yang bisa diakses.
- **Priority:** P0

#### FE-024 — User Active Bisa Akses Semua Halaman
- **Pre-condition:** Login dengan akun aktif.
- **Steps:**
  1. Akses `/dashboard`, `/profile`, `/payment-proof`, `/exam/instruction`.
- **Expected Result:** Semua halaman bisa diakses tanpa redirect yang tidak diinginkan.
- **Priority:** P0

#### FE-025 — User Rejected Hanya Bisa Akses Waiting Approval
- **Pre-condition:** Login dengan akun yang statusnya `rejected`.
- **Steps:**
  1. Login.
  2. Coba akses halaman lain selain `/waiting-approval`.
- **Expected Result:** Redirect ke `/waiting-approval`. Tampil pesan penolakan dengan alasan.
- **Priority:** P0

---

### 4.5 Waiting Approval

#### FE-026 — Tampilan Waiting Approval untuk User Pending
- **Pre-condition:** Login dengan akun `pending_verification`.
- **Steps:**
  1. Setelah login, pastikan redirect ke `/waiting-approval`.
  2. Perhatikan tampilan: status badge, penjelasan, kontak admin.
- **Expected Result:**
  - Badge status: "Menunggu Verifikasi".
  - Informasi rekening bank & kontak WhatsApp tampil.
  - Tombol untuk upload payment proof tersedia (kalau belum upload).
- **Priority:** P0

#### FE-027 — Polling Status di Waiting Approval
- **Pre-condition:** Login dengan akun pending. Buka tab Network di DevTools.
- **Steps:**
  1. Di halaman `/waiting-approval`, biarkan terbuka selama 1–2 menit.
  2. Perhatikan tab Network.
- **Expected Result:** Setiap ~30 detik ada request `GET /api/me`. Response menunjukkan status akun.
- **Priority:** P1

#### FE-028 — Auto-Redirect ke Dashboard saat Status Berubah Jadi Active
- **Pre-condition:** User pending, halaman `/waiting-approval` terbuka.
- **Steps:**
  1. Minta admin untuk approve akun/payment proof user tersebut (di cbt-admin).
  2. Perhatikan halaman fe-cbt yang terbuka.
- **Expected Result:**
  - Setelah beberapa detik, halaman otomatis redirect ke `/dashboard`.
  - Muncul toast/pesan: "Akun Anda telah diverifikasi!"
- **Priority:** P0

#### FE-029 — Tampilan Waiting Approval untuk User Rejected
- **Pre-condition:** Login dengan akun `rejected`.
- **Steps:**
  1. Akses `/waiting-approval`.
  2. Perhatikan tampilan.
- **Expected Result:**
  - Badge status: "Ditolak" (warna merah).
  - Alasan penolakan tampil (jika ada).
  - Tidak ada tombol upload ulang (flow saat ini: reject = akun ditolak total).
- **Priority:** P1

---

### 4.6 Dashboard Peserta

#### FE-030 — Dashboard Load Semua Data
- **Pre-condition:** Login dengan akun aktif yang memiliki data.
- **Steps:**
  1. Login, pastikan redirect ke `/dashboard`.
  2. Tunggu loading selesai (skeleton hilang).
  3. Periksa:
     - Data profil (nama, email, status)
     - Jadwal ujian (kalau ada)
     - Tombol "Mulai Ujian" atau "Lanjutkan Ujian"
     - History hasil ujian (kalau pernah ujian)
- **Expected Result:** Semua data tampil dengan benar. Tidak ada "undefined" atau kosong yang seharusnya ada data.
- **Priority:** P0

#### FE-031 — Dashboard Empty State (User Baru, Belum Ada Jadwal)
- **Pre-condition:** Login dengan akun aktif yang belum pernah di-assign ke sesi ujian.
- **Steps:**
  1. Buka `/dashboard`.
- **Expected Result:** Tampil pesan: "Belum ada jadwal ujian" atau semacamnya. Tidak error.
- **Priority:** P1

#### FE-032 — Tombol "Mulai Ujian" Mengarah ke Instruction Page
- **Pre-condition:** Ada sesi ujian yang bisa dimulai.
- **Steps:**
  1. Di dashboard, klik tombol "Mulai Ujian".
- **Expected Result:** Redirect ke `/exam/instruction`.
- **Priority:** P0

#### FE-033 — Tombol "Lanjutkan Ujian" Muncul saat Ada Attempt Aktif
- **Pre-condition:** User pernah mulai ujian tapi belum submit.
- **Steps:**
  1. Di dashboard, perhatikan CTA (call-to-action) button.
- **Expected Result:** Muncul tombol "Lanjutkan Ujian" (bukan "Mulai Ujian").
- **Priority:** P1

#### FE-034 — Hasil Ujian Hanya Tampil Kalau Sesi Mengizinkan
- **Pre-condition:** User sudah pernah ujian.
- **Steps:**
  1. Di dashboard, lihat bagian history/hasil ujian.
  2. Cek dengan sesi yang `show_result_to_user = true` dan `false`.
- **Expected Result:**
  - Kalau `true`: Skor, benar, salah, kosong tampil.
  - Kalau `false`: Pesan "Hasil belum tersedia" atau semacamnya.
- **Priority:** P1

---

### 4.7 Payment Proof (Upload & History)

#### FE-035 — Upload Payment Proof dari Halaman Payment Proof
- **Pre-condition:** Login dengan akun aktif.
- **Steps:**
  1. Akses `/payment-proof`.
  2. Klik tombol upload.
  3. Pilih file JPG/PNG/PDF valid.
  4. Klik "Kirim" / "Upload".
- **Expected Result:**
  - File berhasil terupload.
  - Muncul di history dengan status "Menunggu Review".
  - Toast sukses muncul.
- **Priority:** P0

#### FE-036 — History Payment Proof Tampil dengan Status
- **Steps:**
  1. Buka `/payment-proof`.
  2. Perhatikan list history.
- **Expected Result:**
  - Setiap item menunjukkan: nama file, tanggal upload, status (badge warna).
  - Status: pending (kuning), approved (hijau), rejected (merah).
  - Kalau rejected, alasan penolakan tampil.
- **Priority:** P1

#### FE-037 — Klik Card Payment Proof ke Detail Page
- **Steps:**
  1. Di list history, klik salah satu card/item.
- **Expected Result:** Buka halaman `/payment-proof/[id]` dengan detail lengkap.
- **Priority:** P2

#### FE-038 — Upload Ulang saat Status Rejected
- **Pre-condition:** Punya payment proof yang pernah direject.
- **Steps:**
  1. Buka `/payment-proof`.
  2. Klik "Upload Ulang".
  3. Pilih file baru.
  4. Kirim.
- **Expected Result:** File baru terupload, status kembali ke "Menunggu Review".
- **Priority:** P1

#### FE-039 — Payment Proof Page Bisa Diakses saat Status Pending
- **Pre-condition:** Login dengan akun pending.
- **Steps:**
  1. Akses `/payment-proof`.
- **Expected Result:** Halaman bisa dibuka. User bisa upload payment proof.
- **Priority:** P1

---

### 4.8 Profile

#### FE-040 — Profile Menampilkan Data Benar
- **Pre-condition:** Login dengan akun aktif.
- **Steps:**
  1. Akses `/profile`.
  2. Periksa: nama, email, no. HP, institusi, status akun.
- **Expected Result:** Semua data sesuai dengan data saat registrasi.
- **Priority:** P1

#### FE-041 — Profile Loading State
- **Steps:**
  1. Buka `/profile`.
  2. Perhatikan saat data sedang loading.
- **Expected Result:** Muncul skeleton loading, bukan layar putih kosong.
- **Priority:** P2

---

### 4.9 Exam Instruction

#### FE-042 — Instruction Page Tampil Lengkap
- **Pre-condition:** Login dengan akun aktif yang memiliki sesi ujian aktif.
- **Steps:**
  1. Dari dashboard, klik "Mulai Ujian".
  2. Perhatikan halaman `/exam/instruction`.
- **Expected Result:**
  - Tampil aturan ujian (durasi, jumlah soal, dsb).
  - Tombol "Mulai Ujian" atau "Saya Siap" tampil.
  - Tidak ada error.
- **Priority:** P0

#### FE-043 — Device Check di Instruction Page (Mobile/Tablet)
- **Pre-condition:** Akses dari HP atau tablet.
- **Steps:**
  1. Dari HP/tablet, buka `/exam/instruction`.
- **Expected Result:**
  - Tampil pesan: "Perangkat Tidak Didukung".
  - Tidak bisa lanjut ke ujian.
  - Icon smartphone + deskripsi penggunaan laptop/desktop.
- **Priority:** P0

#### FE-044 — Device Check di Instruction Page (Desktop Normal)
- **Pre-condition:** Akses dari laptop/desktop.
- **Steps:**
  1. Buka `/exam/instruction` dari laptop.
- **Expected Result:** Halaman instruksi tampil normal. Bisa lanjut ke ujian.
- **Priority:** P0

#### FE-045 — Klik Mulai dari Instruction Memulai Exam
- **Steps:**
  1. Di instruction page, klik tombol mulai.
- **Expected Result:**
  - Request `POST /api/exam-attempts/start` atau similar.
  - Redirect ke `/exam` (halaman ujian).
  - Fullscreen mode aktif (atau diminta untuk fullscreen).
- **Priority:** P0

---

### 4.10 Exam Runtime (Halaman Ujian)

> **Peringatan:** Ini adalah fitur paling kritis. Uji dengan sangat hati-hati.

#### FE-046 — Soal Pertama Tampil Setelah Mulai Ujian
- **Pre-condition:** Ujian sudah dimulai, berada di `/exam`.
- **Steps:**
  1. Mulai ujian.
  2. Perhatikan halaman ujian.
- **Expected Result:**
  - Nomor soal 1 tampil.
  - Pertanyaan tampil (teks, gambar jika ada, audio jika ada).
  - Opsi jawaban (A, B, C, D, E) tampil.
  - Timer countdown aktif.
- **Priority:** P0

#### FE-047 — Timer Countdown Berjalan
- **Steps:**
  1. Saat ujian berlangsung, perhatikan timer di atas.
  2. Tunggu beberapa detik.
- **Expected Result:** Timer berkurang setiap detik. Format: `02:15:30` (jam:menit:detik).
- **Priority:** P0

#### FE-048 — Timer Tidak Reset saat Refresh Browser
- **Steps:**
  1. Catat waktu timer saat ini (misal: 01:45:00).
  2. Tekan `F5` atau refresh browser.
  3. Perhatikan timer setelah refresh.
- **Expected Result:** Timer melanjutkan dari waktu sebelumnya (tidak reset ke 2 jam). Ujian bisa dilanjutkan.
- **Priority:** P0

#### FE-049 — Pilih Jawaban dan Tanda "Tersimpan"
- **Steps:**
  1. Klik salah satu opsi jawaban (misal: A).
  2. Perhatikan indikator status save.
- **Expected Result:**
  - Ada indikator "Menyimpan..." lalu "Tersimpan".
  - Jawaban yang dipilih ditandai (radio button terisi).
- **Priority:** P0

#### FE-050 — Navigasi Soal — Next & Previous
- **Steps:**
  1. Klik tombol "Soal Selanjutnya".
  2. Perhatikan soal berikutnya tampil.
  3. Klik tombol "Soal Sebelumnya".
- **Expected Result:** Soal berpindah dengan benar. Nomor soal di navigator update.
- **Priority:** P0

#### FE-051 — Navigasi Soal — Klik Nomor di Navigator
- **Steps:**
  1. Di panel navigator (biasanya kiri/atas), klik nomor soal 5.
- **Expected Result:** Langsung lompat ke soal nomor 5.
- **Priority:** P0

#### FE-052 — Tandai Soal Ragu-Ragu (Mark Doubtful)
- **Steps:**
  1. Di soal tertentu, klik tombol "Tandai Ragu-Ragu" / checkbox.
  2. Perhatikan nomor soal di navigator.
- **Expected Result:**
  - Nomor soal di navigator berubah warna/indikator (misal: kuning).
  - Status ragu-ragu tersimpan.
- **Priority:** P1

#### FE-053 — Audio Player — Play Audio
- **Pre-condition:** Soal memiliki audio.
- **Steps:**
  1. Klik tombol play audio.
  2. Perhatikan audio berjalan.
- **Expected Result:** Audio berputar. Ada indikator durasi/play count.
- **Priority:** P1

#### FE-054 — Audio Player — Play Count Limit
- **Pre-condition:** Soal memiliki audio dengan batas play (misal: max 2x).
- **Steps:**
  1. Play audio sampai habis.
  2. Coba play lagi sampai melebihi batas.
- **Expected Result:**
  - Setelah melebihi batas, tombol play disabled.
  - Pesan: "Batas pemutaran audio tercapai."
- **Priority:** P1

#### FE-055 — Submit Ujian Manual
- **Pre-condition:** Sudah menjawab beberapa soal.
- **Steps:**
  1. Klik tombol "Selesai Ujian" / "Submit".
  2. Konfirmasi di modal: klik "Ya, Submit".
- **Expected Result:**
  - Request submit ke backend.
  - Redirect ke halaman hasil (atau dashboard, tergantung konfigurasi).
  - Fullscreen mode keluar.
- **Priority:** P0

#### FE-056 — Auto-Submit saat Timer Habis
- **Pre-condition:** Ujian sedang berlangsung.
- **Steps:**
  1. Tunggu sampai timer mencapai 00:00 (atau minta developer set timer pendek untuk test).
- **Expected Result:**
  - Ujian otomatis submit.
  - Redirect ke halaman hasil/dashboard.
  - Pesan: "Waktu ujian telah habis. Jawaban otomatis disimpan."
- **Priority:** P0

#### FE-057 — Progress Soal di Navigator
- **Steps:**
  1. Jawab beberapa soal.
  2. Perhatikan panel navigator.
- **Expected Result:**
  - Soal yang sudah dijawab: ditandai (warna hijau / checkmark).
  - Soal yang ragu-ragu: ditandai (warna kuning).
  - Soal yang belum dijawab: ditandai (warna abu-abu / putih).
- **Priority:** P1

---

### 4.11 Anti-Cheat

> **Anti-cheat adalah fitur keamanan ujian. QA harus mencoba "mengelabui" sistem.**

#### FE-058 — Fullscreen Enforcement
- **Steps:**
  1. Mulai ujian.
  2. Coba tekan `Esc` untuk keluar fullscreen.
  3. Coba klik tombol minimize browser.
- **Expected Result:**
  - Keluar fullscreen terdeteksi.
  - Muncul warning modal: "Anda keluar dari mode fullscreen."
  - Setelah beberapa kali pelanggaran, ujian auto-submit.
- **Priority:** P0

#### FE-059 — Tab Switch Detection
- **Steps:**
  1. Mulai ujian (fullscreen).
  2. Tekan `Alt+Tab` atau klik tab browser lain.
  3. Kembali ke tab ujian.
- **Expected Result:**
  - Muncul warning: "Anda beralih ke tab lain."
  - Pelanggaran tercatat.
  - Setelah threshold, auto-submit.
- **Priority:** P0

#### FE-060 — Right-Click Disabled
- **Steps:**
  1. Saat ujian berlangsung, klik kanan di mana saja di halaman.
- **Expected Result:**
  - Context menu (menu klik kanan) tidak muncul.
  - Atau muncul pesan: "Right click tidak diizinkan selama ujian."
- **Priority:** P1

#### FE-061 — Copy/Paste/Cut Disabled
- **Steps:**
  1. Saat ujian berlangsung, tekan `Ctrl+C`, `Ctrl+V`, `Ctrl+X`.
- **Expected Result:**
  - Tidak ada aksi copy/paste.
  - Atau muncul warning pelanggaran.
- **Priority:** P1

#### FE-062 — Refresh Browser saat Ujian
- **Steps:**
  1. Saat ujian berlangsung, tekan `F5`.
- **Expected Result:**
  - Muncul konfirmasi: "Yakin ingin meninggalkan halaman? Jawaban mungkin tidak tersimpan."
  - Kalau dilanjutkan, ujian resume dari progress terakhir.
- **Priority:** P0

#### FE-063 — Close Browser saat Ujian
- **Steps:**
  1. Saat ujian berlangsung, klik tombol close (X) browser.
- **Expected Result:**
  - Muncul konfirmasi: "Yakin ingin meninggalkan halaman?"
  - Kalau ditutup, attempt tetap aktif di backend (bisa dilanjutkan nanti).
- **Priority:** P0

#### FE-064 — Double Login — Tidak Bisa Login di 2 Browser Bersamaan
- **Pre-condition:** User sedang login & ujian di Browser A.
- **Steps:**
  1. Buka Browser B (beda browser: misal Chrome vs Edge).
  2. Login dengan akun yang sama.
- **Expected Result:**
  - Di Browser A, token invalid / dipaksa logout.
  - Atau di Browser B, tidak bisa login dengan pesan: "Akun sedang aktif di perangkat lain."
- **Priority:** P0

---

### 4.12 Exam Result

#### FE-065 — Halaman Hasil Ujian Tampil
- **Pre-condition:** User sudah menyelesaikan ujian.
- **Steps:**
  1. Setelah submit, perhatikan halaman hasil.
  2. Kalau redirect ke dashboard, cari bagian history hasil.
- **Expected Result:**
  - Total skor tampil.
  - Jumlah benar, salah, kosong tampil.
  - Tidak ada error.
- **Priority:** P0

#### FE-066 — History Hasil Ujian
- **Steps:**
  1. Akses halaman history hasil.
  2. Perhatikan list ujian yang pernah diikuti.
- **Expected Result:**
  - Setiap attempt menunjukkan: nama sesi, tanggal, skor, status.
  - Bisa klik untuk detail.
- **Priority:** P1

---

### 4.13 Mobile Responsiveness & Device Check

#### FE-067 — Landing Page di Mobile
- **Pre-condition:** Buka di HP atau DevTools mobile view.
- **Steps:**
  1. Buka `/` di HP.
  2. Scroll, periksa tombol, teks.
- **Expected Result:** Layout responsive. Teks terbaca. Tombol bisa diklik.
- **Priority:** P1

#### FE-068 — Login Page di Mobile
- **Steps:**
  1. Buka `/login` di HP.
  2. Isi form, klik login.
- **Expected Result:** Form bisa diisi. Tombol login bisa diklik. Redirect berfungsi.
- **Priority:** P1

#### FE-069 — Dashboard di Mobile
- **Steps:**
  1. Login di HP.
  2. Perhatikan dashboard.
- **Expected Result:** Dashboard tampil. Data terbaca. Tidak ada horizontal scroll.
- **Priority:** P1

#### FE-070 — Exam Page Diblokir di Mobile
- **Steps:**
  1. Coba akses `/exam` atau `/exam/instruction` dari HP.
- **Expected Result:**
  - Tampil pesan: "Perangkat Tidak Didukung".
  - Icon smartphone + instruksi pakai laptop.
  - Tidak bisa masuk ke halaman ujian.
- **Priority:** P0

#### FE-071 — Exam Page Diblokir saat Resize Browser Kecil
- **Pre-condition:** Desktop browser.
- **Steps:**
  1. Buka `/exam`.
  2. Resize browser window sampai lebar < 1024px.
- **Expected Result:** Halaman berubah jadi pesan "Perangkat Tidak Didukung".
- **Priority:** P1

---

### 4.14 Logout

#### FE-072 — Logout Berhasil
- **Steps:**
  1. Login dengan akun aktif.
  2. Klik tombol logout (biasanya di navbar atau menu profil).
  3. Konfirmasi logout (jika ada modal).
- **Expected Result:**
  - Token dihapus dari Local Storage & Cookie.
  - Redirect ke `/login` atau `/`.
  - Kalau kembali ke `/dashboard`, redirect ke login.
- **Priority:** P0

---

## 5. CBT-Admin — Test Cases Detail

> **URL:** https://admin-cat.pkumionline.cloud/  
> **Catatan:** Admin panel menggunakan teknologi terbaru (Next.js 16, React 19, Tailwind 4).

---

### 5.1 Login Admin

#### AD-001 — Login dengan Kredensial Admin Valid
- **Pre-condition:** Punya akun admin yang valid.
- **Steps:**
  1. Buka https://admin-cat.pkumionline.cloud/
  2. Isi email & password admin.
  3. Klik "Masuk".
- **Expected Result:**
  - Redirect ke `/admin/dashboard`.
  - Token tersimpan di cookie.
  - Tidak ada error di Console.
- **Priority:** P0

#### AD-002 — Login dengan Password Salah
- **Steps:**
  1. Isi email admin yang benar.
  2. Isi password salah.
  3. Klik "Masuk".
- **Expected Result:** Pesan error: "Email atau password salah."
- **Priority:** P0

#### AD-003 — Login dengan Email Tidak Terdaftar
- **Steps:**
  1. Isi email yang tidak ada di database.
  2. Klik "Masuk".
- **Expected Result:** Pesan error: "Email atau password salah."
- **Priority:** P1

#### AD-004 — Session Expired — Redirect ke Login
- **Pre-condition:** Login sebagai admin. Biarkan beberapa waktu (atau hapus cookie token).
- **Steps:**
  1. Setelah token expired/invalid, refresh halaman atau klik menu.
- **Expected Result:** Redirect ke `/login` dengan pesan: "Sesi Anda telah berakhir. Silakan login kembali."
- **Priority:** P0

#### AD-005 — Form Validasi — Field Kosong
- **Steps:**
  1. Biarkan email & password kosong.
  2. Klik "Masuk".
- **Expected Result:** Error validasi muncul di field yang kosong.
- **Priority:** P1

---

### 5.2 Dashboard Admin

#### AD-006 — Dashboard Statistik Tampil
- **Pre-condition:** Login sebagai admin.
- **Steps:**
  1. Buka `/admin/dashboard`.
  2. Perhatikan:
     - Statistik card (jumlah peserta, pending accounts, pending payments, dll.)
     - Chart/grafik (kalau ada data)
     - Notifikasi pending items
- **Expected Result:** Semua data tampil. Tidak ada "undefined" atau "NaN".
- **Priority:** P0

#### AD-007 — "Review Akun Baru" Button Mengarah ke User List Filtered
- **Steps:**
  1. Di dashboard, klik tombol "Review Akun Baru".
- **Expected Result:**
  - Redirect ke `/admin/users?account_status=pending_verification`.
  - Tabel user ter-filter hanya menunjukkan akun pending.
- **Priority:** P1

#### AD-008 — Dashboard Pending Items Tampil
- **Steps:**
  1. Pastikan ada akun pending atau payment proof pending.
  2. Buka dashboard.
- **Expected Result:**
  - Muncul item "Akun menunggu aktivasi" dengan jumlah.
  - Muncul item payment proof menunggu review.
  - Klik item mengarah ke halaman terkait.
- **Priority:** P1

#### AD-009 — Dashboard Chart Tampil (kalau ada data)
- **Pre-condition:** Ada data ujian/hasil ujian.
- **Steps:**
  1. Di dashboard, perhatikan area chart.
- **Expected Result:** Chart bar/line tampil dengan data yang benar. Tidak error.
- **Priority:** P2

---

### 5.3 User Management

#### AD-010 — List User Tampil
- **Steps:**
  1. Buka `/admin/users`.
- **Expected Result:**
  - Tabel menampilkan: nama, email, status, role, tanggal daftar.
  - Pagination berfungsi.
  - Search/filter berfungsi.
- **Priority:** P0

#### AD-011 — Filter User by Status
- **Steps:**
  1. Di tabel user, pilih filter "Status" = "Pending" / `pending_verification`.
- **Expected Result:** Tabel hanya menunjukkan user dengan status pending.
- **Priority:** P1

#### AD-012 — Detail User Tampil
- **Steps:**
  1. Klik salah satu row user.
- **Expected Result:**
  - Halaman detail user terbuka.
  - Tampil: profil, status akun, payment proof terakhir (jika ada).
- **Priority:** P0

#### AD-013 — Approve User via Payment Proof Approval (Unified Flow)
- **Pre-condition:** Ada user dengan status `pending_verification` dan payment proof pending.
- **Steps:**
  1. Buka detail user.
  2. Lihat section payment proof.
  3. Klik "Approve" / "Setuju".
  4. Isi konfirmasi (jika ada).
- **Expected Result:**
  - Payment proof status berubah jadi "Approved".
  - User account status berubah jadi "Active".
  - Test approval otomatis dibuat.
  - Toast sukses muncul.
- **Priority:** P0

#### AD-014 — Reject User via Payment Proof Rejection (Unified Flow)
- **Pre-condition:** Ada user dengan payment proof pending.
- **Steps:**
  1. Buka detail user.
  2. Lihat section payment proof.
  3. Klik "Reject" / "Tolak".
  4. Isi alasan penolakan.
  5. Konfirmasi.
- **Expected Result:**
  - Payment proof status berubah jadi "Rejected".
  - User account status berubah jadi "Rejected".
  - Toast sukses muncul.
- **Priority:** P0

#### AD-015 — Reset Password User
- **Steps:**
  1. Buka detail user.
  2. Klik "Reset Password".
  3. Isi password baru.
  4. Konfirmasi.
- **Expected Result:** Password user berubah. User bisa login dengan password baru.
- **Priority:** P1

---

### 5.4 Payment Proof Review

#### AD-016 — List Payment Proof Tampil
- **Steps:**
  1. Buka menu Payment Proof (biasanya di sidebar).
- **Expected Result:**
  - Tabel list payment proof.
  - Kolom: nama user, nama file, status, tanggal upload.
  - Filter by status (pending, approved, rejected).
- **Priority:** P0

#### AD-017 — Preview Payment Proof (Gambar)
- **Pre-condition:** Ada payment proof berupa gambar.
- **Steps:**
  1. Klik salah satu payment proof.
  2. Perhatikan preview file.
- **Expected Result:** Gambar tampil di preview. Bisa zoom/dilihat jelas.
- **Priority:** P1

#### AD-018 — Preview Payment Proof (PDF)
- **Pre-condition:** Ada payment proof berupa PDF.
- **Steps:**
  1. Klik payment proof PDF.
- **Expected Result:** PDF bisa dibaca (embedded viewer atau download).
- **Priority:** P1

#### AD-019 — Approve Payment Proof dari List
- **Steps:**
  1. Di list payment proof, klik "Approve" pada item pending.
- **Expected Result:**
  - Status berubah jadi approved.
  - User auto-aktif.
  - Item hilang dari list pending (atau berubah warna).
- **Priority:** P0

#### AD-020 — Reject Payment Proof dari List
- **Steps:**
  1. Di list payment proof, klik "Reject".
  2. Isi alasan penolakan.
- **Expected Result:**
  - Status berubah jadi rejected.
  - User auto-rejected.
- **Priority:** P0

---

### 5.5 Question Bank (Bank Soal)

#### AD-021 — List Question Bank Tampil
- **Steps:**
  1. Buka menu "Bank Soal".
- **Expected Result:**
  - Tabel list bank soal dengan kode (`QB-XXXXXX`), nama, jumlah soal, versi.
  - Pagination & search berfungsi.
- **Priority:** P0

#### AD-022 — Create Question Bank
- **Steps:**
  1. Klik "Tambah Bank Soal".
  2. Isi nama bank soal.
  3. Klik "Simpan".
- **Expected Result:**
  - Bank soal tersimpan.
  - Kode auto-generate (format `QB-XXXXXX`).
  - Redirect ke list dengan toast sukses.
- **Priority:** P0

#### AD-023 — Edit Question Bank
- **Steps:**
  1. Klik "Edit" pada salah satu bank soal.
  2. Ubah nama.
  3. Klik "Simpan".
- **Expected Result:** Data berubah. Versi bank soal bertambah.
- **Priority:** P1

#### AD-024 — Delete Question Bank
- **Steps:**
  1. Klik "Hapus" pada bank soal.
  2. Konfirmasi di modal.
- **Expected Result:**
  - Bank soal terhapus (atau soft-delete).
  - Kalau bank soal dipakai di package aktif, muncul warning.
- **Priority:** P1

---

### 5.6 Question (Soal Individual)

#### AD-025 — List Soal dalam Bank Tampil
- **Steps:**
  1. Buka detail bank soal.
  2. Lihat tab/list soal.
- **Expected Result:**
  - List soal tampil dengan nomor, pertanyaan (snippet), jumlah opsi.
- **Priority:** P0

#### AD-026 — Create Soal Baru
- **Steps:**
  1. Klik "Tambah Soal".
  2. Isi:
     - Pertanyaan (gunakan rich text editor TipTap — bold, italic, dll.)
     - Upload gambar (opsional, max 2MB)
     - Upload audio (opsional, max 2MB)
     - Opsi jawaban (minimal 2, maksimal 5)
     - Pilih 1 opsi yang benar
  3. Klik "Simpan".
- **Expected Result:**
  - Soal tersimpan.
  - Gambar & audio tersimpan.
  - Validasi: harus ada tepat 1 jawaban benar.
- **Priority:** P0

#### AD-027 — Edit Soal
- **Steps:**
  1. Klik "Edit" pada soal.
  2. Ubah pertanyaan atau opsi.
  3. Simpan.
- **Expected Result:** Perubahan tersimpan.
- **Priority:** P1

#### AD-028 — Delete Soal
- **Steps:**
  1. Klik "Hapus" pada soal.
  2. Konfirmasi.
- **Expected Result:** Soal terhapus.
- **Priority:** P1

#### AD-029 — Rich Text Editor TipTap Berfungsi
- **Steps:**
  1. Saat create/edit soal, coba format teks:
     - Bold, italic, underline
     - Heading
     - Align left/center/right
     - Bullet list, numbered list
     - Quote, code block
  2. Simpan, lalu lihat preview.
- **Expected Result:** Format teks tersimpan dan tampil dengan benar.
- **Priority:** P1

#### AD-030 — Upload Gambar pada Soal
- **Steps:**
  1. Saat create soal, upload gambar JPG/PNG (< 2MB).
  2. Simpan.
  3. Lihat preview soal.
- **Expected Result:** Gambar tampil di soal. Tidak error.
- **Priority:** P1

#### AD-031 — Upload Audio pada Soal
- **Steps:**
  1. Saat create soal, upload audio MP3/WAV (< 2MB).
  2. Simpan.
  3. Lihat preview soal.
- **Expected Result:** Audio player muncul. Bisa play.
- **Priority:** P1

#### AD-032 — Import Soal dari CSV
- **Steps:**
  1. Buka menu import soal.
  2. Download template CSV.
  3. Isi template dengan beberapa soal.
  4. Upload CSV.
  5. Konfirmasi import.
- **Expected Result:**
  - Soal dari CSV terimport dengan benar.
  - Kalau ada error di CSV, muncul pesan error dengan nomor baris.
- **Priority:** P1

#### AD-033 — Bulk Create Soal (Google Forms Style)
- **Steps:**
  1. Buka `/admin/questions/batch/` (Bulk Question Creator).
  2. Isi beberapa soal sekaligus dalam satu form.
  3. Klik "Simpan Semua".
- **Expected Result:** Semua soal tersimpan sekaligus.
- **Priority:** P1

---

### 5.7 Exam Package (Paket Ujian)

#### AD-034 — List Exam Package Tampil
- **Steps:**
  1. Buka menu "Paket Ujian".
- **Expected Result:**
  - Tabel list paket dengan kode (`PKG-XXXXXX`), nama, jumlah bank soal.
- **Priority:** P0

#### AD-035 — Create Exam Package
- **Steps:**
  1. Klik "Tambah Paket".
  2. Isi nama, deskripsi.
  3. Mapping bank soal:
     - Pilih bank soal.
     - Tentukan jumlah soal yang diambil dari bank tersebut.
     - Validasi: jumlah soal ≤ stok soal di bank.
  4. Klik "Simpan".
- **Expected Result:**
  - Paket tersimpan dengan kode auto-generate.
  - Kalau jumlah soal melebihi stok, error validasi.
- **Priority:** P0

#### AD-036 — Edit Exam Package
- **Steps:**
  1. Klik "Edit" pada paket.
  2. Ubah mapping bank soal.
  3. Simpan.
- **Expected Result:** Perubahan tersimpan.
- **Priority:** P1

#### AD-037 — Delete Exam Package
- **Steps:**
  1. Klik "Hapus".
  2. Konfirmasi.
- **Expected Result:** Paket terhapus (atau soft-delete).
- **Priority:** P1

---

### 5.8 Exam Session (Sesi Ujian)

#### AD-038 — List Exam Session Tampil
- **Steps:**
  1. Buka menu "Sesi Ujian".
- **Expected Result:**
  - Tabel list sesi dengan kode (`SES-XXXXXX`), nama paket, status, tanggal, kuota.
- **Priority:** P0

#### AD-039 — Create Exam Session
- **Steps:**
  1. Klik "Tambah Sesi".
  2. Pilih paket ujian.
  3. Isi:
     - Nama sesi
     - Tanggal & waktu mulai
     - Tanggal & waktu selesai
     - Durasi ujian
     - Kuota peserta
     - Pengaturan lain (show_result_to_user, dll.)
  4. Klik "Simpan".
- **Expected Result:**
  - Sesi tersimpan dengan status `draft`.
  - Kode auto-generate `SES-XXXXXX`.
- **Priority:** P0

#### AD-040 — Publish Exam Session
- **Pre-condition:** Sesi dengan status `draft`.
- **Steps:**
  1. Buka detail sesi.
  2. Klik "Publish".
  3. Konfirmasi.
- **Expected Result:**
  - Status sesi berubah jadi `published`.
  - Peserta bisa melihat & mulai ujian.
- **Priority:** P0

#### AD-041 — Close Exam Session
- **Pre-condition:** Sesi dengan status `published`.
- **Steps:**
  1. Buka detail sesi.
  2. Klik "Close".
  3. Konfirmasi.
- **Expected Result:**
  - Status sesi berubah jadi `closed`.
  - Peserta tidak bisa lagi mulai ujian baru (yang sedang ujian tetap bisa lanjut).
- **Priority:** P0

#### AD-042 — Finish Exam Session
- **Pre-condition:** Sesi dengan status `closed` atau `published`.
- **Steps:**
  1. Klik "Finish".
  2. Konfirmasi.
- **Expected Result:**
  - Status sesi berubah jadi `finished`.
  - Semua attempt aktif auto-submit.
  - Hasil bisa di-export.
- **Priority:** P0

#### AD-043 — Cancel Exam Session
- **Steps:**
  1. Klik "Cancel".
  2. Konfirmasi.
- **Expected Result:**
  - Status sesi berubah jadi `cancelled`.
  - Peserta tidak bisa ujian.
- **Priority:** P1

#### AD-044 — Status Transition Invalid Ditolak
- **Pre-condition:** Sesi dengan status `finished`.
- **Steps:**
  1. Coba klik "Publish" pada sesi yang sudah `finished`.
- **Expected Result:** Error: "Tidak bisa publish dari status finished."
- **Priority:** P1

---

### 5.9 Participant Management

#### AD-045 — Assign Peserta Manual ke Sesi
- **Pre-condition:** Sesi sudah `published`. Ada user aktif.
- **Steps:**
  1. Buka detail sesi → tab "Peserta".
  2. Klik "Tambah Peserta".
  3. Pilih user dari list.
  4. Klik "Simpan".
- **Expected Result:**
  - User muncul di list peserta sesi.
  - User bisa melihat sesi di dashboard.
- **Priority:** P0

#### AD-046 — Auto-Generate Peserta
- **Steps:**
  1. Di tab peserta, klik "Generate Otomatis".
  2. Tentukan jumlah/kriteria.
- **Expected Result:** Peserta ter-assign secara otomatis berdasarkan kriteria.
- **Priority:** P1

#### AD-047 — Remove Peserta dari Sesi
- **Steps:**
  1. Di list peserta, klik "Hapus" pada salah satu peserta.
  2. Konfirmasi di modal.
- **Expected Result:** Peserta terhapus dari sesi. User tidak lagi bisa akses sesi tersebut.
- **Priority:** P1

---

### 5.10 Live Monitoring

#### AD-048 — Live Monitoring Tampil
- **Pre-condition:** Ada peserta yang sedang ujian.
- **Steps:**
  1. Buka menu "Monitoring".
- **Expected Result:**
  - Tabel list peserta yang sedang ujian.
  - Kolom: nama, sesi, waktu mulai, progress, violations.
  - Auto-refresh setiap 30 detik.
- **Priority:** P0

#### AD-049 — Monitoring Detail — Violations
- **Steps:**
  1. Klik salah satu peserta di monitoring.
- **Expected Result:**
  - Detail attempt tampil: soal keberapa, waktu tersisa, jawaban.
  - List violations (tab switch, fullscreen exit, dll.) dengan severity color coding.
- **Priority:** P1

#### AD-050 — Monitoring Refresh Indicator
- **Steps:**
  1. Biarkan halaman monitoring terbuka.
  2. Perhatikan setelah 30 detik.
- **Expected Result:**
  - Muncul indikator "Refreshing...".
  - Data terupdate otomatis.
  - Timestamp "Sinkron terakhir" update.
- **Priority:** P2

---

### 5.11 Export Results

#### AD-051 — Export Hasil Ujian ke Excel
- **Pre-condition:** Ada sesi yang sudah `finished` dengan peserta yang sudah ujian.
- **Steps:**
  1. Buka detail sesi → tab "Hasil".
  2. Klik "Export Excel".
- **Expected Result:**
  - File Excel terdownload.
  - Isi file: nama peserta, skor, benar, salah, kosong, waktu submit.
- **Priority:** P0

#### AD-052 — Export Hasil Ujian ke CSV
- **Steps:**
  1. Klik "Export CSV".
- **Expected Result:** File CSV terdownload dengan format yang benar.
- **Priority:** P1

---

### 5.12 Analytics

#### AD-053 — Analytics Dashboard Tampil
- **Pre-condition:** Ada data hasil ujian.
- **Steps:**
  1. Buka menu "Analytics".
- **Expected Result:**
  - Chart skor rata-rata per sesi/paket tampil.
  - Data statistik tampil tanpa error.
- **Priority:** P1

---

### 5.13 Activity Logs

#### AD-054 — Activity Logs Tampil
- **Steps:**
  1. Buka menu "Activity Logs".
- **Expected Result:**
  - List log aktivitas admin.
  - Kolom: admin, aksi, waktu, entitas yang diubah.
  - Filter & search berfungsi.
- **Priority:** P1

---

### 5.14 Global Settings

#### AD-055 — Global Settings Tampil & Bisa Diedit
- **Steps:**
  1. Buka menu "Pengaturan" / "Settings".
  2. Ubah beberapa pengaturan (misal: durasi default, passing score).
  3. Klik "Simpan".
- **Expected Result:**
  - Pengaturan tersimpan.
  - Saat create package/session, field otomatis terisi dari global settings.
- **Priority:** P1

---

### 5.15 UI/UX & Responsive Admin Panel

#### AD-056 — Sidebar Navigation Berfungsi
- **Steps:**
  1. Klik setiap menu di sidebar.
- **Expected Result:** Navigasi ke halaman yang benar. Menu aktif ditandai.
- **Priority:** P1

#### AD-057 — Breadcrumb Navigation
- **Steps:**
  1. Buka halaman detail (user detail, sesi detail, dll.).
- **Expected Result:** Breadcrumb tampil di atas: `Dashboard > Users > Nama User`.
- **Priority:** P2

#### AD-058 — Loading State Tombol
- **Steps:**
  1. Klik tombol submit/simpan.
  2. Perhatikan saat request sedang berlangsung.
- **Expected Result:** Tombol menunjukkan spinner/loading state. Tidak bisa diklik dua kali.
- **Priority:** P1

#### AD-059 — Toast Notification
- **Steps:**
  1. Lakukan aksi sukses (simpan, hapus, approve).
- **Expected Result:** Toast notifikasi muncul di pojok kanan atas. Ada warna sukses (hijau), error (merah), warning (kuning).
- **Priority:** P1

#### AD-060 — Form Error Mapping (422 Validation)
- **Steps:**
  1. Submit form dengan data invalid (misal: field kosong yang required).
- **Expected Result:**
  - Error muncul di field yang bersangkutan (bukan cuma toast umum).
  - Pesan error jelas dalam Bahasa Indonesia.
- **Priority:** P1

#### AD-061 — Card View di Mobile / Table View di Desktop
- **Pre-condition:** Buka di HP atau resize browser kecil.
- **Steps:**
  1. Buka list user atau list sesi di HP.
- **Expected Result:**
  - Di desktop: tampil tabel (rows & columns).
  - Di mobile: tampil card view (stacked information).
- **Priority:** P2

---

## 6. End-to-End Flow Testing

> **End-to-End (E2E) Testing** adalah menguji keseluruhan alur dari awal sampai akhir, seolah-olah user nyata menggunakan aplikasi.

### Flow 1: Register → Payment → Exam → Result (Happy Path)

| Step | Aksi di FE-CBT | Verifikasi di CBT-Admin |
|---|---|---|
| 1 | Register akun baru + upload payment proof | — |
| 2 | Redirect ke `/waiting-approval` | — |
| 3 | — | Login admin, cek dashboard → ada notifikasi akun baru |
| 4 | — | Buka `/admin/users?account_status=pending_verification` |
| 5 | — | Klik user, lihat payment proof, klik **Approve** |
| 6 | Di `/waiting-approval`, tunggu polling (~30 detik) | — |
| 7 | Auto-redirect ke `/dashboard` | — |
| 8 | Di dashboard, lihat sesi ujian → klik "Mulai Ujian" | — |
| 9 | Di `/exam/instruction`, klik mulai → masuk fullscreen | — |
| 10 | Kerjakan ujian: pilih jawaban, navigasi soal | — |
| 11 | Submit ujian | — |
| 12 | Lihat hasil ujian | — |
| 13 | — | Buka monitoring → tidak ada peserta aktif |
| 14 | — | Buka hasil sesi → skor peserta tampil |

### Flow 2: Register → Rejected

| Step | Aksi | Expected Result |
|---|---|---|
| 1 | Register akun baru + upload payment proof | Redirect waiting-approval |
| 2 | Admin reject payment proof + isi alasan | — |
| 3 | User di `/waiting-approval` melihat status berubah | Status jadi "Ditolak", alasan tampil |

### Flow 3: Resume Ujian (Browser Ditutup di Tengah Ujian)

| Step | Aksi | Expected Result |
|---|---|---|
| 1 | Mulai ujian, kerjakan beberapa soal | Soal tersimpan |
| 2 | Tutup browser (tanpa submit) | — |
| 3 | Buka kembali, login | Dashboard tampil |
| 4 | Klik "Lanjutkan Ujian" | Masuk ke soal terakhir yang dikerjakan |
| 5 | Jawaban sebelumnya masih ada | Jawaban tersimpan |

### Flow 4: Anti-Cheat — Tab Switch

| Step | Aksi | Expected Result |
|---|---|---|
| 1 | Mulai ujian | Fullscreen aktif |
| 2 | Tekan `Alt+Tab` | Warning modal muncul |
| 3 | Ulangi 3x | Auto-submit ujian |

---

## 7. Cross-Browser & Device Testing

### 7.1 Browser Matrix

Uji setidaknya di 2 browser:

| Browser | Versi Terbaru | Device |
|---|---|---|
| Google Chrome | Latest | Desktop (Windows/Mac) |
| Microsoft Edge | Latest | Desktop (Windows/Mac) |
| Google Chrome Mobile | Latest | Android |
| Safari Mobile | Latest | iPhone (jika tersedia) |

### 7.2 Device Matrix

| Device | Resolusi | Halaman yang Diuji |
|---|---|---|
| Desktop (1920×1080) | Full HD | Semua halaman |
| Laptop (1366×768) | HD | Semua halaman |
| Tablet (768×1024) | iPad | Landing, Login, Dashboard |
| Mobile (375×667) | iPhone SE | Landing, Login, Dashboard, Register |

### 7.3 Cara Simulasi Mobile di Desktop

1. Buka Chrome/Edge.
2. Tekan `F12` untuk buka DevTools.
3. Klik icon **Toggle Device Toolbar** (📱) di pojok kiri atas DevTools.
4. Pilih device dari dropdown (misal: iPhone 12, iPad).
5. Refresh halaman (`F5`).

---

## 8. Cara Melaporkan Bug (Bug Report)

### 8.1 Format Bug Report yang Baik

Setiap bug HARUS mencakup informasi berikut:

```
🐛 Bug Report

ID: BUG-001
Module: Register
Priority: P0 (Critical) / P1 (High) / P2 (Medium) / P3 (Low)
Title: [Singkat dan jelas] Contoh: "Tombol Daftar tidak berfungsi saat upload file >5MB"

Description:
Deskripsikan bug secara detail. Apa yang terjadi? Apa yang seharusnya terjadi?

Steps to Reproduce (Langkah Reproduksi):
1. Buka halaman /register
2. Isi semua field dengan data valid
3. Upload file PDF dengan ukuran 6MB
4. Klik tombol "Daftar"

Expected Result:
Muncul error validasi: "Ukuran file maksimal 5MB"

Actual Result:
Tombol "Daftar" berputar (loading) terus-menerus tanpa ada respons. Tidak ada error message.

Environment:
- Browser: Google Chrome 124.0
- OS: Windows 11
- Device: Laptop Dell XPS 15
- URL: https://cat.pkumionline.cloud/register
- Waktu: 14 Mei 2026, 10:30 WIB

Screenshot/Video:
[Lampirkan screenshot atau screen recording]

Console Error (F12 → Console):
[Copy-paste error merah dari console]

Network Error (F12 → Network):
[Copy-paste request yang failed dengan status merah]
```

### 8.2 Prioritas Bug

| Prioritas | Definisi | Contoh |
|---|---|---|
| **P0 — Critical** | Aplikasi crash, data hilang, fitur utama tidak bisa digunakan | Tidak bisa login, ujian tidak bisa dimulai, submit ujian error |
| **P1 — High** | Fitur penting tidak berfungsi dengan benar | Register tanpa payment proof bisa, timer tidak berkurang |
| **P2 — Medium** | Fitur minor bermasalah atau UX tidak nyaman | Toast tidak muncul, layout sedikit berantakan |
| **P3 — Low** | Typo, warna tidak sesuai, enhancement | Teks "Pembayran" (typo), margin tidak konsisten |

### 8.3 Tips Dokumentasi Bug

- **Screenshot selalu** — Gunakan `Win + Shift + S` (Windows) atau `Cmd + Shift + 4` (Mac).
- **Screen recording untuk bug kompleks** — Gunakan built-in screen recorder.
- **Sertakan Console & Network error** — Ini sangat membantu developer.
- **Reproducible?** — Tulis apakah bug selalu terjadi atau hanya kadang-kadang.
- **One bug per report** — Jangan gabungkan 2 bug berbeda dalam 1 laporan.

---

## 9. Checklist Harian QA

Gunakan checklist ini setiap hari testing:

### Sesi Pagi (Smoke Test — 30 menit)

- [ ] FE-CBT landing page bisa dibuka
- [ ] FE-CBT login berfungsi
- [ ] FE-CBT register berfungsi
- [ ] CBT-Admin login berfungsi
- [ ] CBT-Admin dashboard load tanpa error
- [ ] Tidak ada error merah di Console (F12) saat load halaman utama

### Sesi Siang (Functional Testing)

- [ ] Jalankan test cases sesuai sprint/target hari
- [ ] Dokumentasikan setiap bug yang ditemukan
- [ ] Re-test bug yang sudah di-fix developer

### Sesi Sore (Regression & Report)

- [ ] Re-test fitur yang sudah pernah diuji (apakah masih berfungsi?)
- [ ] Update spreadsheet test case dengan status Pass/Fail
- [ ] Kirim laporan harian ke tim

---

## 10. FAQ & Troubleshooting

### Q: Saya menemukan error merah di Console (F12). Apa yang harus dilakukan?
**A:** Screenshot error-nya. Copy teks error-nya. Lampirkan di bug report. Error console biasanya penting untuk developer.

### Q: Saya tidak bisa login setelah register. Kenapa?
**A:** Kemungkinan:
1. Status akun masih `pending_verification` → normal, harus tunggu admin approve.
2. Password salah → coba reset password.
3. Akun di-reject → cek di `/waiting-approval`.

### Q: Bagaimana cara membuat akun test tanpa mengganggu data production?
**A:** Gunakan email dummy (mailinator.com, guerrillamail.com). Hapus akun test setelah selesai (tanyakan admin untuk hapus).

### Q: Ujian saya tiba-tiba submit sendiri saat saya tidak melakukan apa-apa. Kenapa?
**A:** Kemungkinan anti-cheat mendeteksi pelanggaran (tab switch, fullscreen exit, dll.). Ini **expected behavior**. Ulangi test dengan mematuhi aturan ujian.

### Q: Timer ujian tidak berjalan. Apa yang salah?
**A:**
1. Cek Console untuk error.
2. Cek Network untuk request `GET /api/me` atau heartbeat.
3. Refresh browser (seharusnya resume dari waktu terakhir).

### Q: File payment proof saya upload tapi tidak muncul di history. Kenapa?
**A:**
1. Tunggu beberapa detik (mungkin masih processing).
2. Refresh halaman.
3. Cek Console & Network untuk error upload.
4. Cek ukuran file (max 5MB) dan format (JPG/PNG/PDF).

### Q: Saya baru pertama kali pakai DevTools (F12). Apa yang harus saya perhatikan?
**A:**
- **Tab Console** → Lihat error merah. Kalau ada error merah saat bug terjadi, screenshot/copy.
- **Tab Network** → Lihat request API. Kalau ada yang warna merah (status 400, 401, 403, 500), screenshot.
- **Tab Elements** → Inspeksi elemen HTML (untuk bug tampilan).

### Q: Apa bedanya testing di HP asli vs DevTools mobile simulation?
**A:**
- **DevTools simulation** → Cukup untuk cek layout responsive. Tidak 100% akurat.
- **HP asli** → Lebih akurat untuk touch behavior, performance, dan real-world condition. Usahakan uji di HP asli minimal 1x.

### Q: Saya harus test berapa banyak kombinasi data?
**A:** Minimal:
- 1 happy path (data valid, semua berhasil)
- 1 negative path per validasi (email salah, password kurang, file terlalu besar, dll.)
- 1 edge case (data ekstrem: nama 255 karakter, file 4.99MB, dll.)

---

## Lampiran

### A. Kode Status HTTP yang Sering Muncul

| Kode | Arti | Apa yang Perlu Dilakukan QA? |
|---|---|---|
| 200 | OK | Request berhasil. |
| 201 | Created | Data berhasil dibuat (register, create soal). |
| 400 | Bad Request | Data yang dikirim invalid. Cek form input. |
| 401 | Unauthorized | Token tidak valid / expired. Harus login ulang. |
| 403 | Forbidden | Tidak punya izin. Contoh: user pending coba akses dashboard. |
| 404 | Not Found | Data tidak ditemukan. Contoh: akses soal yang sudah dihapus. |
| 422 | Validation Error | Validasi form gagal. Contoh: email sudah terdaftar. |
| 500 | Internal Server Error | Error di server. Ini selalu lapor sebagai bug. |

### B. Format Email Test yang Bisa Digunakan

Gunakan layanan email throwaway untuk testing:

- `qatest001@mailinator.com`
- `qatest002@mailinator.com`
- `qatest003@mailinator.com`

Cara cek inbox: buka https://www.mailinator.com/ dan masukkan nama email.

### C. File Test untuk Upload

Siapkan file berikut di komputer Anda:

| File | Ukuran | Tipe | Kegunaan |
|---|---|---|---|
| `valid-payment.jpg` | 500 KB | JPG | Upload payment proof valid |
| `valid-payment.png` | 1 MB | PNG | Upload payment proof valid |
| `valid-payment.pdf` | 2 MB | PDF | Upload payment proof valid |
| `oversized.jpg` | 6 MB | JPG | Test batas ukuran file |
| `invalid.exe` | 1 MB | EXE | Test format tidak didukung |
| `valid-question.jpg` | 500 KB | JPG | Upload gambar soal |
| `valid-audio.mp3` | 1 MB | MP3 | Upload audio soal |

---

> **Selamat Testing!** 🎯
>
> Ingat: Tugas QA bukan untuk membuktikan aplikasi bekerja, tapi untuk **menemukan cara agar aplikasi bisa rusak**. Semakin banyak bug yang ditemukan sebelum user nyata menggunakannya, semakin baik kualitas aplikasinya.
>
> Kalau ada pertanyaan, jangan ragu bertanya ke tim developer!
