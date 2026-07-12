# MODUL SISI SERVER

## DAFTAR ISI

Modul ini terdiri dari file-file berikut:

1. SERVER MODULE PHASE 1.docx
2. SERVER MODULE PHASE 1.pdf
3. SERVER MODULE PHASE 1 MEDIA.zip

## PENDAHULUAN

Klien telah meminta Anda untuk membangun sistem backend dan aplikasi web frontend untuk sebuah **Aplikasi Manajemen Keuangan** bernama **PintarMenabung**. Sistem ini harus memungkinkan pengguna untuk mendaftar, mengelola dompet (wallet) mereka, mencatat pemasukan dan pengeluaran, serta melihat laporan keuangan dasar.

## GAMBARAN UMUM PROYEK

Proyek ini terdiri dari dua fase utama: Pada Fase 1, Anda diharuskan mengembangkan sebuah **REST API** menggunakan **Framework Laravel**, dengan mengimplementasikan fitur-fitur inti seperti autentikasi pengguna, manajemen dompet, pencatatan transaksi, dan pelaporan keuangan. Pada Fase 2, Anda akan membangun sebuah **aplikasi web frontend** yang terintegrasi dengan API tersebut, menggunakan **React** atau **Vue** sebagai framework pilihan Anda.

Untuk mendukung pekerjaan Anda, telah disediakan file media berupa Postman Collection dan Environment, Dump Basis Data (.sql), dan Framework Scaffolding.

---

# FASE 1 - Pengembangan Backend (REST API)

## A. Autentikasi

Terdapat tiga endpoint: register, login, dan logout. **Sanctum** harus digunakan untuk membuat token. Token yang dihasilkan akan digunakan untuk mengakses semua endpoint yang tersedia. Fungsi logout hanya boleh menonaktifkan token dari perangkat spesifik yang melakukan request tersebut.

### A1: Register

**POST** `/api/auth/register`

Untuk pengguna membuat akun dalam aplikasi.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |

**Body (JSON)**

| Field | Keterangan |
|---|---|
| full_name | wajib diisi. |
| email | wajib diisi, format email valid, unik. |
| password | wajib diisi, minimal 6 karakter. |

**A1a: Response Sukses**

| | |
|---|---|
| Status Code | 201 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Registration successful",\n  "data": {\n    "name": "Dedi",\n    "email": "dedi@webtech.id",\n    "updated_at": "2025-07-11T16:01:46.000000Z",\n    "created_at": "2025-07-11T16:01:46.000000Z",\n    "id": 6,\n    "token": "1\|wwuOrgoVe4Xyj5dvUmnf6WTPTaA..."\n  }\n}\n``` |

**A1b: Response Field Tidak Valid**

| | |
|---|---|
| Status Code | 422 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Invalid field",\n  "errors": {\n    "name": [\n      "The name field is required."\n    ],\n    "email": [\n      "The email has already been taken."\n    ],\n    "password": [\n      "The password field must be at least 6 characters."\n    ]\n  }\n}\n``` |

### A2: Login

**POST** `/api/auth/login`

Untuk pengguna login ke dalam aplikasi menggunakan email dan password.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |

**Body (JSON)**

| Field | Keterangan |
|---|---|
| email | wajib diisi. |
| password | wajib diisi. |

**A2a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Login successful",\n  "data": {\n    "id": 1,\n    "name": "Budi",\n    "email": "budi@webtech.id",\n    "created_at": "2025-07-11T16:01:38.000000Z",\n    "updated_at": "2025-07-11T16:01:38.000000Z",\n    "token": "2\|0Q2TAdua0FOslp1F4cxfkrzQgkp..."\n  }\n}\n``` |

**A2b: Response Gagal**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Username or password incorrect"\n}\n``` |

### A3: Logout

**POST** `/api/auth/logout`

Untuk pengguna yang sudah login untuk logout dari aplikasi. Endpoint ini secara spesifik menonaktifkan token autentikasi yang terkait dengan perangkat yang sedang melakukan request tersebut.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**A3a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Logout successful"\n}\n``` |

**A3b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

## B. Mata Uang dan Kategori

Aplikasi ini menggunakan sistem Kategori untuk mengklasifikasikan transaksi keuangan. Setiap kategori harus diberi sebuah tipe, yang dapat berupa:

- **Income (Pemasukan)** – untuk transaksi yang menambah saldo dompet (misalnya gaji, bonus)
- **Expense (Pengeluaran)** – untuk transaksi yang mengurangi saldo dompet (misalnya belanja kebutuhan, transportasi)

Kategori membantu pengguna mengatur keuangan mereka dan diperlukan saat membuat sebuah transaksi.

### B1: Ambil Semua Mata Uang

**GET** `/api/currencies`

Untuk pengguna yang sudah login mengambil semua mata uang yang tersedia.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**B1a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "message": "Get all currencies successful",\n  "data": {\n    "currencies": [\n      {\n        "id": 1,\n        "name": "US Dollar",\n        "symbol": "$",\n        "code": "USD",\n        "created_at": "2025-07-11T16:01:37...",\n        "updated_at": "2025-07-11T16:01:37..."\n      },\n      ...\n    ]\n  }\n}\n``` |

**B1b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### B2: Ambil Semua Kategori

**GET** `/api/categories`

Untuk pengguna yang sudah login mengambil semua kategori yang tersedia.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**B2a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Get all categories successful",\n  "data": {\n    "categories": [\n      {\n        "id": 1,\n        "name": "Outgoing Transfer",\n        "icon": "⬆",\n        "type": "EXPENSE",\n        "created_at": "2025-07-11T16:01:38...",\n        "updated_at": "2025-07-11T16:01:38..."\n      },\n      ...\n    ]\n  }\n}\n``` |

**B2b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

## C. Dompet (Wallet)

Wallet adalah wadah keuangan yang digunakan untuk melacak pemasukan dan pengeluaran pengguna. Pengguna dapat membuat beberapa dompet untuk mengatur keuangan mereka (misalnya dompet pribadi, tabungan, atau bisnis).

Setiap dompet harus menampilkan saldo (balance) saat ini, yang dihitung secara otomatis dan akurat berdasarkan semua transaksi yang terkait.

- Transaksi Income akan menambah saldo.
- Transaksi Expense akan mengurangi saldo.

Sistem harus memastikan bahwa saldo dompet selalu terbaru dan mencerminkan total yang benar dari semua transaksi yang dilakukan dalam dompet tersebut.

### C1: Tambah Dompet

**POST** `/api/wallets`

Untuk pengguna yang sudah login menambahkan sebuah dompet.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Body (JSON)**

| Field | Keterangan |
|---|---|
| name | wajib diisi. |
| currency_code | wajib diisi, data currency_code yang valid. |

**C1a: Response Sukses**

| | |
|---|---|
| Status Code | 201 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Wallet added successful",\n  "data": {\n    "name": "Savings",\n    "user_id": 1,\n    "updated_at": "2025-07-11T16:22:44.000000Z",\n    "created_at": "2025-07-11T16:22:44.000000Z",\n    "id": 26,\n    "currency_code": "IDR"\n  }\n}\n``` |

**C1b: Response Field Tidak Valid**

| | |
|---|---|
| Status Code | 422 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Invalid field",\n  "errors": {\n    "name": [\n      "The name field is required."\n    ],\n    "currency_code": [\n      "The selected currency code is invalid."\n    ]\n  }\n}\n``` |

**C1c: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### C2: Perbarui Dompet

**PUT** `/api/wallets/:walletId`

Untuk pengguna yang sudah login memperbarui dompet miliknya sendiri.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Body (JSON)**

| Field | Keterangan |
|---|---|
| name | wajib diisi. |

**C2a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Wallet updated successful",\n  "data": {\n    "id": 1,\n    "user_id": 1,\n    "name": "Cash IDR",\n    "created_at": "2025-07-11T16:01:38.000000Z",\n    "updated_at": "2025-07-11T16:26:25.000000Z",\n    "deleted_at": null,\n    "currency_code": "IDR"\n  }\n}\n``` |

**C2b: Response Tidak Ditemukan**

| | |
|---|---|
| Status Code | 404 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Not found"\n}\n``` |

**C2c: Response Terlarang (Forbidden)**

| | |
|---|---|
| Status Code | 403 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Forbidden access"\n}\n``` |

**C2d: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### C3: Hapus Dompet

**DELETE** `/api/wallets/:walletId`

Untuk pengguna yang sudah login menghapus dompet miliknya sendiri.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**C3a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Wallet deleted successful"\n}\n``` |

**C3b: Response Tidak Ditemukan**

| | |
|---|---|
| Status Code | 404 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Not found"\n}\n``` |

**C3c: Response Terlarang (Forbidden)**

| | |
|---|---|
| Status Code | 403 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Forbidden access"\n}\n``` |

**C3d: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### C4: Ambil Semua Dompet

**GET** `/api/wallets`

Untuk pengguna yang sudah login mengambil semua dompet miliknya sendiri.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**C4a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Get all wallets successful",\n  "data": {\n    "wallets": [\n      {\n        "id": 2,\n        "user_id": 1,\n        "name": "Bank",\n        "created_at": "2025-07-11T16:01:38...",\n        "updated_at": "2025-07-11T16:01:38...",\n        "deleted_at": null,\n        "currency_code": "IDR",\n        "balance": 12250000\n      },\n      ...\n    ]\n  }\n}\n``` |

**C4b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### C5: Ambil Detail Dompet

**GET** `/api/wallets/:walletId`

Untuk pengguna yang sudah login mengambil detail dompet miliknya sendiri.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**C5a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Get detail wallet successful",\n  "data": {\n    "id": 1,\n    "user_id": 1,\n    "name": "Cash IDR",\n    "created_at": "2025-07-11T16:01:38.000000Z",\n    "updated_at": "2025-07-11T16:26:25.000000Z",\n    "deleted_at": null,\n    "currency_code": "IDR",\n    "balance": 4865000\n  }\n}\n``` |

**C5b: Response Terlarang (Forbidden)**

| | |
|---|---|
| Status Code | 403 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Forbidden access"\n}\n``` |

**C5c: Response Tidak Ditemukan**

| | |
|---|---|
| Status Code | 404 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Not found"\n}\n``` |

**C5d: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

## D. Transaksi

Transaksi merepresentasikan sebuah catatan keuangan yang terjadi dalam sebuah dompet tertentu. Setiap transaksi harus terhubung ke sebuah dompet dan ditetapkan ke sebuah kategori, yang menentukan apakah transaksi tersebut adalah pemasukan atau pengeluaran.

Atribut utama sebuah transaksi meliputi:

- **Wallet** – dompet tempat transaksi tersebut berada
- **Category** – menentukan tipe (pemasukan atau pengeluaran) berdasarkan kategori yang dipilih
- **Amount** – nilai moneter dari transaksi (angka positif)
- **Date** – tanggal terjadinya transaksi
- **Note (opsional)** – catatan singkat tentang transaksi

Dampak transaksi terhadap saldo dompet tergantung pada tipe yang ditentukan oleh kategori yang dipilih:

- Kategori Income → menambah saldo dompet
- Kategori Expense → mengurangi saldo dompet

Sistem harus memastikan bahwa transaksi dicatat secara akurat dan saldo dompet diperbarui sesuai dengan itu.

### D1: Tambah Transaksi

**POST** `/api/transactions`

Untuk pengguna yang sudah login menambahkan sebuah transaksi.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Body (JSON)**

| Field | Keterangan |
|---|---|
| wallet_id | wajib diisi, data wallet_id yang valid. |
| category_id | wajib diisi, data category_id yang valid. |
| amount | wajib diisi, bilangan bulat, lebih besar atau sama dengan 1 |
| date | wajib diisi, format tanggal "Y-m-d" |
| note | opsional |

**D1a: Response Sukses**

| | |
|---|---|
| Status Code | 201 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Transaction added successful",\n  "data": {\n    "category_id": 3,\n    "wallet_id": 1,\n    "amount": 50000,\n    "note": "starbucks",\n    "date": "2025-07-31",\n    "updated_at": "2025-07-11T16:44:09.000000Z",\n    "created_at": "2025-07-11T16:44:09.000000Z",\n    "id": 842\n  }\n}\n``` |

**D1b: Response Field Tidak Valid**

| | |
|---|---|
| Status Code | 422 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Invalid field",\n  "errors": {\n    "name": [\n      "The name field is required."\n    ],\n    "currency_code": [\n      "The selected currency code is invalid."\n    ]\n  }\n}\n``` |

**D1c: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### D2: Hapus Transaksi

**DELETE** `/api/transactions/:transactionId`

Untuk pengguna yang sudah login menghapus transaksi miliknya sendiri.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**D2a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Transaction deleted successful"\n}\n``` |

**D2b: Response Tidak Ditemukan**

| | |
|---|---|
| Status Code | 404 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Not found"\n}\n``` |

**D2c: Response Terlarang (Forbidden)**

| | |
|---|---|
| Status Code | 403 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Forbidden access"\n}\n``` |

**D2d: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### D3: Ambil Transaksi

**GET** `/api/transactions`

Untuk pengguna yang sudah login mengambil semua transaksi miliknya sendiri, diurutkan berdasarkan tanggal transaksi secara menurun (descending).

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Request Parameters**

| Parameter | Keterangan |
|---|---|
| page | opsional, bilangan bulat, default 1 |
| per_page | opsional, bilangan bulat, default 25 |
| month | opsional, bilangan bulat, antara 1 dan 12 |
| year | opsional, bilangan bulat |

**D3a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "current_page": 1,\n  "data": [\n    {\n      "id": 842,\n      "category_id": 3,\n      "wallet_id": 1,\n      "amount": 50000,\n      "note": "starbucks",\n      "date": "2025-07-31",\n      "created_at": "2025-07-11T16:44:09.000000Z",\n      "updated_at": "2025-07-11T16:44:09.000000Z",\n      "wallet": {\n        "id": 1,\n        "user_id": 1,\n        "name": "Savings IDR",\n        "created_at": "2025-07-11T16:01:38.000000Z",\n        "updated_at": "2025-07-11T16:26:25.000000Z",\n        "deleted_at": null,\n        "currency_code": "IDR"\n      },\n      "category": {\n        "id": 3,\n        "name": "Food & Drinks",\n        "icon": "🍔",\n        "type": "EXPENSE",\n        "created_at": "2025-07-11T16:01:38.000000Z",\n        "updated_at": "2025-07-11T16:01:38.000000Z"\n      }\n    },\n    ...\n  ],\n  "from": 1,\n  "last_page": 170,\n  "per_page": 1,\n  "to": 1,\n  "total": 170\n}\n``` |

**D3b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

## E. Laporan Keuangan

### E1: Ambil Ringkasan berdasarkan Kategori Pengeluaran

**GET** `/api/reports/summary-by-category/expense`

Untuk pengguna yang sudah login mengambil ringkasan berdasarkan kategori pengeluaran.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Request Parameters**

| Parameter | Keterangan |
|---|---|
| month | opsional, bilangan bulat, antara 1 dan 12 |
| year | opsional, bilangan bulat |

**E1a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Get summary by expense category successful",\n  "data": {\n    "summary": [\n      {\n        "category": {\n          "id": 13,\n          "name": "Groceries",\n          "icon": "🛒",\n          "color": "#55EFC4",\n          "type": "EXPENSE",\n          "created_at": "2025-07-28T02:37:46.000000Z",\n          "updated_at": "2025-07-28T02:37:46.000000Z"\n        },\n        "amount": 50000\n      },\n      ...\n    ]\n  }\n}\n``` |

**E1b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

### E2: Ambil Ringkasan berdasarkan Kategori Pemasukan

**GET** `/api/reports/summary-by-category/income`

Untuk pengguna yang sudah login mengambil ringkasan berdasarkan kategori pemasukan.

**Request Headers**

| Header | Nilai |
|---|---|
| Accept | application/json |
| Authorization | Bearer \<TOKEN\> |

**Request Parameters**

| Parameter | Keterangan |
|---|---|
| month | opsional, bilangan bulat, antara 1 dan 12 |
| year | opsional, bilangan bulat |

**E2a: Response Sukses**

| | |
|---|---|
| Status Code | 200 |
| Body | ```json\n{\n  "status": "success",\n  "message": "Get summary by income category successful",\n  "data": {\n    "summary": [\n      {\n        "category": {\n          "id": 2,\n          "name": "Incoming Transfer",\n          "icon": "⬇",\n          "color": "#00B894",\n          "type": "INCOME",\n          "created_at": "2025-07-28T02:37:46.000000Z",\n          "updated_at": "2025-07-28T02:37:46.000000Z"\n        },\n        "amount": 1050000\n      },\n      ...\n    ]\n  }\n}\n``` |

**E2b: Response Token Tidak Valid**

| | |
|---|---|
| Status Code | 401 |
| Body | ```json\n{\n  "status": "error",\n  "message": "Unauthenticated."\n}\n``` |

---

## DIAGRAM ER (ENTITY RELATIONSHIP)

Diagram ER menampilkan lima tabel yang saling terhubung dalam skema database `money_lover`:

- **users**: id, name, email, password, remember_token, created_at, updated_at
- **wallets**: id, user_id (FK ke users), currency_id (FK ke currencies), name, created_at, updated_at, deleted_at
- **categories**: id, name, icon, type (enum: 'EXPENSE', 'INCOME'), created_at, updated_at
- **transactions**: id, category_id (FK ke categories), wallet_id (FK ke wallets), amount, note, date, created_at, updated_at
- **currencies**: id, name, symbol, code, created_at, updated_at

*(Lihat gambar diagram ER pada dokumen asli untuk representasi visual lengkap dari relasi antar tabel.)*

## FILE MEDIA

- **Postman Collection & Environment** – berisi semua endpoint API yang diperlukan
- **Database Dump (.sql)** – mencakup skema awal dan data contoh
- **Framework Installer** – Laravel

## INSTRUKSI

- Anda tidak diperbolehkan membuat endpoint tambahan.
- Anda dapat menambahkan field tambahan pada response body jika diperlukan, **pastikan semua field yang wajib tetap disertakan dan struktur response tetap benar**.

## STRUKTUR PROYEK & PENGUMPULAN

1. Buat folder root bernama **XX_SERVER_MODULE**, di mana **XX** adalah nomor komputer Anda.
2. Di dalam folder root, susun file-file Anda sebagai berikut:
   - XX_SERVER_MODULE/
     - BACKEND/  &nbsp;&nbsp;# Kode proyek backend Anda
     - db-dump.sql  &nbsp;&nbsp;# Dump basis data yang diekspor
     - db-diagram.pdf  &nbsp;&nbsp;# Diagram ER
3. Kompres (zip) folder root menjadi satu file:
   **XX_SERVER_MODULE_PHASE_1.zip**
4. Kecualikan folder berikut saat melakukan zip:
   - vendor/
   - node_modules/
5. Kumpulkan **XX_SERVER_MODULE_PHASE_1.zip** ke halaman pengumpulan (submission page).

---

*Tanggal: 06.02.2026 | LKS2026_WEBTECH_KISI_KISI_EN | Versi: 1.0 | © webtechindonesia*
*Kisi-kisi LKS 2026 SMA/MA/SMK/MAK/Sederajat*
