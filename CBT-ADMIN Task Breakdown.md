# CBT-ADMIN Task Breakdown

Dokumen ini adalah breakdown pekerjaan admin frontend `cbt-admin` berdasarkan kondisi repo saat ini dan API admin backend.

## Ringkasan Status

`cbt-admin` adalah aplikasi Next.js 16/React 19 dengan Tailwind 4 dan pnpm. Kondisi terbaru sudah memiliki:

- Login admin menggunakan API `POST /login`.
- Token admin disimpan di httpOnly cookie `cbt_admin_token`.
- Guard `/admin/*` melalui layout.
- Logout route untuk clear cookie.
- Admin shell/sidebar.
- Dashboard admin.
- Module list page untuk users, payment proofs, test approvals, question banks, questions, packages, sessions, monitoring, results.
- API client server-side `src/lib/admin-api.ts`.
- Detail page generic `/admin/[section]/[id]`.
- Fetch list/count/filter sebagian modul.

Masih banyak action mutasi, form create/edit, file upload/preview, dan UX detail yang belum selesai.

## Scope MVP Admin

### 1. Environment dan Runtime

Status: Parsial.

Task:

- Pastikan `.env.local.example` tersedia:
  - `CBT_API_BASE_URL=https://be-cbt.miftadigital.cloud/api`
  - `CBT_ADMIN_TOKEN=`
- Catatan runtime:
  - Next.js 16 butuh Node `>=20.9.0`.
  - Local yang memakai Node 18 tidak bisa `next build`.
- Tambahkan dokumentasi run:
  - `pnpm install`
  - `pnpm dev`
  - `pnpm build`
  - `pnpm start`
- Pastikan production tidak memakai `CBT_ADMIN_TOKEN` sebagai global bypass kecuali untuk debugging internal.

Acceptance criteria:

- Admin bisa build di Node 20+.
- Env tidak bocor ke client.

## 2. Admin Auth

Status: Parsial.

Yang sudah ada:

- Login action.
- Cookie token.
- Guard admin layout.
- Logout route.

Task:

- Guard harus memvalidasi token ke `GET /api/me`, bukan hanya mengecek cookie ada.
- Jika token invalid/expired:
  - clear cookie
  - redirect login.
- Jika user role bukan admin:
  - clear cookie
  - redirect login dengan error.
- Tambahkan loading/submit disabled di login.
- Tambahkan handling error API yang tidak redirect loop.
- Tambahkan session expiry UX.

Acceptance criteria:

- Cookie palsu tidak bisa membuka admin.
- Participant token tidak bisa membuka admin.
- Logout benar-benar clear cookie.

## 3. API Client dan Data Fetching

Status: Parsial.

Yang sudah ada:

- Server-only API client.
- Base URL dari env.
- Bearer token dari cookie.
- List/detail request.
- Fallback state jika API gagal.
- Count stats beberapa modul.

Task:

- Hilangkan fallback mock untuk production mode, atau tampilkan fallback hanya development.
- Tambahkan strict empty state:
  - API sukses kosong harus tampil kosong.
  - API gagal harus tampil error.
- Tambahkan shared pagination type:
  - current_page
  - last_page
  - per_page
  - total.
- Tambahkan support query params dari UI:
  - search
  - status
  - page
  - per_page
  - session_id
  - date.
- Tambahkan request mutation helper:
  - POST
  - PATCH
  - DELETE
  - multipart/form-data.
- Tambahkan cache policy final:
  - admin list pakai `no-store`.
  - detail juga `no-store` kecuali ada kebutuhan cache.

Acceptance criteria:

- Data yang tampil bisa dibuktikan berasal dari API.
- Filter dan pagination tidak merusak route.

## 4. Dashboard Admin

Status: Parsial.

Task:

- Dashboard count harus berasal dari endpoint per status:
  - pending users
  - pending payment proofs
  - draft sessions
  - available approvals / pending work.
- Tambahkan newest pending dari API:
  - users pending
  - payments pending
  - sessions draft
  - approvals available.
- Tambahkan link filter yang sesuai:
  - `/admin/users?account_status=pending_verification`
  - `/admin/payment-proofs?status=pending_review`
  - `/admin/exam-sessions?status=draft`
  - `/admin/test-approvals?status=available`
- Tambahkan refresh action jika perlu.
- Jangan tampilkan angka fallback tanpa label development.

Acceptance criteria:

- Dashboard angka sama dengan API.
- Klik card membuka list terfilter.

## 5. Payment Proof Review

Status: List/detail parsial, action belum selesai.

Task:

- List:
  - fetch `GET /admin/payment-proofs`.
  - filter status: pending_review, approved, rejected.
  - count per status via API.
- Detail:
  - fetch `GET /admin/payment-proofs/{id}`.
  - tampilkan user, file_name, mime, amount, payment_date, status, reviewed_by, reviewed_at, rejection_reason.
- File preview:
  - butuh backend endpoint/URL aman untuk preview/download.
  - tampilkan PDF/image preview.
- Approve action:
  - `PATCH /admin/payment-proofs/{id}/approve`.
  - setelah sukses revalidate list/detail.
  - tampilkan generated test approval jika backend mengembalikan.
- Reject action:
  - `PATCH /admin/payment-proofs/{id}/reject`.
  - wajib isi rejection_reason.
- Disable action jika status bukan pending_review.

Acceptance criteria:

- Admin bisa approve/reject dari UI.
- Count berubah setelah action.
- File bukti bisa dilihat.

## 6. User Management

Status: List/detail parsial, action belum selesai.

Task:

- List:
  - `GET /admin/users`
  - filter account_status
  - search nama/email
- Detail:
  - `GET /admin/users/{id}`
  - tampilkan profile dan payment proofs.
- Actions:
  - approve account
  - reject account
  - reset password.
- Tambahkan confirmation modal untuk action penting.
- Tambahkan field rejection reason jika backend mendukung.

Acceptance criteria:

- Admin bisa mengaktifkan user pending.
- Admin bisa melihat riwayat payment user.

## 7. Test Approval Management

Status: List/detail parsial.

Task:

- List:
  - `GET /admin/test-approvals`.
  - filter status/user_id.
- Detail:
  - `GET /admin/test-approvals/{id}`.
  - tampilkan user dan payment proof.
- Tampilkan lifecycle:
  - available
  - assigned
  - consumed
  - expired.
- Link ke:
  - user detail
  - payment proof detail
  - session/attempt jika sudah assigned/consumed.

Acceptance criteria:

- Admin bisa tracking hak tes dari pembayaran sampai attempt.

## 8. Question Bank Management

Status: List/detail parsial.

Task:

- List `GET /admin/question-banks`.
- Detail `GET /admin/question-banks/{id}`.
- Create:
  - code
  - title
  - description
  - is_active.
- Edit.
- Deactivate/delete dengan confirmation.
- Tampilkan questions_count.
- Tambahkan warning jika bank dipakai package/session aktif.

Acceptance criteria:

- Admin bisa create/update bank soal dari UI.
- Bank aktif bisa dipilih pada package builder.

## 9. Question Management

Status: List/detail parsial, form editor belum selesai.

Task:

- List:
  - `GET /admin/questions`
  - filter bank, section, type, search.
- Detail:
  - `GET /admin/questions/{id}`.
- Create/edit multipart:
  - question_bank_id
  - section_type
  - question_type
  - difficulty_level
  - stem_html
  - explanation_html
  - image
  - audio
  - audio_max_play_count
  - options.
- Rich text editor:
  - support Arabic
  - RTL toggle
  - paste HTML safely.
- Option editor:
  - add/remove option
  - mark exactly one correct.
- Media preview:
  - image preview
  - audio preview.
- Delete/deactivate action.

Acceptance criteria:

- Admin bisa membuat soal text/image/audio/mixed.
- Arabic tidak rusak.
- Options terkirim sesuai backend.

## 10. Exam Package Builder

Status: List/detail parsial, builder belum selesai.

Task:

- List `GET /admin/exam-packages`.
- Detail `GET /admin/exam-packages/{id}`.
- Create/edit:
  - code
  - title
  - description
  - duration_minutes
  - shuffle_questions
  - shuffle_options
  - max_tab_switch
  - max_fullscreen_exit
  - is_active
  - banks array.
- Bank mapping UI:
  - question_bank_id
  - section_type
  - question_count
  - sort_order.
- Validasi frontend stok soal jika API menyediakan count.
- Tampilkan backend validation errors per row.

Acceptance criteria:

- Admin bisa membuat package valid untuk session.
- Error stok soal tampil jelas.

## 11. Exam Session Management

Status: List/detail parsial, action belum selesai.

Task:

- List:
  - `GET /admin/exam-sessions`.
  - filter status/date.
- Detail:
  - `GET /admin/exam-sessions/{id}`.
- Create/edit:
  - exam_package_id
  - code
  - title
  - description
  - session_date
  - start_time
  - end_time
  - duration_minutes
  - max_participants
  - show_result_to_user
  - auto_generate_enabled
  - access_code.
- Actions:
  - publish
  - close
  - manual assign participant
  - auto generate participant
  - remove participant.
- Participants tab:
  - `GET /admin/exam-sessions/{id}/participants`.

Acceptance criteria:

- Admin bisa publish session dan assign participant dari UI.
- Participant count update setelah action.

## 12. Monitoring

Status: List/detail parsial.

Task:

- Attempts list:
  - `GET /admin/exam-attempts`.
  - filter session/status.
- Attempt detail:
  - `GET /admin/exam-attempts/{id}`.
  - user
  - session
  - status
  - progress
  - remaining_seconds
  - last_activity_at
  - ip/user_agent.
- Violations:
  - `GET /admin/violations`.
  - filter attempt_id/type.
- Tambahkan live refresh interval opsional.
- Tambahkan severity visual.

Acceptance criteria:

- Admin bisa melihat attempt berjalan dan violation.

## 13. Results dan Export

Status: List/detail parsial.

Task:

- Results list:
  - `GET /admin/results`.
  - filter session_id.
- Detail:
  - skor total dan per section.
  - correct/wrong/unanswered.
- Export:
  - link `GET /admin/export/results?session_id=...`.
  - handle download file.
- Publish/unpublish result jika backend menyediakan endpoint.

Acceptance criteria:

- Admin bisa melihat dan export hasil per session.

## 14. Forms, Mutations, dan Revalidation

Status: Belum matang.

Task:

- Buat server actions atau route handlers untuk action admin.
- Gunakan `revalidatePath` setelah mutation.
- Tampilkan toast sukses/error.
- Mapping validation errors dari Laravel ke form field.
- Confirmation modal untuk destructive/high impact action.
- Loading state tombol action.
- Prevent double submit.

Acceptance criteria:

- Action admin tidak hanya tombol visual.
- Setelah action, data UI berubah sesuai API.

## 15. Detail Page Generic

Status: Parsial.

Task:

- Pastikan detail page semua module menampilkan field penting.
- Tambahkan relation links:
  - payment -> user/test approval
  - approval -> user/payment/session
  - session -> package/participants
  - attempt -> user/session/result/violations.
- Tambahkan action contextual di detail page.
- Jika detail API 404, tampilkan not found yang jelas.

Acceptance criteria:

- Detail page berguna untuk operasi admin, bukan hanya record dump.

## 16. UI/UX Admin

Status: Cukup baik untuk dashboard/list, perlu state polish.

Task:

- Empty state per module.
- Error state API gagal.
- Skeleton loading.
- Filter bar.
- Pagination controls.
- Responsive table mobile.
- Breadcrumb.
- Consistent badge/tone.
- Remove fallback data dari production.

Acceptance criteria:

- Admin bisa membedakan data kosong vs API error.
- Table besar tetap usable di desktop dan mobile.

## 17. Testing dan QA

Status: Belum cukup.

Task:

- Lint dan TypeScript check wajib.
- Add Playwright smoke:
  - redirect `/` ke login.
  - login admin.
  - open payment proofs.
  - approve/reject modal.
  - create question bank.
  - create session.
- Mock API tests untuk mapper `admin-api.ts`.
- Manual QA checklist per module.

Acceptance criteria:

- Critical admin flow tidak regress saat update UI.

## Prioritas Sprint

### P0 - Integrasi Admin Wajib

- Strict auth validation via `/me`.
- Payment proof approve/reject + file preview.
- Users approve/reject/reset.
- Real filter/pagination.
- Remove production fallback mock.

### P1 - Content Management

- Question bank CRUD.
- Question editor with media/RTE.
- Exam package builder.
- Exam session create/publish/assign.

### P2 - Monitoring dan Reporting

- Attempt live monitoring.
- Violation detail.
- Result export.
- Dashboard polish.
