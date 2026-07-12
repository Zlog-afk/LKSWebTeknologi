# Langkah Pengerjaan Project — PintarMenabung (Server Module Phase 1)

> Kisi-kisi: LKS 2026 WebTech — Server Module Phase 1
> Stack: Laravel 11/12 + Sanctum + MySQL
> Media pendukung: `db-dump.sql`, `db-diagram.pdf`, Postman Collection & Environment, Laravel Scaffolding (`XX_SERVER_MODULE_SCAFFOLD.zip`)

---

## 0. Checklist Media yang Sudah Disiapkan

- [x] `db-dump.sql` — schema + data contoh (users, currencies, categories, wallets, transactions)
- [x] `db-diagram.pdf` — ER Diagram 5 tabel
- [x] `PintarMenabung.postman_collection.json` — semua 15 endpoint
- [x] `PintarMenabung.postman_environment.json` — variabel `base_url`, `token`, dll.
- [x] `XX_SERVER_MODULE_SCAFFOLD.zip` — Model, Controller, Migration, Seeder, Routes siap pakai

---

## 1. Persiapan Lingkungan & Setup Awal

1. **Ekstrak** `XX_SERVER_MODULE_SCAFFOLD.zip` ke folder kerja.
2. **Buat project Laravel baru** (karena scaffold hanya berisi file aplikasi, bukan instalasi lengkap):
   ```bash
   composer create-project laravel/laravel pintarmenabung-api
   cd pintarmenabung-api
   composer require laravel/sanctum
   ```
3. **Timpa/copy file dari scaffold** ke project baru:
   ```
   app/Models/*.php                      -> app/Models/
   app/Http/Controllers/Api/             -> app/Http/Controllers/Api/
   app/Http/Controllers/Controller.php   -> app/Http/Controllers/Controller.php
   database/migrations/*.php             -> database/migrations/
   database/seeders/*.php                -> database/seeders/
   routes/api.php                        -> routes/api.php
   bootstrap/app.php                     -> bootstrap/app.php
   config/sanctum.php                    -> config/sanctum.php
   .env.example                          -> .env (sesuaikan kredensial DB)
   ```
4. **Konfigurasi `.env`**: set `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` sesuai MySQL lokal Anda.
5. **Buat database** kosong (misalnya `pintarmenabung`) di MySQL.
6. **Generate app key, migrate, dan seed**:
   ```bash
   php artisan key:generate
   php artisan migrate:fresh --seed
   ```
   *(Alternatif: import langsung `db-dump.sql` via `mysql -u root -p pintarmenabung < db-dump.sql` jika ingin memakai data contoh yang sudah lengkap dengan transaksi.)*
7. **Jalankan server**:
   ```bash
   php artisan serve
   ```
8. **Import Postman Collection & Environment** ke Postman:
   - `PintarMenabung.postman_collection.json`
   - `PintarMenabung.postman_environment.json`
   - Pastikan environment "PintarMenabung - Local" aktif, dan `base_url` sesuai (default `http://localhost:8000/api`).

---

## 2. Pahami Struktur Database (ER Diagram)

Buka `db-diagram.pdf` sebagai referensi. Relasi antar tabel:

| Tabel | Relasi |
|---|---|
| `users` | 1 → banyak `wallets` (via `user_id`) |
| `currencies` | 1 → banyak `wallets` (via `currency_id`) |
| `wallets` | 1 → banyak `transactions` (via `wallet_id`) |
| `categories` | 1 → banyak `transactions` (via `category_id`), punya `type` (EXPENSE/INCOME) |

Model Eloquent (`User`, `Wallet`, `Currency`, `Category`, `Transaction`) pada scaffold sudah memiliki relasi (`hasMany`, `belongsTo`) — tinggal diverifikasi ulang saat coding.

---

## 3. Kerjakan Modul Berdasarkan Urutan Dependensi

### Langkah A — Autentikasi (Sanctum)

1. Cek `AuthController` (`register`, `login`, `logout`) sudah ada di scaffold.
2. `register`: validasi `full_name`, `email` (unique), `password` (min:6) → simpan user → `createToken()`.
3. `login`: validasi email & password, cek kredensial via `Hash::check`.
4. `logout`: hapus token request saat ini via `$request->user()->currentAccessToken()->delete()`.
5. Cocokkan format response (status, message, data) & status code (201/200/401/422) dengan dokumen kisi-kisi.
6. **Test** folder "A. Authentication" di Postman — token otomatis tersimpan ke environment setelah register/login sukses.

### Langkah B — Currency & Category (Read-only)

1. Cek `CurrencyController@index` dan `CategoryController@index`.
2. Data sudah tersedia dari seeder/dump — pastikan endpoint dilindungi middleware `auth:sanctum`.
3. **Test** folder "B. Currency & Category" di Postman.

### Langkah C — Wallet (CRUD + balance calculation)

1. Cek `WalletController` (`store`, `update`, `destroy`, `index`, `show`).
2. Validasi input: `name` (required), `currency_code` (exists di tabel `currencies`).
3. **Balance dihitung dinamis** dari transaksi terkait (accessor `getBalanceAttribute()` di model `Wallet`) — bukan kolom statis.
4. Pastikan hanya pemilik wallet yang bisa update/delete/show → 403 jika bukan pemilik, 404 jika tidak ditemukan.
5. Urutan pengecekan: token (401) → not found (404) → forbidden (403).
6. **Test** folder "C. Wallet" di Postman — `wallet_id` otomatis tersimpan ke environment setelah `Add Wallet` sukses.

### Langkah D — Transaction (inti logika bisnis)

1. Cek `TransactionController` (`store`, `destroy`, `index`).
2. Validasi: `wallet_id` (exists & milik user), `category_id` (exists), `amount` (integer, min:1), `date` (format `Y-m-d`), `note` (opsional).
3. Balance wallet **tidak disimpan ulang** — otomatis konsisten karena dihitung dari total transaksi (lihat Langkah C).
4. `index`: eager load `wallet` & `category`, urutkan `date` descending, pagination (`page`, `per_page`), filter `month`/`year`.
5. `destroy`: cek kepemilikan transaksi via relasi wallet (403/404).
6. **Test** folder "D. Transaction" di Postman — `transaction_id` otomatis tersimpan setelah `Add Transaction` sukses.

### Langkah E — Laporan Keuangan (Reporting)

1. Cek `ReportController` (`summaryByExpenseCategory`, `summaryByIncomeCategory`).
2. Logika: group transaksi user berdasarkan `category_id` (filter `type` kategori EXPENSE/INCOME), `SUM(amount)` per kategori.
3. Terapkan filter opsional `month` dan `year`.
4. **Test** folder "E. Financial Report" di Postman.

---

## 4. Middleware & Konsistensi Response

- `bootstrap/app.php` pada scaffold sudah berisi custom exception handler agar semua error (401/403/404/422) punya format konsisten:
  - 401 → `{"status":"error","message":"Unauthenticated."}`
  - 403 → `{"status":"error","message":"Forbidden access"}`
  - 404 → `{"status":"error","message":"Not found"}`
  - 422 → `{"status":"error","message":"Invalid field","errors":{...}}`
- Cukup verifikasi ulang saat testing, tidak perlu membuat handler baru dari nol.

---

## 5. Verifikasi Routing

Cek `routes/api.php` — pastikan **tidak ada endpoint tambahan** di luar yang diminta dokumen kisi-kisi (instruksi soal melarang endpoint baru):

```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/currencies
GET    /api/categories
POST   /api/wallets
PUT    /api/wallets/{walletId}
DELETE /api/wallets/{walletId}
GET    /api/wallets
GET    /api/wallets/{walletId}
POST   /api/transactions
DELETE /api/transactions/{transactionId}
GET    /api/transactions
GET    /api/reports/summary-by-category/expense
GET    /api/reports/summary-by-category/income
```

---

## 6. Testing Menyeluruh dengan Postman

1. Jalankan seluruh collection secara berurutan: **Register → Login → Currencies → Categories → Wallet → Transaction → Report**.
2. Uji **skenario sukses** *dan* **skenario gagal** (invalid field, unauthenticated, forbidden, not found) untuk setiap endpoint.
3. Perhatikan detail kecil: nama field, tipe data, status code, dan struktur nested JSON harus match persis dengan dokumen kisi-kisi (fungsi `pm.test` di beberapa request sudah otomatis mengecek status code).

---

## 7. Finalisasi & Pengumpulan

1. **Export ulang** database final (setelah data uji Anda) ke `db-dump.sql`.
2. Pastikan `db-diagram.pdf` sudah ada di root folder (sudah disediakan, sesuaikan bila ada perubahan skema).
3. Susun folder submission:
   ```
   XX_SERVER_MODULE/
   ├── BACKEND/          # Kode Laravel Anda (tanpa vendor/)
   ├── db-dump.sql
   └── db-diagram.pdf
   ```
4. **Hapus folder** `vendor/` dan `node_modules/` sebelum zip.
5. Compress menjadi **`XX_SERVER_MODULE_PHASE_1.zip`** dan submit ke halaman pengumpulan.

---

## Referensi Cepat: Urutan Pengecekan Error per Endpoint

| Urutan | Kondisi | Status Code |
|---|---|---|
| 1 | Token tidak ada / tidak valid | 401 Unauthenticated |
| 2 | Validasi field gagal (untuk request POST/PUT) | 422 Invalid field |
| 3 | Resource tidak ditemukan | 404 Not found |
| 4 | Resource ditemukan tapi bukan milik user | 403 Forbidden access |

---

*Selamat mengerjakan! Jika menemukan kendala di salah satu modul (Auth, Wallet, Transaction, atau Report), tanyakan detail error/response yang didapat agar bisa ditelusuri bersama.*
