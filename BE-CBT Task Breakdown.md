# BE-CBT Task Breakdown

Dokumen ini adalah breakdown pekerjaan backend `be-cbt` berdasarkan kondisi repo saat ini, dokumen existing di `cbt-documentation`, dan kontrak API yang sudah muncul di `be-cbt/API_DOCUMENTATION.md`.

## Ringkasan Status

Backend sudah memiliki fondasi Laravel API yang cukup lengkap:

- Auth register/login/logout/me menggunakan Sanctum.
- Role admin dan participant, middleware admin dan active account.
- Entity utama CBT sudah ada lewat migration dan model.
- API admin dasar sudah tersedia untuk users, payment proofs, test approvals, question banks, questions, exam packages, exam sessions, participants, monitoring, results, violations, dan export.
- API user dasar sudah tersedia untuk profile, payment proofs, exam sessions, start exam, resume, question, save answer, mark doubtful, navigate, heartbeat, submit, result, violations, audio play.
- Exam engine sudah membuat snapshot attempt question dan scoring dasar sudah ada.

Status saat ini masih perlu pengerasan di validasi bisnis, security, response consistency, media serving, test coverage, dan production hardening.

## Scope MVP Backend

### 1. Authentication dan Account Lifecycle

Status: Parsial.

Task:

- Pastikan admin seed selalu membuat akun admin default yang aman untuk environment lokal saja.
- Tambahkan endpoint atau flow reset password peserta dan admin jika memang masuk scope MVP.
- Rapikan response login untuk akun pending, rejected, suspended agar frontend bisa membedakan state.
- Tambahkan policy bahwa admin bisa login walaupun bukan participant dan tetap harus `active`.
- Implementasikan anti-double login sesuai SOW:
  - Simpan token/session aktif.
  - Tentukan policy: reject login baru atau revoke session lama.
  - Saat ujian aktif, rekomendasi reject login baru.
- Tambahkan logout yang hanya mematikan token aktif, bukan semua token user kecuali diminta.
- Tambahkan audit log untuk login, logout, failed login, account approve/reject/reset.

Acceptance criteria:

- Pending user tidak bisa login.
- Rejected user menerima alasan yang bisa ditampilkan frontend.
- Admin route tidak bisa diakses participant.
- Session aktif tercatat dan bisa diaudit.

## 2. Registration dan Payment Proof

Status: Parsial.

Yang sudah ada:

- `POST /api/register`
- `GET/POST /api/payment-proofs`
- `GET /api/admin/payment-proofs`
- `GET /api/admin/payment-proofs/{id}`
- `PATCH /api/admin/payment-proofs/{id}/approve`
- `PATCH /api/admin/payment-proofs/{id}/reject`
- Saat approve payment, sistem membuat `test_approval` dan mengaktifkan akun pending.

Task:

- Perbaiki naming request reject:
  - Saat ini reject memakai `ApprovePaymentProofRequest`, lebih tepat dibuat `RejectPaymentProofRequest`.
- Tambahkan validasi file storage dan akses file:
  - Endpoint admin untuk preview/download bukti pembayaran.
  - Jangan expose raw private path.
  - Gunakan signed URL atau controller stream.
- Pastikan upload payment proof user membutuhkan auth dan akun yang valid sesuai bisnis.
- Tambahkan rule retake:
  - User boleh upload payment proof baru setelah approval sebelumnya consumed.
  - Audit trail payment proof harus tetap utuh.
- Tambahkan filter admin:
  - status
  - user_id
  - date range
  - search nama/email
- Tambahkan pagination metadata konsisten.
- Tambahkan idempotency guard approve:
  - Payment proof approved kedua kali harus 422.
  - Tidak boleh membuat duplicate test approval.
- Tambahkan rejection reason pada response detail user.

Acceptance criteria:

- Admin bisa melihat list, detail, file bukti, approve, reject.
- Approve membuat satu `test_approval`.
- Reject menyimpan alasan.
- Payment proof yang sudah diproses tidak bisa diproses ulang.

## 3. Test Approval

Status: Parsial.

Yang sudah ada:

- `GET /api/admin/test-approvals`
- `GET /api/admin/test-approvals/{id}`
- `GET /api/my/test-approvals`
- Status approval tersedia pada model dan enum.

Task:

- Pastikan status lifecycle final:
  - `available`
  - `assigned`
  - `consumed`
  - `expired`
- Saat manual assign participant, status approval harus berubah dari `available` ke `assigned`, bukan baru `consumed` saat start.
- Saat attempt berhasil dibuat, status approval berubah ke `consumed` atau locked sesuai keputusan final.
- Tambahkan constraint agar satu approval tidak bisa assigned ke dua session.
- Tambahkan validasi retake:
  - Approval baru tidak boleh digunakan pada sesi yang sama dengan attempt sebelumnya jika aturan bisnis mengharuskan sesi berbeda.
- Tambahkan endpoint admin untuk melihat approval by user, by payment proof, by status.

Acceptance criteria:

- Satu payment proof approved menghasilkan satu test approval.
- Satu test approval hanya bisa dipakai untuk satu session/attempt.
- Approval consumed tidak bisa dipakai ulang.

## 4. Question Bank dan Question Management

Status: Parsial.

Yang sudah ada:

- CRUD question bank.
- CRUD question.
- Validasi options minimal dua dan tepat satu correct answer.
- Validasi media image/audio berdasarkan question type.
- Storage image/audio di disk public.

Task:

- Tambahkan endpoint update/delete detail yang sesuai API docs dan Postman.
- Tambahkan support upload media multipart dari admin frontend:
  - image
  - audio
  - stem_html
  - explanation_html
  - options array
- Pastikan `utf8mb4` dan Arabic RTL tidak rusak di DB.
- Tambahkan sanitasi HTML untuk `stem_html`, `option_html`, `explanation_html`.
- Tambahkan image/audio URL absolut atau signed URL di resource.
- Tambahkan filter:
  - question_bank_id
  - section_type
  - question_type
  - difficulty_level
  - active
  - search
- Tambahkan import/export bank soal jika masuk scope sprint.
- Tambahkan guard delete:
  - Bank soal yang dipakai package aktif/session aktif tidak boleh hard delete.
  - Question yang sudah masuk attempt snapshot tidak boleh merusak attempt lama.
- Tambahkan versioning ringan jika bank soal perlu diedit saat ada session aktif.

Acceptance criteria:

- Admin bisa membuat soal text, image, audio, mixed.
- Satu soal punya satu jawaban benar.
- Media tampil lewat URL yang bisa diakses frontend.
- Soal lama pada attempt tidak berubah saat bank soal diedit.

## 5. Exam Package

Status: Parsial.

Yang sudah ada:

- CRUD package.
- Package dapat berisi mapping bank, section, question_count, sort_order.
- Validasi stok soal aktif per bank/section.
- Shuffle questions/options tersedia.

Task:

- Pastikan update package juga menyinkronkan `exam_package_banks`, bukan hanya data package.
- Tambahkan detail endpoint dengan banks lengkap.
- Tambahkan validasi package aktif:
  - Tidak bisa publish/activate jika stok soal kurang.
  - Tidak bisa hapus package yang dipakai session draft/published/running.
- Tambahkan field timer on/off jika requirement admin timer dapat dimatikan.
- Tambahkan setting max audio play default di package atau question.
- Tambahkan threshold anti-cheat di package/session:
  - max_tab_switch
  - max_fullscreen_exit
  - action_after_limit

Acceptance criteria:

- Package bisa menjadi blueprint randomisasi fair.
- Package tidak bisa dipakai jika konfigurasi bank tidak valid.
- Package yang dipakai session aktif tidak rusak saat diedit.

## 6. Exam Session dan Participant Assignment

Status: Parsial.

Yang sudah ada:

- CRUD session.
- Publish/close session.
- Manual assign dan auto generate participant.
- List participants.
- Quota validation.

Task:

- Perbaiki validasi waktu:
  - `session_date`, `start_time`, `end_time` saat ini perlu dipastikan formatnya konsisten.
  - `start_time` dan `end_time` harus dibandingkan dengan tanggal session, bukan date-only/time-only yang ambigu.
- Tambahkan status lifecycle session:
  - draft
  - published
  - running
  - closed/finished
  - cancelled
- Tambahkan endpoint update status close/cancel/finish jika diperlukan admin.
- Manual assign harus memilih user dengan available test approval.
- Auto generate harus:
  - mengambil approval available.
  - menghormati max_participants.
  - tidak assign user yang sudah punya registration di session yang sama.
  - menandai approval assigned.
- Remove participant harus restore approval ke available jika attempt belum dimulai.
- Tambahkan participant detail dengan user, approval, attempt, status.

Acceptance criteria:

- User hanya melihat session yang assigned ke dirinya.
- Admin tidak bisa melebihi kuota.
- Approval tidak bisa double assign.
- Sesi published siap dipakai exam runtime.

## 7. Exam Runtime Engine

Status: Parsial, perlu pengerasan.

Yang sudah ada:

- Start attempt.
- Generate snapshot attempt questions.
- Randomisasi option order.
- Get question by display number.
- Save answer.
- Mark doubtful.
- Navigate.
- Heartbeat.
- Submit.
- Scoring.

Task kritis:

- Simpan relasi `test_approval_id` pada `exam_attempt` jika belum ada di schema. Ini penting untuk audit satu approval satu attempt.
- Perbaiki check session window:
  - Saat ini start membandingkan `$now` dengan `$session->start_time`/`end_time`. Pastikan nilai tersebut sudah full datetime berdasarkan `session_date`.
- Saat timeout di save answer/heartbeat:
  - Auto-submit dan scoring harus dipanggil konsisten.
  - Response harus memberi status expired/auto_submitted.
- Save answer harus memastikan `selected_option_id` termasuk opsi dari attempt question tersebut.
- Get question harus memakai `question_snapshot_json` sebagai sumber tampilan utama agar edit bank soal tidak memengaruhi attempt.
- `ExamQuestionResource` perlu memastikan option order dari `option_order_json` diterapkan.
- `audio-play` sebaiknya berbasis `attempt_question_id`, bukan raw `question_id`, supaya sesuai snapshot dan attempt.
- Submit harus idempotent:
  - Submit kedua mengembalikan status already submitted, bukan hitung ulang berisiko.
- Auto-resume:
  - Endpoint resume harus mengembalikan progress answered/doubtful/current/remaining.
- Tambahkan clear answer jika required oleh UI.

Acceptance criteria:

- Refresh/resume tidak mengubah urutan soal dan opsi.
- User tidak bisa memilih option dari soal lain.
- Timer backend tetap benar walaupun browser refresh.
- Submit dua kali tidak menggandakan hasil.

## 8. Scoring dan Result Visibility

Status: Parsial.

Yang sudah ada:

- `ScoringService` menghitung total, listening, structure, reading.
- Result resource tersedia.
- Admin results endpoint tersedia.

Task:

- Result user harus menghormati `show_result_to_user`.
- Jika hasil hidden, response user tidak boleh mengandung score/detail.
- Admin selalu bisa melihat result.
- Tambahkan publish/unpublish result jika dibutuhkan.
- Tambahkan export result dengan filter session/status.
- Tambahkan scoring scale jika TOAFL/TOEFL memakai formula khusus, bukan persentase sederhana.
- Tambahkan per-section result untuk package dengan section dinamis.

Acceptance criteria:

- User selesai ujian melihat score hanya jika session mengizinkan.
- Admin bisa export hasil.
- Skor tidak bocor pada session hidden.

## 9. Anti-Cheat dan Monitoring

Status: Parsial.

Yang sudah ada:

- Violation log endpoint.
- Audio play log endpoint.
- Admin list attempts/results/violations.

Task:

- Tambahkan threshold enforcement:
  - Jika tab switch/fullscreen exit melewati limit, tentukan warning/auto-submit.
- Tambahkan event types:
  - tab_switch
  - fullscreen_exit
  - right_click
  - copy_attempt
  - paste_attempt
  - reconnect
  - audio_replay_blocked
- Validasi violation hanya boleh dibuat oleh owner attempt.
- Simpan action_taken jika threshold terlewati.
- Monitoring endpoint perlu filter:
  - session_id
  - attempt_id
  - user_id
  - violation_type
  - severity
- Tambahkan summary monitoring per session.

Acceptance criteria:

- Pelanggaran tercatat per attempt.
- Admin bisa melihat pelanggaran dan attempt status.
- Threshold bisa memicu action sesuai package/session.

## 10. Settings

Status: Belum terlihat implementasi endpoint.

Task:

- Implement `GET /api/admin/settings/exam`.
- Implement `PATCH /api/admin/settings/exam`.
- Tentukan setting global vs setting package/session.
- Minimal setting:
  - default_duration_minutes
  - default_audio_max_play
  - enable_fullscreen_required
  - enable_tab_switch_detection
  - auto_submit_on_violation_limit
  - result_visibility_default

Acceptance criteria:

- Admin bisa membaca dan mengubah setting exam.
- Setting default tidak menimpa konfigurasi session yang sudah eksplisit.

## 11. API Response, Error, dan Documentation

Status: Parsial.

Task:

- Konsistenkan response Resource pagination. Saat ini resource collection Laravel bisa membungkus data/meta berbeda dari helper custom.
- Pastikan semua validation error berbentuk standar.
- Update `API_DOCUMENTATION.md` sesuai route final.
- Tambahkan contoh payload untuk:
  - register
  - login
  - upload payment proof multipart
  - create question multipart
  - create exam package
  - create session
  - start/resume/save/submit exam
- Update Postman collection setelah endpoint final.

Acceptance criteria:

- Frontend bisa mengandalkan response shape yang stabil.
- Docs sama dengan route aktual.

## 12. Testing

Status: Belum memadai.

Task:

- Feature tests auth:
  - register
  - login active
  - login pending/rejected
  - admin middleware
- Feature tests payment:
  - upload file
  - approve creates approval
  - reject stores reason
  - double approve rejected
- Feature tests question:
  - create text question
  - create audio question with file
  - reject invalid options
- Feature tests session:
  - create/publish
  - manual assign
  - quota full
  - auto generate
- Feature tests exam runtime:
  - start creates snapshot
  - save answer
  - invalid option rejected
  - resume preserves order
  - timeout auto submit
  - submit idempotent
- Feature tests results visibility.

Acceptance criteria:

- `php artisan test` mencakup core business rule.
- Bug randomisasi/timer/approval double use tertangkap test.

## 13. Deployment dan Operations

Status: Parsial.

Task:

- Pastikan `.env.example` lengkap:
  - DB
  - Sanctum
  - filesystem disk
  - queue
  - app URL
  - CORS frontend/admin URL
- Storage link dan permission deployment.
- Queue worker untuk job async bila digunakan.
- Scheduler jika perlu auto-close session/auto-submit stale attempt.
- Log rotation.
- Backup DB dan storage payment/media.
- Production CORS dan Sanctum stateful domains.

Acceptance criteria:

- Backend bisa deploy repeatable.
- File payment/media tetap bisa diakses sesuai permission.
- API aman dipanggil frontend dan admin domain.

## Prioritas Sprint

### P0 - Harus Diselesaikan Sebelum Demo Integrasi

- Response shape pagination final.
- Admin/user token auth stabil.
- Payment proof file preview/download.
- Session datetime validation.
- Attempt relasi ke approval.
- Result visibility guard.
- Save answer option ownership validation.

### P1 - MVP Production

- Full test coverage core flow.
- Settings endpoint.
- Anti-double login.
- Anti-cheat threshold.
- Admin export result.
- Strong audit log.

### P2 - Enhancement

- Import bank soal.
- Versioning bank/package.
- Advanced analytics.
- Notification email/WhatsApp.
- Proctoring webcam jika scope berubah.
