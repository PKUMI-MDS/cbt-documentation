# Fix Attempt Not Found / Not In Progress (Frontend)

## Tujuan
- Menjaga UI ujian selalu sinkron dengan status attempt di backend.
- Menghentikan UI dan mengarahkan user bila attempt sudah tidak valid.
- Menghindari toast error berulang yang membuat UI terasa macet.

## Ringkasan Masalah
- Pesan "Attempt not found or not in progress" berasal dari backend dan diteruskan ke UI via ApiError.
- Error pada sendHeartbeat dan logViolation diabaikan sehingga UI tetap aktif walau attempt sudah ditutup server.
- attempt_id pada URL belum divalidasi dengan resumeExam sehingga bisa memakai attempt yang sudah tidak valid.
- Fallback exam settings bisa lebih longgar dari backend, memicu auto-submit server tanpa diketahui UI.

## Rencana Perbaikan
1. Deteksi attempt invalid
   - Buat helper isAttemptInvalidError(err) yang memeriksa:
     - err instanceof ApiError
     - err.code termasuk 404/409/410/422/403 (sesuaikan dengan backend)
     - atau err.message mengandung "attempt not found", "not in progress", "expired", "closed"
2. Handler terpusat
   - Buat handleAttemptInvalid(err, source):
     - Stop timerRef dan heartbeatRef
     - Lepas beforeunload
     - Keluar fullscreen bila aktif
     - Tampilkan toast/pesan yang ramah
     - Redirect ke /dashboard atau /exam/completed sesuai kebutuhan
3. Validasi attempt_id dari URL
   - Jika attempt_id ada, panggil resumeExam(attemptId) terlebih dulu.
   - Jika sukses, gunakan remaining_seconds, current_question_number, total_questions dari response.
   - Jika gagal karena invalid, panggil handleAttemptInvalid.
4. Tangani error heartbeat
   - Pada catch sendHeartbeat:
     - Jika invalid, panggil handleAttemptInvalid.
     - Jika bukan, tampilkan toast ringan (hindari spam).
5. Tangani error aksi soal
   - Pada getQuestion, saveAnswer, navigateQuestion, logViolation:
     - Jika invalid, panggil handleAttemptInvalid.
     - Jika bukan, tampilkan toast normal.
6. Sinkron limit pelanggaran
   - Jika getExamSettings gagal, tampilkan toast bahwa aturan tidak dapat dimuat.
   - Pertimbangkan fallback ke limit lebih ketat agar UI tidak lebih longgar dari backend.

## Perubahan per File
### app/exam/page.tsx
- Tambah helper isAttemptInvalidError dan handleAttemptInvalid.
- Update initExam:
  - attempt_id ada -> resumeExam(attemptId)
  - attempt_id tidak ada -> getActiveAttempt()
- Update sendHeartbeat catch untuk memanggil handleAttemptInvalid.
- Update error handling pada:
  - loadQuestion
  - saveAnswer
  - navigateQuestion
  - logViolation
- (Opsional) Rate-limit toast error agar tidak muncul berulang.

### lib/auth-api.ts
- Pastikan resumeExam digunakan dan hasilnya dipakai untuk state awal.
- (Opsional) Tambahkan helper khusus untuk parsing error attempt invalid jika perlu.

### lib/api.ts
- Tidak wajib diubah, tetapi bisa ditambah mapping error code ke pesan yang lebih ramah bila diperlukan.

## Kriteria Sukses
- Saat attempt sudah tidak valid, UI berhenti dan user diarahkan dengan pesan jelas.
- Navigasi soal dan simpan jawaban tidak lagi memicu toast error berulang.
- Status pelanggaran di UI konsisten dengan server (atau lebih ketat bila fallback).

## Langkah Uji Manual
1. Buka ujian normal, pindah soal, simpan jawaban -> berjalan normal.
2. Simulasikan attempt kadaluarsa/ditutup server:
   - UI menampilkan pesan dan redirect ke dashboard/selesai.
3. Buka /exam?attempt_id=invalid:
   - UI tidak crash dan langsung diarahkan.
4. Putuskan koneksi sesaat saat heartbeat:
   - UI tidak spam toast, tetap recover saat koneksi kembali.

## Catatan
- Pesan error backend tetap boleh ditampilkan, namun untuk error attempt invalid sebaiknya ditampilkan versi yang lebih ramah.
- Pastikan cleanup event listener dan interval agar tidak ada kebocoran state.
