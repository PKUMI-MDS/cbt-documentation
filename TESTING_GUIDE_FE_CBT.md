# 🧪 Testing Guide — FE-CBT (Participant Frontend)

Panduan lengkap untuk menguji fitur-fitur utama di frontend peserta CBT-TOAFL.

---

## 📋 Pre-requisite

Pastikan semua service berjalan:

```bash
# 1. BE-CBT (Backend)
cd /var/www/be-cbt
php artisan migrate        # Jalankan migration terbaru
php artisan serve          # atau via nginx/php-fpm

# 2. FE-CBT (Participant Frontend)
cd /var/www/fe-cbt
npm install
npm run build && npm start # Production mode
# atau: npm run dev         # Development mode
```

**Environment variables yang harus di-set:**
```bash
NEXT_PUBLIC_API_BASE_URL=https://be-cbt.miftadigital.cloud/api
```

---

## 🧪 Test Case 1: Registration Period — Ditutup

### Tujuan
Memastikan peserta tidak bisa mendaftar/login saat periode pendaftaran ditutup.

### Pre-Condition (via Admin Panel)
1. Login admin → menu **Settings**
2. Set periode registrasi:
   - **Tanggal Buka:** `2025-01-01 00:00` (masa lalu)
   - **Tanggal Tutup:** `2025-01-02 00:00` (masa lalu)
   - **Pesan saat Ditutup:** "Pendaftaran peserta ujian TOAFL periode 2026 telah ditutup."
3. Klik **Simpan**

### Langkah Manual — Halaman Register
1. Buka `https://fe-cbt-url/register`
2. **Verifikasi UI:**
   - [ ] Muncul **banner amber** di atas form dengan pesan: "Pendaftaran peserta ujian TOAFL periode 2026 telah ditutup."
   - [ ] Semua input form **disabled** (Nama, Email, WhatsApp, Institusi, Password, Konfirmasi Password)
   - [ ] Upload file bukti pembayaran **disabled**
   - [ ] Tombol **"Submit Registrasi" disabled**, text berubah jadi **"Pendaftaran Ditutup"**
   - [ ] Input tidak bisa difocus/diketik

### Langkah Manual — Halaman Login
1. Buka `https://fe-cbt-url/login`
2. **Verifikasi UI:**
   - [ ] Muncul **banner amber** dengan pesan yang sama
   - [ ] Input email dan password **disabled**
   - [ ] Checkbox "Remember me" **disabled**
   - [ ] Tombol **"Login" disabled**, text berubah jadi **"Pendaftaran Ditutup"**

### Langkah Manual — Coba Submit Paksa
1. Buka DevTools → Console
2. Jalankan:
   ```javascript
   document.querySelector('form').submit();
   ```
3. **Verifikasi:** Form tidak submit (handler di-guard dengan `if (!registrationStatus.is_open) return`)

### Verifikasi via API
```bash
# Cek status registrasi (tanpa auth)
curl -s https://be-cbt.miftadigital.cloud/api/settings/registration | jq .

# Expected:
# {
#   "data": {
#     "is_open": false,
#     "message": "Pendaftaran peserta ujian TOAFL periode 2026 telah ditutup.",
#     "open_date": "2025-01-01T00:00:00.000000Z",
#     "close_date": "2025-01-02T00:00:00.000000Z"
#   }
# }
```

---

## 🧪 Test Case 2: Registration Period — Terbuka (Registrasi Normal)

### Tujuan
Memastikan peserta bisa mendaftar dengan normal saat periode buka.

### Pre-Condition (via Admin Panel)
1. Login admin → menu **Settings**
2. Set periode registrasi:
   - **Tanggal Buka:** `2026-01-01 00:00` (masa lalu)
   - **Tanggal Tutup:** `2026-12-31 23:59` (masa depan)
3. Klik **Simpan**

### Langkah Manual
1. Buka `https://fe-cbt-url/register`
2. **Verifikasi:** Tidak ada banner amber, semua input aktif
3. Isi form registrasi:
   | Field | Value |
   |-------|-------|
   | Nama Lengkap | Test Peserta |
   | Email | testpeserta@example.com |
   | No. WhatsApp | 081234567890 |
   | Institusi | Universitas Test |
   | Password | password123 |
   | Konfirmasi Password | password123 |
4. Upload bukti pembayaran (file JPG/PNG/PDF, max 5MB)
5. Klik **"Submit Registrasi"**

### Verifikasi
- [ ] Loading state: tombol berubah jadi "Mengirim..."
- [ ] Setelah sukses, redirect ke `/waiting-approval?registered=1&email=testpeserta@example.com`
- [ ] Di halaman waiting approval, muncul pesan sukses registrasi

### Edge Cases

#### 2a. Email Sudah Terdaftar
1. Coba daftar lagi dengan email yang sama
2. **Verifikasi:** Error "Email sudah terdaftar", form tidak submit

#### 2b. Password < 8 Karakter
1. Isi password: "12345"
2. **Verifikasi:** Error validasi "Password minimal 8 karakter"

#### 2c. No. WhatsApp Tidak Valid
1. Isi phone: "123" atau "abc"
2. **Verifikasi:** Error "No. WhatsApp harus 11-13 digit angka"

#### 2d. File Terlalu Besar
1. Upload file > 5MB
2. **Verifikasi:** Error "Ukuran file maksimal 5MB"

#### 2e. Format File Salah
1. Upload file .txt atau .exe
2. **Verifikasi:** Error "Format file harus JPG, PNG, atau PDF"

---

## 🧪 Test Case 3: Login & Waiting Approval

### Tujuan
Memastikan flow login dan status akun ditangani dengan benar.

### Pre-Condition
Peserta sudah terdaftar (Test Case 2) tapi belum di-approve oleh admin.

### Langkah Manual — Login Akun Pending
1. Buka `https://fe-cbt-url/login`
2. Isi email dan password akun yang belum di-approve
3. Klik **Login**

### Verifikasi
- [ ] Redirect ke `/waiting-approval`
- [ ] Muncul pesan: "Akun Anda masih menunggu persetujuan admin"
- [ ] Tidak bisa akses `/dashboard` (akan redirect ke waiting-approval)

### Langkah Manual — Setelah Admin Approve
1. Login admin → menu **Participants**
2. Cari peserta "Test Peserta"
3. Klik **Approve**
4. Di frontend peserta, refresh halaman `/waiting-approval`

### Verifikasi
- [ ] Status berubah jadi "Akun Anda telah disetujui!"
- [ ] Muncul tombol "Masuk ke Dashboard"
- [ ] Klik tombol → redirect ke `/dashboard`

### Langkah Manual — Login Akun Rejected
1. Admin reject akun peserta
2. Peserta coba login
3. **Verifikasi:** Error "Akun Anda ditolak oleh admin"

### Verifikasi via API
```bash
# Login sebagai peserta
USER_TOKEN=$(curl -s -X POST https://be-cbt.miftadigital.cloud/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"testpeserta@example.com","password":"password123"}' \
  | jq -r '.data.token')

# Cek profile (untuk verifikasi status)
curl -s -H "Authorization: Bearer $USER_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/user/profile | jq '.data.account_status'
```

---

## 🧪 Test Case 4: Dashboard Peserta

### Tujuan
Memastikan dashboard menampilkan informasi ujian dengan benar.

### Pre-Condition
Peserta sudah di-approve dan ada exam session yang published.

### Langkah Manual
1. Login sebagai peserta yang sudah di-approve
2. **Verifikasi Dashboard:**
   - [ ] Nama peserta tampil di header/sidebar
   - [ ] Card/session ujian yang tersedia tampil
   - [ ] Nama ujian sesuai dengan exam settings di admin
   - [ ] Durasi ujian sesuai
   - [ ] Tanggal/waktu session sesuai
   - [ ] Status session (Aktif/Belum Mulai/Selesai)

### Verifikasi via API
```bash
curl -s -H "Authorization: Bearer $USER_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/dashboard | jq .
```

---

## 🧪 Test Case 5: Mulai Ujian & Exam Flow

### Tujuan
Memastikan peserta bisa mulai ujian, navigasi soal, dan submit.

### Pre-Condition
- Peserta di-approve
- Ada exam session published yang berjalan
- Peserta sudah di-assign ke session

### Langkah Manual
1. Di dashboard, klik session yang aktif
2. **Halaman Instruksi:**
   - [ ] Instruksi ujian tampil
   - [ ] Informasi durasi, jumlah soal, aturan tampil
   - [ ] Tombol **"Mulai Ujian"** aktif
3. Klik **"Mulai Ujian"**

#### Verifikasi Halaman Ujian
- [ ] Timer countdown tampil dan berjalan
- [ ] Nomor soal tampil
- [ ] Audio player muncul untuk soal listening (jika ada)
- [ ] Pilihan jawaban (A/B/C/D) tampil
- [ ] Navigation panel menampilkan semua nomor soal
- [ ] Tombol **"Next"**, **"Previous"**, **"Flag"** berfungsi

#### Verifikasi Navigasi Soal
- [ ] Klik nomor di navigation panel → lompat ke soal tersebut
- [ ] Jawab soal → nomor berubah warna (menandai sudah dijawab)
- [ ] Klik **"Flag"** → nomor ditandai dengan ikon bendera
- [ ] Tombol **"Next"** di soal terakhir berubah jadi **"Review"**

#### Verifikasi Submit
1. Klik **"Review"** atau navigasi ke soal terakhir lalu klik **"Submit"**
2. **Modal Konfirmasi:**
   - [ ] Muncul daftar soal yang belum dijawab (jika ada)
   - [ ] Peringatan "Apakah Anda yakin ingin submit?"
3. Klik **"Ya, Submit"**
4. **Verifikasi:**
   - [ ] Redirect ke `/exam/completed`
   - [ ] Tidak bisa kembali ke halaman ujian (browser back akan redirect)

---

## 🧪 Test Case 6: Hasil Ujian dengan Weighted Scoring

### Tujuan
Memastikan hasil ujian menampilkan total score dan section scores dengan bobot yang benar.

### Pre-Condition
- Peserta sudah menyelesaikan ujian (Test Case 5)
- Exam package memiliki bobot:
  - Listening = 2.5
  - Structure = 1.0
  - Reading = 1.0

### Langkah Manual

#### 6a. Halaman `/exam/completed`
1. Setelah submit, peserta berada di `/exam/completed`
2. **Verifikasi:**
   - [ ] **Total Score** menampilkan angka (bukan persentase)
   - [ ] **Section Scores** muncul:
     - Listening: score sesuai (benar × 2.5)
     - Structure: score sesuai (benar × 1.0)
     - Reading: score sesuai (benar × 1.0)
   - [ ] Answer Breakdown: Correct, Wrong, Unanswered
   - [ ] Tombol "Lihat Detail" mengarah ke halaman score

#### 6b. Halaman `/exam/score?attempt_id=X`
1. Klik **"Lihat Detail"**
2. **Verifikasi:**
   - [ ] Total Score besar di tengah
   - [ ] Section Scores muncul dengan warna berbeda:
     - Listening: sky blue
     - Structure: violet
     - Reading: amber
   - [ ] Detail per soal (benar/salah/tidak dijawab)
   - [ ] Audio player untuk soal listening (bisa diputar ulang)

#### 6c. Halaman `/exam/history`
1. Buka menu **Riwayat Ujian**
2. **Verifikasi:**
   - [ ] List hasil ujian tampil
   - [ ] Setiap item menampilkan total score
   - [ ] Bisa klik untuk lihat detail

### Contoh Perhitungan Expected
| Section | Soal | Bobot | Benar | Score |
|---------|------|-------|-------|-------|
| Listening | 20 | 2.5 | 15 | 37.5 |
| Structure | 25 | 1.0 | 20 | 20.0 |
| Reading | 25 | 1.0 | 18 | 18.0 |
| **Total** | **70** | | **53** | **75.5** |

**Verifikasi:** Total Score = **75.5**

---

## 🧪 Test Case 7: Registration Period Ditutup Saat Peserta Sudah Terdaftar

### Tujuan
Memastikan peserta yang sudah terdaftar dan di-approve **tetap bisa login** meskipun periode registrasi ditutup.

### Pre-Condition
- Peserta sudah terdaftar dan di-approve
- Periode registrasi sedang terbuka

### Langkah Manual
1. Pastikan peserta bisa login dan masuk dashboard
2. Tutup periode registrasi (via admin panel):
   - Tanggal Tutup: masa lalu
3. Di frontend peserta, refresh halaman atau logout → login lagi

### Verifikasi
- [ ] Peserta yang sudah di-approve **tetap bisa login**
- [ ] Dashboard tampil normal
- [ ] Tidak ada banner "Pendaftaran Ditutup"
- [ ] Ujian tetap bisa dikerjakan (jika ada session aktif)

### Catatan Penting
Blocking hanya terjadi saat **registrasi** (register) dan **login pertama kali**. Peserta yang sudah punya token/session yang valid tidak terpengaruh. Tapi, backend juga memblokir login jika periode tutup — jadi ini perlu diverifikasi apakah backend mengizinkan existing users.

> **Tip:** Jika Anda ingin existing users tetap bisa login meski registrasi ditutup, pastikan backend `AuthController::login()` memeriksa apakah user sudah ada sebelumnya. Saat ini implementasi memblokir SEMUA login saat registrasi ditutup. Perlu diskusi apakah ini behavior yang diinginkan.

---

## 🧪 Test Case 8: Forgot Password

### Tujuan
Memastikan fitur lupa password berfungsi.

### Langkah Manual
1. Buka `https://fe-cbt-url/login`
2. Klik **"Lupa password?"**
3. Isi email yang terdaftar
4. Klik **"Kirim Link Reset"**

### Verifikasi
- [ ] Toast/alert sukses: "Link reset password telah dikirim ke email"
- [ ] Cek email (atau log mail) ada link reset password
- [ ] Klik link → halaman reset password
- [ ] Isi password baru → sukses redirect ke login

---

## 🧪 Test Case 9: Responsive & UI

### Tujuan
Memastikan frontend responsif di berbagai ukuran layar.

### Langkah Manual
1. **Desktop (1920x1080)**
   - [ ] Layout dua kolom di dashboard
   - [ ] Tabel riwayat ujian penuh
   - [ ] Timer ujian tidak terpotong
2. **Tablet (768px)**
   - [ ] Layout menumpuk, masih readable
   - [ ] Navigation soal masih terjangkau
   - [ ] Audio player masih berfungsi
3. **Mobile (375px)**
   - [ ] Layout single column
   - [ ] Navigation soal jadi scrollable/grid kecil
   - [ ] Tombol submit cukup besar untuk tap
   - [ ] Audio player responsif
   - [ ] Modal konfirmasi submit penuh layar

---

## 🧪 Test Case 10: Network & Error Handling

### Tujuan
Memastikan frontend handle error network dengan baik.

### Langkah Manual

#### 10a. Backend Down
1. Matikan backend (stop php-fpm/nginx)
2. Buka frontend dan coba login
3. **Verifikasi:** Error message jelas, tidak stuck loading

#### 10b. Slow Network
1. Throttle network di DevTools → "Slow 3G"
2. Coba login atau load dashboard
3. **Verifikasi:** Loading state tampil, tidak crash

#### 10c. Invalid Token
1. Hapus token dari localStorage:
   ```javascript
   localStorage.removeItem('cbt_participant_token');
   ```
2. Refresh halaman protected (`/dashboard`)
3. **Verifikasi:** Redirect ke `/login`

#### 10d. Session Expired
1. Tunggu token expired (atau ubah expiry di backend untuk testing)
2. Coba request API
3. **Verifikasi:** Redirect ke login dengan pesan sesi habis

---

## 🧪 Test Case 11: API Endpoint Test (Manual)

```bash
# 1. Registrasi peserta baru
curl -s -X POST https://be-cbt.miftadigital.cloud/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "API Test Peserta",
    "email": "apitest@example.com",
    "phone": "081234567891",
    "institution": "Universitas API",
    "password": "password123",
    "password_confirmation": "password123"
  }' | jq .

# 2. Login peserta
USER_TOKEN=$(curl -s -X POST https://be-cbt.miftadigital.cloud/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"apitest@example.com","password":"password123"}' \
  | jq -r '.data.token')

echo "Token: $USER_TOKEN"

# 3. Cek dashboard
curl -s -H "Authorization: Bearer $USER_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/dashboard | jq .

# 4. Cek exam sessions
curl -s -H "Authorization: Bearer $USER_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/exam-sessions | jq .

# 5. Cek my results
curl -s -H "Authorization: Bearer $USER_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/my/results | jq '.data[0]'
```

---

## ❌ Troubleshooting Frontend Peserta

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|---------------------|--------|
| Banner "Pendaftaran Ditutup" tidak muncul | `/settings/registration` gagal fetch | Cek backend running, CORS, network tab |
| Form tetap aktif meski registrasi ditutup | `registrationStatus` tidak ke-render | Cek console error, cek prop drilling |
| Login gagal dengan error generic | API return 403 tapi tidak di-handle | Cek `AuthController::login()` message |
| Redirect loop login ↔ waiting-approval | Middleware/token check salah | Cek `use-auth-session.ts`, cek `middleware.ts` |
| Score tidak muncul di `/exam/completed` | API tidak return section scores | Cek `ExamResultResource.php` |
| Timer ujian berhenti/mundur | Timezone mismatch | Cek timezone backend (Asia/Jakarta) |
| Audio tidak diputar | Browser autoplay policy | Klik halaman terlebih dahulu sebelum audio play |
| Upload bukti pembayaran gagal | File size > 5MB atau format salah | Cek validasi client + backend |
| Hasil ujian tidak tampil | Attempt ID tidak ditemukan | Cek URL param, cek database |

---

## ✅ Checklist Akhir Frontend Peserta

### Registrasi & Login
- [ ] Banner ditutup muncul saat periode tutup (register & login)
- [ ] Form disabled saat periode tutup
- [ ] Registrasi berhasil saat periode buka
- [ ] Validasi form berfungsi (email, password, phone, file)
- [ ] Login berhasil untuk akun approved
- [ ] Login redirect ke waiting-approval untuk akun pending
- [ ] Login error untuk akun rejected
- [ ] Existing user tetap bisa login meski registrasi ditutup

### Ujian
- [ ] Dashboard menampilkan session aktif
- [ ] Instruksi ujian tampil sebelum mulai
- [ ] Timer berjalan dengan benar
- [ ] Navigasi soal berfungsi (next, prev, flag, jump)
- [ ] Audio listening bisa diputar
- [ ] Submit ujian dengan konfirmasi
- [ ] Tidak bisa kembali ke ujian setelah submit

### Hasil
- [ ] Halaman completed tampil total score + section scores
- [ ] Halaman score detail tampil breakdown per soal
- [ ] Halaman history tampil list hasil
- [ ] Score calculation sesuai bobot (listening 2.5x)

### Umum
- [ ] Responsive di mobile/tablet
- [ ] Error network di-handle dengan baik
- [ ] Token expired redirect ke login
- [ ] Tidak ada error di console browser

---

*Last Updated: 2026-05-09*
