# Modul 2 — Pengenalan Laravel Sanctum

> **Versi:** Laravel 12.x · Sanctum 4.x · PHP 8.3+ · **Durasi:** 2–3 jam

---

## Daftar Isi

1. [Apa itu Laravel Sanctum?](#1-apa-itu-laravel-sanctum)
2. [Sanctum vs Passport vs JWT](#2-sanctum-vs-passport-vs-jwt)
3. [Dua Mode Sanctum](#3-dua-mode-sanctum)
4. [Instalasi & Konfigurasi Sanctum](#4-instalasi--konfigurasi-sanctum)
5. [Tabel `personal_access_tokens`](#5-tabel-personal_access_tokens)
6. [Middleware `auth:sanctum`](#6-middleware-authsanctum)
7. [Alur Kerja Token Authentication](#7-alur-kerja-token-authentication)
8. [Latihan](#8-latihan)
9. [Proyek Mini](#9-proyek-mini)

---

## 1. Apa itu Laravel Sanctum?

Laravel Sanctum adalah package autentikasi ringan yang disertakan secara default dalam Laravel 12. Sanctum dirancang untuk dua skenario utama:

1. **API Token Authentication** — untuk aplikasi mobile, CLI tools, atau third-party consumer
2. **SPA Authentication** — untuk Single Page Application (Vue, React, dll.) yang hosted di domain yang sama

### Mengapa Sanctum?

- **Ringan dan sederhana** — tidak memerlukan konfigurasi OAuth yang kompleks
- **Built-in di Laravel** — tersedia langsung, tidak perlu instalasi tambahan di Laravel 12
- **Token granular** — setiap token bisa diberi kemampuan (abilities/scopes) yang berbeda
- **Multi-device** — satu user bisa punya banyak token aktif (untuk banyak device)
- **Mudah di-revoke** — token bisa dicabut kapan saja

---

## 2. Sanctum vs Passport vs JWT

| Fitur                    | Sanctum              | Passport             | JWT (tymon)          |
|--------------------------|----------------------|----------------------|----------------------|
| **Kompleksitas**         | Rendah ✅            | Tinggi ❌            | Sedang               |
| **OAuth 2.0**            | Tidak                | Ya ✅                | Tidak                |
| **SPA Support**          | Ya ✅                | Terbatas             | Tidak                |
| **Mobile API**           | Ya ✅                | Ya                   | Ya ✅                |
| **Token Storage**        | Database             | Database             | Stateless (JWT)      |
| **Token Revocation**     | Mudah ✅             | Mudah                | Sulit ❌             |
| **Built-in Laravel 12**  | Ya ✅                | Tidak (install manual)| Tidak (third-party) |
| **Cocok untuk**          | API + SPA sederhana  | OAuth server         | Microservices        |

### Kapan Memilih Sanctum?

✅ Gunakan Sanctum jika:
- Membangun API untuk aplikasi mobile atau SPA sendiri
- Tidak membutuhkan fitur OAuth lengkap (seperti login via Google/GitHub)
- Ingin implementasi cepat dan mudah dipahami

❌ Pertimbangkan Passport jika:
- Aplikasi Anda menjadi **OAuth provider** (misalnya membuat sistem "Login dengan App Anda")
- Membutuhkan `authorization_code` grant, `refresh_token` yang kompleks

---

## 3. Dua Mode Sanctum

### Mode 1: API Token Authentication

Cocok untuk aplikasi **mobile**, **desktop**, atau **third-party** yang mengakses API Anda.

**Cara kerja:**
1. User login → server menerbitkan token berupa string panjang
2. Client menyimpan token tersebut (di local storage atau secure storage)
3. Setiap request berikutnya menyertakan token di header `Authorization`
4. Server memvalidasi token di database

```
Client                          Server
  │                               │
  │  POST /api/login               │
  │  { email, password }  ──────► │
  │                               │  Validasi kredensial
  │  { token: "abc123..." } ◄──── │  Buat token di DB
  │                               │
  │  GET /api/profile              │
  │  Authorization: Bearer abc123 ─►│
  │                               │  Cari token di DB → OK
  │  { data: { user } }   ◄────── │
```

### Mode 2: SPA Authentication (Cookie-based)

Cocok untuk **Single Page Application** (Vue, React, Angular) yang di-serve dari domain yang sama atau subdomain.

**Cara kerja:**
- Menggunakan cookie session, bukan token yang disimpan di client
- Lebih aman karena cookie httpOnly tidak bisa diakses JavaScript
- Memerlukan konfigurasi CORS dan CSRF

> **Di modul ini kita fokus pada Mode 1 (API Token)** karena lebih umum digunakan dan lebih mudah dipahami untuk pemula.

---

## 4. Instalasi & Konfigurasi Sanctum

### Pengecekan — Sanctum Sudah Ada di Laravel 12

Sanctum sudah termasuk dalam Laravel 12 secara default. Cek di `composer.json`:

```json
"require": {
    "laravel/sanctum": "^4.0",
    ...
}
```

Jika belum ada (misal project lama):

```bash
composer require laravel/sanctum
```

### Publish Konfigurasi & Migration

```bash
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

Perintah ini akan membuat:
- `config/sanctum.php` — file konfigurasi
- Migration untuk tabel `personal_access_tokens`

### Jalankan Migration

```bash
php artisan migrate
```

### Konfigurasi `config/sanctum.php`

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Stateful Domains
    |--------------------------------------------------------------------------
    | Domain-domain ini akan menggunakan cookie-based auth (SPA mode).
    | Untuk API token mode, bagian ini tidak terlalu relevan.
    */
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
        '%s%s',
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
        Sanctum::currentApplicationUrlWithPort()
    ))),

    /*
    |--------------------------------------------------------------------------
    | Sanctum Guards
    |--------------------------------------------------------------------------
    */
    'guard' => ['web'],

    /*
    |--------------------------------------------------------------------------
    | Expiration Time (menit)
    |--------------------------------------------------------------------------
    | null = token tidak pernah expired
    | 60   = token expired setelah 60 menit
    | 60 * 24 * 30 = 30 hari
    */
    'expiration' => null,  // Ubah sesuai kebutuhan

    /*
    |--------------------------------------------------------------------------
    | Token Prefix
    |--------------------------------------------------------------------------
    */
    'token_prefix' => env('SANCTUM_TOKEN_PREFIX', ''),

    /*
    |--------------------------------------------------------------------------
    | Sanctum Middleware
    |--------------------------------------------------------------------------
    */
    'middleware' => [
        'authenticate_session' => Laravel\Sanctum\Http\Middleware\AuthenticateSession::class,
        'encrypt_cookies'      => Illuminate\Cookie\Middleware\EncryptCookies::class,
        'validate_csrf_token'  => Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
    ],

];
```

### Tambahkan Trait `HasApiTokens` ke Model User

```php
<?php
// app/Models/User.php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens; // ← Import ini

class User extends Authenticatable
{
    use HasApiTokens, Notifiable; // ← Tambahkan HasApiTokens

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password'          => 'hashed', // Laravel 12: hash otomatis
    ];
}
```

> **Penting:** Trait `HasApiTokens` memberikan method `createToken()`, `tokens()`, dan `currentAccessToken()` pada model User.

---

## 5. Tabel `personal_access_tokens`

### Struktur Tabel

Setelah `php artisan migrate`, tabel `personal_access_tokens` akan dibuat dengan struktur:

```sql
CREATE TABLE personal_access_tokens (
  id             BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  tokenable_type VARCHAR(255) NOT NULL,   -- Nama model (App\Models\User)
  tokenable_id   BIGINT UNSIGNED NOT NULL, -- ID user
  name           VARCHAR(255) NOT NULL,   -- Nama token (misal: "MyApp Mobile")
  token          VARCHAR(64) NOT NULL UNIQUE, -- Hash SHA-256 dari token
  abilities      TEXT NULL,              -- JSON array kemampuan token
  last_used_at   TIMESTAMP NULL,         -- Terakhir digunakan
  expires_at     TIMESTAMP NULL,         -- Waktu expired
  created_at     TIMESTAMP NULL,
  updated_at     TIMESTAMP NULL
);
```

### Penjelasan Kolom Penting

| Kolom            | Penjelasan                                                       |
|------------------|------------------------------------------------------------------|
| `tokenable_type` | Polymorphic: nama class model pemilik token                      |
| `tokenable_id`   | ID pemilik token                                                 |
| `name`           | Label untuk token (misal: "iPhone 14", "Dashboard")              |
| `token`          | **Hash SHA-256** dari token — bukan token aslinya!               |
| `abilities`      | Array JSON berisi kemampuan, misal: `["read", "write"]`          |
| `last_used_at`   | Diupdate otomatis setiap token digunakan                         |
| `expires_at`     | Null = tidak expired                                             |

### Token Asli vs Hash

Ini poin penting yang sering membingungkan:

```
Token asli   : 1|AbCdEfGhIjKlMnOpQrStUvWxYz1234567890abcdef
               ↑                  ↑
            ID token         Token string (random)

Yang disimpan di DB: SHA256("AbCdEfGhIjKlMnOpQrStUvWxYz1234567890abcdef")
```

- **Token asli** hanya dikirim **sekali** saat pembuatan — setelah itu tidak bisa dilihat lagi
- **Di database**, hanya tersimpan **hash-nya** — mirip password hashing
- Jika user kehilangan token, mereka harus login ulang untuk mendapat token baru

### Melihat Token via Tinker

```bash
php artisan tinker

# Lihat semua token
\Laravel\Sanctum\PersonalAccessToken::all();

# Lihat token milik user ID 1
\App\Models\User::find(1)->tokens;

# Hapus semua token user ID 1
\App\Models\User::find(1)->tokens()->delete();
```

---

## 6. Middleware `auth:sanctum`

### Cara Kerja Middleware

Ketika request masuk dengan header `Authorization: Bearer <token>`, middleware `auth:sanctum` akan:

1. Mengambil token dari header
2. Mencari hash token di tabel `personal_access_tokens`
3. Memastikan token belum expired
4. Memuat model user terkait (`tokenable`)
5. Set user sebagai `authenticated` untuk request tersebut

### Menggunakan di Route

```php
// routes/api.php

use App\Http\Controllers\Api\ProductController;
use App\Http\Controllers\Api\ProfileController;

// Route publik — tidak perlu autentikasi
Route::post('/login', [AuthController::class, 'login']);
Route::post('/register', [AuthController::class, 'register']);

// Route yang membutuhkan autentikasi
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/profile', [ProfileController::class, 'show']);
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::apiResource('products', ProductController::class);
});
```

### Mendapatkan User yang Terotentikasi

```php
// Di dalam controller yang dilindungi middleware auth:sanctum
public function profile(): JsonResponse
{
    $user = auth()->user();
    // atau
    $user = request()->user();
    // atau
    $user = Auth::user();

    return response()->json([
        'success' => true,
        'data'    => $user,
    ]);
}
```

### Menggunakan di Controller (Tanpa Route Middleware)

```php
public function sensitiveAction(): JsonResponse
{
    // Cek manual apakah user terotentikasi
    if (!auth('sanctum')->check()) {
        return response()->json(['message' => 'Unauthorized'], 401);
    }

    $user = auth('sanctum')->user();
    // ...
}
```

---

## 7. Alur Kerja Token Authentication

### Memahami Siklus Hidup Token

```
┌─────────────────────────────────────────────────────────┐
│                  Siklus Token Sanctum                    │
│                                                         │
│  [Register/Login]                                       │
│       │                                                 │
│       ▼                                                 │
│  createToken('nama')  →  Token disimpan (hash) di DB    │
│       │                                                 │
│       ▼                                                 │
│  Token asli dikirim ke client (sekali saja!)            │
│       │                                                 │
│       ▼                                                 │
│  Client kirim request + "Authorization: Bearer <token>" │
│       │                                                 │
│       ▼                                                 │
│  Middleware auth:sanctum cek token di DB                │
│       │                                                 │
│       ├── Token valid → Request diteruskan ke controller│
│       └── Token tidak ada / expired → 401 Unauthorized  │
│                                                         │
│  [Logout]                                               │
│       │                                                 │
│       ▼                                                 │
│  currentAccessToken()->delete()  →  Token dihapus di DB │
└─────────────────────────────────────────────────────────┘
```

### Implementasi AuthController (Preview — detail di Modul 3)

```php
<?php
// app/Http/Controllers/Api/AuthController.php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|min:8|confirmed',
        ]);

        $user  = User::create($validated);
        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'success'      => true,
            'message'      => 'Registrasi berhasil',
            'data'         => $user,
            'access_token' => $token,
            'token_type'   => 'Bearer',
        ], 201);
    }

    public function login(Request $request): JsonResponse
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json([
                'success' => false,
                'message' => 'Email atau password salah',
            ], 401);
        }

        // Hapus token lama (opsional — untuk single device login)
        // $user->tokens()->delete();

        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'success'      => true,
            'message'      => 'Login berhasil',
            'data'         => $user,
            'access_token' => $token,
            'token_type'   => 'Bearer',
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'success' => true,
            'message' => 'Logout berhasil',
        ]);
    }
}
```

### Format Token yang Diterima Client

```json
{
  "success": true,
  "access_token": "1|AbCdEfGhIjKlMnOpQrStUvWxYz1234567890abcdefgh",
  "token_type": "Bearer"
}
```

Token ini kemudian digunakan di header request:

```
Authorization: Bearer 1|AbCdEfGhIjKlMnOpQrStUvWxYz1234567890abcdefgh
```

### Token Abilities (Scopes)

Token bisa diberi kemampuan spesifik:

```php
// Token dengan semua akses
$token = $user->createToken('full-access')->plainTextToken;

// Token hanya bisa read
$token = $user->createToken('read-only', ['read'])->plainTextToken;

// Token untuk admin
$token = $user->createToken('admin', ['read', 'write', 'delete'])->plainTextToken;

// Cek kemampuan di controller
if ($request->user()->tokenCan('write')) {
    // user boleh menulis
}

// Atau di route menggunakan middleware
Route::middleware(['auth:sanctum', 'abilities:read,write'])->group(function () {
    Route::post('/products', [ProductController::class, 'store']);
});
```

---

## 8. Latihan

### Latihan 1 — Pemahaman Konsep

Jawab pertanyaan berikut:

1. Apa perbedaan utama antara Sanctum API Token mode dan SPA mode?
2. Mengapa token asli tidak disimpan di database melainkan hash-nya?
3. Apa yang terjadi ketika user logout menggunakan Sanctum?
4. Kapan sebaiknya menggunakan Passport dibandingkan Sanctum?

---

### Latihan 2 — Setup Sanctum

Lanjutkan dari proyek Mini Modul 1 (Toko API), kemudian:

1. Pastikan Sanctum sudah terinstall dan `HasApiTokens` sudah ada di model User
2. Jalankan migration
3. Verifikasi tabel `personal_access_tokens` sudah terbuat di database
4. Buka tabel via database GUI (TablePlus, DBeaver, dll.) dan perhatikan strukturnya

```bash
# Cek apakah tabel sudah ada
php artisan tinker
\Schema::hasTable('personal_access_tokens'); // harus true
```

---

### Latihan 3 — Eksplorasi via Tinker

Gunakan `php artisan tinker` untuk:

```php
// 1. Buat user baru
$user = \App\Models\User::create([
    'name'     => 'Budi Santoso',
    'email'    => 'budi@example.com',
    'password' => 'password123',
]);

// 2. Buat token untuk user tersebut
$tokenResult = $user->createToken('test-token');

// 3. Lihat token asli (plaintext)
echo $tokenResult->plainTextToken;

// 4. Lihat data token di database
$tokenResult->accessToken;

// 5. Buat token dengan abilities
$readToken = $user->createToken('read-only', ['read']);

// 6. Lihat semua token milik user
$user->tokens;

// 7. Hapus token pertama
$user->tokens()->first()->delete();

// 8. Hapus semua token
$user->tokens()->delete();
```

---

### Latihan 4 — Protect Routes

Pada project Toko API, tambahkan autentikasi ke route products:

```php
// routes/api.php

// Publik
Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);

// Protected
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::get('/me', [AuthController::class, 'me']);
    Route::apiResource('products', ProductController::class);
});
```

Uji di Postman:
- Akses `GET /api/products` **tanpa** token → harus 401
- Login → dapatkan token
- Akses `GET /api/products` **dengan** token → harus 200

---

## 9. Proyek Mini

### 🔐 Toko API — Tambah Sistem Autentikasi

Lanjutkan proyek Mini Modul 1 dengan menambahkan sistem autentikasi lengkap.

#### Endpoint Autentikasi yang Ditambahkan

| Method | Endpoint          | Keterangan                   | Auth?     |
|--------|-------------------|------------------------------|-----------|
| POST   | /api/v1/register  | Daftar akun baru             | Tidak     |
| POST   | /api/v1/login     | Login, dapatkan token        | Tidak     |
| POST   | /api/v1/logout    | Logout, hapus token          | Ya        |
| GET    | /api/v1/me        | Lihat profil diri sendiri    | Ya        |
| GET    | /api/v1/products  | Daftar produk                | Ya        |

#### Langkah Pengerjaan

**Step 1: Buat AuthController**

```bash
php artisan make:controller Api/AuthController
```

**Step 2: Isi AuthController**

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\Rules\Password;

class AuthController extends Controller
{
    public function register(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users,email',
            'password' => ['required', 'confirmed', Password::min(8)],
        ]);

        $user  = User::create($validated);
        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'success'      => true,
            'message'      => 'Registrasi berhasil',
            'data'         => $user,
            'access_token' => $token,
            'token_type'   => 'Bearer',
        ], 201);
    }

    public function login(Request $request): JsonResponse
    {
        $request->validate([
            'email'    => 'required|email',
            'password' => 'required|string',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            return response()->json([
                'success' => false,
                'message' => 'Kredensial tidak valid',
            ], 401);
        }

        $token = $user->createToken('auth_token')->plainTextToken;

        return response()->json([
            'success'      => true,
            'message'      => 'Login berhasil',
            'data'         => $user,
            'access_token' => $token,
            'token_type'   => 'Bearer',
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'success' => true,
            'message' => 'Logout berhasil',
        ]);
    }

    public function me(Request $request): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data'    => $request->user(),
        ]);
    }
}
```

**Step 3: Update `routes/api.php`**

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\AuthController;
use App\Http\Controllers\Api\ProductController;
use App\Http\Controllers\Api\CategoryController;

Route::prefix('v1')->group(function () {

    // Route publik
    Route::post('/register', [AuthController::class, 'register']);
    Route::post('/login',    [AuthController::class, 'login']);

    // Route yang perlu autentikasi
    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/logout',       [AuthController::class, 'logout']);
        Route::get('/me',            [AuthController::class, 'me']);
        Route::apiResource('categories', CategoryController::class);
        Route::apiResource('products',   ProductController::class);
    });

});
```

**Step 4: Pastikan model User memiliki `HasApiTokens`**

```php
// app/Models/User.php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
    // ...
}
```

**Step 5: Jalankan migration & seeder**

```bash
php artisan migrate:fresh --seed
php artisan serve
```

**Step 6: Test di Postman**

Buat Environment Postman dengan variabel:
- `base_url` = `http://localhost:8000/api/v1`
- `token` = (kosong dulu, akan diisi setelah login)

Buat Collection "Toko API Auth" dengan request berikut:

```
1. POST {{base_url}}/register
   Body: { "name": "Test User", "email": "test@test.com",
           "password": "password123", "password_confirmation": "password123" }

2. POST {{base_url}}/login
   Body: { "email": "test@test.com", "password": "password123" }
   → Salin nilai "access_token" dari response ke variabel {{token}}

3. GET {{base_url}}/me
   Header: Authorization: Bearer {{token}}

4. GET {{base_url}}/products
   Header: Authorization: Bearer {{token}}

5. POST {{base_url}}/logout
   Header: Authorization: Bearer {{token}}

6. GET {{base_url}}/products        ← Setelah logout
   Header: Authorization: Bearer {{token}}
   → Harus return 401 Unauthorized
```

#### Kriteria Keberhasilan

- [ ] Register berhasil membuat user dan mengembalikan token
- [ ] Login mengembalikan token valid
- [ ] Endpoint `/me` mengembalikan data user yang login
- [ ] Akses tanpa token ke route protected → 401
- [ ] Logout menghapus token, akses setelahnya → 401
- [ ] Tabel `personal_access_tokens` berisi record token setelah login
- [ ] Setelah logout, record token terhapus dari database

---

## Ringkasan Modul 2

| Konsep                    | Yang Dipelajari                                              |
|---------------------------|--------------------------------------------------------------|
| Laravel Sanctum           | Pengertian, keunggulan, kapan menggunakan                    |
| Sanctum vs alternatif     | Perbandingan dengan Passport dan JWT                         |
| API Token vs SPA          | Dua mode Sanctum dan perbedaannya                            |
| Instalasi                 | Publish config, migration, trait `HasApiTokens`              |
| `personal_access_tokens`  | Struktur tabel, perbedaan token asli vs hash                 |
| Middleware `auth:sanctum` | Cara kerja, penggunaan di route                              |
| Token abilities           | Memberi izin granular pada token                             |
| Auth flow                 | Register → Login → gunakan token → Logout                    |

> **Selanjutnya → Modul 3:** Implementasi lengkap Register, Login, Logout beserta Form Request Validation dan pengelolaan token multi-device.
