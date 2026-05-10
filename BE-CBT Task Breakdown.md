# BE-CBT Task Breakdown

Dokumen ini adalah breakdown pekerjaan backend `be-cbt` berdasarkan kondisi repo saat ini, dokumen existing di `cbt-documentation`, dan kontrak API yang sudah muncul di `be-cbt/API_DOCUMENTATION.md`.

**Last Updated:** 2026-05-09

---

## Ringkasan Status

Backend sudah memiliki fondasi Laravel API yang cukup lengkap:

| Area | Status |
|------|--------|
| **Auth** | ✅ Sanctum, role, middleware |
| **Database** | ✅ 24 migrations, 20 models, relasi lengkap |
| **API Endpoints** | ✅ ~47 endpoint (admin + user) |
| **Exam Engine** | ✅ Snapshot, randomisasi, timer, scoring |
| **Payment Proof** | ✅ Upload, approve, reject + preview endpoint |
| **Question API** | ✅ CRUD + validasi + bank sync |
| **Session Validation** | ✅ `date_format:H:i` untuk time |
| **Question Import** | ✅ CSV import + template download |
| **Feature Tests** | ✅ 21 test cases (Auth, Payment, Question Bank, User Management) |
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
- ✅ Anti-double login: token lama di-revoke saat login baru

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
- ✅ Pagination metadata

### 3. Test Approval
- ✅ `GET /api/admin/test-approvals`
- ✅ `GET /api/admin/test-approvals/{id}`
- ✅ `GET /api/my/test-approvals`
- ✅ Status: available, assigned, consumed, expired

### 4. Question Bank dan Question Management
- ✅ CRUD question bank
- ✅ CRUD question dengan multipart
- ✅ Validasi options (minimal 2, exactly 1 correct)
- ✅ Validasi media berdasarkan question type
- ✅ `difficulty_level` field
- ✅ `audio_max_play_count` default 1
- ✅ Guard delete: blok hapus bank soal yang dipakai sesi aktif
- ✅ Storage image/audio di disk public
- ✅ **Import soal dari CSV** (`POST /api/admin/questions/import`)
- ✅ **Download template CSV** (`GET /api/admin/questions/import/template`)

### 5. Exam Package
- ✅ CRUD package
- ✅ Bank sync via transaction
- ✅ Validasi stok soal aktif per bank/section
- ✅ Validasi unique bank+section combination
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
- ✅ `ScoringService` menghitung total, listening, structure, reading
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
| 2 | **Anti-double login** | ✅ Selesai | Token lama di-revoke, hanya 1 sesi aktif |
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
| 4 | **Advanced analytics** | ✅ Selesai | Endpoint `/api/admin/analytics` dengan group_by session/package/section |

### Deployment dan Operations
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **`.env.example` lengkap** | ✅ Selesai | Semua env var tercantum |
| 2 | **Queue worker** | ✅ Selesai | `ExportResultsJob` + dokumentasi |
| 3 | **Scheduler** | ✅ Selesai | Auto-close session, auto-submit stale |
| 4 | **Log rotation** | ✅ Selesai | daily channel + PowerShell script |
| 5 | **Backup DB dan storage** | ✅ Selesai | PowerShell + Bash scripts |

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