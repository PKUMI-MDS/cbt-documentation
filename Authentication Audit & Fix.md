# Authentication Audit & Fix: Unauthenticated Error on Questions Page

**Date:** 2026-05-20
**Reporter:** Kimi Code CLI
**Affected Systems:** `cbt-admin` (Next.js Frontend), `be-cbt` (Laravel Backend)
**Severity:** High — Admin cannot access Questions module after login

---

## 1. Problem Statement

After a successful login, admin users see **"Data belum bisa dimuat. Unauthenticated."** when navigating to the **Questions** page (`/admin/questions`). The issue persists even after refreshing the page or logging in again. Other admin pages (Dashboard, Users, etc.) may work, but the Questions module consistently fails with an authentication error.

### Screenshot

![Unauthenticated Error](https://i.imgur.com/placeholder.png)
*Error banner showing "Data belum bisa dimuat. Unauthenticated." on the Questions page while the user is logged in (Admin badge visible in top-right).*

---

## 2. Root Cause Analysis

The frontend (`cbt-admin`) and backend (`be-cbt`) are hosted on **different subdomains** (cross-origin). Several misconfigurations in both the cookie policy and CORS handling prevented the authentication token from being transmitted correctly.

### 2.1 Issue Matrix

| # | Issue | File | Line | Impact |
|---|-------|------|------|--------|
| 1 | Cookie `SameSite: "lax"` | `cbt-admin/src/lib/admin-api/auth.ts` | 49 | Cookie is **not sent** to a different subdomain |
| 2 | Cookie `secure` depends on `NODE_ENV` | `cbt-admin/src/lib/admin-api/auth.ts` | 50 | Cookie may be rejected on HTTPS cross-origin |
| 3 | Missing `credentials: "include"` | `cbt-admin/src/lib/admin-api/core.ts` | ~310 | `fetch()` does not send cookies at all |
| 4 | No CORS middleware on BE | `be-cbt/bootstrap/app.php` | — | BE rejects cross-origin requests |
| 5 | Sanctum `guard` only `['web']` | `be-cbt/config/sanctum.php` | 37 | Sanctum cannot authenticate via Bearer token fallback |

### 2.2 Authentication Flow Diagram (Before Fix)

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   Admin Login   │         │  cbt-admin (FE) │         │  be-cbt (BE)    │
└────────┬────────┘         └────────┬────────┘         └────────┬────────┘
         │                           │                           │
         │  1. POST /login           │                           │
         │──────────────────────────>│                           │
         │                           │  2. Receives token        │
         │                           │  3. Saves to localStorage │
         │                           │  4. Saves to httpOnly     │
         │                           │     cookie (SameSite=lax) │
         │                           │                           │
         │  5. Navigate to /admin/questions                     │
         │                           │                           │
         │                           │  6. Server Component      │
         │                           │     calls getAdminToken() │
         │                           │     → reads cookie ✓      │
         │                           │                           │
         │                           │  7. fetch(/admin/questions)│
         │                           │     with Bearer token     │
         │                           │     BUT no credentials    │
         │                           │     AND cookie blocked    │
         │                           │     by SameSite=lax       │
         │                           │                           │
         │                           │──────────────────────────>│
         │                           │                           │ 8. Token invalid
         │                           │                           │    or missing
         │                           │<──────────────────────────│
         │                           │  9. 401 Unauthenticated   │
         │                           │                           │
```

### 2.3 Why the Token Was Not Reaching the Backend

1. **Cookie Scope**: The token cookie was set with `SameSite: "lax"` and `secure: process.env.NODE_ENV === "production"`. When FE and BE are on different subdomains, `lax` prevents the cookie from being included in cross-origin requests.

2. **Fetch Credentials**: The `fetch()` calls in `core.ts` did not specify `credentials: "include"`, so even if the cookie policy were correct, the browser would not send cookies.

3. **CORS Missing**: The Laravel backend had **no CORS middleware** registered. Without `Access-Control-Allow-Credentials: true`, the browser blocks any cross-origin request that includes cookies or authorization headers.

4. **Sanctum Guard Configuration**: `config/sanctum.php` had `'guard' => ['web']`. While Sanctum falls back to Bearer token authentication when session auth fails, the lack of proper CORS meant the Bearer token request was also blocked or mishandled.

---

## 3. Files Audited

### Frontend (`cbt-admin`)

| File | Purpose |
|------|---------|
| `src/lib/admin-api/core.ts` | API client: `request()`, `mutation()`, `multipartMutation()`, `getAdminToken()`, `validateAdminSession()` |
| `src/lib/admin-api/auth.ts` | Login logic, cookie management (`setAdminToken`, `clearAdminToken`) |
| `src/lib/client-auth.ts` | Client-side localStorage token helpers |
| `src/app/login/login-form.tsx` | Login form component |
| `src/app/login/actions.ts` | Server action for login |
| `src/app/admin/layout.tsx` | Admin layout with session validation |
| `src/app/admin/[section]/page.tsx` | Generic admin section page |
| `src/app/admin/questions/page.tsx` | Questions page wrapper |
| `src/components/admin-shell.tsx` | Admin shell with navigation |
| `src/components/module-page.tsx` | Module page layout with action buttons |
| `src/lib/admin-api/module-pages.ts` | Data fetching for module pages |
| `src/lib/admin-api/questions.ts` | Questions-specific API calls |

### Backend (`be-cbt`)

| File | Purpose |
|------|---------|
| `routes/api.php` | API route definitions including `/admin/questions` |
| `app/Http/Controllers/Api/AuthController.php` | Login, logout, me endpoints |
| `app/Http/Middleware/EnsureUserIsAdmin.php` | Admin authorization middleware |
| `app/Http/Middleware/EnsureAccountIsActive.php` | Active account middleware |
| `config/sanctum.php` | Sanctum configuration |
| `config/auth.php` | Auth guard configuration |
| `bootstrap/app.php` | Application bootstrap, middleware registration |
| `app/Models/User.php` | User model with `HasApiTokens` trait |

---

## 4. Implementation Fix

### 4.1 Backend — Add CORS Middleware

**New file:** `be-cbt/app/Http/Middleware/HandleCors.php`

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class HandleCors
{
    public function handle(Request $request, Closure $next): Response
    {
        $allowedOrigins = [
            'http://localhost:3000',
            'http://localhost',
            'https://cbt-admin.pkumionline.cloud',
            'https://admin.pkumionline.cloud',
        ];

        $origin = $request->header('Origin');
        $allowOrigin = in_array($origin, $allowedOrigins, true) ? $origin : '';

        // Handle preflight OPTIONS request
        if ($request->isMethod('OPTIONS')) {
            $response = response('', 204);
        } else {
            $response = $next($request);
        }

        $response->headers->set('Access-Control-Allow-Origin', $allowOrigin);
        $response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
        $response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With, Accept, X-XSRF-TOKEN');
        $response->headers->set('Access-Control-Allow-Credentials', 'true');
        $response->headers->set('Access-Control-Max-Age', '86400');

        return $response;
    }
}
```

### 4.2 Backend — Register CORS Middleware for API Routes

**Modified file:** `be-cbt/bootstrap/app.php`

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'active' => \App\Http\Middleware\EnsureAccountIsActive::class,
        'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
    ]);

    // Global CORS middleware for API routes
    $middleware->api(prepend: [
        \App\Http\Middleware\HandleCors::class,
    ]);
})
```

### 4.3 Frontend — Fix Cookie Policy for Cross-Origin

**Modified file:** `cbt-admin/src/lib/admin-api/auth.ts`

**Before:**
```typescript
export async function setAdminToken(token: string) {
  const cookieStore = await cookies();
  cookieStore.set(TOKEN_COOKIE, token, {
    httpOnly: true,
    sameSite: "lax",
    secure: process.env.NODE_ENV === "production",
    path: "/",
    maxAge: 7 * 24 * 60 * 60,
  });
}
```

**After:**
```typescript
export async function setAdminToken(token: string) {
  const cookieStore = await cookies();
  cookieStore.set(TOKEN_COOKIE, token, {
    httpOnly: true,
    sameSite: "none",
    secure: true,
    path: "/",
    maxAge: 7 * 24 * 60 * 60,
  });
}
```

### 4.4 Frontend — Add `credentials: "include"` to All Fetch Calls

**Modified file:** `cbt-admin/src/lib/admin-api/core.ts`

#### Change 1 — `request()` function (GET requests)

**Before:**
```typescript
const response = await fetch(url, {
  headers,
  cache: "no-store",
});
```

**After:**
```typescript
const response = await fetch(url, {
  headers,
  cache: "no-store",
  credentials: "include",
});
```

#### Change 2 — `mutation()` function (POST/PATCH/DELETE JSON)

**Before:**
```typescript
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: init.method,
  headers,
  body: init.body ? JSON.stringify(init.body) : undefined,
  cache: "no-store",
});
```

**After:**
```typescript
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: init.method,
  headers,
  body: init.body ? JSON.stringify(init.body) : undefined,
  cache: "no-store",
  credentials: "include",
});
```

#### Change 3 — `multipartMutation()` function (POST with FormData)

**Before:**
```typescript
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: "POST",
  headers,
  body: formData,
  cache: "no-store",
});
```

**After:**
```typescript
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: "POST",
  headers,
  body: formData,
  cache: "no-store",
  credentials: "include",
});
```

### 4.5 Backend — Sanctum Configuration Cleanup

**Modified file:** `be-cbt/config/sanctum.php`

Removed an unnecessary commented line for cleanliness:

**Before:**
```php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort(),
    // Sanctum::currentRequestHost(),
))),
```

**After:**
```php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort(),
))),
```

---

## 5. Authentication Flow Diagram (After Fix)

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   Admin Login   │         │  cbt-admin (FE) │         │  be-cbt (BE)    │
└────────┬────────┘         └────────┬────────┘         └────────┬────────┘
         │                           │                           │
         │  1. POST /login           │                           │
         │──────────────────────────>│                           │
         │                           │  2. Receives token        │
         │                           │  3. Saves to localStorage │
         │                           │  4. Saves to httpOnly     │
         │                           │     cookie (SameSite=none │
         │                           │     secure=true)          │
         │                           │                           │
         │  5. Navigate to /admin/questions                     │
         │                           │                           │
         │                           │  6. Server Component      │
         │                           │     calls getAdminToken() │
         │                           │     → reads cookie ✓      │
         │                           │                           │
         │                           │  7. fetch(/admin/questions)│
         │                           │     with Bearer token     │
         │                           │     AND credentials:include│
         │                           │     AND cookie included   │
         │                           │                           │
         │                           │──────────────────────────>│
         │                           │                           │ 8. CORS preflight
         │                           │                           │    → Allowed ✓
         │                           │                           │ 9. Validate Bearer
         │                           │                           │    token ✓
         │                           │<──────────────────────────│
         │                           │  10. 200 OK with data     │
         │                           │                           │
```

---

## 6. Deployment Checklist

### Backend (`be-cbt`)

```bash
# Upload changed files:
# - app/Http/Middleware/HandleCors.php (new)
# - bootstrap/app.php (modified)
# - config/sanctum.php (modified)

# Clear all caches
php artisan config:clear
php artisan cache:clear
php artisan route:clear

# Verify .env settings
APP_URL=https://be-cbt.pkumionline.cloud
SANCTUM_STATEFUL_DOMAINS=cbt-admin.pkumionline.cloud,localhost,localhost:3000
```

### Frontend (`cbt-admin`)

```bash
# Upload changed files:
# - src/lib/admin-api/core.ts (modified)
# - src/lib/admin-api/auth.ts (modified)

# Rebuild
sudo pnpm run build

# Restart service (PM2, systemd, etc.)
```

### Post-Deployment User Actions

1. **Logout** from admin panel
2. **Login again** (to receive the new cookie with `SameSite: none`)
3. **Navigate to Questions** page — data should load successfully

---

## 7. Security Considerations

| Change | Risk | Mitigation |
|--------|------|------------|
| `SameSite: "none"` | Cookie sent on all cross-origin requests | `secure: true` ensures HTTPS only; token is still `httpOnly` |
| `credentials: "include"` | Cookies sent to all API calls | Backend CORS restricts allowed origins explicitly |
| CORS allows multiple origins | Potential misconfiguration | Origin list is hardcoded in middleware, not wildcard (`*`) |

---

## 8. Related Documentation

- [Laravel Sanctum Documentation](https://laravel.com/docs/sanctum)
- [MDN: SameSite Cookie Attribute](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [MDN: Fetch API — credentials](https://developer.mozilla.org/en-US/docs/Web/API/fetch#credentials)
- [OWASP CORS Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/CORS_Security_Cheat_Sheet.html)

---

## 9. Appendix: Full File Contents (Post-Fix)

### `be-cbt/app/Http/Middleware/HandleCors.php`

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class HandleCors
{
    public function handle(Request $request, Closure $next): Response
    {
        $allowedOrigins = [
            'http://localhost:3000',
            'http://localhost',
            'https://cbt-admin.pkumionline.cloud',
            'https://admin.pkumionline.cloud',
        ];

        $origin = $request->header('Origin');
        $allowOrigin = in_array($origin, $allowedOrigins, true) ? $origin : '';

        if ($request->isMethod('OPTIONS')) {
            $response = response('', 204);
        } else {
            $response = $next($request);
        }

        $response->headers->set('Access-Control-Allow-Origin', $allowOrigin);
        $response->headers->set('Access-Control-Allow-Methods', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
        $response->headers->set('Access-Control-Allow-Headers', 'Content-Type, Authorization, X-Requested-With, Accept, X-XSRF-TOKEN');
        $response->headers->set('Access-Control-Allow-Credentials', 'true');
        $response->headers->set('Access-Control-Max-Age', '86400');

        return $response;
    }
}
```

### `be-cbt/bootstrap/app.php`

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->alias([
            'active' => \App\Http\Middleware\EnsureAccountIsActive::class,
            'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
        ]);

        // Global CORS middleware for API routes
        $middleware->api(prepend: [
            \App\Http\Middleware\HandleCors::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions): void {
        //
    })->create();
```

### `cbt-admin/src/lib/admin-api/auth.ts`

```typescript
import "server-only";

import { cookies } from "next/headers";
import { apiBaseUrl } from "./core";
import type { ApiEnvelope } from "./types";

const TOKEN_COOKIE = "cbt_admin_token";

export async function loginAdmin(email: string, password: string) {
  const response = await fetch(`${apiBaseUrl()}/login`, {
    method: "POST",
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ email, password }),
    cache: "no-store",
  });

  const body = (await response.json().catch(() => null)) as ApiEnvelope<{
    token?: string;
    user?: { role?: string; name?: string; email?: string };
  }> | null;

  if (!response.ok || body?.status === "error") {
    throw new Error(body?.message ?? "Login gagal.");
  }

  const data = body?.data;

  if (data?.user?.role !== "admin") {
    throw new Error("Akun ini bukan admin.");
  }

  if (!data.token) {
    throw new Error("Token admin tidak ditemukan di response login.");
  }

  return {
    token: data.token,
    user: data.user,
  };
}

export async function setAdminToken(token: string) {
  const cookieStore = await cookies();
  cookieStore.set(TOKEN_COOKIE, token, {
    httpOnly: true,
    sameSite: "none",
    secure: true,
    path: "/",
    maxAge: 7 * 24 * 60 * 60, // 7 hari
  });
}

export async function clearAdminToken() {
  const cookieStore = await cookies();
  cookieStore.delete(TOKEN_COOKIE);
}
```

### `cbt-admin/src/lib/admin-api/core.ts` (relevant fetch sections)

```typescript
// request() — GET with query params
const response = await fetch(url, {
  headers,
  cache: "no-store",
  credentials: "include",
});

// mutation() — POST/PATCH/DELETE JSON
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: init.method,
  headers,
  body: init.body ? JSON.stringify(init.body) : undefined,
  cache: "no-store",
  credentials: "include",
});

// multipartMutation() — POST with FormData
const response = await fetch(`${apiBaseUrl()}${path}`, {
  method: "POST",
  headers,
  body: formData,
  cache: "no-store",
  credentials: "include",
});
```

---

*End of Document*
