# FE-CBT Task Breakdown

Dokumen ini adalah breakdown pekerjaan frontend peserta `fe-cbt` berdasarkan kondisi repo saat ini dan kontrak API backend terbaru.

**Last Updated:** 2026-05-17

---

## Ringkasan Status

`fe-cbt` adalah aplikasi Next.js 14 untuk peserta CBT. Mayoritas alur utama sudah memakai real API backend, dengan beberapa bagian yang masih menunggu endpoint BE.

| Area | Status |
|------|--------|
| **Auth Flow** | DONE - register, login, session, logout |
| **Payment Proof** | DONE - upload file dan history real API, limit 5MB sesuai BE terbaru |
| **Dashboard** | DONE - real data dari profile, approval, session, active attempt, results |
| **Exam Engine** | DONE - start/resume, question, answer save, doubtful, navigate, heartbeat, submit |
| **Audio Player** | DONE - log audio-play memakai `question_id` sesuai BE |
| **Anti-Cheat** | DONE - fullscreen, tab switch, right-click/copy prevention, violation logging memakai `violation_type` |
| **Hasil Ujian** | DONE - total score dan answer stats, tanpa section score karena BE sudah drop section score columns |
| **API Integration** | DONE - adapter response ada di `lib/auth-api.ts` untuk menormalkan response BE terbaru |
| **Server Middleware** | DONE - `middleware.ts` aktif, `/reset-password` sudah ditambahkan ke public paths |
| **Global UI States** | DONE - `loading.tsx`, `error.tsx`, `not-found.tsx` |
| **E2E Tests** | DONE - Playwright updated untuk kontrak FE/BE terbaru dan berhasil dijalankan |
| **Forgot Password** | DONE — `/forgot-password` & `/reset-password` fungsional |
| **Edit Profile** | DONE — `PATCH /api/my/profile` integrated, form inline dengan React Query mutation |
| **Dynamic Violation Limits** | DONE — `GET /api/settings/exam` integrated, anti-cheat menggunakan nilai dari backend |
| **next/image** | DONE — gambar soal menggunakan `next/image` dengan signed URL support |
| **Dead Code Cleanup** | DONE — `FlowNav` dihapus dari semua halaman |
| **Exam Page Refactor** | DONE — `QuestionPanel`, `QuestionNavigator`, `SubmitExamModal`, `ViolationModal` diekstrak |

---

## Yang Sudah Selesai

### 1. Environment dan API Client

- `NEXT_PUBLIC_API_BASE_URL` dengan fallback ke `http://localhost:8000/api`.
- `apiRequest` support JSON dan FormData.
- Token participant disimpan di `localStorage` dan disinkronkan ke cookie untuk middleware.
- Interceptor behavior:
  - 401 clear token dan redirect login.
  - 403 redirect waiting approval hanya untuk error status akun, bukan semua error exam.
  - 422 field errors dinormalisasi.
- Typed wrappers dan adapter endpoint ada di `lib/auth-api.ts`.
- Adapter response terbaru sudah menangani:
  - `startExam`
  - `resumeExam`
  - `getQuestion`
  - `submitExam`
  - `getAttemptResult`
  - `getResultHistory`

### 2. Auth Pages

- Login real API ke `POST /api/login`.
- Register real API ke `POST /api/register`.
- Loading state dan disabled submit.
- Role check: hanya `participant` yang boleh masuk FE peserta.
- Account status redirect ke waiting approval.
- Logout participant.
- Validasi register mengikuti BE terbaru:
  - `name`
  - `email`
  - `password`
  - `password_confirmation`
  - `phone` 11-13 digit
  - `institution`

Catatan: `exam_type` tidak dikirim dari FE karena BE `RegisterRequest` terbaru tidak menerima field tersebut.

### 3. Payment Proof Peserta

- Page `/payment-proof` untuk upload dan history.
- Form upload sekarang hanya mengirim:
  - `file`
- Field lama `amount` dan `payment_date` sudah tidak dipakai karena BE migration terbaru menghapus kolom tersebut.
- Validasi FE:
  - JPG/JPEG/PNG/PDF
  - max 5MB, sesuai BE `max:5120`
  - required file
- History menampilkan:
  - `file_name`
  - `status`
  - `created_at`
  - `rejection_reason` jika ada
- CTA upload ulang tersedia saat proof ditolak.

### 4. Waiting Approval / Account Status

- Fetch status via `GET /api/me`.
- Polling status akun setiap 30 detik.
- Tampilan mengikuti status:
  - pending
  - rejected
  - active
- Jika status berubah menjadi active, user diberi notifikasi dan redirect countdown ke dashboard.

### 5. Dashboard Peserta

- Fetch real data:
  - `GET /api/my/profile`
  - `GET /api/my/test-approvals`
  - `GET /api/my/exam-sessions`
  - `GET /api/my/active-attempt`
  - `GET /api/my/results`
- Tombol start/resume mengikuti state API.
- Empty state untuk data kosong.
- Dashboard result visibility mengikuti `show_result_to_user`.
- Data result sudah dinormalisasi agar cocok dengan response `ExamResultResource` terbaru.

### 6. Exam Detail dan Instruction

- Detail session dari backend via daftar session peserta.
- Start exam real API ke `POST /api/exam-sessions/{session_id}/start`.
- Jika sudah ada active attempt, FE redirect resume ke `/exam?attempt_id=...`.
- Error bisnis ditampilkan ke user.

### 7. Exam Engine

- Load active attempt dari:
  - `GET /api/my/active-attempt`
  - query `attempt_id`
- Load question via `GET /api/exam-attempts/{attempt}/questions/{number}`.
- Adapter question terbaru membaca struktur BE:
  - `display_number`
  - `snapshot.stem_html`
  - `snapshot.image_url`
  - `snapshot.audio_url`
  - `snapshot.audio_max_play_count`
  - `options`
  - `selected_option_id`
  - `is_doubtful`
- Save answer ke `POST /api/exam-attempts/{attempt}/answers`.
- Mark doubtful ke `POST /api/exam-attempts/{attempt}/mark-doubt`.
- Navigate ke `POST /api/exam-attempts/{attempt}/navigate`.
- Timer dari `remaining_seconds`.
- Heartbeat berkala.
- Submit final dengan confirmation modal.
- Auto-resume saat refresh.
- Auto-submit saat timer habis.

### 8. Audio Player

- Render audio dari `audio_url`.
- Play count limit dari backend.
- Log audio-play ke:
  - `POST /api/exam-attempts/{attempt}/audio-play`
- Payload sudah disesuaikan dengan BE terbaru:

```json
{
  "question_id": 123
}
```

- Response BE `max_play_count` dinormalisasi menjadi `max_play` untuk UI.

### 9. Anti-Cheat Client Events

- Fullscreen enforcement.
- Detect fullscreen exit.
- Detect visibility change/tab switch.
- Disable right click/copy/cut/paste/selectstart di area exam.
- Violation logging ke:
  - `POST /api/exam-attempts/{attempt}/violations`
- Payload sudah disesuaikan dengan BE terbaru:

```json
{
  "violation_type": "tab_switch",
  "severity": "medium"
}
```

- Warning modal muncul saat threshold terlewati.
- Auto-submit menggunakan fallback FE karena endpoint public settings belum tersedia untuk peserta.

### 10. Exam Completed, Result, History

- Completed page membaca submit response melalui adapter.
- Result visibility mengikuti:
  - `show_result_to_user`
  - `show_result`
- Detail skor memakai endpoint dedicated:
  - `GET /api/exam-attempts/{attempt}/result`
- History fetch dari:
  - `GET /api/my/results`
- History detail link memakai `exam_attempt_id`, bukan `result.id`.
- UI skor per-section sudah dihapus karena BE migration terbaru menghapus:
  - `listening_score`
  - `structure_score`
  - `reading_score`
- UI hasil sekarang menampilkan:
  - total score
  - correct count
  - wrong count
  - unanswered count

### 11. Profile Page

- Fetch `GET /api/my/profile`.
- Menampilkan:
  - name
  - email
  - phone
  - institution
  - account_status

Catatan: **edit profile sudah tersedia di BE** (`PATCH /api/my/profile`) — FE tinggal integrate.

### 12. Route Guard

- Public route:
  - `/`
  - `/login`
  - `/register`
  - `/forgot-password`
  - `/waiting-approval`
  - `/panduan`
- Auth route:
  - `/dashboard`
  - `/profile`
  - `/payment-proof`
  - `/exam/*`
- Middleware redirect user tanpa token ke login.
- Client `AuthGuard` tetap memvalidasi session via API.
- User non-participant diarahkan keluar dari FE peserta.

### 13. Global UI States

- `app/loading.tsx`.
- `app/error.tsx`.
- `app/not-found.tsx`.

### 14. Polish

- Loading skeleton:
  - dashboard
  - history
  - exam
  - profile
- State management memakai `@tanstack/react-query`.
- Responsive mobile layout untuk exam.
- Accessibility dasar:
  - focus states
  - aria dialog
  - keyboard-friendly controls
- Lucide icons dipakai di dashboard/profile/payment.

### 15. Testing dan QA

- E2E Playwright files tersedia:
  - `e2e/auth.spec.ts`
  - `e2e/login.spec.ts`
  - `e2e/protected-routes.spec.ts`
  - `e2e/dashboard.spec.ts`
  - `e2e/exam.spec.ts`
  - `playwright.config.ts`
- Testing terakhir yang sudah dijalankan pada FE:
  - `npx tsc --noEmit --pretty false` - PASS
  - `npm run lint` - PASS
  - `npm run build` - PASS
  - `npm run test:e2e` - PASS, 41 passed / 13 skipped karena skenario auth/data-dependent dan 1 loading-state test yang flaky.

---

## Yang Masih Belum / Perlu Perbaikan

### P1 - Exam Reliability

| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | Fullscreen enforcement | DONE | Request fullscreen saat exam start, detect exit |
| 2 | Tab switch detection | DONE | Detect visibility change, kirim violation |
| 3 | Disable right click/copy | DONE | Di area exam |
| 4 | Warning modal violation | DONE | Saat threshold terlewati |
| 5 | Route guard saat exam aktif | DONE | Cegah keluar tanpa konfirmasi |
| 6 | Auto-submit on violation limit | DONE | FE fallback aktif, sudah integrate `GET /api/settings/exam` |
| 7 | Dynamic violation limits | DONE | `GET /api/settings/exam` public tersedia — FE sudah integrate |

### P2 - Polish & Optimization

| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | Loading skeleton | DONE | Dashboard, history, exam, profile |
| 2 | State management | DONE | `@tanstack/react-query` |
| 3 | E2E tests execution | DONE | Playwright suite sudah disesuaikan kontrak terbaru dan pass |
| 4 | Responsive mobile exam | DONE | Layout soal dan navigasi grid |
| 5 | Accessibility | DONE | Focus state, aria, keyboard navigation dasar |
| 6 | Score detail fetch | DONE | `getAttemptResult()` dedicated endpoint |

### P3 - Backend Integration & Settings

| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | Global Exam Settings Integration | DONE | `GET /api/settings/exam` sudah diintegrasikan |
| 2 | Media Proxy / Signed URL | DONE | FE memakai `image_url`/`audio_url`, ada `onError` handler |
| 3 | Account Status Notification | DONE | Polling `GET /api/me` setiap 30 detik, tidak polling saat exam |
| 4 | Show Result to User enforcement | DONE | History, Dashboard, Completed, Score mengikuti visibility |

### P4 - Missing Features / Placeholder

| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | Forgot Password | DONE | `/forgot-password` & `/reset-password` sudah fungsional |
| 2 | Edit Profile | DONE | `PATCH /api/my/profile` sudah integrate di `ProfileContent.tsx` |
| 3 | Exam Type di Register | NOT ACTIVE | BE terbaru tidak menerima `exam_type`, jadi FE tidak mengirim field ini |

### P5 - Client Feedback Refinement

| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | Validasi No. WhatsApp | DONE | FE validasi 11-13 digit, sama dengan BE |
| 2 | Sederhanakan Upload Bukti Bayar | DONE | Hanya upload file; `amount` dan `payment_date` dihapus |

---

## Perubahan Terbaru

### 17 Mei 2026

**Audit & Bug Fix:**
- **BUG FIX KRITIS**: `/reset-password` ditambahkan ke `PUBLIC_PATHS` di `middleware.ts` — sebelumnya user tidak bisa reset password dari link email karena middleware redirect ke `/login`.
- **FIX**: Timer warning color di `ExamHeader` sekarang menggunakan `remainingSeconds` (prop numerik) sebagai basis perbandingan, bukan string comparison yang tidak akurat untuk jam > 0. Timer berubah merah + pulse saat ≤ 5 menit tersisa.
- **CLEANUP**: Removed unused `useRouter` dari `ExamPage` dan `usePathname` dari `AuthGuard` yang menyebabkan lint error.
- **FIX TS**: Type assertion `TS2352` pada violation response di `exam/page.tsx` diperbaiki dengan double cast `as unknown as Record`.

**Refactor Exam Page:**
- `QuestionPanel` diekstrak ke `components/exam/QuestionPanel.tsx` — berisi stem, image, audio player, options, dan nav prev/next.
- `QuestionNavigator` diekstrak ke `components/exam/QuestionNavigator.tsx` — sidebar grid navigasi soal.
- `SubmitExamModal` diekstrak ke `components/exam/SubmitExamModal.tsx` — modal konfirmasi submit ujian.
- `ViolationModal` diekstrak ke `components/exam/ViolationModal.tsx` — modal pelanggaran dengan auto-submit state.
- `audioRef` sekarang dikelola di dalam `QuestionPanel`, tidak di parent `ExamPage`.
- `exam/page.tsx` berkurang dari 847 baris menjadi ~627 baris, hanya berisi state logic dan efek.

**Verifikasi:**
- `npx tsc --noEmit --pretty false` — PASS
- `npm run lint` — PASS (hanya 1 warning pre-existing di `payment-proof/[id]`)
- `npm run build` — PASS (exit code 0, 20 routes)

### 13 Mei 2026

- Navbar `History` disembunyikan untuk guest dan hanya muncul setelah participant login.
- Exam page mulai dipecah ke komponen kecil; `ExamHeader` sudah diekstrak dari `app/exam/page.tsx`.
- Resume exam diperbaiki agar initial question yang dimuat mengikuti `current_question_number` dari backend, bukan selalu state awal soal 1.
- Audio play tidak lagi fallback dari `question_id` ke `attempt_question.id`; jika backend tidak mengirim `question_id`, FE menampilkan error aman dan tidak mengirim payload yang salah.
- BE menambahkan `section_type` pada question bank (termasuk `fahm_al_maqru`, `fahm_al_kitabah`, `fahm_al_masmu`) dan server akan menonaktifkan shuffle untuk bank dengan `section_type = fahm_al_masmu`.
- E2E tests disesuaikan dengan kontrak terbaru:
  - register tidak lagi memakai `exam_type`
  - payment proof tidak lagi memakai `amount` dan `payment_date`
  - limit upload proof disamakan ke 5MB
  - URL assertion tidak hardcode `localhost:3000`
- Verifikasi:
  - `npx tsc --noEmit --pretty false` - PASS
  - `npm run lint` - PASS
  - `npm run build` - PASS
  - `npm run test:e2e` - PASS, 41 passed / 13 skipped

### 12 Mei 2026

- Sinkron FE dengan BE terbaru setelah push backend/admin.
- Payment proof limit disamakan menjadi 5MB sesuai `StorePaymentProofRequest` BE (`max:5120`).
- Payment proof form tetap hanya mengirim `file`; field `amount` dan `payment_date` tidak lagi dikirim.
- Menambahkan adapter response di `lib/auth-api.ts` untuk:
  - `startExam`
  - `resumeExam`
  - `getQuestion`
  - `submitExam`
  - `getAttemptResult`
  - `getResultHistory`
- Adapter question menormalisasi response BE terbaru dari `snapshot.*` ke shape UI yang dipakai exam page.
- Audio play payload diubah dari `question_number` menjadi `question_id`.
- Violation payload diubah dari `type` menjadi `violation_type`.
- History score link diubah memakai `exam_attempt_id`.
- UI skor per-section peserta dihapus karena BE sudah drop section score columns.
- Unused variables dibersihkan sehingga lint kembali pass.
- Verifikasi:
  - `npx tsc --noEmit --pretty false` - PASS
  - `npm run lint` - PASS
  - `npm run build` - PASS

### Catatan Dari BE/Admin Terbaru

- BE punya media proxy `GET /api/media/{path}` dan signed URL untuk `image_url` / `audio_url`.
- BE settings exam saat ini tersedia di admin route:
  - `GET /api/admin/settings/exam`
  - `PATCH /api/admin/settings/exam`
- ✅ Public participant route tersedia:
  - `GET /api/settings/exam` (tanpa auth, return safe defaults)
- BE result resource terbaru tidak mengirim section score.
- Admin repo masih punya beberapa UI/typing lama terkait section score/section type. Itu di luar scope FE peserta, tetapi bisa memengaruhi konsistensi data admin.

---

## Next Tasks

Semua task utama sudah selesai. Tidak ada task FE yang menunggu BE saat ini.

**Sisa pekerjaan (opsional / nice to have):**

1. **E2E test update** — Playwright tests perlu dijalankan ulang untuk memverifikasi komponen baru (`QuestionPanel`, `QuestionNavigator`, `SubmitExamModal`, `ViolationModal`).

2. **Optimasi `payment-proof/[id]`** — Ada warning `<img>` vs `<Image />` dari Next.js. Jika halaman ini aktif, pertimbangkan migrasi ke `next/image`.

---

## Prioritas Sprint

### P0 - Integrasi Wajib - DONE

- Login real API.
- Register real API.
- Route guard server + client.
- Dashboard real API.
- Upload payment proof.
- Start/resume exam.
- Get question/save answer/submit.

### P1 - Exam Reliability - DONE

- Fullscreen + tab switch detection.
- Violation logging client events.
- Result visibility.
- Loading skeleton.
- Auto-submit on violation limit, FE fallback aktif dan sudah integrate dynamic settings dari BE.
- Dynamic max tab/fullscreen limit dari `GET /api/settings/exam`.

### P2 - Polish - DONE

- Better UX for retake.
- Richer history and profile.
- State management layer.
- Responsive and accessibility.
- Timer warning color akurat menggunakan detik (bukan string comparison).

### P3 - Backend Integration & Settings - DONE

- Global exam settings integration: done via `GET /api/settings/exam`.
- Media proxy / signed URL authentication: done.
- Real-time/polling notifikasi status akun: done.
- Show result enforcement: done.

### P4 - Missing Features - DONE

- Forgot password: done (`/forgot-password` & `/reset-password` fungsional).
- Edit profile: done (`ProfileContent.tsx` sudah inline form + `PATCH /api/my/profile`).
- Exam type register: tidak aktif karena tidak ada di kontrak BE terbaru.

### P5 - Client Feedback Refinement - DONE

- Validasi No. WhatsApp.
- Upload bukti bayar disederhanakan.

### P6 - Refactor & Code Quality - DONE

- Exam page refactor: `QuestionPanel`, `QuestionNavigator`, `SubmitExamModal`, `ViolationModal` diekstrak.
- Bug fix: `/reset-password` ditambahkan ke `PUBLIC_PATHS` middleware.
- Unused imports dibersihkan (`router` di `ExamPage`, `pathname` di `AuthGuard`).
- TypeScript error TS2352 pada `exam/page.tsx` diperbaiki.
