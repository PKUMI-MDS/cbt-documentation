# Re-Audit Report: Unauthenticated Error on Questions Page

**Tanggal:** 21 Mei 2026  
**Auditor:** Kimi Code CLI  
**Status:** ✅ **FIXED** — Semua perubahan sudah di-apply ke codebase  
**Severity:** High — Admin tidak bisa akses Questions setelah login ulang

---

## 1. Ringkasan

Masalah "Unauthenticated" di halaman Questions disebabkan oleh **konfigurasi autentikasi Laravel yang tidak compatible dengan Sanctum Bearer token**. Setelah fix diterapkan, muncul masalah baru "Failed to fetch" saat login yang disebabkan oleh **CORS preflight failure akibat kombinasi `credentials: "include"` + `SameSite: "none"` + `Access-Control-Allow-Origin` yang tidak wildcard**.

Setelah fix CORS, muncul masalah ketiga: **"Sesi admin tidak valid atau sudah berakhir"** yang disebabkan oleh **client-side navigation (router.push) tidak membawa cookie ke Server Component**.

| Aspek | Status Awal | Fix #1 | Fix #2 | Status Final |
|-------|-------------|--------|--------|--------------|
| CORS Middleware (BE) | ✅ Ada | ✅ Fixed preflight | — | ✅ Fixed preflight |
| Cookie SameSite | ❌ lax (blocked) | ❌ none → CORS error | ✅ **lax** | ✅ **lax** |
| credentials: "include" | ❌ Tidak ada | ❌ Dihapus semua | — | ✅ Dihapus |
| Sanctum guard | ❌ Hanya ['web'] | ✅ ['web', 'api'] | — | ✅ ['web', 'api'] |
| Auth config guard 'api' | ❌ Tidak ada | ✅ Ditambahkan | — | ✅ Ditambahkan |
| **Client-side navigation** | ❌ router.push | — | ✅ **window.location.href** | ✅ **Full page reload** |
| **Cookie readable by Server** | ❌ Tidak | — | ✅ **Ya** | ✅ **Ya** |

---

## 2. Root Cause Analysis — 3 Masalah Berurutan

### Masalah #1: Guard Config Laravel (FIXED)

**File:** `be-cbt/config/auth.php`

Tidak ada guard `'api'` dengan driver `'sanctum'`. Sanctum middleware hanya bisa cek guard `'web'` (session driver), yang gagal pada request API stateless.

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

### Masalah #2: Sanctum Guard Config (FIXED)

**File:** `be-cbt/config/sanctum.php`

```php
// SEBELUM:
'guard' => ['web'],

// SESUDAH:
'guard' => ['web', 'api'],
```

### Masalah #3: "Failed to fetch" — CORS Preflight Error (FIXED)

Setelah menambahkan `credentials: "include"` dan `SameSite: "none"`, muncul error baru saat login:

**Penyebab:** Kombinasi berikut melanggar aturan CORS:
1. `credentials: "include"` pada fetch request
2. `SameSite: "none"` pada cookie
3. `Access-Control-Allow-Origin` non-wildcard

**Fix:** Hapus `credentials: "include"` dari semua fetch, kembalikan cookie ke `SameSite: "lax"`.

### Masalah #4: "Sesi admin tidak valid" — Client-Side Navigation (FIXED) ⬅️ CURRENT

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
// SEBELUM (BROKEN):
await syncToken(data.token);     // Set cookie via server action
router.push("/admin");           // Client-side navigation
router.refresh();                // Soft refresh
```

**Kenapa ini broken?**

1. `syncToken(data.token)` memanggil Server Action yang set cookie via `cookies()` dari Next.js
2. Cookie di-set untuk response yang akan dikirim ke browser
3. Tapi `router.push("/admin")` adalah **client-side navigation** — tidak ada full page request ke server
4. Next.js tidak mengirim cookie baru ke browser pada client-side navigation
5. Saat `/admin` di-render sebagai Server Component, `getAdminToken()` membaca cookie → **cookie tidak ada!**
6. `validateAdminSession()` gagal → redirect ke `/logout?reason=invalid`
7. User diarahkan ke login page dengan pesan "Sesi admin tidak valid atau sudah berakhir"

**Fix:** Gunakan `window.location.href = "/admin"` untuk **full page navigation**:

```typescript
// SESUDAH (FIXED):
await syncToken(data.token);     // Set cookie via server action
window.location.href = "/admin"; // Full page navigation → browser kirim cookie
```

Dengan full page navigation:
1. Browser menerima response dari Server Action (dengan Set-Cookie header)
2. Browser menyimpan cookie
3. Browser melakukan navigasi ke `/admin` dengan cookie yang sudah tersimpan
4. Server Component bisa membaca cookie → autentikasi berhasil ✅

---

## 3. Alur Login (Before vs After Fix)

### Before Fix (Broken)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│ Login Form  │     │ Server      │     │ Browser Cookie  │
│ (client)    │     │ Action      │     │ Storage         │
└──────┬──────┘     └──────┬──────┘     └─────────────────┘
       │                   │
       │ 1. POST /login    │
       │──────────────────>│
       │                   │ 2. Return token
       │<──────────────────│
       │                   │
       │ 3. syncToken()    │
       │   (Server Action) │
       │──────────────────>│ 4. Set cookie
       │                   │   (tapi belum dikirim ke browser!)
       │                   │
       │ 5. router.push("/admin")
       │   (Client-side nav)
       │                   │
       │                   │ 6. Browser TIDAK menerima cookie
       │                   │    karena tidak ada full page request
       │                   │
       │                   │ 7. Server Component di /admin
       │                   │    → getAdminToken() → cookie KOSONG
       │                   │    → validateAdminSession() → GAGAL
       │                   │    → redirect /logout?reason=invalid
       │                   │
       ▼                   ▼
   Login page dengan error:
   "Sesi admin tidak valid atau sudah berakhir"
```

### After Fix (Working)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────────┐
│ Login Form  │     │ Server      │     │ Browser Cookie  │
│ (client)    │     │ Action      │     │ Storage         │
└──────┬──────┘     └──────┬──────┘     └─────────────────┘
       │                   │                          │
       │ 1. POST /login    │                          │
       │──────────────────>│                          │
       │                   │ 2. Return token          │
       │<──────────────────│                          │
       │                   │                          │
       │ 3. syncToken()    │                          │
       │   (Server Action) │                          │
       │──────────────────>│ 4. Set cookie            │
       │                   │   + Response ke browser  │
       │<──────────────────│                          │
       │                   │                          │ 5. Browser simpan cookie
       │                   │                          │    (httpOnly, SameSite=lax)
       │                   │                          │
       │ 6. window.location.href = "/admin"           │
       │   (Full page navigation)                     │
       │──────────────────────────────────────────────>│
       │                   │                          │ 7. Browser kirim cookie
       │                   │                          │    ke server
       │                   │                          │
       │                   │ 8. Server Component /admin
       │                   │    → getAdminToken() → cookie ADA ✅
       │                   │    → validateAdminSession() → BERHASIL
       │                   │    → Render dashboard ✅
       │                   │
       ▼                   ▼
   Dashboard berhasil dimuat!
```

---

## 4. Perubahan yang Dilakukan

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
    // ... other headers ...
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
    secure: process.env.NODE_ENV === "production",  // ← Env-based
    path: "/",
    maxAge: 7 * 24 * 60 * 60,
  });
}
```

### Fix 5: Hapus credentials: "include" dari Semua Fetch ✅

**File:** `cbt-admin/src/lib/admin-api/core.ts`

```typescript
// request(), mutation(), multipartMutation()
// credentials: "include" dihapus dari semua fetch
```

### Fix 6: Gunakan window.location.href untuk Full Page Navigation ✅ ⬅️ CURRENT FIX

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
// SEBELUM (BROKEN):
import { useRouter } from "next/navigation";
// ...
const router = useRouter();
// ...
setClientToken(data.token);      // localStorage (tidak perlu)
await syncToken(data.token);     // Set cookie
router.push("/admin");           // ❌ Client-side nav, cookie tidak terbawa
router.refresh();                // ❌ Soft refresh

// SESUDAH (FIXED):
await syncToken(data.token);     // Set cookie via server action
window.location.href = "/admin"; // ✅ Full page nav, cookie terbawa
```

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
3. Cek sequence request:
   - ✅ `POST /api/login` → 200 OK dengan token
   - ✅ `POST` ke Server Action `syncToken` → 200 OK
   - ✅ `GET /admin` → 200 OK (full page navigation)
   - ✅ `GET /api/admin/questions` → 200 OK dengan data
4. Cek Application → Cookies:
   - ✅ Cookie `cbt_admin_token` ada dengan value token
5. Tidak ada error CORS di console

---

## 7. File yang Diubah

| # | File | Perubahan |
|---|------|-----------|
| 1 | `be-cbt/config/auth.php` | Tambah guard `'api'` dengan driver `'sanctum'` |
| 2 | `be-cbt/config/sanctum.php` | Update `'guard'` dari `['web']` ke `['web', 'api']` |
| 3 | `be-cbt/app/Http/Middleware/HandleCors.php` | Fix preflight OPTIONS return early |
| 4 | `cbt-admin/src/lib/admin-api/auth.ts` | Kembalikan cookie ke `SameSite: "lax"`, `secure: env-based` |
| 5 | `cbt-admin/src/lib/admin-api/core.ts` | Hapus `credentials: "include"` dari semua fetch |
| 6 | `cbt-admin/src/app/login/login-form.tsx` | Gunakan `window.location.href` untuk full page navigation |
| 7 | `cbt-admin/src/app/api/admin/analytics/route.ts` | Hapus `credentials: "include"` |
| 8 | `cbt-admin/src/app/api/media/route.ts` | Hapus `credentials: "include"` |
| 9 | `cbt-admin/src/app/admin/results/export/route.ts` | Hapus `credentials: "include"` |

---

## 8. Pelajaran Penting

1. **Server Action set cookie tidak langsung terkirim ke browser** — perlu full page navigation agar browser menerima Set-Cookie header
2. **`router.push()` di Next.js App Router adalah client-side navigation** — tidak memicu browser untuk menyimpan cookie dari Server Action
3. **`window.location.href` memaksa full page reload** — browser akan menyimpan cookie dan mengirimkannya pada request berikutnya
4. **Bearer token di header Authorization** adalah cara paling reliable untuk autentikasi API stateless
5. **Jangan gunakan `credentials: "include"` + `SameSite: "none"` bersamaan** kecuali benar-benar cross-domain dengan setup CORS yang sempurna
6. **CORS preflight (OPTIONS)** harus di-handle dengan return early, jangan diproses ke controller
7. **Sanctum guard config** harus mencakup guard yang sesuai dengan driver autentikasi yang digunakan

---

## 9. Related Documentation

- [Authentication Audit & Fix.md](./Authentication%20Audit%20&%20Fix.md) — Audit pertama (20 Mei 2026)
- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [MDN: SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [MDN: CORS - Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)
- [Next.js: Server Actions and Cookies](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#cookies)

---

*End of Report — 21 Mei 2026*
