# Modul 1 — Fondasi: Laravel & REST API

> **Versi:** Laravel 12.x · PHP 8.3+ · **Durasi:** 3–4 jam

---

## Daftar Isi

1. [Konsep REST API](#1-konsep-rest-api)
2. [Instalasi Laravel 12](#2-instalasi-laravel-12)
3. [Struktur Project untuk API](#3-struktur-project-untuk-api)
4. [Routing API](#4-routing-api)
5. [Controller & Response JSON](#5-controller--response-json)
6. [Eloquent ORM Dasar](#6-eloquent-orm-dasar)
7. [Testing dengan Postman](#7-testing-dengan-postman)
8. [Latihan](#8-latihan)
9. [Proyek Mini](#9-proyek-mini)

---

## 1. Konsep REST API

### Apa itu REST API?

REST (Representational State Transfer) adalah gaya arsitektur untuk membangun web service. API (Application Programming Interface) berbasis REST memungkinkan komunikasi antar aplikasi melalui protokol HTTP.

### HTTP Methods (Verbs)

| Method   | Fungsi              | Contoh URL            |
|----------|---------------------|-----------------------|
| `GET`    | Mengambil data      | `GET /api/products`   |
| `POST`   | Membuat data baru   | `POST /api/products`  |
| `PUT`    | Update data penuh   | `PUT /api/products/1` |
| `PATCH`  | Update data sebagian| `PATCH /api/products/1`|
| `DELETE` | Menghapus data      | `DELETE /api/products/1`|

### HTTP Status Codes

| Kode | Arti                        | Kapan Digunakan                      |
|------|-----------------------------|--------------------------------------|
| 200  | OK                          | Request berhasil (GET, PUT, PATCH)   |
| 201  | Created                     | Data berhasil dibuat (POST)          |
| 204  | No Content                  | Berhasil, tidak ada body (DELETE)    |
| 400  | Bad Request                 | Input tidak valid                    |
| 401  | Unauthorized                | Belum autentikasi                    |
| 403  | Forbidden                   | Tidak punya izin                     |
| 404  | Not Found                   | Data tidak ditemukan                 |
| 422  | Unprocessable Entity        | Validasi gagal                       |
| 500  | Internal Server Error       | Error di server                      |

### Format JSON Response

Gunakan struktur response yang konsisten di seluruh API:

```json
// Response sukses (single item)
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Laptop ASUS",
    "price": 8500000
  },
  "message": "Data berhasil diambil"
}

// Response sukses (koleksi)
{
  "success": true,
  "data": [...],
  "meta": {
    "total": 50,
    "per_page": 10,
    "current_page": 1
  }
}

// Response error
{
  "success": false,
  "message": "Data tidak ditemukan",
  "errors": {
    "id": ["Record tidak ada di database"]
  }
}
```

### Prinsip RESTful

- **Stateless** — setiap request harus membawa semua informasi yang dibutuhkan server
- **Uniform Interface** — gunakan URL yang konsisten dan deskriptif
- **Resource-based** — URL merepresentasikan resource (kata benda, bukan kata kerja)

```
// Baik ✅
GET    /api/articles
GET    /api/articles/1
POST   /api/articles
PUT    /api/articles/1
DELETE /api/articles/1

// Buruk ❌
GET  /api/getArticles
POST /api/deleteArticle/1
GET  /api/article_list
```

---

## 2. Instalasi Laravel 12

### Persyaratan Sistem

- PHP 8.3 atau lebih baru
- Composer 2.x
- Database: MySQL 8+, PostgreSQL 15+, atau SQLite 3.26+
- Node.js (opsional, untuk assets)

### Instalasi via Composer

```bash
# Buat project baru
composer create-project laravel/laravel toko-api

# Masuk ke folder project
cd toko-api

# Jalankan server development
php artisan serve
```

### Instalasi via Laravel Installer

```bash
# Install Laravel Installer secara global
composer global require laravel/installer

# Buat project baru
laravel new toko-api

# Pilih opsi: No starter kit, SQLite atau MySQL, tidak perlu testing framework dulu
```

### Konfigurasi Awal `.env`

```env
APP_NAME=TokoAPI
APP_ENV=local
APP_KEY=                    # Di-generate otomatis saat install
APP_DEBUG=true
APP_URL=http://localhost:8000

# Untuk SQLite (lebih mudah untuk development)
DB_CONNECTION=sqlite
# DB_DATABASE=/absolute/path/to/database.sqlite

# Untuk MySQL
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=toko_api
DB_USERNAME=root
DB_PASSWORD=
```

### Membuat Database SQLite (Development)

```bash
# Buat file database SQLite
touch database/database.sqlite

# Jalankan migration
php artisan migrate
```

---

## 3. Struktur Project untuk API

### Struktur Direktori Penting

```
toko-api/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   └── Api/           ← Buat subfolder khusus API
│   │   ├── Middleware/
│   │   └── Requests/          ← Form Request Validation
│   ├── Models/
│   └── Providers/
├── bootstrap/
│   └── app.php                ← Konfigurasi utama di Laravel 12
├── config/
│   └── sanctum.php            ← Nanti di Modul 2
├── database/
│   ├── migrations/
│   └── seeders/
├── routes/
│   ├── api.php                ← Semua route API ada di sini
│   └── web.php
└── .env
```

### Konfigurasi `bootstrap/app.php` (Laravel 12)

Di Laravel 12, konfigurasi middleware dan route dilakukan di `bootstrap/app.php`, bukan `Kernel.php`:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',   // ← Route API
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        // Konfigurasi middleware di sini
    })
    ->withExceptions(function (Exceptions $exceptions) {
        // Konfigurasi exception handler di sini
    })->create();
```

> **Catatan Laravel 12:** File `app/Http/Kernel.php` sudah tidak ada. Semua konfigurasi middleware ada di `bootstrap/app.php`.

---

## 4. Routing API

### File `routes/api.php`

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ProductController;

// Route dasar tanpa middleware
Route::get('/products', [ProductController::class, 'index']);
Route::get('/products/{id}', [ProductController::class, 'show']);
Route::post('/products', [ProductController::class, 'store']);
Route::put('/products/{id}', [ProductController::class, 'update']);
Route::delete('/products/{id}', [ProductController::class, 'destroy']);

// Cara lebih ringkas menggunakan apiResource
Route::apiResource('products', ProductController::class);
```

### Perbedaan `Route::resource` vs `Route::apiResource`

`apiResource` tidak membuat route `create` dan `edit` (karena API tidak butuh halaman form HTML).

| Method  | URI                  | Action  | Route Name          |
|---------|----------------------|---------|---------------------|
| GET     | /products            | index   | products.index      |
| POST    | /products            | store   | products.store      |
| GET     | /products/{id}       | show    | products.show       |
| PUT     | /products/{id}       | update  | products.update     |
| DELETE  | /products/{id}       | destroy | products.destroy    |

### Grouping Route

```php
// Prefix versi API
Route::prefix('v1')->group(function () {
    Route::apiResource('products', ProductController::class);
    Route::apiResource('categories', CategoryController::class);
});

// Atau menggunakan nama
Route::prefix('v1')->name('v1.')->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

### Melihat Semua Route

```bash
php artisan route:list
php artisan route:list --path=api   # Filter hanya route API
```

---

## 5. Controller & Response JSON

### Membuat Controller API

```bash
php artisan make:controller Api/ProductController --api
```

### Struktur Controller

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class ProductController extends Controller
{
    // GET /api/products
    public function index(): JsonResponse
    {
        $products = Product::all();

        return response()->json([
            'success' => true,
            'data'    => $products,
            'message' => 'Data produk berhasil diambil',
        ], 200);
    }

    // GET /api/products/{id}
    public function show(int $id): JsonResponse
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'success' => false,
                'message' => 'Produk tidak ditemukan',
            ], 404);
        }

        return response()->json([
            'success' => true,
            'data'    => $product,
        ], 200);
    }

    // POST /api/products
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'        => 'required|string|max:255',
            'description' => 'nullable|string',
            'price'       => 'required|numeric|min:0',
            'stock'       => 'required|integer|min:0',
        ]);

        $product = Product::create($validated);

        return response()->json([
            'success' => true,
            'data'    => $product,
            'message' => 'Produk berhasil ditambahkan',
        ], 201);
    }

    // PUT /api/products/{id}
    public function update(Request $request, int $id): JsonResponse
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'success' => false,
                'message' => 'Produk tidak ditemukan',
            ], 404);
        }

        $validated = $request->validate([
            'name'        => 'sometimes|string|max:255',
            'description' => 'nullable|string',
            'price'       => 'sometimes|numeric|min:0',
            'stock'       => 'sometimes|integer|min:0',
        ]);

        $product->update($validated);

        return response()->json([
            'success' => true,
            'data'    => $product,
            'message' => 'Produk berhasil diupdate',
        ], 200);
    }

    // DELETE /api/products/{id}
    public function destroy(int $id): JsonResponse
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'success' => false,
                'message' => 'Produk tidak ditemukan',
            ], 404);
        }

        $product->delete();

        return response()->json([
            'success' => true,
            'message' => 'Produk berhasil dihapus',
        ], 200);
    }
}
```

### Helper Function Response (Opsional)

Buat trait untuk response yang konsisten:

```php
<?php
// app/Traits/ApiResponse.php

namespace App\Traits;

use Illuminate\Http\JsonResponse;

trait ApiResponse
{
    protected function successResponse(mixed $data, string $message = 'Berhasil', int $code = 200): JsonResponse
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data'    => $data,
        ], $code);
    }

    protected function errorResponse(string $message, int $code = 400, mixed $errors = null): JsonResponse
    {
        $response = [
            'success' => false,
            'message' => $message,
        ];

        if ($errors) {
            $response['errors'] = $errors;
        }

        return response()->json($response, $code);
    }
}
```

Gunakan di controller:

```php
use App\Traits\ApiResponse;

class ProductController extends Controller
{
    use ApiResponse;

    public function index(): JsonResponse
    {
        $products = Product::all();
        return $this->successResponse($products, 'Data produk berhasil diambil');
    }
}
```

---

## 6. Eloquent ORM Dasar

### Membuat Model & Migration

```bash
# Buat model sekaligus migration, factory, dan seeder
php artisan make:model Product -mfs
```

### Migration

```php
<?php
// database/migrations/xxxx_create_products_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->decimal('price', 15, 2);
            $table->integer('stock')->default(0);
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

### Model

```php
<?php
// app/Models/Product.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory;

    // Kolom yang boleh diisi massal
    protected $fillable = [
        'name',
        'description',
        'price',
        'stock',
        'is_active',
    ];

    // Casting tipe data otomatis
    protected $casts = [
        'price'     => 'decimal:2',
        'stock'     => 'integer',
        'is_active' => 'boolean',
    ];

    // Menyembunyikan kolom dari response
    // protected $hidden = ['created_at', 'updated_at'];
}
```

### Factory & Seeder

```php
<?php
// database/factories/ProductFactory.php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class ProductFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name'        => fake()->words(3, true),
            'description' => fake()->paragraph(),
            'price'       => fake()->randomFloat(2, 10000, 5000000),
            'stock'       => fake()->numberBetween(0, 100),
            'is_active'   => fake()->boolean(80), // 80% kemungkinan true
        ];
    }
}
```

```php
<?php
// database/seeders/ProductSeeder.php

namespace Database\Seeders;

use App\Models\Product;
use Illuminate\Database\Seeder;

class ProductSeeder extends Seeder
{
    public function run(): void
    {
        Product::factory(20)->create();
    }
}
```

```php
<?php
// database/seeders/DatabaseSeeder.php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            ProductSeeder::class,
        ]);
    }
}
```

```bash
# Jalankan migration dan seeder
php artisan migrate --seed

# Atau jika ingin reset semua
php artisan migrate:fresh --seed
```

### Query Eloquent yang Sering Dipakai

```php
// Mengambil semua data
$products = Product::all();

// Mengambil dengan kondisi
$products = Product::where('is_active', true)->get();

// Dengan pengurutan
$products = Product::orderBy('created_at', 'desc')->get();

// Dengan pagination
$products = Product::paginate(10);

// Mengambil satu data berdasarkan ID
$product = Product::find(1);
$product = Product::findOrFail(1); // Otomatis 404 jika tidak ada

// Membuat data baru
$product = Product::create([
    'name'  => 'Laptop ASUS',
    'price' => 8500000,
    'stock' => 10,
]);

// Update data
$product->update(['price' => 9000000]);

// Hapus data
$product->delete();
```

---

## 7. Testing dengan Postman

### Setup Postman

1. Download dan install [Postman](https://www.postman.com/downloads/)
2. Buat **Collection** baru bernama "Toko API"
3. Set **Base URL** sebagai variable: `{{base_url}}` = `http://localhost:8000/api`

### Konfigurasi Header Default

Tambahkan header berikut di setiap request:

```
Content-Type: application/json
Accept:       application/json
```

> **Penting:** Header `Accept: application/json` membuat Laravel mengembalikan error dalam format JSON, bukan halaman HTML.

### Contoh Request di Postman

**GET semua produk:**
```
Method: GET
URL:    {{base_url}}/products
```

**POST buat produk baru:**
```
Method: POST
URL:    {{base_url}}/products
Body (raw JSON):
{
    "name": "Laptop ASUS VivoBook",
    "description": "Laptop untuk pelajar dan profesional",
    "price": 8500000,
    "stock": 15
}
```

**PUT update produk:**
```
Method: PUT
URL:    {{base_url}}/products/1
Body (raw JSON):
{
    "name": "Laptop ASUS VivoBook 15",
    "price": 8750000
}
```

**DELETE hapus produk:**
```
Method: DELETE
URL:    {{base_url}}/products/1
```

---

## 8. Latihan

### Latihan 1 — HTTP Methods & Status Codes

Jawab pertanyaan berikut tanpa melihat materi:

1. Method HTTP apa yang digunakan untuk **membuat data baru**?
2. Apa perbedaan status code `200`, `201`, dan `204`?
3. Kapan kita menggunakan `PUT` vs `PATCH`?
4. Mengapa URL `/api/getProducts` dianggap tidak RESTful?

---

### Latihan 2 — Routing & Controller

Buat routing dan controller untuk resource **Category** dengan ketentuan:

- Model: `Category` dengan kolom `name` (string) dan `description` (text, nullable)
- Gunakan `apiResource` untuk routing
- Kembalikan response JSON dengan format yang konsisten
- Handle error 404 jika data tidak ditemukan

**Langkah-langkah:**

```bash
# 1. Buat model, migration, factory, seeder
php artisan make:model Category -mfs

# 2. Buat controller
php artisan make:controller Api/CategoryController --api
```

Isi migration, model, controller, dan tambahkan route di `api.php`.

---

### Latihan 3 — Validasi Input

Tambahkan validasi pada `ProductController@store` dan `ProductController@update` dengan aturan:

- `name`: wajib diisi, string, maksimal 255 karakter
- `price`: wajib diisi, numerik, minimal 0
- `stock`: wajib diisi, integer, minimal 0
- `description`: opsional, string

Uji dengan Postman: kirim request tanpa field `name` dan lihat response error 422.

---

### Latihan 4 — Query Eloquent

Tambahkan fitur filter dan sort di endpoint `GET /api/products`:

```php
// Endpoint yang diharapkan:
// GET /api/products?search=laptop        → cari berdasarkan nama
// GET /api/products?sort=price&order=asc → urutkan berdasarkan harga
// GET /api/products?per_page=5           → pagination 5 item per halaman
```

**Petunjuk:**

```php
public function index(Request $request): JsonResponse
{
    $query = Product::query();

    // Filter pencarian
    if ($request->has('search')) {
        $query->where('name', 'like', '%' . $request->search . '%');
    }

    // Sorting
    $sortBy    = $request->get('sort', 'created_at');
    $sortOrder = $request->get('order', 'desc');
    $query->orderBy($sortBy, $sortOrder);

    // Pagination
    $perPage  = $request->get('per_page', 10);
    $products = $query->paginate($perPage);

    return response()->json([
        'success' => true,
        'data'    => $products,
    ]);
}
```

---

## 9. Proyek Mini

### 📦 Toko API — CRUD Produk & Kategori

Bangun API sederhana untuk manajemen toko dengan fitur lengkap.

#### Spesifikasi

**Tabel `categories`:**
- `id`, `name` (unique), `description`, `timestamps`

**Tabel `products`:**
- `id`, `category_id` (FK), `name`, `description`, `price`, `stock`, `is_active`, `timestamps`

#### Endpoints yang Harus Dibuat

| Method | Endpoint                  | Deskripsi                       |
|--------|---------------------------|---------------------------------|
| GET    | /api/v1/categories        | Daftar semua kategori           |
| POST   | /api/v1/categories        | Tambah kategori baru            |
| GET    | /api/v1/categories/{id}   | Detail kategori                 |
| PUT    | /api/v1/categories/{id}   | Update kategori                 |
| DELETE | /api/v1/categories/{id}   | Hapus kategori                  |
| GET    | /api/v1/products          | Daftar produk (dengan filter)   |
| POST   | /api/v1/products          | Tambah produk baru              |
| GET    | /api/v1/products/{id}     | Detail produk beserta kategori  |
| PUT    | /api/v1/products/{id}     | Update produk                   |
| DELETE | /api/v1/products/{id}     | Hapus produk                    |

#### Langkah Pengerjaan

**Step 1: Setup Project**

```bash
composer create-project laravel/laravel toko-api
cd toko-api
touch database/database.sqlite
```

**Step 2: Buat Model & Migration**

```bash
php artisan make:model Category -mfs
php artisan make:model Product -mfs
```

**Step 3: Isi Migration Category**

```php
Schema::create('categories', function (Blueprint $table) {
    $table->id();
    $table->string('name')->unique();
    $table->text('description')->nullable();
    $table->timestamps();
});
```

**Step 4: Isi Migration Product**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->foreignId('category_id')->constrained()->cascadeOnDelete();
    $table->string('name');
    $table->text('description')->nullable();
    $table->decimal('price', 15, 2);
    $table->integer('stock')->default(0);
    $table->boolean('is_active')->default(true);
    $table->timestamps();
});
```

**Step 5: Isi Model Product (dengan relasi)**

```php
protected $fillable = ['category_id', 'name', 'description', 'price', 'stock', 'is_active'];

protected $casts = [
    'price'     => 'decimal:2',
    'is_active' => 'boolean',
];

public function category()
{
    return $this->belongsTo(Category::class);
}
```

**Step 6: Tambahkan Relasi di Model Category**

```php
public function products()
{
    return $this->hasMany(Product::class);
}
```

**Step 7: Route `routes/api.php`**

```php
Route::prefix('v1')->group(function () {
    Route::apiResource('categories', Api\CategoryController::class);
    Route::apiResource('products', Api\ProductController::class);
});
```

**Step 8: Controller Product (dengan relasi)**

Di method `show`, sertakan data kategori:

```php
public function show(int $id): JsonResponse
{
    $product = Product::with('category')->find($id);

    if (!$product) {
        return response()->json(['success' => false, 'message' => 'Produk tidak ditemukan'], 404);
    }

    return response()->json(['success' => true, 'data' => $product]);
}
```

**Step 9: Seeder**

```php
// CategorySeeder
Category::factory()->createMany([
    ['name' => 'Elektronik',  'description' => 'Produk elektronik'],
    ['name' => 'Fashion',     'description' => 'Pakaian dan aksesoris'],
    ['name' => 'Makanan',     'description' => 'Produk makanan & minuman'],
]);

// ProductSeeder
Product::factory(30)->create();
```

```bash
php artisan migrate:fresh --seed
php artisan serve
```

#### Kriteria Keberhasilan

- [ ] Semua 10 endpoint berjalan dengan response JSON yang konsisten
- [ ] Validasi input berfungsi dan mengembalikan error 422 dengan pesan jelas
- [ ] Response 404 ketika data tidak ditemukan
- [ ] Endpoint `GET /api/v1/products/{id}` menyertakan data kategori
- [ ] Endpoint `GET /api/v1/products` mendukung filter `?search=` dan `?category_id=`
- [ ] Seeder berhasil mengisi data dummy

---

## Ringkasan Modul 1

| Konsep          | Yang Dipelajari                                              |
|-----------------|--------------------------------------------------------------|
| REST API        | HTTP methods, status codes, format JSON response             |
| Laravel 12      | Instalasi, struktur project, `bootstrap/app.php`             |
| Routing         | `apiResource`, prefix, grouping                              |
| Controller      | CRUD lengkap, validasi, response JSON                        |
| Eloquent        | Model, migration, factory, seeder, query dasar               |
| Testing         | Postman setup, request GET/POST/PUT/DELETE                   |

> **Selanjutnya → Modul 2:** Pengenalan Laravel Sanctum dan sistem autentikasi berbasis token.
