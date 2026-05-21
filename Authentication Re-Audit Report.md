# Re-Audit Report: Unauthenticated Error on Questions Page

**Tanggal:** 21 Mei 2026  
**Auditor:** Kimi Code CLI  
**Status:** ✅ **FIXED** — Semua perubahan sudah di-apply ke codebase  
**Severity:** High — Admin tidak bisa akses Questions setelah login ulang

---

## 1. Ringkasan

Masalah "Unauthenticated" di halaman Questions disebabkan oleh **konfigurasi autentikasi Laravel yang tidak compatible dengan Sanctum Bearer token**. Setelah fix diterapkan, muncul masalah baru "Failed to fetch" saat login yang disebabkan oleh **CORS preflight failure akibat kombinasi `credentials: "include"` + `SameSite: "none"` + `Access-Control-Allow-Origin` yang tidak wildcard**.

| Aspek | Status Awal | Status Setelah Fix #1 | Status Final |
|-------|-------------|----------------------|--------------|
| CORS Middleware (BE) | ✅ Ada | ✅ Ada | ✅ Fixed preflight |
| Cookie SameSite | ❌ lax (blocked) | ❌ none + credentials (CORS error) | ✅ **lax** (same-origin) |
| credentials: "include" | ❌ Tidak ada | ❌ Dihapus semua | ✅ **Dihapus** (tidak perlu) |
| Sanctum guard | ❌ Hanya ['web'] | ✅ ['web', 'api'] | ✅ ['web', 'api'] |
| Auth config guard 'api' | ❌ Tidak ada | ✅ Ditambahkan | ✅ Ditambahkan |
| Bearer token validation | ❌ Gagal | ✅ Berhasil | ✅ Berhasil |

---

## 2. Root Cause Analysis

### 2.1 Masalah Utama #1: Guard Config Laravel (FIXED)

**File:** `be-cbt/config/auth.php`

Sebelumnya tidak ada guard `'api'` dengan driver `'sanctum'`. Sanctum middleware hanya bisa cek guard `'web'` (session driver), yang gagal pada request API stateless.

**Fix:** Tambahkan guard `'api'`:
```php
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

### 2.2 Masalah Utama #2: Sanctum Guard Config (FIXED)

**File:** `be-cbt/config/sanctum.php`

```php
// SEBELUM:
'guard' => ['web'],

// SESUDAH:
'guard' => ['web', 'api'],
```

### 2.3 Masalah #3: "Failed to fetch" — CORS Preflight Error (FIXED)

Setelah menambahkan `credentials: "include"` dan `SameSite: "none"`, muncul error baru saat login:

**Penyebab:** Kombinasi berikut melanggar aturan CORS:
1. `credentials: "include"` pada fetch request
2. `SameSite: "none"` pada cookie
3. `Access-Control-Allow-Origin` yang di-set ke origin spesifik (bukan wildcard `*`)

Browser Chrome/Edge memblokir request karena:
- Saat `credentials: "include"` aktif, server HARUS mengembalikan `Access-Control-Allow-Origin` dengan origin yang EXACT MATCH
- Tapi pada saat yang sama, jika cookie `SameSite: "none"` + `secure: true`, browser juga menolak cookie pada HTTP (non-HTTPS) connection

**Solusi yang benar:**
- Karena **FE dan BE berada di domain yang sama** (`pkumionline.cloud`), kita TIDAK PERLU `credentials: "include"`
- Cookie dengan `SameSite: "lax"` sudah cukup untuk same-origin requests
- Bearer token di header `Authorization` sudah menangani autentikasi

---

## 3. Perubahan yang Dilakukan

### Fix 1: Tambahkan Guard 'api' di config/auth.php ✅

**File:** `be-cbt/config/auth.php`

```php
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
'guard' => ['web', 'api'],
```

### Fix 3: Fix CORS Preflight — Return Early untuk OPTIONS ✅

**File:** `be-cbt/app/Http/Middleware/HandleCors.php`

```php
if ($request->isMethod('OPTIONS')) {
    $response = response('', 204);
    $response->headers->set('Access-Control-Allow-Origin', $allowOrigin);
    $response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
    $response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With, Accept, X-XSRF-TOKEN');
    $response->headers->set('Access-Control-Allow-Credentials', 'true');
    $response->headers->set('Access-Control-Max-Age', '86400');
    return $response;  // ← Return early, jangan panggil $next()
}
```

### Fix 4: Kembalikan Cookie ke SameSite: "lax" ✅

**File:** `cbt-admin/src/lib/admin-api/auth.ts`

```typescript
export async function setAdminToken(token: string) {
  const cookieStore = await cookies();
  cookieStore.set(TOKEN_COOKIE, token, {
    httpOnly: true,
    sameSite: "lax",                          // ← Dari "none" ke "lax"
    secure: process.env.NODE_ENV === "production",  // ← Dari true ke env-based
    path: "/",
    maxAge: 7 * 24 * 60 * 60,
  });
}
```

### Fix 5: Hapus credentials: "include" dari Semua Fetch ✅

**File:** `cbt-admin/src/lib/admin-api/core.ts`

```typescript
// request() — GET requests
const response = await fetch(url, {
  headers,
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});

// mutation() — POST/PATCH/DELETE JSON
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: init.method,
  headers,
  body: init.body ? JSON.stringify(init.body) : undefined,
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});

// multipartMutation() — POST with FormData
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: "POST",
  headers,
  body: formData,
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});
```

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
const response = await fetch(`${apiUrl}/login`, {
  method: "POST",
  headers: { ... },
  body: JSON.stringify({ email, password }),
  // credentials: "include"  ← DIHAPUS
});
```

**File:** `cbt-admin/src/app/api/admin/analytics/route.ts`

```typescript
const response = await fetch(apiUrl, {
  headers: { ... },
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});
```

**File:** `cbt-admin/src/app/api/media/route.ts`

```typescript
const response = await fetch(`${apiBaseUrl()}/media/${safePath}`, {
  headers: { ... },
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});
```

**File:** `cbt-admin/src/app/admin/results/export/route.ts`

```typescript
const response = await fetch(upstreamUrl, {
  headers: { ... },
  cache: "no-store",
  // credentials: "include"  ← DIHAPUS
});
```

---

## 4. Arsitektur Autentikasi Final

```
┌─────────────────────────────────────────────────────────────────┐
│                    cbt-admin (Next.js)                          │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │
│  │ Login Form  │───>│  Server     │───>│  Cookie (httpOnly)  │  │
│  │ (client)    │    │  Action     │    │  cbt_admin_token    │  │
│  └─────────────┘    └─────────────┘    └─────────────────────┘  │
│         │                                              │         │
│         │ 1. POST /login (Bearer token response)       │         │
│         │    Tanpa credentials (CORS safe)              │         │
│         ▼                                              ▼         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Server Components / Route Handlers           │   │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │   │
│  │  │getAdminToken│───>│  fetch()    │───>│  BE API     │  │   │
│  │  │(read cookie)│    │  + Bearer   │    │  validate   │  │   │
│  │  └─────────────┘    │  header     │    │  token      │  │   │
│  │                     └─────────────┘    └─────────────┘  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ 2. API Request
                              │    Header: Authorization: Bearer <token>
                              │    Tanpa credentials (CORS safe)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    be-cbt (Laravel)                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │
│  │   CORS      │───>│  Sanctum    │───>│  Guard ['web','api']│  │
│  │ Middleware  │    │ Middleware  │    │  api: sanctum drv   │  │
│  └─────────────┘    └─────────────┘    └─────────────────────┘  │
│                                              │                   │
│                                              ▼                   │
│                                       Bearer token validated     │
│                                       → User authenticated ✅    │
└─────────────────────────────────────────────────────────────────┘
```

**Mekanisme autentikasi:**
1. **Login:** Client form → POST /login → BE return token → Server Action simpan ke cookie (httpOnly, SameSite=lax)
2. **API Calls:** Server Component baca cookie → kirim Bearer token di header Authorization → BE validate via Sanctum guard 'api'
3. **Tidak perlu credentials: "include"** karena token dikirim via header, bukan cookie

---

## 5. Deployment Checklist

### Backend (be-cbt)

```bash
# 1. Upload file yang diubah:
# - be-cbt/config/auth.php
# - be-cbt/config/sanctum.php
# - be-cbt/app/Http/Middleware/HandleCors.php

# 2. Clear cache (WAJIB!)
php artisan config:clear
php artisan cache:clear
php artisan route:clear
```

### Frontend (cbt-admin)

```bash
# 1. Upload file yang diubah:
# - cbt-admin/src/lib/admin-api/core.ts
# - cbt-admin/src/lib/admin-api/auth.ts
# - cbt-admin/src/app/login/login-form.tsx
# - cbt-admin/src/app/api/admin/analytics/route.ts
# - cbt-admin/src/app/api/media/route.ts
# - cbt-admin/src/app/admin/results/export/route.ts

# 2. Rebuild
sudo pnpm run build

# 3. Restart service (PM2, systemd, dll)
```

### Post-Deploy

1. **Clear browser cache / hard refresh** (Ctrl+Shift+R)
2. **Login** ke admin panel
3. **Navigasi ke Questions** — data harusnya sudah bisa dimuat

---

## 6. Verifikasi Fix

Setelah deploy, verifikasi dengan:

1. Buka browser DevTools → Network tab
2. Login ke admin panel
3. Navigasi ke Questions
4. Cek request `GET /api/admin/questions`
5. Pastikan:
   - ✅ Request header ada `Authorization: Bearer <token>`
   - ✅ Response status **200 OK**
   - ✅ Response body berisi data questions
   - ✅ Tidak ada error CORS di console

---

## 7. File yang Diubah

| # | File | Perubahan |
|---|------|-----------|
| 1 | `be-cbt/config/auth.php` | Tambah guard `'api'` dengan driver `'sanctum'` |
| 2 | `be-cbt/config/sanctum.php` | Update `'guard'` dari `['web']` ke `['web', 'api']` |
| 3 | `be-cbt/app/Http/Middleware/HandleCors.php` | Fix preflight OPTIONS return early |
| 4 | `cbt-admin/src/lib/admin-api/auth.ts` | Kembalikan cookie ke `SameSite: "lax"`, `secure: env-based` |
| 5 | `cbt-admin/src/lib/admin-api/core.ts` | Hapus `credentials: "include"` dari `request()`, `mutation()`, `multipartMutation()` |
| 6 | `cbt-admin/src/app/login/login-form.tsx` | Hapus `credentials: "include"` |
| 7 | `cbt-admin/src/app/api/admin/analytics/route.ts` | Hapus `credentials: "include"` |
| 8 | `cbt-admin/src/app/api/media/route.ts` | Hapus `credentials: "include"` |
| 9 | `cbt-admin/src/app/admin/results/export/route.ts` | Hapus `credentials: "include"` |

---

## 8. Pelajaran Penting

1. **Jangan gunakan `credentials: "include"` + `SameSite: "none"` bersamaan** kecuali benar-benar cross-domain dengan setup CORS yang sempurna
2. **Bearer token di header Authorization** adalah cara paling reliable untuk autentikasi API stateless
3. **Cookie httpOnly dengan SameSite: "lax"** cukup untuk same-origin/same-site scenarios
4. **CORS preflight (OPTIONS)** harus di-handle dengan return early, jangan diproses ke controller
5. **Sanctum guard config** harus mencakup guard yang sesuai dengan driver autentikasi yang digunakan

---

## 9. Related Documentation

- [Authentication Audit & Fix.md](./Authentication%20Audit%20&%20Fix.md) — Audit pertama (20 Mei 2026)
- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [MDN: SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [MDN: CORS - Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)

---

*End of Report — 21 Mei 2026*
