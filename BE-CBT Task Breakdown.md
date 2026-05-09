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
| **API Endpoints** | ✅ ~45 endpoint (admin + user) |
| **Exam Engine** | ✅ Snapshot, randomisasi, timer, scoring |
| **Payment Proof** | ✅ Upload, approve, reject + **preview endpoint** (baru) |
| **Question API** | ✅ CRUD + validasi + **bank sync** (baru) |
| **Session Validation** | ✅ `date_format:H:i` untuk time (baru) |

---

## Yang Sudah Selesai ✅

### 1. Authentication dan Account Lifecycle
- ✅ Auth register/login/logout/me menggunakan Sanctum
- ✅ Role admin dan participant
- ✅ Middleware admin dan active account
- ✅ Response login membedakan status (pending, rejected, suspended)
- ✅ Admin bisa login (role check)

### 2. Registration dan Payment Proof
- ✅ `POST /api/register`
- ✅ `GET/POST /api/payment-proofs`
- ✅ `GET /api/admin/payment-proofs`
- ✅ `GET /api/admin/payment-proofs/{id}`
- ✅ **`GET /api/admin/payment-proofs/{id}/preview`** (baru - 9 Mei 2026)
- ✅ `PATCH /api/admin/payment-proofs/{id}/approve`
- ✅ `PATCH /api/admin/payment-proofs/{id}/reject`
- ✅ Saat approve, sistem membuat `test_approval` dan aktifkan akun
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
- ✅ **`difficulty_level` field** (baru - 9 Mei 2026)
- ✅ **`audio_max_play_count` default 1** (baru - 9 Mei 2026)
- ✅ Storage image/audio di disk public

### 5. Exam Package
- ✅ CRUD package
- ✅ **Bank sync via transaction** (baru - 9 Mei 2026)
- ✅ Validasi stok soal aktif per bank/section
- ✅ Validasi unique bank+section combination
- ✅ Shuffle questions/options

### 6. Exam Session dan Participant Assignment
- ✅ CRUD session
- ✅ Publish/close session
- ✅ Manual assign dan auto generate participant
- ✅ List participants
- ✅ Quota validation
- ✅ **`date_format:H:i` validation** untuk start/end time (baru - 9 Mei 2026)

### 7. Exam Runtime Engine
- ✅ Start attempt
- ✅ Generate snapshot attempt questions
- ✅ Randomisasi option order
- ✅ Get question by display number
- ✅ Save answer
- ✅ Mark doubtful
- ✅ Navigate
- ✅ Heartbeat
- ✅ Submit
- ✅ Scoring
- ✅ **Fix relasi `questionOptions`** (baru - 9 Mei 2026)

### 8. Scoring dan Result Visibility
- ✅ `ScoringService` menghitung total, listening, structure, reading
- ✅ Result resource
- ✅ Admin results endpoint
- ✅ Export results to Excel

### 9. Anti-Cheat dan Monitoring
- ✅ Violation log endpoint
- ✅ Audio play log endpoint
- ✅ Admin list attempts/results/violations

### 10. API Response dan Documentation
- ✅ Standard response format
- ✅ `API_DOCUMENTATION.md` lengkap
- ✅ Payload examples

---

## Yang Masih Belum / Perlu Perbaikan 🔧

### P0 - Harus Sebelum Demo Integrasi
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Authorization Policies** | ❌ Kosong | Folder `app/Policies/` kosong |
| 2 | **Attempt relasi ke approval** | ❌ Belum | `test_approval_id` di `exam_attempts` |
| 3 | **Result visibility guard** | ⚠️ Parsial | Perlu cek user response |
| 4 | **Save answer option ownership** | ⚠️ Parsial | Validasi selected_option_id |

### P1 - MVP Production
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Feature tests** | ❌ Kosong | Hanya `ExampleTest.php` default |
| 2 | **Anti-double login** | ❌ Belum | `user_sessions` tabel ada, belum dipakai |
| 3 | **Audit log** | ⚠️ Parsial | Spatie terinstall, belum digunakan |
| 4 | **Settings endpoint** | ❌ Belum | `GET/PATCH /api/admin/settings/exam` |
| 5 | **Idempotency guard approve** | ❌ Belum | Payment proof approved 2x harus 422 |
| 6 | **Session lifecycle** | ⚠️ Parsial | Belum ada `running`, `cancelled` |
| 7 | **HTML sanitasi** | ❌ Belum | `stem_html`, `option_html`, `explanation_html` |
| 8 | **Signed URL media** | ❌ Belum | Image/audio URL |
| 9 | **Guard delete** | ❌ Belum | Bank soal dipakai aktif |

### P2 - Enhancement
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **Import bank soal** | ❌ Belum | |
| 2 | **Versioning bank/package** | ❌ Belum | |
| 3 | **Notification email/WhatsApp** | ❌ Belum | |
| 4 | **Advanced analytics** | ❌ Belum | |
| 5 | **Proctoring webcam** | ❌ Belum | Jika scope berubah |

### Deployment dan Operations
| # | Fitur | Status | Catatan |
|---|-------|--------|---------|
| 1 | **`.env.example` lengkap** | ⚠️ Parsial | Perlu cek semua env var |
| 2 | **Queue worker** | ❌ Belum | Untuk job async |
| 3 | **Scheduler** | ❌ Belum | Auto-close session, auto-submit stale |
| 4 | **Log rotation** | ❌ Belum | |
| 5 | **Backup DB dan storage** | ❌ Belum | |

---

## Prioritas Sprint

### P0 - Harus Diselesaikan Sebelum Demo Integrasi
- Authorization Policies
- Attempt relasi ke approval (`test_approval_id`)
- Result visibility guard
- Save answer option ownership validation

### P1 - MVP Production
- Full test coverage core flow
- Settings endpoint
- Anti-double login
- Anti-cheat threshold enforcement
- Strong audit log
- HTML sanitasi

### P2 - Enhancement
- Import bank soal
- Versioning bank/package
- Advanced analytics
- Notification email/WhatsApp
