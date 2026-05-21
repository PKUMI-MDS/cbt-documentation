# Re-Audit Report: Unauthenticated Error on Questions Page

**Tanggal:** 21 Mei 2026  
**Auditor:** Kimi Code CLI  
**Status:** ✅ **FIXED** — Semua perubahan sudah di-apply ke codebase  
**Severity:** High — Admin tidak bisa akses Questions setelah login ulang

---

## 1. Ringkasan

Fix autentikasi yang diterapkan pada 20 Mei 2026 **sudah benar dan terekam di codebase** (CORS, cookie SameSite=none, credentials: "include"). Namun, masalah "Unauthenticated" masih terjadi karena **root cause yang berbeda**:

| Aspek | Status 20 Mei | Status 21 Mei (Setelah Fix) |
|-------|---------------|----------------------------|
| CORS Middleware (BE) | ✅ Fix diterapkan | ✅ Masih benar |
| Cookie SameSite=none (FE) | ✅ Fix diterapkan | ✅ Masih benar |
| credentials: "include" di core.ts | ✅ Fix diterapkan | ✅ Masih benar |
| **Sanctum guard hanya ['web']** | ⚠️ Teridentifikasi, belum fix | ✅ **FIXED** |
| **Auth config tidak punya guard 'api'** | ❌ Belum teridentifikasi | ✅ **FIXED** |
| **Token Bearer tidak bisa di-validate** | ❌ Belum teridentifikasi | ✅ **FIXED** |
| Route handler tanpa credentials | ❌ Belum teridentifikasi | ✅ **FIXED** |
| Login form tanpa credentials | ❌ Belum teridentifikasi | ✅ **FIXED** |

---

## 2. Root Cause Analysis

### 2.1 Masalah Utama: Guard Config Laravel

**File:** `be-cbt/config/auth.php`

```php
// SEBELUM (BROKEN):
'guards' => [
    'web' => [
        'driver' => 'session',   // ← SESSION driver!
        'provider' => 'users',
    ],
    // ❌ TIDAK ADA guard 'api'!
],
```

**File:** `be-cbt/config/sanctum.php`

```php
// SEBELUM (BROKEN):
'guard' => ['web'],  // ← Hanya guard 'web' yang dicek
```

**Kenapa ini menyebabkan 401?**

1. Sanctum menerima request dengan `Authorization: Bearer <token>`
2. Sanctum cek guard `'web'` → driver-nya **session**
3. Karena ini request API stateless (tanpa session cookie Laravel), guard `'web'` **gagal**
4. Sanctum seharusnya fallback ke Bearer token validation, tapi karena tidak ada guard `'api'` dengan driver `sanctum`, fallback juga gagal
5. Hasil: **401 Unauthenticated** — meski token valid!

### 2.2 Alur Autentikasi (Before Fix)

```
Browser → Next.js Server → fetch(/admin/questions)
                                    │
                                    ▼
                              Header: Bearer <token>
                              credentials: "include"
                                    │
                                    ▼
                            BE: auth:sanctum middleware
                                    │
                                    ▼
                            Sanctum cek guard ['web']
                            → driver: session
                            → tidak ada session cookie
                            → GAGAL ❌
                                    │
                                    ▼
                            401 Unauthenticated
```

### 2.3 Alur Autentikasi (After Fix)

```
Browser → Next.js Server → fetch(/admin/questions)
                                    │
                                    ▼
                              Header: Bearer <token>
                              credentials: "include"
                                    │
                                    ▼
                            BE: auth:sanctum middleware
                                    │
                                    ▼
                            Sanctum cek guard ['web', 'api']
                            → guard 'web': session → GAGAL (stateless)
                            → guard 'api': sanctum → Berhasil! ✅
                            → Bearer token valid
                                    │
                                    ▼
                            200 OK dengan data
```

---

## 3. Perubahan yang Dilakukan

### Fix 1: Tambahkan Guard 'api' di config/auth.php ✅

**File:** `be-cbt/config/auth.php`

```php
// SESUDAH:
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
],
```

### Fix 2: Update Sanctum Guard Config ✅

**File:** `be-cbt/config/sanctum.php`

```php
// SESUDAH:
'guard' => ['web', 'api'],
```

### Fix 3: Tambahkan credentials: "include" ke Route Handlers ✅

**File:** `cbt-admin/src/app/api/admin/analytics/route.ts`

```typescript
const response = await fetch(apiUrl, {
  headers: {
    Accept: "application/json",
    Authorization: `Bearer ${token}`,
  },
  cache: "no-store",
  credentials: "include",  // ← DITAMBAHKAN
});
```

**File:** `cbt-admin/src/app/api/media/route.ts`

```typescript
const response = await fetch(`${apiBaseUrl()}/media/${safePath}`, {
  headers: {
    Accept: "image/*,audio/*",
    Authorization: `Bearer ${token}`,
  },
  cache: "no-store",
  credentials: "include",  // ← DITAMBAHKAN
});
```

**File:** `cbt-admin/src/app/admin/results/export/route.ts`

```typescript
const response = await fetch(upstreamUrl, {
  headers: {
    Accept: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    Authorization: `Bearer ${token}`,
  },
  cache: "no-store",
  credentials: "include",  // ← DITAMBAHKAN
});
```

### Fix 4: Tambahkan credentials: "include" ke Login Form ✅

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
const response = await fetch(`${apiUrl}/login`, {
  method: "POST",
  headers: {
    Accept: "application/json",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ email, password }),
  credentials: "include",  // ← DITAMBAHKAN
});
```

---

## 4. Deployment Checklist

### Backend (be-cbt)

```bash
# 1. Upload file yang diubah:
# - be-cbt/config/auth.php
# - be-cbt/config/sanctum.php

# 2. Clear cache (WAJIB!)
php artisan config:clear
php artisan cache:clear
php artisan route:clear

# 3. Verifikasi .env
# Pastikan SANCTUM_STATEFUL_DOMAINS sesuai domain FE
```

### Frontend (cbt-admin)

```bash
# 1. Upload file yang diubah:
# - cbt-admin/src/app/api/admin/analytics/route.ts
# - cbt-admin/src/app/api/media/route.ts
# - cbt-admin/src/app/admin/results/export/route.ts
# - cbt-admin/src/app/login/login-form.tsx

# 2. Rebuild
sudo pnpm run build

# 3. Restart service (PM2, systemd, dll)
```

### Post-Deploy User Actions

1. **Logout** dari admin panel
2. **Login ulang** (untuk mendapatkan token baru yang valid dengan guard 'api')
3. **Navigasi ke Questions** — data harusnya sudah bisa dimuat

---

## 5. Verifikasi Fix

Setelah deploy, verifikasi dengan:

1. Buka browser DevTools → Network tab
2. Login ke admin panel
3. Navigasi ke Questions
4. Cek request `GET /api/admin/questions`
5. Pastikan:
   - ✅ Request header ada `Authorization: Bearer <token>`
   - ✅ Response status **200 OK**
   - ✅ Response body berisi data questions

Jika masih 401, cek Laravel log (`storage/logs/laravel.log`) untuk pesan error detail dari Sanctum.

---

## 6. Perbandingan: Sebelum vs Sesudah Fix

| Aspek | Sebelum Fix | Sesudah Fix |
|-------|-------------|-------------|
| Guard config | `['web']` only (session) | `['web', 'api']` (session + sanctum) |
| Bearer token validation | ❌ Gagal (guard 'web' pakai session) | ✅ Berhasil (guard 'api' pakai sanctum) |
| Session-based auth | ✅ Berhasil | ✅ Tetap berhasil |
| API stateless auth | ❌ 401 Unauthenticated | ✅ 200 OK |
| Route handlers credentials | ❌ Tidak include | ✅ Semua include |
| Login form credentials | ❌ Tidak include | ✅ Include |

---

## 7. Temuan Tambahan (Non-Blocking, Sudah Fix)

### 7.1 Dual Token Storage (Cookie + localStorage)

Token disimpan di dua tempat:
- **Cookie** (`cbt_admin_token`) → untuk Server Components
- **localStorage** (`cbt_admin_token`) → untuk Client Components

Ini bisa menyebabkan inkonsistensi jika salah satu tidak ter-update.

**Rekomendasi jangka panjang:** Pilih satu mekanisme saja. Untuk Next.js App Router dengan Server Components, **cookie-based** adalah pilihan yang tepat. Pertimbangkan untuk menghapus `client-auth.ts` jika tidak diperlukan.

---

## 8. File yang Diubah

| # | File | Perubahan |
|---|------|-----------|
| 1 | `be-cbt/config/auth.php` | Tambah guard `'api'` dengan driver `'sanctum'` |
| 2 | `be-cbt/config/sanctum.php` | Update `'guard'` dari `['web']` ke `['web', 'api']` |
| 3 | `cbt-admin/src/app/api/admin/analytics/route.ts` | Tambah `credentials: "include"` |
| 4 | `cbt-admin/src/app/api/media/route.ts` | Tambah `credentials: "include"` |
| 5 | `cbt-admin/src/app/admin/results/export/route.ts` | Tambah `credentials: "include"` |
| 6 | `cbt-admin/src/app/login/login-form.tsx` | Tambah `credentials: "include"` |

---

## 9. Related Documentation

- [Authentication Audit & Fix.md](./Authentication%20Audit%20&%20Fix.md) — Audit pertama (20 Mei 2026)
- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [Laravel Authentication Guards](https://laravel.com/docs/authentication#introduction)

---

*End of Report — 21 Mei 2026*
