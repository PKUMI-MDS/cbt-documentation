# FE-CBT Task Breakdown

Dokumen ini adalah breakdown pekerjaan frontend peserta `fe-cbt` berdasarkan kondisi repo saat ini dan kontrak API backend.

**Last Updated:** 2026-05-10

---

## Ringkasan Status

`fe-cbt` adalah aplikasi Next.js 14 dengan integrasi API backend yang sudah sangat lengkap:

| Area | Status |
|------|--------|
| **Auth Flow** | ✅ Lengkap (register → login → dashboard/logout) |
| **Payment Proof** | ✅ Upload & history real API (limit 2MB - sesuai BE) |
| **Dashboard** | ✅ Real data dari backend |
| **Exam Engine** | ✅ Full integration (soal, audio, timer, navigasi, auto-save, submit) |
| **Anti-Cheat** | ✅ Full (fullscreen, tab switch, right-click, violation modal) |
| **Hasil Ujian** | ✅ Score breakdown per section + fetch dedicated endpoint |
| **API Integration** | ✅ 100% real fetching |
| **Server Middleware** | ✅ `middleware.ts` aktif |
| **Global UI States** | ✅ `loading.tsx`, `error.tsx`, `not-found.tsx` |
| **E2E Tests** | ✅ File lengkap (Playwright) - eksekusi & refinement ditunda |

---

## Yang Sudah Selesai ✅

### 1. Environment dan API Client
- ✅ `NEXT_PUBLIC_API_BASE_URL` dengan fallback ke `http://localhost:8000/api`
- ✅ `apiRequest` support JSON dan FormData
- ✅ Token participant di `localStorage` + **sync ke cookie** untuk middleware
- ✅ Interceptor: 401 redirect login, 403 pending redirect waiting-approval, 422 field errors
- ✅ Typed wrappers semua endpoint ada di `lib/auth-api.ts`

### 2. Auth Pages
- ✅ Login real API dengan loading state dan disabled submit
- ✅ Register real API dengan validasi field
- ✅ Role check (hanya participant boleh login)
- ✅ Account status redirect (pending → waiting-approval)
- ✅ Logout participant

### 3. Payment Proof Peserta
- ✅ Page `/payment-proof` untuk upload dan riwayat
- ✅ Form fields: file, amount, payment_date
- ✅ Validasi file type (JPG/PNG/PDF) dan **size max 2MB** (sesuai BE limit)
- ✅ History dengan status dan rejection_reason

### 4. Waiting Approval / Account Status
- ✅ Fetch `GET /api/me`
- ✅ Polling status akun setiap 30 detik
- ✅ Tampilan sesuai status (pending/rejected/active)

### 5. Dashboard Peserta
- ✅ Fetch real data: profile, test-approvals, exam-sessions, active-attempt, results
- ✅ Tombol start/resume mengikuti state API
- ✅ Empty states

### 6. Exam Detail dan Instruction
- ✅ Detail session dari backend
- ✅ Start exam dengan validasi
- ✅ Error handling bisnis

### 7. Exam Engine
- ✅ Load active attempt (start/resume)
- ✅ Load question by number
- ✅ Render soal dari API (stem_html, image, audio, options)
- ✅ Save answer ke backend
- ✅ Mark doubtful
- ✅ Navigate soal
- ✅ Timer dari `remaining_seconds` backend
- ✅ Heartbeat berkala
- ✅ Submit dengan konfirmasi
- ✅ Auto-resume saat refresh
- ✅ Auto-submit saat timer habis

### 8. Audio Player
- ✅ Render audio dari `audio_url`
- ✅ Play count limit dari backend
- ✅ Tampilan max play dan remaining play
- ✅ Log audio-play ke backend (`POST /audio-play`)

### 9. Anti-Cheat Client Events (P1 - Lengkap)
- ✅ **Fullscreen enforcement** — request fullscreen saat exam start, detect exit
- ✅ **Tab switch detection** — visibility change, kirim violation
- ✅ **Disable right click/copy** — di area exam
- ✅ **Warning modal violation** — saat threshold terlewati
- ✅ **Route guard saat exam aktif** — cegah keluar tanpa konfirmasi (`beforeunload`)
- ✅ Violation logging endpoint

### 10. Exam Completed, Result, History
- ✅ Completed page membaca submit response
- ✅ Result visibility mengikuti `show_result_to_user`
- ✅ **Detail skor pakai endpoint dedicated** `GET /exam-attempts/{id}/result` (bukan fetch all + filter)
- ✅ History fetch dari API
- ✅ Score breakdown: listening, structure, reading, benar/salah/tidak dijawab

### 11. Profile Page
- ✅ Fetch `GET /api/my/profile`
- ✅ Tampil semua field user

### 12. Route Guard
- ✅ **Server-side middleware** (`middleware.ts`) - redirect berbasis cookie
- ✅ Client-side `AuthGuard` component
- ✅ Redirect authenticated users dari login/register

### 13. Global UI States
- ✅ `app/loading.tsx` - Loading spinner global
- ✅ `app/error.tsx` - Error boundary dengan retry & dashboard link
- ✅ `app/not-found.tsx` - Halaman 404 dengan navigasi

### 14. Polish (P2)
- ✅ Loading skeleton (Dashboard, History, Exam, Profile)
- ✅ State management dengan @tanstack/react-query
- ✅ Responsive mobile exam layout
- ✅ Accessibility (focus state, aria-labels, keyboard navigation)
- ✅ Lucide icons di dashboard & profile

### 15. E2E Tests (Playwright)
- ✅ `e2e/auth.spec.ts`
- ✅ `e2e/login.spec.ts`
- ✅ `e2e/protected-routes.spec.ts`
- ✅ `e2e/dashboard.spec.ts`
- ✅ `e2e/exam.spec.ts`
- ✅ `playwright.config.ts`

---

## Yang Masih Belum / Perlu Perbaikan 🔧

### P1 - Exam Reliability
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Fullscreen enforcement** | ✅ Selesai | Request fullscreen saat exam start, detect exit |
| 2 | **Tab switch detection** | ✅ Selesai | Detect visibility change, kirim violation |
| 3 | **Disable right click/copy** | ✅ Selesai | Di area exam |
| 4 | **Warning modal violation** | ✅ Selesai | Saat threshold terlewati |
| 5 | **Route guard saat exam aktif** | ✅ Selesai | Cegah keluar tanpa konfirmasi |
| 6 | **Auto-submit on violation limit** | ❌ Belum | Integrasi `auto_submit_on_violation_limit` dari global settings BE |
| 7 | **Dynamic violation limits** | ❌ Belum | `max_tab_switch` & `max_fullscreen_exit` masih hardcode, harusnya dari BE |

### P2 - Polish & Optimization
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Loading skeleton** | ✅ Selesai | Dashboard, history, exam, profile |
| 2 | **State management** | ✅ Selesai | Menggunakan @tanstack/react-query |
| 3 | **E2E tests execution** | ⏸️ Ditunda | File lengkap, perlu validasi & refinement |
| 4 | **Responsive mobile exam** | ✅ Selesai | Layout soal & navigasi grid |
| 5 | **Accessibility** | ✅ Selesai | Focus state, aria, keyboard navigation |
| 6 | **Score detail fetch** | ✅ Selesai | Refactor ke `getAttemptResult()` dedicated endpoint |

### P3 - Backend Integration & Settings (NEW — sinkron BE 10 Mei 2026)
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Global Exam Settings Integration** | ❌ Belum | FE harus baca settings dinamis saat start exam: `auto_submit_on_violation_limit`, `max_tab_switch`, `max_fullscreen_exit`, `shuffle_questions`, `shuffle_options`, `show_result_to_user` |
| 2 | **Media Proxy / Signed URL** | ❌ Belum | BE sekarang pakai signed URL untuk media. FE harus pakai `image_url`/`audio_url` dari response API, bukan hardcode path |
| 3 | **Account Status Notification** | ❌ Belum | Implementasi websocket/polling global untuk update status akun (bukan cuma di waiting-approval) |
| 4 | **Show Result to User enforcement** | ⚠️ Partial | FE sudah ada logic tapi perlu ensure respect global setting dari admin |

### P4 - Missing Features / Placeholder
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Forgot Password** | ⚠️ Placeholder | Page ada tapi hanya static info. BE belum sediakan endpoint reset password |
| 2 | **Edit Profile** | ❌ Belum | Belum ada page & endpoint `PATCH /my/profile` |
| 3 | **Exam Type di Register** | ❌ Belum | Field `exam_type` ada di UI tapi tidak masuk payload API (`RegisterPayload` tidak punya field ini) |

---

## Perubahan Terbaru (10 Mei 2026)

### ✅ Committed
- **Image upload limit** 5MB → 2MB (sesuai BE `max:2048`)
- **Score detail fetch** refactor dari fetch all + client-side filter ke dedicated endpoint `GET /exam-attempts/{id}/result`
- Query param history → score berubah dari `result_id` ke `attempt_id`

### 📋 Next Tasks (Rekomendasi Urutan Pengerjaan)
1. **Global Exam Settings Integration** (P3 #1) — paling urgent karena BE admin sudah bisa atur
2. **Media Proxy / Signed URL** (P3 #2) — security fix
3. **Auto-submit on violation limit** (P1 #6) — terkait dengan #1
4. **Dynamic violation limits** (P1 #7) — terkait dengan #1
5. **Exam Type di Register** (P4 #3) — quick fix
6. **Show Result to User enforcement** (P3 #4) — polish
7. **Edit Profile** (P4 #2) — butuh BE support dulu
8. **Forgot Password** (P4 #1) — butuh BE support dulu
9. **Account Status Notification** (P3 #3) — nice to have
10. **E2E tests execution** (P2 #3) — bisa parallel

---

## Prioritas Sprint

### P0 - Integrasi Wajib ✅ DONE
- ✅ Login real API
- ✅ Register real API
- ✅ Route guard (server + client)
- ✅ Dashboard real API
- ✅ Upload payment proof
- ✅ Start/resume exam
- ✅ Get question/save answer/submit

### P1 - Exam Reliability ✅ DONE (core)
- ✅ Fullscreen + tab switch detection
- ✅ Violation logging client events
- ✅ Result visibility
- ✅ Loading skeleton
- ❌ Auto-submit on violation limit (butuh global settings)
- ❌ Dynamic max tab/fullscreen limit (butuh global settings)

### P2 - Polish
- ✅ Better UX for retake
- ✅ Richer history and profile
- ✅ State management layer
- ✅ Responsive & accessibility
- ⏸️ E2E tests execution & refinement (Ditunda)

### P3 - Backend Integration & Settings (ACTIVE)
- ❌ Global exam settings integration (`auto_submit`, `max_tab_switch`, `max_fullscreen_exit`, `shuffle`, `show_result`)
- ❌ Media proxy signed URL authentication
- ❌ Real-time/polling notifikasi status akun global
- ⚠️ Show result enforcement

### P4 - Missing Features / Nice to Have
- ⚠️ Forgot password (menunggu BE)
- ❌ Edit profile (menunggu BE)
- ❌ Connect exam_type saat register
