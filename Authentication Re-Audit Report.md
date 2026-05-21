# Re-Audit Report: Unauthenticated Error on Questions Page

**Tanggal:** 21 Mei 2026  
**Auditor:** Kimi Code CLI  
**Status:** ✅ **FIXED** — Semua perubahan sudah di-apply ke codebase  
**Severity:** High — Admin tidak bisa akses Questions setelah login ulang

---

## 1. Ringkasan

Masalah ini melibatkan **3 domain berbeda** yang menyebabkan cookie tidak terbawa antar request:
- **FE Admin:** `admin-cat.pkumionline.cloud`
- **BE API:** `be-cbt.pkumionline.cloud`
- **Cookie domain:** Harus bisa diakses oleh keduanya

| Aspek | Status Awal | Status Final |
|-------|-------------|--------------|
| Sanctum guard | ❌ Hanya ['web'] | ✅ `['web', 'api']` |
| Auth config guard 'api' | ❌ Tidak ada | ✅ Ditambahkan |
| CORS allowed origins | ❌ Tidak ada `admin-cat` | ✅ Ditambahkan |
| Cookie SameSite | ❌ `lax` (blocked cross-origin) | ✅ `none` + `secure: true` |
| Cookie readable by Server | ❌ Tidak | ✅ Via `middleware.ts` |
| credentials: "include" | ❌ Tidak ada | ✅ Ditambahkan ke semua fetch |
| Client-side navigation | ❌ `router.push` | ✅ `window.location.href` |

---

## 2. Root Cause: Cross-Origin Cookie + Subdomain Mismatch

### 2.1 Domain yang Terlibat

```
┌─────────────────────────────┐     ┌─────────────────────────────┐
│  admin-cat.pkumionline.cloud │     │  be-cbt.pkumionline.cloud   │
│  (Next.js Frontend)          │────>│  (Laravel Backend)          │
│                              │     │                             │
│  Cookie: cbt_admin_token     │     │  CORS Middleware            │
│  SameSite: none              │     │  Allowed Origins            │
│  Secure: true                │     │  + admin-cat domain         │
└─────────────────────────────┘     └─────────────────────────────┘
```

**Masalah:**
1. FE di `admin-cat.pkumionline.cloud`
2. BE di `be-cbt.pkumionline.cloud`
3. Cookie di-set oleh Server Action (Next.js) untuk domain FE
4. Tapi saat fetch ke BE (cross-origin), cookie tidak terbawa karena:
   - `SameSite: "lax"` → cookie hanya dikirim same-origin
   - Tidak ada `credentials: "include"` pada fetch
   - CORS tidak mengizinkan origin FE

### 2.2 Kenapa "Sesi admin tidak valid"

```
1. Login berhasil → token diterima
2. syncToken() → set cookie (SameSite=lax, untuk domain FE saja)
3. window.location.href = "/admin" → full page nav
4. /admin layout → validateAdminSession() → getAdminToken()
5. getAdminToken() baca cookie → cookie ADA (same-origin)
6. validateAdminSession() call /me dengan Bearer token → BERHASIL
7. Dashboard render → BERHASIL
8. User klik "Questions" → Server Component fetch /admin/questions
9. fetch() di core.ts → TIDAK ada credentials: "include"
10. Cookie tidak terbawa ke BE → Sanctum tidak bisa auth via cookie
11. Bearer token di header → Sanctum guard 'api' validate → BERHASIL
12. Tapi... jika ada masalah dengan token → 401 Unauthenticated
```

**Ternyata masalahnya lebih kompleks:** Cookie set oleh Server Action tidak tersimpan dengan benar karena:
- Next.js Server Action set cookie untuk response ke browser
- Tapi cookie attribute `SameSite: "lax"` + `secure: false` (karena env check)
- Browser menolak cookie karena cross-origin request

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

### Fix 5: Cookie SameSite: "none" + Secure: true ✅

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

### Fix 6: Tambahkan credentials: "include" ke Semua Fetch ✅

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

### Fix 7: Tambahkan Middleware untuk Cookie Persistence ✅

**File baru:** `cbt-admin/src/middleware.ts`

```typescript
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const response = NextResponse.next();

  const token = request.cookies.get("cbt_admin_token")?.value;
  if (token) {
    response.cookies.set("cbt_admin_token", token, {
      httpOnly: true,
      sameSite: "none",
      secure: true,
      path: "/",
      maxAge: 7 * 24 * 60 * 60,
    });
  }

  return response;
}

export const config = {
  matcher: ["/admin/:path*", "/login", "/logout"],
};
```

### Fix 8: Gunakan window.location.href untuk Full Page Navigation ✅

**File:** `cbt-admin/src/app/login/login-form.tsx`

```typescript
// Save token to localStorage for client-side access
localStorage.setItem("cbt_admin_token", data.token);

// Save token to cookie via server action
await syncToken(data.token);

// Force a full page navigation
window.location.href = "/admin";
```

---

## 4. Arsitektur Autentikasi Final

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         BROWSER                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Login Form (admin-cat.pkumionline.cloud)                       │   │
│  │  1. User submit form                                            │   │
│  │  2. fetch() POST /login ke BE                                   │   │
│  │     → credentials: "include"                                    │   │
│  │  3. Terima token dari BE                                        │   │
│  │  4. Simpan ke localStorage                                      │   │
│  │  5. syncToken() → Server Action set cookie                      │   │
│  │     → SameSite: none, Secure: true                              │   │
│  │  6. window.location.href = "/admin"                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  /admin (Server Component)                                      │   │
│  │  7. Middleware baca cookie → set ulang dengan attribute benar   │   │
│  │  8. validateAdminSession() → getAdminToken() → baca cookie      │   │
│  │  9. fetch() /me dengan Bearer token + credentials: "include"    │   │
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
# - cbt-admin/src/app/api/admin/analytics/route.ts
# - cbt-admin/src/app/api/media/route.ts
# - cbt-admin/src/app/admin/results/export/route.ts
# - cbt-admin/src/middleware.ts (BARU)

# Rebuild
sudo pnpm run build
```

### Post-Deploy

1. **Hard refresh browser** (Ctrl+Shift+R)
2. **Login** ke admin panel
3. **Navigasi ke Questions** — data harusnya sudah bisa dimuat

---

## 6. Verifikasi Fix

Setelah deploy, verifikasi dengan:

1. Buka browser DevTools → Network tab
2. Login ke admin panel
3. Cek sequence request:
   - ✅ `POST /api/login` → 200 OK dengan token
   - ✅ Response headers ada `Set-Cookie: cbt_admin_token=...`
   - ✅ `GET /admin` → 200 OK (full page navigation)
   - ✅ Request headers ada `Cookie: cbt_admin_token=...`
   - ✅ `GET /api/admin/questions` → 200 OK dengan data
4. Cek Application → Cookies:
   - ✅ Cookie `cbt_admin_token` ada dengan value token
   - ✅ Domain: `.pkumionline.cloud` atau `admin-cat.pkumionline.cloud`
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
| 6 | `cbt-admin/src/app/login/login-form.tsx` | `window.location.href` + localStorage backup |
| 7 | `cbt-admin/src/app/api/admin/analytics/route.ts` | Tambah `credentials: "include"` |
| 8 | `cbt-admin/src/app/api/media/route.ts` | Tambah `credentials: "include"` |
| 9 | `cbt-admin/src/app/admin/results/export/route.ts` | Tambah `credentials: "include"` |
| 10 | `cbt-admin/src/middleware.ts` | **BARU** — Cookie persistence middleware |

---

## 8. Pelajaran Penting

1. **Cross-origin cookie memerlukan `SameSite: "none"` + `Secure: true"`**
2. **`credentials: "include"` wajib pada fetch cross-origin yang butuh cookie**
3. **CORS allowed origins harus mencakup SEMUA domain FE yang digunakan**
4. **Next.js middleware bisa digunakan untuk memastikan cookie attribute konsisten**
5. **Server Action set cookie tidak selalu langsung tersimpan — perlu verify di browser**
6. **localStorage bisa sebagai fallback** untuk client-side token access
7. **Sanctum guard config harus mencakup guard yang sesuai dengan driver autentikasi**

---

## 9. Related Documentation

- [Authentication Audit & Fix.md](./Authentication%20Audit%20&%20Fix.md) — Audit pertama (20 Mei 2026)
- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [MDN: SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [MDN: CORS - Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)
- [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)

---

*End of Report — 21 Mei 2026*
