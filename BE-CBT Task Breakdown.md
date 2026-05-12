# BE-CBT Task Breakdown

Dokumen ini adalah breakdown pekerjaan backend `be-cbt` berdasarkan kondisi repo saat ini, dokumen existing di `cbt-documentation`, dan kontrak API yang sudah muncul di `be-cbt/API_DOCUMENTATION.md`.

**Last Updated:** 2026-05-12 (rev. audit)

---

## Perubahan Terbaru (2026-05-12)

### Schema & Field Changes (Lanjutan)
- ✅ Removed `section_type` dari tabel `questions`, `exam_package_banks`, `exam_attempt_questions` (migration `2026_05_12_000000`)
- ✅ Removed `listening_score`, `structure_score`, `reading_score` dari tabel `exam_results` (migration `2026_05_12_000000`)
- ✅ Migration dilengkapi `try-catch` pada `dropIndex` agar aman dijalankan di VPS tanpa `migrate:fresh`

### Code Cleanup
- ✅ `ExamPackageController::syncBanks()` — dihapus `section_type` dari `ExamPackageBank::create()`
- ✅ `QuestionImportController` — dihapus validasi dan assignment `section_type` dari CSV import
- ✅ `NotificationService` dan template `result_published` — dihapus `listeningScore`, `structureScore`, `readingScore` dari email notifikasi
- ✅ `ResultsExport` — dihapus kolom section score dari heading dan mapping export Excel
- ✅ `ResponseFormatter::success()` sekarang reset `code/status/message/data` agar response error tidak bocor ke response sukses berikutnya
- ✅ `AuthController::logout()` menghapus bearer token Sanctum secara eksplisit dan reset guard cache setelah logout
- ✅ Factories dan feature tests dibersihkan dari kolom schema lama (`section_type`, `question_type`, `difficulty_level`, section score)

### Frontend (cbt-admin)
- ✅ Removed field `Code` display dari `QuestionBankForm`, `ExamPackageForm`, `ExamSessionForm` — code kini sepenuhnya di-generate backend
- ✅ Removed `Amount` dan `Payment Date` dari detail view bukti pembayaran di admin panel
- ✅ Removed `Type` dan `Difficulty` dari detail view soal di admin panel
- ✅ Removed `Question Type` dari halaman import soal admin karena `question_type` sudah tidak dipakai
- ✅ Create form `ExamPackage` dan `ExamSession` sekarang prefill dari `GET /api/admin/settings/exam`
- ✅ Create `ExamSession` tidak lagi mengirim `code` kosong; backend menerima `code` nullable agar auto-generate `SES-*` berjalan

---

## Perubahan Terbaru (2026-05-11)

### Schema & Field Changes
- ✅ Removed `question_type` dan `difficulty_level` dari `questions` table (migrations + code cleanup)
- ✅ Removed `amount` dan `payment_date` dari `payment_proofs` table (migrations + code cleanup)
- ✅ Migration files untuk drop column tersedia untuk production rollout

### New Features
- ✅ Auto-generate `code` untuk QuestionBank (prefix `QB-`), ExamPackage (`PKG-`), ExamSession (`SES-`) — generated saat create jika kosong
- ✅ Phone number validation: digits only, 11-13 characters (di `RegisterRequest` dan nullable)
- ✅ `registrations:mark-absent` command — auto-mark registrations absent jika session finished dan no attempt (scheduled setiap 5 menit)

---

## Ringkasan Status

Backend sudah memiliki fondasi Laravel API yang cukup lengkap:

| Area | Status |
|------|--------|
| **Auth** | ✅ Sanctum, role, middleware |
| **Database** | ✅ 29 migrations, 19 models, relasi lengkap |
| **API Endpoints** | ✅ ~74 endpoint (admin + user) |
| **Exam Engine** | ✅ Snapshot, randomisasi, timer, scoring |
| **Payment Proof** | ✅ Upload, approve, reject + preview endpoint (schema: removed amount, payment_date) |
| **Question API** | ✅ CRUD + validasi + bank sync (schema: removed question_type, difficulty_level, section_type) |
| **Question Import** | ✅ CSV import + template download (tidak perlu section_type lagi) |
| **Exam Package** | ✅ CRUD + bank sync tanpa section_type |
| **Exam Result** | ✅ Scoring hanya total_score (removed listening/structure/reading score) |
| **Session Validation** | ✅ `date_format:H:i` untuk time |
| **Feature Tests** | ✅ Test/factory schema lama sudah diupdate (section_type dan section score dibersihkan) |
| **Auto-generate Code** | ✅ QB/PKG/SES codes auto-generated — FE tidak perlu kirim code |
| **Registrations Absent** | ✅ Scheduled command marks registrations absent when session finished |
| **Phone Validation** | ✅ Digits only, 11-13 chars |
| **Queue Worker** | ✅ ExportResultsJob + jobs table + dokumentasi |
| **Log Rotation** | ✅ daily channel + PowerShell script + dokumentasi |
| **Backup** | ✅ PowerShell + Bash scripts + dokumentasi |
| **.env.example** | ✅ Lengkap (66+ env vars) |

---

## Yang Sudah Selesai ✅

### 1. Authentication dan Account Lifecycle
- ✅ Auth register/login/logout/me menggunakan Sanctum
- ✅ Role admin dan participant
- ✅ Middleware admin dan active account
- ✅ Response login membedakan status (pending, rejected, suspended)
- ✅ Admin bisa login (role check)
- ✅ Anti-double login: token lama di-revoke saat login baru — berlaku untuk **participant**. Admin secara by-design dikecualikan (`AuthController.php` baris 62: `if ($user->role !== 'admin')`) agar admin bisa login dari banyak device sekaligus.

### 2. Registration dan Payment Proof
- ✅ `POST /api/register`
- ✅ `GET/POST /api/payment-proofs`
- ✅ `GET /api/admin/payment-proofs`
- ✅ `GET /api/admin/payment-proofs/{id}`
- ✅ `GET /api/admin/payment-proofs/{id}/preview`
- ✅ `PATCH /api/admin/payment-proofs/{id}/approve`
- ✅ `PATCH /api/admin/payment-proofs/{id}/reject`
- ✅ Saat approve, sistem membuat `test_approval` dan aktifkan akun
- ✅ Approve/reject hanya untuk status pending (idempotency guard 422)
- ✅ Filter admin: status, user_id
- ✅ Pagination metadata- ✅ **Phone validation**: digits only, 11-13 characters (Register form)
- ✅ **Schema Cleanup**: Removed `amount` dan `payment_date` fields (redundant for proof upload)
### 3. Test Approval
- ✅ `GET /api/admin/test-approvals`
- ✅ `GET /api/admin/test-approvals/{id}`
- ✅ `GET /api/my/test-approvals`
- ✅ Status: available, assigned, consumed, expired

### 4. Question Bank dan Question Management
- ✅ CRUD question bank
- ✅ CRUD question dengan multipart
- ✅ Validasi options (minimal 2, exactly 1 correct)
- ✅ Audio play count default 1
- ✅ Guard delete: blok hapus bank soal yang dipakai sesi aktif
- ✅ Storage image/audio di disk public
- ✅ **Import soal dari CSV** (`POST /api/admin/questions/import`)
- ✅ **Download template CSV** (`GET /api/admin/questions/import/template`)
- ✅ **Schema Cleanup**: Removed `question_type` dan `difficulty_level` fields (no longer used for question differentiation)

### 5. Exam Package
- ✅ CRUD package
- ✅ Bank sync via transaction
- ✅ Validasi stok soal aktif per bank
- ✅ Validasi unique bank dalam package
- ✅ Shuffle questions/options

### 6. Exam Session dan Participant Assignment
- ✅ CRUD session
- ✅ Publish/close/finish/cancel session dengan guard transisi
- ✅ Manual assign dan auto generate participant
- ✅ List participants
- ✅ Delete participant (guard: tidak bisa hapus yang sudah start attempt)
- ✅ Quota validation
- ✅ `date_format:H:i` validation untuk start/end time

### 7. Exam Runtime Engine
- ✅ Start attempt
- ✅ Generate snapshot attempt questions
- ✅ Randomisasi option order
- ✅ Get question by display number
- ✅ Save answer dengan validasi ownership option
- ✅ Mark doubtful
- ✅ Navigate
- ✅ Heartbeat dengan auto-submit saat timeout
- ✅ Submit
- ✅ Scoring
- ✅ Fix relasi `questionOptions`
- ✅ Attempt relasi ke `test_approval_id`

### 8. Scoring dan Result Visibility
- ✅ `ScoringService` menghitung `total_score`, `correct_count`, `wrong_count`, `unanswered_count`
- ✅ Result resource
- ✅ Admin results endpoint
- ✅ Export results to Excel
- ✅ Result visibility guard: user tidak melihat skor jika `show_result_to_user = false`

### 9. Anti-Cheat dan Monitoring
- ✅ Violation log endpoint
- ✅ Audio play log endpoint
- ✅ Admin list attempts/results/violations/activity-logs
- ✅ Violation disimpan dengan payload JSON

### 10. Settings
- ✅ `GET/PATCH /api/admin/settings/exam` dengan type casting (int/bool)
- ✅ Default values: duration=120, tab_switch=3, fullscreen_exit=3, shuffle=true

### 11. Audit Log
- ✅ `LogsActivity` trait di model: PaymentProof, TestApproval, ExamSession, ExamAttempt
- ✅ `GET /api/admin/activity-logs` dengan filter (log_name, subject_type, event, causer_id)

### 12. Media
- ✅ `GET /api/media/{path}` dengan signed URL validation
- ✅ Directory traversal protection
- ✅ Cache-Control header

### 13. Scheduler
- ✅ `sessions:auto-close` — auto-close session yang lewat end_time (setiap 5 menit)
- ✅ `attempts:auto-submit-stale` — auto-submit attempt timeout (setiap 5 menit)
- ✅ `registrations:mark-absent` — mark registrations absent when session finished and no attempt (setiap 5 menit)

### 14. API Response dan Documentation
- ✅ Standard response format
- ✅ `API_DOCUMENTATION.md` lengkap
- ✅ Payload examples

### 15. Feature Tests
- ✅ `AuthFlowTest` — register, login/logout, double-login, wrong password
- ✅ `AdminPaymentFlowTest` — list, filter, show, approve, reject, guard, non-admin
- ✅ `AdminQuestionBankFlowTest` — CRUD bank, CRUD question, validation, guard delete
- ✅ `AdminUserManagementTest` — list, search, show, approve, reject, reset password
- ✅ `HtmlSanitizerTest` — 9 tests
- ✅ `SessionStatusTransitionTest` — 10 tests

### 16. Operations
- ✅ `.env.example` lengkap (66+ env vars)
- ✅ Queue worker: `ExportResultsJob` dengan tries=3, timeout=300
- ✅ Jobs table migration tersedia
- ✅ Log rotation: daily channel + PowerShell script
- ✅ Backup: PowerShell + Bash scripts dengan retention

### 17. Auto-Generated Features
- ✅ Auto-generate `code` untuk QuestionBank (prefix `QB-` + 6 random chars)
- ✅ Auto-generate `code` untuk ExamPackage (prefix `PKG-` + 6 random chars)
- ✅ Auto-generate `code` untuk ExamSession (prefix `SES-` + 6 random chars)
- ✅ Model boot logic tetap membolehkan override manual saat create


---

## Yang Masih Belum / Perlu Perbaikan 🔧

### P0 - Harus Sebelum Demo Integrasi
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Authorization Policies** | ✅ Selesai | Policies + `authorize()` dipakai di controller |
| 2 | **Attempt relasi ke approval** | ✅ Selesai | `exam_attempts.test_approval_id` + relasi model |
| 3 | **Result visibility guard** | ✅ Selesai | Endpoint user tidak bocorkan metadata saat `show_result_to_user = false` |
| 4 | **Save answer option ownership** | ✅ Selesai | `selected_option_id` divalidasi milik soal di attempt via `SaveAnswerRequest` |

### P1 - MVP Production
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Feature tests** | ✅ Selesai | 21 tests: Auth, Payment, Question Bank, User Management |
| 2 | **Anti-double login** | ✅ Selesai | Token lama di-revoke untuk participant. Admin by-design dikecualikan (bisa multi-device). |
| 3 | **Audit log** | ✅ Selesai | `LogsActivity` di model + endpoint activity log |
| 4 | **Settings endpoint** | ✅ Selesai | `GET/PATCH /api/admin/settings/exam` tersedia |
| 5 | **Session lifecycle** | ✅ Selesai | Endpoint finish/cancel + guard transisi |
| 6 | **HTML sanitasi** | ✅ Selesai | HtmlSanitizer di create/update question |
| 7 | **Signed URL media** | ✅ Selesai | Media proxy + signed URL validation |
| 8 | **Anti-cheat threshold enforcement** | ✅ Selesai | Auto-submit saat tab_switch/fullscreen_exit melebihi threshold; bisa dimatikan via setting |

### P2 - Enhancement
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Import bank soal** | ✅ Selesai | CSV import + template download |
| 2 | **Versioning bank/package** | ✅ Selesai | Auto-increment version saat update; package_version & bank_versions_json tersimpan di attempt |
| 3 | **Notification email/WhatsApp** | ✅ Selesai | Email notifikasi: approval, rejection, session assigned, result published. Bisa dimatikan via config. |
| 4 | **Advanced analytics** | ✅ Selesai | Endpoint `/api/admin/analytics` dengan group_by session/package |

### Deployment dan Operations
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **`.env.example` lengkap** | ✅ Selesai | Semua env var tercantum |
| 2 | **Queue worker** | ✅ Selesai | `ExportResultsJob` + dokumentasi |
| 3 | **Scheduler** | ✅ Selesai | Auto-close session, auto-submit stale, mark absent |
| 4 | **Log rotation** | ✅ Selesai | daily channel + PowerShell script |
| 5 | **Backup DB dan storage** | ✅ Selesai | PowerShell + Bash scripts |
| 6 | **Schema cleanup migrations** | ✅ Selesai | Drop question_type/difficulty_level (2026-05-11), drop amount/payment_date (2026-05-11), drop section_type dari 3 tabel + drop listening/structure/reading score dari exam_results (2026-05-12) |

---

## ⚠️ Item Yang Memerlukan Tindak Lanjut

### 1. Test Coverage Pending
Tidak ada lagi test/factory aktif yang memakai kolom schema lama (`section_type`, `question_type`, `difficulty_level`, `listening_score`, `structure_score`, `reading_score`). Tindak lanjut yang masih relevan adalah menambah coverage khusus untuk default exam setting pada create package/session.

### 2. Session `publish()` Tidak Ada Guard Transisi
- **File:** `app/Http/Controllers/Api/Admin/ExamSessionController.php` baris 75–87
- **Masalah:** Method `publish()` langsung update status ke `PUBLISHED` tanpa memanggil `canTransitionTo()`, sementara `close()`, `finish()`, dan `cancel()` sudah pakai guard.
- **Dampak:** Session berstatus `FINISHED` atau `CANCELLED` bisa di-publish ulang.
- **Fix:** Tambahkan guard sebelum update, contoh: `if (!$this->canTransitionTo($session->status, ExamSessionStatus::PUBLISHED)) { return $this->error(...); }`

### 3. `API_DOCUMENTATION.md` Masih Menyebut `section_type`
- **File:** `API_DOCUMENTATION.md` baris 193, 221, 247, 293, 306, 309
- **Masalah:** Field `section_type` sudah dihapus dari schema via migration `2026_05_12_000000`, tetapi dokumentasi API masih mencantumkannya sebagai required field di CSV import dan endpoint question/package.
- **Dampak:** Menyesatkan developer yang mengintegrasikan API.
- **Fix:** Hapus semua referensi `section_type` dari `API_DOCUMENTATION.md` dan perbarui contoh payload CSV import.

### 4. Belum Ada Endpoint Public Settings untuk Peserta
- **File:** `routes/api.php`
- **Masalah:** `GET /api/admin/settings/exam` hanya bisa diakses admin. FE peserta (`fe-cbt`) membutuhkan endpoint tanpa auth untuk membaca nilai `max_tab_switch` dan `max_fullscreen_exit`, sehingga saat ini FE menggunakan hardcode fallback (3/3).
- **Fix:** Tambahkan route publik atau peserta: `GET /api/settings/exam` yang hanya mengembalikan field yang aman dibaca peserta (bukan semua setting admin).

---

## Prioritas Sprint

### P0 - Harus Diselesaikan Sebelum Demo Integrasi
- Authorization Policies ✅
- Attempt relasi ke approval (`test_approval_id`) ✅
- Result visibility guard ✅
- Save answer option ownership validation ✅

### P1 - MVP Production
- Full test coverage core flow ✅
- Settings endpoint ✅
- Anti-double login ✅
- Anti-cheat threshold enforcement ✅ (auto-submit saat threshold tercapai, bisa dimatikan via setting)
- Strong audit log ✅
- HTML sanitasi ✅

### P2 - Enhancement
- Import bank soal ✅
- Versioning bank/package ✅
- Advanced analytics ✅
- Notification email/WhatsApp ✅

---

## Urutan Pengerjaan (Termudah -> Tersulit)

Catatan: urutan ini berbasis effort implementasi, bukan prioritas bisnis. Tag prioritas tetap [P0]/[P1]/[P2]/[OPS].

### Exam Setting Integration (P1)

Status: implementasi inti sudah selesai, tetapi test coverage khusus create package/session masih pending.

Tujuan bagian ini adalah memakai exam setting sebagai data awal saat create exam package dan exam session, lalu tetap membolehkan admin mengubah nilainya sebelum submit. Source of truth final tetap berada di tabel package/session, bukan di settings.

#### Field mapping yang paling relevan
| Area | Field dari exam setting | Target pemakaian |
|------|--------------------------|------------------|
| Package | `default_duration_minutes` | `duration_minutes` |
| Package | `default_shuffle_questions` | `shuffle_questions` |
| Package | `default_shuffle_options` | `shuffle_options` |
| Package | `default_max_tab_switch` | `max_tab_switch` |
| Package | `default_max_fullscreen_exit` | `max_fullscreen_exit` |
| Session | `default_duration_minutes` | `duration_minutes` |
| Session | `default_show_result_to_user` | `show_result_to_user` |
| Session | `default_auto_generate_enabled` | `auto_generate_enabled` |

#### Urutan pengerjaan BE-CBT dari termudah ke tersulit

1. [P1] Buat helper/resolver untuk membaca exam settings yang sudah ter-cast dan siap dipakai ulang. ✅

	- Effort: S
	- Scope: satu pintu baca default exam setting dari tabel `settings`.
	- DoD: helper mengembalikan nilai default yang konsisten untuk package dan session.

2. [P1] Terapkan default exam setting saat create exam package. ✅

	- Effort: S-M
	- Scope: isi nilai awal `duration_minutes`, `shuffle_questions`, `shuffle_options`, `max_tab_switch`, `max_fullscreen_exit`.
	- DoD: jika FE tidak mengirim field tertentu, BE tetap mengisi nilai default.

3. [P1] Terapkan default exam setting saat create exam session. ✅

	- Effort: S-M
	- Scope: isi nilai awal `duration_minutes`, `show_result_to_user`, `auto_generate_enabled`.
	- DoD: session tetap bisa dibuat walau FE hanya mengirim field minimum, selama default tersedia.

4. [P1] Pastikan override manual selalu menang atas default. ✅

	- Effort: S
	- Scope: nilai dari request final tidak boleh ditimpa default.
	- DoD: admin bisa mengubah semua nilai awal sebelum submit.

5. [P1] Tambahkan test untuk create package dan session. TODO

	- Effort: M
	- Scope: test fallback default, test override manual, test field mapping.
	- DoD: ada test yang memastikan exam setting memang dipakai hanya saat create.
	- **Catatan (2026-05-12):** Belum ada test file khusus untuk skenario ini. Tidak blocking deployment, tapi perlu diselesaikan untuk full coverage.

#### Catatan implementasi

- FE `cbt-admin` sudah prefill form create package/session dari `GET /api/admin/settings/exam` agar UX konsisten dengan setting global.
- BE tetap wajib punya fallback default supaya API aman dipanggil langsung.
- Update existing record tidak perlu dipaksa mengikuti exam setting lagi.
- Setting global tidak dipakai langsung oleh runtime attempt; runtime tetap baca package/session final.

### Core (P0/P1)

1. [P0] Result visibility guard ✅
	- Effort: S
	- Scope: sembunyikan detail skor saat `show_result_to_user = false` di semua endpoint user.
	- DoD: user hanya melihat status selesai dan flag result visible; admin tetap melihat skor penuh.

2. [P0] Validasi ownership `selected_option_id` ✅
	- Effort: S
	- Scope: pastikan opsi yang dipilih memang milik question yang sedang dijawab pada attempt.
	- DoD: validasi 422 bila opsi tidak match; opsi kosong tetap boleh untuk skip.

3. [P1] Settings endpoint exam ✅
	- Effort: S-M
	- Scope: `GET/PATCH /api/admin/settings/exam` untuk konfigurasi umum.
	- DoD: storage konsisten (DB), validasi request, hanya admin dapat akses.

4. [P0] Authorization policies ✅
	- Effort: M
	- Scope: buat policy untuk resource utama dan gunakan `authorize()` di controller.
	- DoD: semua endpoint admin terlindungi policy; user hanya bisa akses resource miliknya.

5. [P1] Session lifecycle (finish/cancel) ✅
	- Effort: M
	- Scope: endpoint/status transition `finished` dan `cancelled`.
	- DoD: aturan transisi jelas; attempt baru ditolak jika session tidak aktif.

6. [P1] Anti-double login enforcement ✅
	- Effort: M
	- Scope: blokir sesi login ganda atau invalidate session lama.
	- DoD: hanya 1 session aktif per user; audit/log tersedia saat diblokir.

7. [P0] Attempt relasi ke approval ✅
	- Effort: M-L
	- Scope: simpan `test_approval_id` ke `exam_attempts` saat start.
	- DoD: migrasi + relasi dipakai untuk validasi.

8. [P1] Audit log activity ✅
	- Effort: M-L
	- Scope: aktifkan `LogsActivity` di model kritikal.
	- DoD: event approve/reject/publish/submit tercatat dengan actor.

9. [P1] HTML sanitasi konten soal ✅
	- Effort: M-L
	- Scope: sanitize `stem_html`, `option_html`, `explanation_html` pada create/update.
	- DoD: payload berbahaya dibersihkan; content valid tetap utuh.

10. [P1] Signed URL media ✅
	 - Effort: L
	 - Scope: generate signed URL untuk image/audio atau proxy endpoint.
	 - DoD: URL hanya berlaku sementara; akses tanpa token ditolak.

11. [P1] Feature tests core flow ✅
	 - Effort: L
	 - Scope: test alur register -> approval -> session -> start -> submit -> result.
	 - DoD: coverage minimal untuk happy path dan negative cases penting.

### Ops (Deployment/Operations)

1. [OPS] Lengkapi `.env.example` ✅
	- Effort: S
	- DoD: semua env yang dipakai config dan service tercantum dengan nilai contoh.

2. [OPS] Scheduler jobs ✅
	- Effort: M
	- Scope: auto-close session dan auto-submit attempt stale.
	- DoD: command terjadwal dan safe-guard untuk idempotency.

3. [OPS] Queue worker ✅
	- Effort: M
	- Scope: setup queue connection dan job async (export results).
	- DoD: job berjalan di worker, retry dan timeout terdefinisi.

4. [OPS] Log rotation ✅
	- Effort: M
	- Scope: logging harian atau size-based.
	- DoD: log tidak menumpuk; retention jelas.

5. [OPS] Backup DB dan storage ✅
	- Effort: L
	- Scope: backup terjadwal untuk database dan file upload.
	- DoD: dokumentasi restore dan lokasi backup ada.

### Enhancement (P2)

1. [P2] Import bank soal ✅
	- Effort: M
	- Scope: import CSV untuk question bank + options.
	- DoD: validasi template dan report error per baris.

2. [P2] Versioning bank/package ✅
	- Effort: L
	- Scope: simpan versi soal/paket untuk audit dan rollback.
	- DoD: attempt menyimpan versi snapshot yang digunakan.

3. [P2] Notification email/WhatsApp ✅
	- Effort: L
	- Scope: notifikasi approval, session assigned, dan hasil.
	- DoD: provider configurable dan bisa dimatikan.

4. [P2] Advanced analytics ✅
	- Effort: L
	- Scope: agregasi hasil per sesi/paket/section.
	- DoD: endpoint admin + export ringkas.

### Infrastructure & UX Enhancement (P2)

1. [P2] Auto-generate code untuk resource ✅
	- Effort: S-M
	- Scope: QB/PKG/SES codes otomatis (prefix + random chars).
	- DoD: FE tidak perlu generate code; admin bisa override manual.

2. [P2] Phone validation ✅
	- Effort: S
	- Scope: digits only, 11-13 chars untuk Register form.
	- DoD: validasi di backend enforce consistent format.

3. [P2] Mark absent registrations ✅
	- Effort: M
	- Scope: scheduled job `registrations:mark-absent` saat session finished.
	- DoD: registrasi tanpa attempt otomatis marked absent; safe guard untuk idempotency.

### Schema Cleanup (P1)

1. [P1] Remove question_type & difficulty_level ✅
	- Effort: M
	- Scope: hapus field tidak dipakai; migration untuk production.
	- DoD: code di-refactor, import/create updated, test di-update.

2. [P1] Remove payment_date & amount dari proof ✅
	- Effort: M
	- Scope: hapus field redundant; migration untuk production.
	- DoD: controller/model/resource di-cleanup, tidak ada referensi sisa.

### Client Feedback Refinement (P3)

1. [P3] Fitur Bulk Create Soal ✅
	- Effort: M
	- Scope: Endpoint `POST /api/admin/questions/bulk` menerima `multipart/form-data` array soal (teks, gambar, audio).
	- DoD: Bisa menyimpan banyak soal sekaligus dengan efisien untuk UI bergaya Google Forms di CBT-Admin.
