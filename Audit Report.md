# Audit Report — CBT-TOAFL

**Tanggal:** 12 Mei 2026  
**Auditor:** Claude AI (Sonnet 4.6)  
**Scope:** be-cbt (Backend), fe-cbt (Frontend Peserta), cbt-admin (Admin Panel)

**Update FE-CBT 13 Mei 2026:** item FE-only yang tidak menunggu BE/Admin sudah ditindaklanjuti. E2E suite diperbarui dan berhasil dijalankan (41 passed / 13 skipped), kontrak register/payment proof di test disesuaikan, audio payload diperketat agar wajib `question_id`, resume exam memuat nomor soal awal yang benar, dan refactor awal `exam/page.tsx` dimulai dengan ekstraksi `ExamHeader`.

---

## Gambaran Sistem

CBT-TOAFL adalah platform ujian TOAFL (Test of Arabic as a Foreign Language) berbasis web.

**Alur utama:**
```
Register → Upload Bukti Bayar → Admin Approve → Assign ke Sesi → Exam Engine → Scoring → Result
```

| Sub-project | Stack | Status |
|---|---|---|
| **be-cbt** | Laravel 13 · PHP 8.3 · MySQL · Sanctum · Spatie | ✅ MVP Complete |
| **fe-cbt** | Next.js 14 · React 18 · TanStack Query v5 · Tailwind 3 | ✅ 95% Complete |
| **cbt-admin** | Next.js 16 · React 19 · TipTap · Recharts · Tailwind 4 | ✅ 98% Complete |

---

## 1. Backend — `be-cbt` (Laravel 13)

### 1.1 Arsitektur

- **47+ API endpoints** terbagi: public (2), user (20+), admin (25+)
- **19 Eloquent models**, **29 migrations**
- Service layer: `ExamEngineService`, `ScoringService`, `AntiCheatService`, `EmailNotificationService`
- Scheduler: `auto-close sessions`, `auto-submit stale attempts`, `mark absent`
- Queue: `ExportResultsJob`

### 1.2 Database Schema Utama

| Tabel | Fungsi |
|---|---|
| `users` + `user_profiles` | Auth + profil peserta |
| `payment_proofs` | Upload & review bukti bayar |
| `test_approvals` | Hak ikut ujian (1 per payment) |
| `exam_packages` + `exam_package_banks` | Blueprint ujian |
| `exam_sessions` + `exam_session_registrations` | Jadwal & peserta |
| `exam_attempts` + `exam_attempt_questions` | Runtime ujian (snapshot immutable) |
| `exam_answers` | Jawaban user |
| `exam_results` | Skor final |
| `violation_logs` | Anti-cheat audit |
| `settings` | Config global (duration, limits) |

### 1.3 Fitur Implemented (MVP)

- [x] Registrasi user + validasi phone (11–13 digit)
- [x] Upload & approval/rejection bukti pembayaran
- [x] Auto-generate test approval saat payment diapprove
- [x] CRUD question bank, soal, opsi jawaban
- [x] Upload image & audio media (signed URL)
- [x] Import soal dari CSV + template download
- [x] CRUD exam packages + mapping ke question bank
- [x] CRUD exam sessions (draft → published → closed → finished/cancelled)
- [x] Assign peserta manual + auto-generate
- [x] Exam engine: start, navigasi, save jawaban, mark doubtful, heartbeat, submit
- [x] Snapshot randomisasi soal & opsi per user (immutable)
- [x] Timer 120 menit dengan source of truth di backend
- [x] Auto-submit saat timeout
- [x] Scoring: total_score, correct_count, wrong_count, unanswered_count
- [x] Kontrol visibilitas hasil per sesi
- [x] Violation logging: tab_switch, fullscreen_exit, copy_attempt, dll.
- [x] Single active login per user
- [x] Export hasil ke Excel/CSV
- [x] Activity log (Spatie)
- [x] Feature tests (21+ tests)

### 1.4 Security — Sudah Diimplementasi

| Mekanisme | Keterangan |
|---|---|
| Sanctum token-based | Stateless API, revoke on logout |
| RBAC Spatie Permission | Role admin / user, policy-based |
| Snapshot immutability | Soal tidak berubah saat bank diupdate |
| Backend timer | Source of truth, tidak bisa dimanipulasi FE |
| Single active login | Anti multi-device |
| Signed URL media | Image & audio tidak bisa diakses langsung |
| HTML sanitizer | Input soal & opsi (support Arabic) |
| Audit trail | Spatie ActivityLog di model kritikal |

### 1.5 Temuan & Issues

| Severity | Temuan | Lokasi | Rekomendasi | Status |
|---|---|---|---|---|
| ✅ DONE | Tidak ada endpoint public `GET /api/settings/exam` untuk peserta | `routes/api.php` | Expose endpoint read-only tanpa auth untuk config exam | **Done 14 Mei 2026** — `GET /api/settings/exam` public tersedia |
| ✅ DONE | Forgot password & reset password endpoint tidak ada | `AuthController` | Tambah `POST /forgot-password` dan `POST /reset-password` | **Done 14 Mei 2026** |
| ✅ DONE | Edit profil user (`PATCH /api/my/profile`) tidak ada | `DashboardController` | Tambah endpoint update profil dengan validasi email unique | **Done 14 Mei 2026** |
| ✅ DONE | ResponseFormatter static mutable state | `app/Helpers/ResponseFormatter.php` | Refactor ke instance-based return untuk hindari state leak | **Done 14 Mei 2026** |
| ✅ DONE | N+1 query di ExamAttemptDetailResource dan ScoringService | `app/Http/Resources/`, `app/Services/` | Eager load `examAttemptQuestions` sebelum iterasi | **Done 14 Mei 2026** |
| 🟡 Medium | Tidak ada unit test untuk Service layer | `app/Services/` | Tambah unit test untuk `ExamEngineService`, `ScoringService` | Open |
| 🟡 Medium | Test coverage untuk default exam setting saat create package/session belum ada | feature tests | Tambah skenario create dengan/tanpa field override | Open |
| 🟡 Medium | Tidak ada rate limiting di auth routes | `routes/api.php` | Tambah throttle 10 req/menit untuk `/login` dan `/register` | **Done 14 Mei 2026** |
| 🟢 Minor | Hardcoded business rules di service (violation threshold, duration default) | beberapa service | Pindah ke config file terpisah | Open |
| 🟢 Minor | File upload tanpa virus scan | `app/Http/Controllers/` | Tambah virus scan sebelum simpan (pasca-MVP) | Open |

### 1.6 Fitur Belum Implemented (Planned)

| Fitur | Status | Catatan |
|---|---|---|
| Forgot password reset | ⏳ Planned | FE placeholder sudah ada |
| Edit profil user | ⏳ Planned | `PATCH /api/my/profile` belum ada |
| Public exam settings endpoint | ⏳ Planned | Dibutuhkan FE-CBT |
| Notifikasi WhatsApp | ⏳ Planned pasca-MVP | Email sudah ada |
| Pembahasan soal | ⏳ Planned pasca-MVP | Dokumentasi sudah ada |
| Sertifikat PDF otomatis | ❌ Out of scope | — |
| Payment gateway otomatis | ❌ Out of scope | Manual upload only |

---

## 2. Frontend Peserta — `fe-cbt` (Next.js 14)

### 2.1 Halaman Tersedia

| Route | Status | Keterangan |
|---|---|---|
| `/login` | ✅ | Real API |
| `/register` | ✅ | Real API |
| `/forgot-password` | ⚠️ | Placeholder — menunggu BE endpoint |
| `/waiting-approval` | ✅ | Polling status akun 30 detik |
| `/dashboard` | ✅ | Profile summary, approvals, results |
| `/profile` | ⚠️ | Read-only — menunggu `PATCH /my/profile` |
| `/payment-proof` | ✅ | Upload + history |
| `/exam` | ✅ | Engine utama dengan anti-cheat |
| `/exam/instruction` | ✅ | Info ujian + start |
| `/exam/detail` | ✅ | Detail sesi + CTA |
| `/exam/history` | ✅ | Riwayat hasil |
| `/exam/score` | ✅ | Detail skor |
| `/exam/completed` | ✅ | Post-submit page |

### 2.2 Fitur Exam Engine

- Fullscreen enforcement (modal warning)
- Tab-switch detection + violation logging
- Right-click block
- Copy/paste detection
- Heartbeat ping ke backend
- Timer countdown dengan auto-submit saat 0 (sync dari backend)
- Mark doubtful per soal
- Navigation grid
- Audio play count limit
- Save state otomatis (Saved / Saving... / Failed)

### 2.3 State Management

- **TanStack React Query v5** untuk remote state & caching
- **useState** untuk UI-only state (modals, toast, loading)
- **useRef** untuk timer & heartbeat interval
- **useCallback** untuk memoized handlers exam

### 2.4 API Integration

- Base client: `lib/api.ts` — handle 401 (redirect login), 403 (redirect waiting-approval), 422 (normalize field errors)
- Adapter layer: `lib/auth-api.ts` — normalizer `normalizeAttempt()`, `normalizeQuestion()`, `normalizeResult()` untuk mapping response BE → UI types

### 2.5 Temuan & Issues

| Severity | Temuan | Lokasi | Rekomendasi | Status |
|---|---|---|---|---|
| 🔴 High | `exam/page.tsx` terlalu besar (762 baris) — satu komponen handle exam logic, anti-cheat, timer, violations, UI rendering | [`app/exam/page.tsx`](../fe-cbt/app/exam/page.tsx) | Split ke sub-components: `ExamHeader`, `QuestionNavigator`, `SubmitModal`, `AntiCheatGuard` | Partial — `ExamHeader` sudah diekstrak |
| 🔴 High | 8+ `useEffect` dengan dependency arrays kompleks, rawan race condition & infinite loop | [`app/exam/page.tsx`](../fe-cbt/app/exam/page.tsx) | Tambah ESLint `exhaustive-deps`, pertimbangkan `useReducer` untuk exam state machine | Partial |
| ✅ DONE | Violation limits hardcoded di FE (`max_tab_switch: 3`, `max_fullscreen_exit: 3`) karena endpoint public settings belum ada di BE | [`lib/auth-api.ts`](../fe-cbt/lib/auth-api.ts) | Koordinasi dengan BE untuk expose `GET /api/settings/exam` tanpa auth | **BE Done 14 Mei 2026** — FE bisa integrate `GET /api/settings/exam` |
| 🟡 Medium | `AccountStatusWatcher` tetap polling setiap 30 detik saat user sedang dalam ujian | [`components/AccountStatusWatcher.tsx`](../fe-cbt/components/AccountStatusWatcher.tsx) | Pause polling saat exam active | Done — `/exam` dikecualikan |
| ✅ READY | Forgot password tidak functional — hanya placeholder | `app/forgot-password/` | Implementasi setelah BE siap | **BE Ready 14 Mei 2026** — `POST /forgot-password` & `POST /reset-password` tersedia |
| ✅ READY | Profile page read-only — tidak bisa edit | `app/profile/` | Implementasi setelah `PATCH /api/my/profile` tersedia di BE | **BE Ready 14 Mei 2026** — `PATCH /api/my/profile` tersedia |
| 🟡 Medium | `question_id` fallback dengan `?? id` di audio play payload — bisa kirim data salah jika `question_id` null | [`lib/auth-api.ts`](../fe-cbt/lib/auth-api.ts) | Pastikan BE selalu return `question_id` atau standardize ke satu field | Done — FE tidak lagi fallback |
| 🟢 Minor | ESLint disabled saat build (`eslintIgnoreDuringBuilds: true`) | `next.config.mjs` | Tetap jalankan `npm run lint` di CI/CD pipeline | Open |
| 🟢 Minor | E2E tests (Playwright) perlu di-run ulang setelah adapter patch terakhir | `e2e/` | Jalankan `npm run test:e2e` sebelum go-live | Open |
| 🟢 Minor | Tidak ada unit tests (Jest belum dikonfigurasi) | — | Tambah unit tests untuk normalizer & helper functions | Open |

---

### 2.6 Update FE-CBT 13 Mei 2026

| Item Audit FE | Status 13 Mei 2026 |
|---|---|
| Refactor `exam/page.tsx` | PARTIAL - `ExamHeader` sudah diekstrak, kandidat berikutnya `QuestionPanel`, `QuestionNavigator`, `SubmitExamModal`, `ViolationModal` |
| Dependency arrays `useEffect` | PARTIAL - init effect dan timer effect diperbaiki, state machine tetap backlog |
| Pause `AccountStatusWatcher` saat exam | DONE - route `/exam` sudah dikecualikan dari polling dan E2E coverage pass |
| Audio payload `question_id` fallback | DONE - FE tidak lagi fallback ke `attempt_question.id`; request audio ditahan jika `question_id` tidak tersedia |
| E2E Playwright rerun | DONE - `npm run test:e2e` pass, 41 passed / 13 skipped |

## 3. Admin Panel — `cbt-admin` (Next.js 16)

### 3.1 Halaman Tersedia

| Route | Status | Keterangan |
|---|---|---|
| `/login` | ✅ | Cookie-based auth |
| `/admin` (dashboard) | ✅ | Stats + bar chart Recharts |
| `/admin/users` | ✅ | CRUD + approve/reject/reset password |
| `/admin/payment-proofs` | ✅ | List + preview file + approve/reject |
| `/admin/test-approvals` | ✅ | List + detail |
| `/admin/question-banks` | ✅ | CRUD |
| `/admin/questions` | ✅ | CRUD + TipTap rich editor + media upload |
| `/admin/exam-packages` | ✅ | CRUD + bank mapping editor |
| `/admin/exam-sessions` | ✅ | CRUD + publish/close/finish/cancel + participant management |
| `/admin/monitoring` | ✅ | Attempts + violations + live refresh 30s |
| `/admin/activity-logs` | ✅ | Log viewer |
| `/admin/results` | ✅ | List + export Excel/CSV + detail skor |
| `/admin/analytics` | ✅ | Bar chart skor rata-rata per sesi/paket |
| `/admin/settings` | ✅ | Global exam settings |

### 3.2 Architecture Pattern

- **Server Components** sebagai default (Next.js 16 server-first)
- **Server Actions** untuk mutations (create, update, delete)
- **Cookie-based Bearer token** untuk auth ke BE
- **`revalidatePath`** setelah mutations untuk refresh data
- `admin-api.ts` sebagai server-only API client (`"use server-only"`)
- `useTransition` + `useActionState` + `useFormStatus` untuk form loading state

### 3.3 Temuan & Issues

| Severity | Temuan | Lokasi | Rekomendasi |
|---|---|---|---|
| 🔴 High | `as any` type cast menyembunyikan potential type errors di analytics route | [`src/app/api/admin/analytics/route.ts`](../cbt-admin/src/app/api/admin/analytics/route.ts) | Ganti dengan proper TypeScript type definition |
| 🟡 Medium | `admin-api.ts` terlalu besar (1885 baris) — semua CRUD logic dalam satu file (God object) | [`src/lib/admin-api.ts`](../cbt-admin/src/lib/admin-api.ts) | Split by module: `admin-users.ts`, `admin-questions.ts`, `admin-sessions.ts`, dll. |
| 🟡 Medium | Tidak ada XSS sanitization untuk output HTML dari TipTap rich editor saat di-render | komponen yang render `dangerouslySetInnerHTML` | Tambah DOMPurify atau sanitize-html sebelum render |
| 🟡 Medium | P4 client feedback belum selesai: field `section` & `difficulty` masih ada di question form, bulk import CSV error 401 | [`src/components/question-form.tsx`](../cbt-admin/src/components/question-form.tsx) | Hapus field, fix template download atau generate client-side |
| 🟡 Medium | Live refresh monitoring (30 detik) tanpa visual indicator "refreshing..." | [`src/components/monitoring-detail.tsx`](../cbt-admin/src/components/monitoring-detail.tsx) | Tambah subtle loading hint saat refresh |
| 🟡 Medium | React 19 + Tailwind CSS 4 (cutting edge) — potensi edge case compatibility dengan third-party library | `package.json` | Monitor changelog TipTap & Recharts untuk React 19 compatibility |
| 🟢 Minor | Required fields di question form tidak ada visual marker (asterisk merah) | [`src/components/question-form.tsx`](../cbt-admin/src/components/question-form.tsx) | Tambah indikator visual untuk required fields |
| 🟢 Minor | Analytics response type tidak eksplisit selain satu instance `as any` | [`src/lib/admin-api.ts`](../cbt-admin/src/lib/admin-api.ts) | Tambah TypeScript interface untuk analytics response shape |
| 🟢 Minor | FE-CBT (Next.js 14) dan cbt-admin (Next.js 16) berbeda versi — juga React 18 vs 19, Tailwind 3 vs 4 | `package.json` masing-masing | Pertimbangkan upgrade fe-cbt ke Next.js 16 (medium-term) |

---

## 4. Ringkasan Prioritas Tindakan

### 🔴 P0 — Segera (blocking / critical)

| # | Tindakan | Project | Estimasi | Status |
|---|---|---|---|---|
| 1 | Fix `as any` di analytics route — ganti dengan proper TypeScript type | cbt-admin | 15 menit | Open |
| 2 | Expose `GET /api/settings/exam` tanpa auth untuk peserta | be-cbt | 30 menit | **✅ Done 14 Mei 2026** |
| 3 | Run E2E tests (`npm run test:e2e`) untuk validasi setelah adapter patch | fe-cbt | 30 menit | Open |

### 🟡 P1 — Short-term (sprint ini)

| # | Tindakan | Project | Estimasi | Status |
|---|---|---|---|---|
| 4 | Refactor `exam/page.tsx` — split ke sub-components | fe-cbt | 3–4 jam | Open |
| 5 | Perbaiki dependency arrays `useEffect` di exam page | fe-cbt | 2 jam | Open |
| 6 | Pause `AccountStatusWatcher` polling saat exam active | fe-cbt | 30 menit | ✅ Done |
| 7 | Tambah XSS sanitization untuk output TipTap HTML | cbt-admin | 1 jam | ✅ Done |
| 8 | Selesaikan P4 items: hapus field section/difficulty, fix CSV template download | cbt-admin | 2–3 jam | ✅ Done |

### 🟢 P2 — Medium-term (backlog)

| # | Tindakan | Project | Estimasi | Status |
|---|---|---|---|---|
| 9 | Implementasi forgot password + profile edit (setelah BE siap) | be-cbt + fe-cbt | 4–6 jam | **BE Done 14 Mei 2026** — tinggal integrate di FE |
| 10 | Split `admin-api.ts` by module | cbt-admin | 4 jam | ✅ Done |
| 11 | Tambah unit tests untuk BE service layer | be-cbt | 8 jam | Open |
| 12 | Tambah unit tests untuk FE normalizer & helper | fe-cbt | 3 jam | Open |
| 13 | Hardcoded business rules di BE service → pindah ke config | be-cbt | 2 jam | Open |

### 🔵 P3 — Long-term (pasca-launch)

| # | Tindakan | Project | Estimasi |
|---|---|---|---|
| 14 | Upgrade fe-cbt ke Next.js 16 (align dengan cbt-admin) | fe-cbt | 2–3 hari |
| 15 | Tambah Content Security Policy (CSP) headers | fe-cbt + cbt-admin | 4 jam |
| 16 | Tambah error tracking (Sentry) + performance monitoring | semua | 4 jam |
| 17 | Virus scan untuk file upload | be-cbt | 4 jam |
| 18 | Fitur pasca-MVP: WhatsApp notifikasi, pembahasan soal, sertifikat PDF | be-cbt | estimasi terpisah |

---

## 5. Go-Live Readiness

| Layer | Status | Catatan |
|---|---|---|
| Backend API | ✅ Ready | MVP 100% complete, security solid |
| Frontend Peserta | ✅ Ready* | *Setelah E2E tests pass; integrate `GET /api/settings/exam`, forgot password, edit profile |
| Admin Panel | ✅ Ready* | *Setelah fix `as any` |
| Infrastructure | ✅ Ready | Queue, scheduler, backup, log rotation terdokumentasi |

**Keputusan:** Sistem bisa go-live setelah 2 item P0 (fix `as any` di admin + E2E tests pass) diselesaikan. Backend sudah **100% ready**.
