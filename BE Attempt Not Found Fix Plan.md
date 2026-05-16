# Rencana Perbaikan - Attempt Not Found Saat Navigasi Ujian

## Konteks
Pengguna melihat toast "Attempt not found or not in progress" dan tidak bisa pindah/pilih soal walaupun jumlah pelanggaran belum melewati batas.

## Akar masalah saat ini (terkonfirmasi)
1) Endpoint API hanya menerima attempt dengan status `in_progress` (getQuestion, navigate, saveAnswer, markDoubtful). Saat status berubah, navigasi gagal.
2) Deteksi timeout tidak konsisten: `checkTimeout` membandingkan enum dengan string, sehingga heartbeat bisa tidak menandai timeout, sementara scheduler kemudian auto-submit dan mengubah status.
3) Batas pelanggaran dibaca dari paket (`max_tab_switch`, `max_fullscreen_exit`), tetapi UI sering membaca default global, sehingga angka yang tampil bisa lebih tinggi dari batas paket yang sebenarnya.
4) Scheduler auto-submit attempt setelah `ends_at` + grace. Jika `ends_at` atau timezone salah, status bisa berubah lebih awal.
5) Respons error API terlalu generik (404) dan tidak mengembalikan status atau alasan sebenarnya, sehingga UI tidak bisa menampilkan kondisi yang benar.

## Rencana perbaikan (backend)
### A) Normalisasi logika timeout
- Perbaiki perbandingan enum di `checkTimeout` agar enum-ke-enum (atau value-ke-value).
- Pastikan perhitungan timeout memakai timezone yang konsisten (Asia/Jakarta).

### B) Kembalikan status yang jelas saat navigasi gagal
- Saat query `IN_PROGRESS` gagal, ambil attempt berdasarkan ID + user lalu kembalikan status dan alasan, bukan 404.
- Contoh respons (HTTP 409):

```json
{
  "status": "error",
  "message": "Attempt is not in progress",
  "data": {
    "attempt_id": 123,
    "attempt_status": "auto_submitted",
    "submitted_at": "2026-05-16T08:11:00+07:00",
    "reason": "timeout"
  }
}
```

### C) Ekspose batas anti-cheat level paket ke client
- Tambahkan `max_tab_switch` dan `max_fullscreen_exit` (dan opsional `auto_submit_on_violation_limit`) ke resource detail attempt agar UI memakai batas yang benar.

### D) Log alasan auto-submit dari scheduler
- Saat scheduler auto-submit, tambahkan violation log (misalnya `suspicious_disconnect`) atau activity log dengan alasan `timeout`.

### E) Samakan guard status di semua endpoint runtime
- `getQuestion`, `navigate`, `saveAnswer`, `markDoubtful`, `heartbeat`, `submit` harus mengembalikan payload status yang konsisten saat bukan `IN_PROGRESS`.

## Rencana perbaikan (frontend)
1) Gunakan batas dari response attempt/session/package, bukan default global.
2) Saat API mengembalikan status != `in_progress`, tampilkan pesan blocking lalu arahkan ke halaman hasil atau resume.
3) Tangani respons `403` auto-submitted dari endpoint pelanggaran dengan menghentikan navigasi dan menampilkan alasan yang jelas.

## Touchpoint kode yang disarankan (backend)
- app/Services/ExamEngineService.php
- app/Http/Controllers/Api/User/ExamController.php
- app/Http/Resources/ExamAttemptDetailResource.php
- app/Console/Commands/AutoSubmitStaleAttempts.php
- app/Services/AntiCheatService.php (opsional: tambah payload alasan atau log)

## Rencana pengujian
1) Unit test: `checkTimeout` bernilai true saat now >= ends_at.
2) Feature test: heartbeat auto-submit saat timeout tercapai.
3) Feature test: navigate mengembalikan payload status saat attempt berstatus `auto_submitted`.
4) Feature test: pelanggaran di bawah batas paket tidak auto-submit.

## Checklist verifikasi
- Attempt tetap bisa dinavigasi sampai timeout atau pelanggaran melewati batas.
- Jika attempt berakhir, API mengembalikan status yang jelas dan UI memblokir dengan alasan yang benar.
- UI menampilkan batas yang benar per paket.

## Rollout
- Deploy perubahan backend.
- Update FE agar memakai field status/limit baru.
- Monitor log scheduler untuk event auto-submit dan bandingkan dengan laporan UI.
