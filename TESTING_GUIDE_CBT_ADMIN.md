# 🧪 Testing Guide — CBT-Admin (Admin Panel)

Panduan lengkap untuk menguji fitur-fitur utama di panel admin CBT-TOAFL.

---

## 📋 Pre-requisite

Pastikan semua service berjalan:

```bash
# 1. BE-CBT (Backend)
cd /var/www/be-cbt
php artisan migrate        # Jalankan migration terbaru
php artisan serve          # atau via nginx/php-fpm

# 2. CBT-ADMIN (Admin Panel)
cd /var/www/cbt-admin
pnpm install
pnpm build && pnpm start   # Production mode
# atau: pnpm dev            # Development mode
```

**Environment variables yang harus di-set:**
```bash
NEXT_PUBLIC_API_BASE_URL=https://be-cbt.miftadigital.cloud/api
API_BASE_URL=https://be-cbt.miftadigital.cloud/api
```

---

## 🧪 Test Case 1: Login & Cookie Authentication

### Tujuan
Memastikan login berhasil dan cookie `cbt_admin_token` tersimpan dengan benar (Route Handler + Middleware pattern).

### Langkah Manual
1. Buka halaman login: `https://admin-panel-url/login`
2. Masukkan email dan password admin yang valid
3. Klik **Login**

### Verifikasi
- [ ] Redirect ke `/admin/dashboard` (full page reload, bukan client-side navigation)
- [ ] Di DevTools → Application → Cookies, terlihat cookie `cbt_admin_token` dengan:
  - `HttpOnly: true`
  - `Secure: true`
  - `SameSite: Lax`
- [ ] Di DevTools → Network, request ke `/api/login` mengembalikan cookie di response header `Set-Cookie`
- [ ] Refresh halaman (`F5`), user tetap logged in (tidak redirect ke login)
- [ ] Navigasi antar menu (klik sidebar) tetap authenticated

### Verifikasi via curl
```bash
# Login dan simpan cookie
ADMIN_TOKEN=$(curl -s -c cookies.txt -X POST https://be-cbt.miftadigital.cloud/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"password"}' \
  | jq -r '.data.token')

echo "Token: $ADMIN_TOKEN"

# Cek cookie tersimpan
cat cookies.txt

# Request ke endpoint admin (pakai token)
curl -s -H "Authorization: Bearer $ADMIN_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/admin/dashboard-stats \
  | jq .
```

---

## 🧪 Test Case 2: Registration Period Settings (CRUD)

### Tujuan
Mengatur periode buka/tutup pendaftaran peserta dan memastikan pesan tampil di frontend.

### Langkah Manual

#### 2a. Set Periode Pendaftaran (Terbuka)
1. Login admin → menu **Settings**
2. Scroll ke bagian **"Pengaturan Periode Registrasi"**
3. Isi:
   - **Tanggal Buka:** `2026-05-01 00:00`
   - **Tanggal Tutup:** `2026-12-31 23:59`
   - **Pesan saat Ditutup:** (kosongkan atau isi opsional)
4. Klik **"Simpan Pengaturan"**

#### Verifikasi
- [ ] Toast sukses muncul: "Pengaturan berhasil disimpan"
- [ ] Field tanggal tetap menampilkan value yang baru disimpan setelah refresh

#### 2b. Set Periode Pendaftaran (Ditutup)
1. Di menu **Settings**, ubah:
   - **Tanggal Buka:** `2025-01-01 00:00` (tanggal masa lalu)
   - **Tanggal Tutup:** `2025-01-02 00:00` (tanggal masa lalu)
   - **Pesan saat Ditutup:** "Pendaftaran peserta ujian TOAFL periode 2026 telah ditutup. Silakan hubungi admin untuk informasi lebih lanjut."
2. Klik **"Simpan Pengaturan"**

#### Verifikasi via API
```bash
# Cek public endpoint (tanpa auth)
curl -s https://be-cbt.miftadigital.cloud/api/settings/registration | jq .

# Expected:
# {
#   "data": {
#     "is_open": false,
#     "message": "Pendaftaran peserta ujian TOAFL periode 2026 telah ditutup...",
#     "open_date": "2025-01-01T00:00:00.000000Z",
#     "close_date": "2025-01-02T00:00:00.000000Z"
#   }
# }
```

#### 2c. Verifikasi Impact di Frontend Peserta
1. Buka `https://fe-cbt-url/register`
2. **Verifikasi:** Muncul banner amber dengan pesan di atas form
3. **Verifikasi:** Semua input form disabled, tombol "Submit Registrasi" disabled
4. Coba login di `https://fe-cbt-url/login`
5. **Verifikasi:** Muncul banner amber, input disabled, tombol "Login" berubah jadi "Pendaftaran Ditutup"

#### 2d. Buka Kembali Pendaftaran
1. Ubah **Tanggal Tutup** ke tanggal masa depan
2. Klik **Simpan**
3. Refresh frontend → **Verifikasi:** Form aktif kembali, banner hilang

---

## 🧪 Test Case 3: Exam Settings

### Tujuan
Mengatur pengaturan ujian dan memastikan tersimpan dengan benar.

### Langkah Manual
1. Login admin → menu **Settings**
2. Isi field-field berikut:
   | Field | Value Contoh |
   |-------|-------------|
   | Nama Ujian | Ujian TOAFL Periode Mei 2026 |
   | Durasi (menit) | 120 |
   | Passing Grade | 60 |
   | Nilai per Soal | 1 |
   | Tampilkan Hasil | Ya |
   | Tampilkan Timer | Ya |
3. Klik **"Simpan Pengaturan"**

### Verifikasi
- [ ] Toast sukses muncul
- [ ] Refresh halaman, semua field tetap terisi value yang sama
- [ ] Di frontend peserta (`/dashboard`), nama ujian dan durasi sesuai

### Verifikasi via API
```bash
curl -s https://be-cbt.miftadigital.cloud/api/settings/exam | jq .
```

---

## 🧪 Test Case 4: Participant Management (Approve/Reject)

### Tujuan
Mengelola peserta: melihat list, approve/reject, filter, export.

### Langkah Manual

#### 4a. Lihat List Peserta
1. Login admin → menu **Participants**
2. **Verifikasi:**
   - [ ] Tabel menampilkan kolom: Nama, Email, No. WhatsApp, Institusi, Status Akun, Status Pembayaran
   - [ ] Status badge berwarna: Pending (amber), Approved (emerald), Rejected (rose)
   - [ ] Pagination aktif (jika data > 10)
   - [ ] Search/filter berfungsi

#### 4b. Approve Peserta
1. Cari peserta dengan status **"Pending"**
2. Klik tombol **"Approve"** pada baris tersebut
3. **Verifikasi:**
   - [ ] Modal konfirmasi muncul
   - [ ] Setelah klik "Ya, Approve", status berubah jadi **"Approved"**
   - [ ] Toast sukses muncul
   - [ ] Peserta bisa login di frontend dan masuk ke dashboard

#### 4c. Reject Peserta
1. Cari peserta dengan status **"Pending"**
2. Klik tombol **"Reject"**
3. **Verifikasi:**
   - [ ] Status berubah jadi **"Rejected"**
   - [ ] Peserta tidak bisa login (error "Akun ditolak")

#### 4d. Detail Peserta
1. Klik nama peserta di tabel
2. **Verifikasi:**
   - [ ] Modal/detail drawer menampilkan data lengkap peserta
   - [ ] Ada preview bukti pembayaran (gambar/PDF)
   - [ ] Tombol Approve/Reject tersedia

### Verifikasi via API
```bash
# List peserta
curl -s -H "Authorization: Bearer $ADMIN_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/admin/participants | jq '.data.items[0]'

# Approve peserta (ganti ID sesuai)
curl -s -X PATCH -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  https://be-cbt.miftadigital.cloud/api/admin/participants/1/approve \
  | jq .

# Reject peserta
curl -s -X PATCH -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"reason":"Data tidak lengkap"}' \
  https://be-cbt.miftadigital.cloud/api/admin/participants/1/reject \
  | jq .
```

---

## 🧪 Test Case 5: Exam Package & Bank Mapping (Weighted Scoring)

### Tujuan
Membuat exam package dengan bank soal dan mengatur bobot skor per section.

### Langkah Manual

#### 5a. Buat Exam Package Baru
1. Login admin → menu **Exam Packages** → **New**
2. Isi:
   - **Nama Package:** TOAFL Standard 2026
   - **Deskripsi:** Package standar TOAFL
3. Di bagian **"Daftar Bank Soal"**, tambah:
   | Bank Soal | Jumlah Soal | Bobot |
   |-----------|-------------|-------|
   | Bank Listening (Fahm al-Masmu) | 20 | **2.5** |
   | Bank Structure (Fahm al-Kitabah) | 25 | **1.0** |
   | Bank Reading (Fahm al-Maqru) | 25 | **1.0** |
4. **Verifikasi UI:**
   - [ ] Kolom "Bobot" tampil di tabel
   - [ ] Saat pilih Listening, default bobot = 2.5
   - [ ] Bobot bisa diedit manual
5. Klik **"Create Package"**

#### 5b. Verifikasi Detail Package
1. Klik package yang baru dibuat
2. **Verifikasi:**
   - [ ] Section "Banks" menampilkan: `Bank Listening (20 soal, bobot 2.5)`
   - [ ] Total soal = 70

#### 5c. Edit Bobot
1. Klik **"Edit"** pada package
2. Ubah bobot Listening dari 2.5 → 3.0
3. Simpan
4. **Verifikasi:** Bobot tersimpan baru

### Verifikasi via API
```bash
# Detail package (harus ada score_weight)
curl -s -H "Authorization: Bearer $ADMIN_TOKEN" \
  https://be-cbt.miftadigital.cloud/api/admin/exam-packages/1 \
  | jq '.data.banks[] | {question_bank_id, question_count, score_weight}'
```

---

## 🧪 Test Case 6: Exam Session Management

### Tujuan
Membuat dan mengelola sesi ujian.

### Langkah Manual

#### 6a. Buat Exam Session
1. Login admin → menu **Exam Sessions** → **New**
2. Isi:
   - **Nama Session:** TOAFL Batch 1 - Mei 2026
   - **Exam Package:** Pilih "TOAFL Standard 2026"
   - **Tanggal Mulai:** (pilih tanggal & waktu)
   - **Tanggal Selesai:** (pilih tanggal & waktu)
   - **Status:** Published
3. Klik **"Create Session"**

#### Verifikasi
- [ ] Session muncul di list
- [ ] Detail session menampilkan package yang dipilih
- [ ] Status badge = "Published"

#### 6b. Assign Peserta ke Session
1. Buka detail session
2. Tab **"Peserta"**
3. Klik **"Tambah Peserta"**
4. Pilih peserta yang sudah di-approve
5. **Verifikasi:** Peserta muncul di list session

#### 6c. Publish/Unpublish Session
1. Di list session, klik tombol **"Unpublish"**
2. **Verifikasi:** Status berubah jadi "Draft", peserta tidak bisa mulai ujian
3. Klik **"Publish"** kembali
4. **Verifikasi:** Status kembali "Published"

---

## 🧪 Test Case 7: Results & Monitoring

### Tujuan
Melihat hasil ujian peserta dengan section scores.

### Langkah Manual

#### 7a. List Results
1. Login admin → menu **Exam Results**
2. **Verifikasi:**
   - [ ] Tabel menampilkan: Nama Peserta, Session, Total Score, Status
   - [ ] Total Score dalam bentuk angka (bukan persentase)
   - [ ] Bisa filter per session

#### 7b. Detail Result
1. Klik salah satu result
2. **Verifikasi:**
   - [ ] **Total Score** besar di atas
   - [ ] **Section Scores** muncul:
     - Listening (L): score sesuai
     - Structure (S): score sesuai
     - Reading (R): score sesuai
   - [ ] Answer Breakdown: Correct, Wrong, Unanswered
   - [ ] Metadata: Attempt ID, Waktu Mulai, Waktu Selesai

#### 7c. Real-time Monitoring (jika ada)
1. Menu **Monitoring**
2. **Verifikasi:**
   - [ ] List peserta yang sedang ujian (status "In Progress")
   - [ ] Bisa force-submit ujian
   - [ ] Bisa lihat live progress

---

## 🧪 Test Case 8: Dashboard Statistics

### Tujuan
Memastikan dashboard menampilkan statistik dengan benar.

### Langkah Manual
1. Login admin → **Dashboard**
2. **Verifikasi:**
   - [ ] Card "Total Peserta" menampilkan jumlah
   - [ ] Card "Peserta Approved" menampilkan jumlah
   - [ ] Card "Peserta Pending" menampilkan jumlah
   - [ ] Card "Total Ujian" menampilkan jumlah
   - [ ] Grafik/chart (jika ada) menampilkan data
   - [ ] Data berubah saat ada perubahan di backend

---

## 🧪 Test Case 9: Responsive & UI

### Tujuan
Memastikan UI responsif di berbagai ukuran layar.

### Langkah Manual
1. Buka admin panel di desktop (1920x1080)
   - [ ] Sidebar tampil penuh
   - [ ] Tabel tidak terpotong
2. Buka di tablet (768px)
   - [ ] Sidebar bisa collapse/expand
   - [ ] Tabel bisa di-scroll horizontal
3. Buka di mobile (375px)
   - [ ] Sidebar jadi hamburger menu
   - [ ] Form menumpuk vertikal
   - [ ] Tabel tampil sebagai card/list

---

## ❌ Troubleshooting Admin Panel

| Masalah | Kemungkinan Penyebab | Solusi |
|---------|---------------------|--------|
| Login redirect loop | Cookie tidak tersimpan | Cek `secure` flag (harus true di HTTPS), cek domain cookie |
| 401 setelah login | Token tidak terpropagasi ke header | Cek middleware.ts, cek `x-admin-token` header |
| Data tidak update setelah edit | Cache atau state tidak invalidate | Refresh halaman (F5), clear browser cache |
| Form settings tidak tersimpan | Field validation gagal | Cek console error, cek network tab response |
| Section scores tidak muncul | API tidak return field | Cek `ExamResultResource.php`, cek database |
| Upload gambar gagal | File size terlalu besar | Cek batas upload PHP (`upload_max_filesize`) |

---

## ✅ Checklist Akhir Admin Panel

- [ ] Login berhasil, cookie tersimpan
- [ ] Registration period bisa di-set buka/tutup
- [ ] Exam settings bisa di-edit dan tersimpan
- [ ] Peserta bisa di-approve/reject
- [ ] Exam package bisa dibuat dengan bobot per bank
- [ ] Exam session bisa dibuat dan di-publish
- [ ] Results menampilkan section scores
- [ ] Dashboard statistik akurat
- [ ] UI responsif di mobile/tablet
- [ ] Tidak ada error di console browser

---

*Last Updated: 2026-05-09*
