# CBT-ADMIN Task Breakdown

Dokumen ini adalah breakdown pekerjaan admin frontend `cbt-admin` berdasarkan kondisi repo saat ini dan API admin backend.

**Last Updated:** 2026-05-09

---

## Ringkasan Status

`cbt-admin` adalah aplikasi Next.js 16/React 19 dengan Tailwind 4 dan pnpm. Kondisi terbaru sudah memiliki:

| Area | Status |
|------|--------|
| **Auth** | ✅ Cookie-based, layout guard, logout |
| **Dashboard** | ✅ Stats real-time dari API |
| **Question Banks** | ✅ CRUD lengkap |
| **Questions** | ✅ CRUD + media upload + RTE |
| **Exam Packages** | ✅ CRUD + bank mapping |
| **Exam Sessions** | ✅ CRUD + publish/close + **participant management** |
| **User Management** | ✅ Approve/reject/reset password (actions ready) |
| **Payment Proofs** | ✅ Approve/reject + **file preview via API** |
| **Result Export** | ✅ Export dengan filter session |
| **Related Links** | ✅ Navigasi antar entitas di detail page |

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

### 4. Dashboard Admin
- ✅ Count dari endpoint per status
- ✅ Newest pending dari API
- ✅ Link filter ke list terfilter

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

### 9. Question Management
- ✅ CRUD dengan multipart
- ✅ HTML editor (stem, explanation)
- ✅ Option editor dengan exactly one correct
- ✅ Media upload (image/audio)
- ✅ `difficulty_level` field

### 10. Exam Package Builder
- ✅ CRUD dengan banks array
- ✅ Bank mapping UI
- ✅ Validasi stok soal dari backend

### 11. Exam Session Management (Updated - 9 Mei 2026)
- ✅ List dengan filter status/date
- ✅ Detail dengan semua field
- ✅ Create/edit form
- ✅ Publish/close actions
- ✅ **Participant Management UI** (baru):
  - ✅ Manual assign dengan User ID input
  - ✅ Auto generate participants button
  - ✅ Remove participant per row
  - ✅ Kuota info (`X / Y peserta`)
- ✅ **Export results link** per session

### 12. Monitoring
- ✅ Attempts list dengan filter
- ✅ Violations list

### 13. Results dan Export
- ✅ Results list dengan filter session_id
- ✅ Export Excel/CSV via route handler
- ✅ **Export dengan session_id filter** (baru)

### 14. Forms, Mutations, dan Revalidation
- ✅ Server actions untuk semua modul
- ✅ `revalidatePath` setelah mutation
- ✅ Error redirect dengan query params

### 15. Detail Page Generic (Updated - 9 Mei 2026)
- ✅ Semua module menampilkan field penting
- ✅ **Related links** (baru):
  - User → Payment proofs, Test approvals
  - Payment Proof → User detail, Test approval
  - Test Approval → User detail, Payment proof
  - Question → Question bank
  - Exam Session → Exam package, Results
  - Monitoring → User detail, Session detail
- ✅ Action contextual di detail page

### 16. UI/UX Admin
- ✅ Empty state per module
- ✅ Error state API gagal
- ✅ Filter bar
- ✅ Pagination controls
- ✅ Consistent badge/tone

---

## Yang Masih Belum / Perlu Perbaikan 🔧

### P0 - Integrasi Admin Wajib
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Remove production fallback mock** | ❌ Belum | `admin-data.ts` masih punya fallback data |
| 2 | **Loading state tombol action** | ❌ Belum | Prevent double submit |
| 3 | **Confirmation modal** | ❌ Belum | Destructive actions (delete, reject) |
| 4 | **Toast notification** | ❌ Belum | Sukses/error feedback |
| 5 | **Validation error mapping** | ❌ Belum | Laravel errors → form field |

### P1 - Content Management
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Warning bank dipakai aktif** | ❌ Belum | Saat delete/deactivate |
| 2 | **Media preview di question detail** | ⚠️ Parsial | Hanya filename, belum preview |
| 3 | **Rich text editor Arabic** | ⚠️ Parsial | HTML textarea, belum RTE proper |

### P2 - Monitoring dan Reporting
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Live refresh interval** | ❌ Belum | Auto-refresh monitoring |
| 2 | **Severity visual** | ❌ Belum | Color coding violations |
| 3 | **Custom monitoring detail page** | ⚠️ Parsial | Generic page, belum custom |
| 4 | **Custom violation detail page** | ⚠️ Parsial | Generic page, belum custom |

### UX Polish
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Skeleton loading** | ❌ Belum | Saat fetch data |
| 2 | **Breadcrumb** | ❌ Belum | Navigation trail |
| 3 | **Responsive table mobile** | ⚠️ Parsial | Overflow-x ada, tapi bisa diperbaiki |

---

## Prioritas Sprint

### P0 - Integrasi Admin Wajib
- Remove production fallback mock
- Loading state tombol action
- Confirmation modal
- Toast notification

### P1 - Content Management ✅ MOSTLY DONE
- Question bank CRUD ✅
- Question editor with media/RTE ✅
- Exam package builder ✅
- Exam session create/publish/assign ✅

### P2 - Monitoring dan Reporting
- Attempt live monitoring (auto-refresh)
- Violation detail custom page
- Dashboard polish
