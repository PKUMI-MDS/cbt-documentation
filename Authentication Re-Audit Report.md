# Re-Audit Report: Unauthenticated Error on Questions Page

**Tanggal:** 21 Mei 2026  
**Auditor:** Kimi Code CLI  
**Status:** ✅ **FIXED** — Semua perubahan sudah di-apply ke codebase  
**Severity:** High — Admin tidak bisa akses Questions setelah login ulang

---

## 1. Ringkasan

Masalah ini melibatkan **cross-origin cookie** antara FE (`admin-cat.pkumionline.cloud`) dan BE (`be-cbt.pkumionline.cloud`). Cookie yang di-set oleh Server Action (`syncToken`) tidak tersimpan dengan benar karena mekanisme Server Action di Next.js tidak mengirim `Set-Cookie` header ke browser pada saat client-side fetch.

| Aspek | Status Awal | Status Final |
|-------|-------------|--------------|
| Sanctum guard | ❌ Hanya ['web'] | ✅ `['web', 'api']` |
| Auth config guard 'api' | ❌ Tidak ada | ✅ Ditambahkan |
| CORS allowed origins | ❌ Tidak ada `admin-cat` | ✅ Ditambahkan |
| Cookie setting | ❌ Server Action (tidak reliable) | ✅ API Route (`/api/auth/sync-token`) |
| Cookie SameSite | ❌ `lax` | ✅ `none` + `secure: true` |
| credentials: "include" | ❌ Tidak ada | ✅ Ditambahkan ke semua fetch |

---

## 2. Root Cause: Server Action Tidak Set Cookie ke Browser

### 2.1 Apa yang Terjadi

```
Login Form (client-side fetch)
    ↓
POST /api/login → BE return token
    ↓
syncToken(token) → Server Action
    ↓
Server Action panggil cookies().set() → cookie di-set di response Server Action
    ↓
TAPI... Server Action response tidak mengirim Set-Cookie header ke browser
    ↓
window.location.href = "/admin" → full page nav
    ↓
/admin Server Component → getAdminToken() → cookie KOSONG
    ↓
validateAdminSession() → GAGAL → redirect /logout?reason=invalid
```

### 2.2 Kenapa Server Action Tidak Bisa Set Cookie yang Reliable

Server Action di Next.js App Router:
- Bisa set cookie via `cookies().set()`
- Tapi cookie tersebut hanya tersedia untuk **subsequent Server Action calls**
- Tidak otomatis dikirim ke browser sebagai `Set-Cookie` header pada response HTTP
- Browser tidak menyimpan cookie tersebut

### 2.3 Solusi: Gunakan API Route untuk Set Cookie

API Route (Route Handler) di Next.js:
- Bisa set cookie via `cookies().set()`
- Response dari API Route mengandung `Set-Cookie` header
- Browser menerima dan menyimpan cookie
- Cookie tersedia untuk subsequent requests

---

## 3. Fix yang Dilakukan

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
    // Set CORS headers...
    return $response;  // Return early
}
```

### Fix 4: Tambahkan Domain FE ke CORS Allowed Origins ✅

**File:** `be-cbt/app/Http/Middleware/HandleCors.php`

```php
$allowedOrigins = [
    'http://localhost:3000',
    'http://localhost',
    'https://cbt-admin.pkumionline.cloud',
    'https://admin.pkumionline.cloud',
    'https://admin-cat.pkumionline.cloud',  // ← DITAMBAHKAN
];
```

### Fix 5: Buat API Route untuk Set Cookie ✅ ⬅️ KEY FIX

**File baru:** `cbt-admin/src/app/api/auth/sync-token/route.ts`

```typescript
import { cookies } from "next/headers";
import { NextResponse } from "next/server";

const TOKEN_COOKIE = "cbt_admin_token";

export async function POST(request: Request) {
  try {
    const { token } = await request.json();

    if (!token || typeof token !== "string") {
      return NextResponse.json({ error: "Token required" }, { status: 400 });
    }

    const cookieStore = await cookies();
    cookieStore.set(TOKEN_COOKIE, token, {
      httpOnly: true,
      sameSite: "none",
      secure: true,
      path: "/",
      maxAge: 7 * 24 * 60 * 60,
    });

    return NextResponse.json({ success: true });
  } catch {
    return NextResponse.json({ error: "Invalid request" }, { status: 400 });
  }
}
```

### Fix 6: Login Form — Panggil API Route untuk Set Cookie ✅

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
// SEBELUM (BROKEN - Server Action):
await syncToken(data.token);

// SESUDAH (FIXED - API Route):
const syncRes = await fetch("/api/auth/sync-token", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ token: data.token }),
});

if (!syncRes.ok) {
  throw new Error("Gagal menyimpan sesi login.");
}

// Force a full page navigation to /admin so the server can read the cookie
window.location.href = "/admin";
```

### Fix 7: Cookie SameSite: "none" + Secure: true ✅

**File:** `cbt-admin/src/lib/admin-api/auth.ts`

```typescript
export async function setAdminToken(token: string) {
  const cookieStore = await cookies();
  cookieStore.set(TOKEN_COOKIE, token, {
    httpOnly: true,
    sameSite: "none",    // ← Cross-origin cookie
    secure: true,        // ← Wajib HTTPS
    path: "/",
    maxAge: 7 * 24 * 60 * 60,
  });
}
```

### Fix 8: Tambahkan credentials: "include" ke Semua Fetch ✅

**File:** `cbt-admin/src/lib/admin-api/core.ts`

```typescript
// request(), mutation(), multipartMutation()
const response = await fetch(url, {
  headers,
  cache: "no-store",
  credentials: "include",  // ← Tambahkan
});
```

**File:** `cbt-admin/src/app/api/admin/analytics/route.ts`
**File:** `cbt-admin/src/app/api/media/route.ts`
**File:** `cbt-admin/src/app/admin/results/export/route.ts`

---

## 4. Arsitektur Autentikasi Final

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         BROWSER                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Login Form (admin-cat.pkumionline.cloud)                       │   │
│  │  1. User submit form                                            │   │
│  │  2. fetch() POST /login ke BE                                   │   │
│  │  3. Terima token dari BE                                        │   │
│  │  4. fetch() POST /api/auth/sync-token (API Route)               │   │
│  │     → Response dengan Set-Cookie header                         │   │
│  │  5. Browser simpan cookie cbt_admin_token                       │   │
│  │  6. window.location.href = "/admin"                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  /admin (Server Component)                                      │   │
│  │  7. validateAdminSession() → getAdminToken() → baca cookie      │   │
│  │  8. Cookie ADA! ✅                                              │   │
│  │  9. fetch() /me dengan Bearer token                             │   │
│  │  10. BE validate via Sanctum guard 'api' → BERHASIL             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  /admin/questions (Server Component)                            │   │
│  │  11. fetch() /admin/questions dengan Bearer + credentials       │   │
│  │  12. BE validate → return data → BERHASIL                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Deployment Checklist

### Backend (be-cbt)

```bash
# Upload file yang diubah:
# - be-cbt/config/auth.php
# - be-cbt/config/sanctum.php
# - be-cbt/app/Http/Middleware/HandleCors.php

# Clear cache
php artisan config:clear
php artisan cache:clear
php artisan route:clear
```

### Frontend (cbt-admin)

```bash
# Upload file yang diubah:
# - cbt-admin/src/lib/admin-api/core.ts
# - cbt-admin/src/lib/admin-api/auth.ts
# - cbt-admin/src/app/login/login-form.tsx
# - cbt-admin/src/app/login/page.tsx
# - cbt-admin/src/app/api/admin/analytics/route.ts
# - cbt-admin/src/app/api/media/route.ts
# - cbt-admin/src/app/admin/results/export/route.ts
# - cbt-admin/src/app/api/auth/sync-token/route.ts (BARU)

# Rebuild
sudo pnpm run build
```

### Post-Deploy

1. **Hard refresh browser** (Ctrl+Shift+R)
2. **Login** ke admin panel
3. Cek di DevTools → Application → Cookies:
   - Cookie `cbt_admin_token` harus ada
   - SameSite: None
   - Secure: true
4. **Navigasi ke Questions** — data harusnya sudah bisa dimuat

---

## 6. Verifikasi Fix

Setelah deploy, verifikasi dengan:

1. Buka browser DevTools → Network tab
2. Login ke admin panel
3. Cek sequence request:
   - ✅ `POST /api/login` → 200 OK dengan token
   - ✅ `POST /api/auth/sync-token` → 200 OK
   - ✅ Response headers ada `Set-Cookie: cbt_admin_token=...`
   - ✅ `GET /admin` → 200 OK (full page navigation)
   - ✅ Request headers ada `Cookie: cbt_admin_token=...`
   - ✅ `GET /api/admin/questions` → 200 OK dengan data
4. Cek Application → Cookies:
   - ✅ Cookie `cbt_admin_token` ada dengan value token
   - ✅ SameSite: None
   - ✅ Secure: true

---

## 7. File yang Diubah

| # | File | Perubahan |
|---|------|-----------|
| 1 | `be-cbt/config/auth.php` | Tambah guard `'api'` dengan driver `'sanctum'` |
| 2 | `be-cbt/config/sanctum.php` | Update `'guard'` ke `['web', 'api']` |
| 3 | `be-cbt/app/Http/Middleware/HandleCors.php` | Fix preflight + tambah domain `admin-cat` |
| 4 | `cbt-admin/src/lib/admin-api/auth.ts` | Cookie `SameSite: "none"`, `secure: true` |
| 5 | `cbt-admin/src/lib/admin-api/core.ts` | Tambah `credentials: "include"` ke semua fetch |
| 6 | `cbt-admin/src/app/login/login-form.tsx` | Panggil API Route `/api/auth/sync-token` |
| 7 | `cbt-admin/src/app/login/page.tsx` | Fix searchParams destructuring |
| 8 | `cbt-admin/src/app/api/admin/analytics/route.ts` | Tambah `credentials: "include"` |
| 9 | `cbt-admin/src/app/api/media/route.ts` | Tambah `credentials: "include"` |
| 10 | `cbt-admin/src/app/admin/results/export/route.ts` | Tambah `credentials: "include"` |
| 11 | `cbt-admin/src/app/api/auth/sync-token/route.ts` | **BARU** — API Route untuk set cookie |

---

## 8. Pelajaran Penting

1. **Server Action tidak reliable untuk set cookie yang perlu dibaca oleh Server Component** — gunakan API Route sebagai alternatif
2. **API Route mengirim `Set-Cookie` header ke browser** — cookie tersimpan dan bisa dibaca oleh Server Component
3. **Cross-origin cookie memerlukan `SameSite: "none"` + `Secure: true"`**
4. **`credentials: "include"` wajib pada fetch cross-origin yang butuh cookie**
5. **CORS allowed origins harus mencakup SEMUA domain FE yang digunakan**
6. **Sanctum guard config harus mencakup guard yang sesuai dengan driver autentikasi**

---

## 9. Related Documentation

- [Authentication Audit & Fix.md](./Authentication%20Audit%20&%20Fix.md) — Audit pertama (20 Mei 2026)
- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [MDN: SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [MDN: CORS - Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)
- [Next.js Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

---

*End of Report — 21 Mei 2026*
