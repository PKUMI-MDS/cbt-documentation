# Alignment BE-FE - Attempt Not Found / Not In Progress

Tanggal: 2026-05-16
Sumber: Rencana BE dan FE terkait "Attempt not found / not in progress"

## Ringkasan Alignment
- BE dan FE sama-sama mengutamakan status attempt sebagai sumber kebenaran.
- FE akan menghentikan UI bila attempt tidak valid dan mengarahkan user dengan pesan jelas.
- BE menambahkan payload status yang jelas saat navigasi gagal.
- Keduanya sepakat limit anti-cheat harus mengikuti paket (bukan default global).

## Gap yang Perlu Diselaraskan
1) Kontrak status code dan payload error belum final (FE masih mengandalkan substring pesan).
2) Sumber limit pelanggaran di FE belum pasti (resumeExam / attempt detail vs getExamSettings).
3) Mapping status attempt -> redirect halaman belum didefinisikan bersama.
4) Penanganan khusus auto-submit karena pelanggaran perlu konsisten dengan status code.

## Kontrak Respons Error (Usulan)
Gunakan payload yang konsisten untuk semua endpoint runtime (getQuestion, navigate, saveAnswer, markDoubtful, heartbeat, submit, logViolation).

### Skema Payload Error (umum)
```
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

### Matriks Status Code
| Skenario | HTTP | attempt_status | reason | Catatan FE |
| --- | --- | --- | --- | --- |
| Attempt tidak ditemukan | 404 | - | - | Tampilkan pesan umum dan redirect ke dashboard |
| Attempt bukan in_progress | 409 | submitted / auto_submitted / closed | timeout / manual / violation | Block UI dan redirect ke hasil/selesai |
| Attempt expired / closed | 410 | closed | expired | Block UI dan redirect ke hasil/selesai |
| Auto-submit karena pelanggaran | 403 | auto_submitted | violation | Tampilkan alasan pelanggaran dan redirect |

Catatan: FE sebaiknya memprioritaskan `data.attempt_status` dan `data.reason` bila ada.

## Sumber Limit Pelanggaran
- BE: expose `max_tab_switch`, `max_fullscreen_exit`, dan opsional `auto_submit_on_violation_limit` di detail attempt.
- FE: gunakan limit dari response attempt/resumeExam, hindari fallback yang lebih longgar.

## Mapping Status -> Redirect (Usulan)
| attempt_status | Tujuan FE | Catatan |
| --- | --- | --- |
| submitted | /exam/completed | Tampilkan hasil/selesai |
| auto_submitted | /exam/completed | Tampilkan alasan (timeout/violation) |
| closed | /exam/completed | Attempt ditutup sistem |
| invalid / not_found | /dashboard | Attempt tidak valid |

## Checklist Implementasi
### Backend
- Samakan guard status di semua endpoint runtime.
- Saat status bukan in_progress, return payload status (bukan 404 generik).
- Tambahkan limit paket ke resource attempt.
- Log alasan auto-submit dari scheduler.

### Frontend
- Implement `isAttemptInvalidError` berbasis status code + payload.
- `handleAttemptInvalid` menghentikan UI dan melakukan redirect.
- Validasi `attempt_id` dari URL via `resumeExam`.
- Pastikan limit di UI memakai data attempt, bukan default global.

## Kriteria Sukses
- UI berhenti dengan pesan jelas saat attempt tidak valid.
- Navigasi tetap normal sampai timeout atau pelanggaran melewati batas.
- Limit pelanggaran di UI konsisten dengan paket.

## Langkah Uji Manual (Ringkas)
1) Attempt normal: navigate, save, heartbeat berjalan normal.
2) Attempt timeout: UI menerima status, berhenti, redirect ke selesai.
3) Auto-submit karena pelanggaran: UI menampilkan alasan dan redirect.
4) Attempt invalid via URL: UI tidak crash, redirect ke dashboard.
