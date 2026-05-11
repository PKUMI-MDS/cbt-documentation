# CBT-ADMIN Task Breakdown

Dokumen ini adalah breakdown pekerjaan admin frontend `cbt-admin` berdasarkan kondisi repo saat ini dan API admin backend.

**Last Updated:** 2026-05-11

---

## Ringkasan Status

`cbt-admin` adalah aplikasi Next.js 16/React 19 dengan Tailwind 4 dan pnpm. Kondisi terbaru sudah memiliki:

| Area | Status |
|------|--------|
| **Auth** | ✅ Cookie-based, layout guard, logout |
| **Dashboard** | ✅ Stats real-time dari API + chart |
| **Question Banks** | ✅ CRUD lengkap + warning usage |
| **Questions** | ✅ CRUD + media upload + **RTE TipTap** + preview image/audio |
| **Exam Packages** | ✅ CRUD + bank mapping |
| **Exam Sessions** | ✅ CRUD + publish/close + **participant management** |
| **User Management** | ✅ Approve/reject/reset password (actions ready) |
| **Payment Proofs** | ✅ Approve/reject + **file preview via API** |
| **Result Export** | ✅ Export dengan filter session |
| **Related Links** | ✅ Navigasi antar entitas di detail page |
| **Loading State** | ✅ SubmitButton dengan useFormStatus |
| **Confirmation Modal** | ✅ DeleteConfirmForm + RemoveParticipantForm |
| **Toast Notification** | ✅ Sonner installed di layout root |
| **Validation Error** | ✅ Laravel 422 parsing dengan useActionState |
| **Skeleton Loading** | ✅ Dashboard + module page |
| **Breadcrumb** | ✅ ModuleDetailPage, MonitoringDetailPage, ResultDetailPage, ViolationDetailPage |
| **Monitoring** | ✅ Custom detail page + live refresh fetch data baru |
| **Violations** | ✅ Custom detail page + severity color coding |
| **Card View Mobile** | ✅ Table di desktop, card view di mobile |

---

## Yang Sudah Selesai ✅

### 1. Environment dan Runtime
- ✅ `.env.local.example` tersedia
- ✅ Node 20+ requirement documented
- ✅ Production `CBT_ADMIN_TOKEN` hanya untuk debug

### 2. Admin Auth
- ✅ Login action dengan API
- ✅ Cookie token `cbt_admin_token`
- ✅ Guard admin layout dengan validasi ke `/api/me`
- ✅ Logout route

### 3. API Client dan Data Fetching
- ✅ Server-only API client
- ✅ Bearer token dari cookie
- ✅ List/detail request
- ✅ Pagination metadata
- ✅ Filter dan query params
- ✅ Mutation helpers (POST/PATCH/DELETE/multipart)
- ✅ Cache policy `no-store`
- ✅ **Remove production fallback mock data** — error state jelas saat API gagal

### 4. Dashboard Admin
- ✅ Count dari endpoint per status
- ✅ Newest pending dari API
- ✅ Link filter ke list terfilter
- ✅ **Skeleton loading** saat data kosong
- ✅ **Stats chart** — Bar chart dengan recharts

### 5. Payment Proof Review
- ✅ List dengan filter status
- ✅ Detail dengan file preview
- ✅ **Preview via API endpoint** (`/api/admin/payment-proofs/{id}/preview`)
- ✅ Approve/reject actions
- ✅ Disable action jika bukan pending

### 6. User Management
- ✅ List dengan filter account_status
- ✅ Detail dengan profile dan payment proofs
- ✅ Actions: approve, reject, reset password

### 7. Test Approval Management
- ✅ List dengan filter status/user_id
- ✅ Detail dengan user dan payment proof
- ✅ Lifecycle display (available/assigned/consumed/expired)

### 8. Question Bank Management
- ✅ CRUD lengkap
- ✅ Questions count
- ✅ **Warning bank dipakai aktif** — tampil package yang menggunakan bank saat detail

### 9. Question Management
- ✅ CRUD dengan multipart
- ✅ **Rich text editor (TipTap)** — toolbar dengan bold, italic, underline, heading, align, list, quote, code block
- ✅ Option editor dengan exactly one correct
- ✅ Media upload (image/audio)
- ✅ `difficulty_level` field
- ✅ **Media preview** — image (`<img>`) dan audio (`<audio>`) di form dan detail

### 10. Exam Package Builder
- ✅ CRUD dengan banks array
- ✅ Bank mapping UI
- ✅ Validasi stok soal dari backend

### 11. Exam Session Management
- ✅ List dengan filter status/date
- ✅ Detail dengan semua field
- ✅ Create/edit form
- ✅ Publish/close actions
- ✅ **Participant Management UI**:
  - ✅ Manual assign dengan User ID input
  - ✅ Auto generate participants button
  - ✅ Remove participant per row
  - ✅ Kuota info (`X / Y peserta`)
- ✅ **Export results link** per session

### 12. Monitoring
- ✅ Attempts list dengan filter
- ✅ Violations list
- ✅ **Custom detail page** dengan live countdown, progress bar, stats grid
- ✅ **Live refresh** fetch data baru setiap 30 detik via `/api/monitoring/{id}`

### 13. Results dan Export
- ✅ Results list dengan filter session_id
- ✅ Export Excel/CSV via route handler
- ✅ **Export dengan session_id filter**
- ✅ **Custom detail page** dengan score visualization
- ✅ **Analytics Dashboard** — Visualisasi skor rata-rata per sesi, paket, dan section beserta fix struktur paginasi API (`route.ts`).

### 14. Forms, Mutations, dan Revalidation
- ✅ Server actions untuk semua modul
- ✅ `revalidatePath` setelah mutation
- ✅ Error redirect dengan query params
- ✅ **Loading state tombol action** — SubmitButton dengan useFormStatus
- ✅ **Confirmation modal** — DeleteConfirmForm, RemoveParticipantForm
- ✅ **Toast notification** — Sonner di root layout
- ✅ **Validation error mapping** — Laravel 422 errors ke field-level dengan useActionState

### 15. Detail Page Generic
- ✅ Semua module menampilkan field penting
- ✅ **Related links**:
  - User → Payment proofs, Test approvals
  - Payment Proof → User detail, Test approval
  - Test Approval → User detail, Payment proof
  - Question → Question bank
  - Exam Session → Exam package, Results
  - Monitoring → User detail, Session detail
- ✅ Action contextual di detail page
- ✅ **Breadcrumb navigation** — Dashboard > Module > Detail

### 16. UI/UX Admin
- ✅ Empty state per module
- ✅ Error state API gagal
- ✅ Filter bar
- ✅ Pagination controls
- ✅ Consistent badge/tone
- ✅ **Skeleton loading** — Dashboard + module page
- ✅ **Responsive table mobile** — table di desktop, card view di mobile

### 17. Bug Fixes & Improvements
- ✅ **Route Shadowing Fix** — Menambahkan `page.tsx` secara eksplisit pada setiap folder modul di `src/app/admin/*` untuk mencegah Next.js static folder shadowing terhadap *dynamic route* `[section]`.
- ✅ **React 19 Server Components** — Memperbarui pemanggilan komponen `AdminSectionPage` menjadi format JSX `<AdminSectionPage params={...} />` agar sesuai dengan konvensi Server Component di React 19.
- ✅ **Question Media URL Fix** — Menghapus prefix `questions/` dari `imageProxyUrl` dan `audioProxyUrl` di `admin-api.ts` agar tidak terjadi duplikasi *path* yang menyebabkan error 404 saat memuat gambar/audio soal melalui `MediaController`.

### 18. New Features (Sprint P3)
- ✅ **Question Import** — Fitur Bulk Import soal via CSV dengan download template.
- ✅ **Global Exam Settings** — Modul pengaturan default ujian (durasi, tab switch, shuffle, show result) di `/admin/settings`.
- ✅ **Exam Session Management** — Tambahan aksi *Finish* dan *Cancel* untuk manajemen paksa siklus ujian.
- ✅ **Activity Logs** — Halaman pantauan log aktivitas admin dan peserta.
- ✅ **Analytics Dashboard** — Visualisasi performa tes per sesi, paket, atau section.

---

## Yang Masih Belum / Perlu Perbaikan 🔧

- 🎉 **Semua task untuk sprint P0 - P3 telah diselesaikan!** Tidak ada backlog atau task yang tertunda saat ini. Semua pembaruan dari Backend sudah terintegrasi penuh.

---

## Prioritas Sprint

### P0 - Integrasi Admin Wajib ✅ DONE
- Remove production fallback mock ✅
- Loading state tombol action ✅
- Confirmation modal ✅
- Toast notification ✅
- Validation error mapping ✅

### P1 - Content Management ✅ DONE
- Question bank CRUD ✅
- Question editor with media/RTE ✅
- Exam package builder ✅
- Exam session create/publish/assign ✅
- Warning bank dipakai ✅
- RTE proper (TipTap) ✅

### P2 - Monitoring dan Reporting ✅ DONE
- Attempt live monitoring (auto-refresh fetch data) ✅
- Violation detail custom page ✅
- Severity color coding ✅
- Skeleton loading ✅
- Breadcrumb navigation ✅
- Dashboard chart ✅
- Card view mobile ✅

### P3 - New Features & Enhancements ✅ DONE
- Fitur Import Soal via CSV ✅
- Pengaturan Ujian Global (Settings) ✅
- Menu Activity Logs ✅
- Analytics Dashboard ✅
- Aksi Finish/Cancel pada Exam Session ✅
- Penyesuaian Media Proxy URL ✅

### P4 - Client Feedback Refinement ❌
- **Hapus Section & Difficulty (Buat Soal):** Menghilangkan field "Section" dan "Difficulty" di halaman `question-form.tsx`. ❌
- **Hapus Section (Exam Package):** Menghilangkan kolom "Section" di halaman `bank-mapping-editor.tsx`. ❌
- **Perbaikan Input Jumlah Soal:** Memperbaiki bug pada input "Jumlah" di Exam Package agar bisa diketik dan dihapus dengan leluasa. ❌
- **Fix Download Template Import:** Mengubah tombol Download Template agar *generate* CSV langsung di frontend (mengatasi error 401). ❌
- **[FITUR BARU] UI Buat Soal Massal:** Membuat halaman baru (`/admin/questions/batch`) dengan gaya *Google Forms* (berbasis kartu). Form ini hanya akan berisi *Pertanyaan* dan *Opsi A-D* (tanpa Section/Difficulty). ❌
