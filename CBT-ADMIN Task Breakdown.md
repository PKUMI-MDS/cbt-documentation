# CBT-ADMIN Task Breakdown

Dokumen ini adalah breakdown pekerjaan admin frontend `cbt-admin` berdasarkan kondisi repo saat ini dan API admin backend.

**Last Updated:** 2026-05-14

---

## Ringkasan Status

`cbt-admin` adalah aplikasi Next.js 16/React 19 dengan Tailwind 4 dan pnpm. Kondisi terbaru sudah memiliki:

| Area | Status |
|------|--------|
| **Auth** | ✅ Cookie-based, layout guard, logout |
| **Dashboard** | ✅ Stats real-time dari API + chart |
| **Question Banks** | ✅ CRUD lengkap + warning usage |
| **Questions** | ✅ CRUD + media upload + **RTE TipTap** + preview image/audio + **batch creator** |
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
| **Security Hardening** | ✅ Sanitasi HTML TipTap + CSP/security headers |
| **Lint / Build** | ✅ 0 warnings, 0 errors — build pass 29 pages |

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
- ✅ `admin-api.ts` sudah menjadi facade re-export; logic dipisah per modul di `src/lib/admin-api/*`
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
- ✅ **Section Type Metadata** — form create/edit, list, dan detail Question Bank menampilkan `section_type` dengan opsi final `fahm_al_maqru`, `fahm_al_kitabah`, dan `fahm_al_masmu`.

### 9. Question Management
- ✅ CRUD dengan multipart
- ✅ **Rich text editor (TipTap)** — toolbar dengan bold, italic, underline, heading, align, list, quote, code block
- ✅ Option editor dengan exactly one correct
- ✅ Media upload (image/audio)
- ✅ **Media preview** — image (`<img>`) dan audio (`<audio>`) di form dan detail
- ✅ **Field `section` dan `difficulty_level` sudah dihapus** dari form (`question-form.tsx`) — sesuai client feedback P4
- ✅ **Bulk/Batch Question Creator** — halaman `/admin/questions/batch` dengan gaya Google Forms (kartu per soal, stem + opsi A–D, tanpa section/difficulty)

### 10. Exam Package Builder
- ✅ CRUD dengan banks array
- ✅ Bank mapping UI
- ✅ Validasi stok soal dari backend
- ✅ **Kolom `section` sudah dihapus** dari `bank-mapping-editor.tsx` — kolom yang tersisa: Bank Soal, Jumlah, Urutan, Action
- ✅ **Input Jumlah Soal** — field dinormalisasi minimal `1` saat blur dan saat serialize `banks_json`
- ✅ **Indikator No Shuffle Fahm al-Masmu** — bank mapping menampilkan label section dan badge `No shuffle` untuk `section_type = fahm_al_masmu`.

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
- ✅ **Refresh indicator** — tombol menampilkan `Refreshing...` dan waktu sinkron terakhir

### 13. Results dan Export
- ✅ Results list dengan filter session_id
- ✅ Export Excel/CSV via route handler
- ✅ **Export dengan session_id filter**
- ✅ **Custom detail page** dengan score visualization
- ✅ **Analytics Dashboard** — Visualisasi skor rata-rata per sesi dan paket beserta fix struktur paginasi API (`route.ts`).

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
- ✅ **Analytics Dashboard** — Visualisasi performa tes per sesi dan paket.

### 19. Client Feedback Refinement (Sprint P4)
- ✅ **Hapus Section & Difficulty (Buat Soal)** — Field "Section" dan "Difficulty" sudah tidak ada di `question-form.tsx`.
- ✅ **Hapus Section (Exam Package)** — Kolom "Section" sudah tidak ada di `bank-mapping-editor.tsx`.
- ✅ **Fix Download Template Import** — Tombol Download Template men-*generate* CSV langsung di frontend via `data:text/csv` URL (tidak lagi hit API yang error 401).
- ✅ **UI Buat Soal Massal** — Halaman `/admin/questions/batch` sudah tersedia dengan komponen `BulkQuestionCreator` gaya Google Forms (kartu per soal, stem + opsi A–D).
- ✅ **Perbaikan Input Jumlah Soal** — Field "Jumlah" di bank mapping tidak lagi terkirim sebagai `0`; nilai kosong dinormalisasi minimal `1`.

### 20. Feature Completion Review Hardening
- ✅ **Analytics route type-safety** — `as any` diganti dengan interface response eksplisit.
- ✅ **Sanitasi HTML TipTap** — semua `dangerouslySetInnerHTML` untuk konten soal melewati `sanitizeHtml`.
- ✅ **Refactor Admin API Client** — `admin-api.ts` dipecah menjadi facade + modul `analytics`, `auth`, `module-pages`, `questions`, `exam-packages`, `exam-sessions`, `monitoring`, dan `settings`.
- ✅ **Input Question Count** — `question_count` bank mapping dinormalisasi minimal `1` saat blur dan serialize.
- ✅ **Monitoring refresh feedback** — tombol refresh disabled saat fetch dan menampilkan indikator putar.
- ✅ **Security headers** — CSP, `X-Content-Type-Options`, `Referrer-Policy`, dan `X-Frame-Options` ditambahkan di `next.config.ts`.
- ✅ **Lint cleanup** — 24 ESLint warnings (unused imports + `<img>` tags) dibersihkan menjadi **0 warnings, 0 errors**.
- ✅ **`<img>` → `<Image />`** — 4 komponen migrasi ke `next/image` (login, admin-shell, media-upload, question-media).
- ✅ **`.env.local.example`** — Dokumentasi variabel lingkungan wajib tersedia.

#### Mapping dari `Feature Completion Review`

| Item Review | Lokasi Implementasi | Status | Catatan |
|---|---|---|---|
| Fix `as any` di analytics route | `src/app/api/admin/analytics/route.ts` | ✅ DONE | Response analytics sudah memakai tipe eksplisit, tidak lagi memakai `as any`. |
| Tambah XSS sanitization untuk output TipTap HTML | `src/lib/sanitize-html.ts`, `src/components/module-detail-page.tsx`, `src/components/question-form-fields.tsx` | ✅ DONE | Konten stem, explanation, option, dan preview HTML disanitasi sebelum `dangerouslySetInnerHTML`. |
| Split `admin-api.ts` by module | `src/lib/admin-api.ts`, `src/lib/admin-api/*` | ✅ DONE | `admin-api.ts` menjadi facade re-export; logic dipisah ke modul domain. |
| Fix input `question_count` bank mapping | `src/components/bank-mapping-editor.tsx` | ✅ DONE | Nilai kosong dinormalisasi minimal `1` saat blur dan saat serialize `banks_json`. |
| Tambah visual indicator `Refreshing...` di monitoring detail | `src/components/monitoring-detail.tsx` | ✅ DONE | Tombol refresh disabled saat fetch, icon berputar, dan waktu sinkron terakhir ditampilkan. |
| Tambah CSP/security headers untuk `cbt-admin` | `next.config.ts` | ✅ DONE | CSP, `nosniff`, `Referrer-Policy`, dan `X-Frame-Options` sudah aktif lewat `headers()`. |
| Bersihkan ESLint warnings (24 → 0) | 12 file di `src/` | ✅ DONE | Hapus unused imports & variables, migrasi `<img>` ke `<Image />`. |
| Tambah `.env.local.example` | Root `cbt-admin/` | ✅ DONE | Dokumentasi `CBT_API_BASE_URL`, `NEXT_PUBLIC_API_BASE_URL`, `CBT_ADMIN_TOKEN`. |

---

## Yang Masih Belum / Perlu Perbaikan 🔧

- Tidak ada task admin fungsional yang masih terbuka di breakdown ini.
- Catatan testing: `cbt-admin` belum memiliki suite E2E resmi di repo. Validasi task `section_type` saat ini dilakukan via `npm run build` dan `npm run lint`; skenario manual/UAT yang perlu dicek adalah create/edit Question Bank dengan `fahm_al_masmu`, lalu pastikan Exam Package Builder menampilkan badge `No shuffle`.

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

### P4 - Client Feedback Refinement ✅ DONE
- **Hapus Section & Difficulty (Buat Soal):** ✅ Field sudah tidak ada di `question-form.tsx`.
- **Hapus Section (Exam Package):** ✅ Kolom sudah tidak ada di `bank-mapping-editor.tsx`.
- **Fix Download Template Import:** ✅ Template di-generate langsung di frontend via `data:text/csv` URL.
- **[FITUR BARU] UI Buat Soal Massal:** ✅ Halaman `/admin/questions/batch` sudah tersedia dengan komponen `BulkQuestionCreator`.
- **Perbaikan Input Jumlah Soal:** ✅ Field kosong dinormalisasi minimal `1` di `bank-mapping-editor.tsx`.

### Feature Completion Review Hardening ✅ DONE
- Fix `as any` di analytics route ✅
- Tambah XSS sanitization untuk output TipTap HTML ✅
- Split `admin-api.ts` by module ✅
- Fix input `question_count` bank mapping ✅
- Tambah visual indicator `Refreshing...` di monitoring detail ✅
- Tambah CSP/security headers untuk `cbt-admin` ✅
