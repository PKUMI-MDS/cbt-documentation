# Perencanaan Implementasi Exam Setting

## Tujuan

Exam setting dipakai sebagai default awal saat admin membuat exam package dan exam session. Setelah nilai default diambil, admin tetap boleh mengubahnya sebelum data disimpan.

## Scope

- FE admin untuk prefill form create/edit exam package.
- FE admin untuk prefill form create/edit exam session.
- BE untuk menyediakan endpoint exam setting.
- BE untuk merge default exam setting ke request create package/session.
- BE tetap menjadi source of truth agar default tidak hanya bergantung pada UI.

## Referensi Implementasi Saat Ini

- Exam setting controller: [g:/Project/CBT/app/Http/Controllers/Api/Admin/ExamSettingController.php](g:/Project/CBT/app/Http/Controllers/Api/Admin/ExamSettingController.php)
- Create/update package: [g:/Project/CBT/app/Http/Controllers/Api/Admin/ExamPackageController.php](g:/Project/CBT/app/Http/Controllers/Api/Admin/ExamPackageController.php)
- Validasi package: [g:/Project/CBT/app/Http/Requests/Admin/StoreExamPackageRequest.php](g:/Project/CBT/app/Http/Requests/Admin/StoreExamPackageRequest.php)
- Validasi session: [g:/Project/CBT/app/Http/Requests/Admin/StoreExamSessionRequest.php](g:/Project/CBT/app/Http/Requests/Admin/StoreExamSessionRequest.php)
- Runtime exam attempt: [g:/Project/CBT/app/Http/Resources/ExamAttemptDetailResource.php](g:/Project/CBT/app/Http/Resources/ExamAttemptDetailResource.php)

## FE Plan

### 1. Ambil exam setting saat halaman dibuka
- Panggil GET `/api/admin/settings/exam`.
- Simpan response ke state form create/edit.
- Gunakan nilai default tersebut untuk mengisi field awal.

### 2. Prefill form create exam package
- Gunakan `default_duration_minutes`.
- Gunakan `default_shuffle_questions`.
- Gunakan `default_shuffle_options`.
- Gunakan `default_max_tab_switch`.
- Gunakan `default_max_fullscreen_exit`.

### 3. Prefill form create exam session
- Gunakan `default_duration_minutes`.
- Gunakan `default_show_result_to_user`.
- Gunakan `default_auto_generate_enabled`.

### 4. Tetap izinkan override manual
- Admin boleh mengubah semua field default sebelum submit.
- FE hanya membantu prefill, bukan memaksa nilai final.

### 5. UX fallback
- Jika endpoint exam setting gagal, gunakan fallback dari UI.
- Tampilkan form tetap bisa dipakai walau default tidak ter-load.

## BE Plan

### 1. Sediakan sumber default yang konsisten
- Endpoint GET `/api/admin/settings/exam` tetap dipakai sebagai sumber default.
- Setting disimpan di tabel `settings` dengan group `exam`.

### 2. Merge default ke create package
- Saat create exam package, jika FE tidak mengirim field tertentu, BE ambil default dari exam setting.
- Field yang relevan:
  - `duration_minutes`
  - `shuffle_questions`
  - `shuffle_options`
  - `max_tab_switch`
  - `max_fullscreen_exit`

### 3. Merge default ke create session
- Saat create exam session, jika FE tidak mengirim field tertentu, BE ambil default dari exam setting.
- Field yang relevan:
  - `duration_minutes`
  - `show_result_to_user`
  - `auto_generate_enabled`

### 4. Tetap pertahankan validasi backend
- FormRequest tidak boleh bergantung penuh pada FE.
- BE harus tetap validasi input final setelah default diterapkan.

### 5. Pisahkan default global dan nilai final
- Exam setting hanya menjadi input awal.
- Nilai final tetap disimpan di `exam_packages` dan `exam_sessions`.
- Runtime ujian membaca data dari package/session, bukan dari settings langsung.

## Alur yang Diinginkan

### Create Exam Package
1. FE load exam settings.
2. FE isi form package dengan default.
3. Admin ubah bila perlu.
4. FE kirim payload final ke BE.
5. BE simpan package ke database.

### Create Exam Session
1. FE load exam settings.
2. FE isi form session dengan default.
3. Admin ubah bila perlu.
4. FE kirim payload final ke BE.
5. BE simpan session ke database.

## Catatan Penting

- Jangan jadikan exam setting sebagai data runtime utama.
- Jangan biarkan package/session bergantung hanya pada FE.
- BE tetap harus bisa menyelesaikan request walaupun FE tidak mengirim default.
- Jika nanti ada perubahan default setting, itu hanya memengaruhi form awal, bukan data yang sudah tersimpan.

## Acceptance Criteria

- Form create exam package menampilkan nilai awal dari exam setting.
- Form create exam session menampilkan nilai awal dari exam setting.
- Admin bisa mengubah nilai default sebelum submit.
- Data final tersimpan di package/session, bukan di settings.
- API tetap aman jika request datang tanpa prefill dari FE.
- Runtime ujian tetap konsisten memakai data package/session yang tersimpan.

## Rekomendasi Implementasi

- FE: wajib untuk UX prefill.
- BE: wajib untuk konsistensi, validasi, dan fallback.
- Jangan hanya implementasi FE.
- Tambahkan fallback default di BE supaya behavior tidak tergantung UI.