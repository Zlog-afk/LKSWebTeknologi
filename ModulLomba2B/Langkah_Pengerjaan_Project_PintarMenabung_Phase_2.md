# Langkah Pengerjaan Project — PintarMenabung (Server Module Phase 2)

> Kisi-kisi: LKS 2026 WebTech — Server Module Phase 2 (Frontend Web Application)
> Stack: React 18 + Vite + Tailwind CSS + shadcn/ui + react-router-dom + chart.js
> Media pendukung: `XX_SERVER_MODULE_PHASE_2_SCAFFOLD.zip`, HTML Template, Utility Scripts, Chart-js example (dari panitia)
> Prasyarat: API Fase 1 sudah berjalan (lihat `Langkah_Pengerjaan_Project_PintarMenabung.md`)

---

## 0. Checklist Media yang Sudah Disiapkan

- [x] `XX_SERVER_MODULE_PHASE_2_SCAFFOLD.zip` — scaffold React siap pakai (routing, auth, dialog, chart sudah terpasang)
- [ ] HTML Template dari panitia — jadikan acuan layout dasar/navbar (media resmi lomba)
- [ ] Utility Scripts dari panitia — cek apakah ada helper currency/date yang wajib dipakai
- [ ] Chart-js example dari panitia — bandingkan dengan implementasi `WalletDetail.jsx`

---

## 1. Persiapan Lingkungan & Setup Awal

1. **Ekstrak** `XX_SERVER_MODULE_PHASE_2_SCAFFOLD.zip` ke folder kerja, atau gunakan sebagai referensi jika ingin scaffold ulang dari HTML Template panitia.
2. **Install dependencies**:
   ```bash
   npm install
   ```
3. **Konfigurasi environment**:
   ```bash
   cp .env.example .env
   ```

   gunakan perintah berikut di commandpromt Windows:
     ```bash
     xcopy .env.example .env
     ```

   Isi `VITE_API_URL` sesuai backend Fase 1 Anda:
   - Lokal: `http://localhost:8000/api`
   - Saat lomba: `http://XX.api-server-module.lksn.id/api` (ganti `XX` dengan nomor komputer)
4. **Pastikan backend Fase 1 sudah jalan** (`php artisan serve`) dan bisa diakses dari browser sebelum lanjut — test dulu via Postman kalau perlu.
5. **Jalankan dev server frontend**:
   ```bash
   npm run dev
   ```
   Buka `http://localhost:5173`.
6. **Cross-check dengan media panitia**: buka HTML Template yang disediakan, bandingkan struktur navbar/layout dasarnya dengan `src/components/Navbar.jsx` dan `src/App.jsx` — sesuaikan bila juri mengharapkan markup/kelas tertentu dari template tersebut.

---

## 2. Pahami Struktur Routing & Auth Flow

| Path | Halaman | Akses |
|---|---|---|
| `/register` | Register | Guest only (redirect ke `/` jika sudah login) |
| `/login` | Login | Guest only (redirect ke `/` jika sudah login) |
| `/` | Overview | Harus login |
| `/wallets/:walletId` | Detail Wallet | Harus login |

Alur autentikasi sudah diimplementasikan di:
- `AuthContext.jsx` — state `user`, `token`, fungsi `register()`, `login()`, `logout()`
- `ProtectedRoute.jsx` / `GuestRoute.jsx` — guard otomatis
- `lib/api.js` — axios interceptor menyisipkan `Authorization: Bearer <token>` dan auto-redirect ke `/login` saat 401

Verifikasi ulang bagian ini dulu sebelum lanjut ke halaman lain, karena semua page bergantung padanya.

---

## 3. Kerjakan Modul Berdasarkan Urutan Dependensi

### Langkah A — Halaman Register & Login

1. Cek `pages/Register.jsx` dan `pages/Login.jsx` — form sudah terhubung ke `AuthContext`.
2. Pastikan **pesan error spesifik per field** tampil benar:
   - Register: error `full_name`/`name`, `email`, `password` dari response 422.
   - Login: response gagal (401) hanya memberi `message` umum ("Username or password incorrect") — bukan per-field, karena API Fase 1 memang begitu.
3. Verifikasi **redirect setelah sukses** → ke `/` (Overview).
4. Verifikasi **document title** berubah ("Register", "Login") — via `useDocumentTitle()`.
5. **Test manual**: coba register dengan email yang sudah dipakai → pastikan error email muncul di bawah field email, bukan alert generik.

### Langkah B — Navbar & Logout

1. Cek `components/Navbar.jsx`.
2. Pastikan tombol **Logout hanya muncul setelah login**.
3. Klik Logout → pastikan token terhapus (`localStorage`) dan redirect balik ke `/login`.

### Langkah C — Halaman Overview (`/`)

1. Cek `pages/Overview.jsx`.
2. **Wallet list**: pastikan `WalletCard.jsx` menampilkan nama & saldo (`balance`) dari `GET /api/wallets`.
3. **Transaksi terbaru**: cek `TransactionList.jsx` —
   - Tanggal tampil human-readable ("Jun 7, 2025") via `formatDate()`.
   - Tanggal **disembunyikan jika sama dengan transaksi sebelumnya** (grouping) — logika `dateKey()`.
   - Icon kategori merah (expense) / hijau (income).
   - Amount diberi prefix `-`/`+` dan format currency (`formatCurrency()`).
   - Note tampil jika ada.
4. **Add Wallet**: klik tombol "+" → `AddWalletDialog` → isi nama & pilih currency → Save → list wallet refresh otomatis.
5. **Add Transaction**: klik tombol "Add Transaction" → `AddTransactionDialog` → isi wallet, kategori, amount, tanggal, note → Save.
6. **Delete Transaction**: double-click item transaksi → `ConfirmDialog` muncul → konfirmasi → transaksi terhapus & balance ter-update.
7. **Test shortcut**: `Alt+W` buka Add Wallet, `Alt+N` buka Add Transaction, `Esc` menutup dialog yang terbuka.

### Langkah D — Halaman Detail Wallet (`/wallets/:walletId`)

1. Cek `pages/WalletDetail.jsx`.
2. **Navigasi**: klik salah satu `WalletCard` di Overview → harus masuk ke halaman ini dengan `walletId` yang benar.
3. **Info wallet**: nama & balance tampil di atas.
4. **Filter transaksi bulan/tahun**:
   - Dropdown bulan (Januari–Desember) & tahun (rentang **2015–2030**, cek `YEAR_OPTIONS`).
   - Ganti bulan/tahun → transaksi ter-refresh sesuai filter.
   - *Catatan*: karena endpoint `GET /transactions` tidak punya filter `wallet_id`, data difilter ulang di client — pastikan performanya masih wajar dengan `per_page` yang cukup besar (default 100 di scaffold).
5. **Doughnut chart ringkasan**:
   - Data diambil dari `GET /reports/summary-by-category/expense` & `/income`.
   - Label harus terlihat **di bagian bawah chart** — cek `plugins.legend.position: 'bottom'` di `chartData` config.
6. **Edit nama wallet**: klik nama wallet → jadi input text → ubah nama → tekan Enter → tersimpan (`PUT /wallets/:walletId`).
7. **Hapus wallet**: klik nama wallet → kosongkan isi form → tekan Enter → dialog konfirmasi muncul → klik "Ok" → wallet terhapus, redirect ke Overview.
8. **Add Transaction dari Detail Wallet**: field wallet pada form harus **otomatis ter-pilih** sesuai wallet aktif (`currentWalletId` prop) — verifikasi ini secara khusus karena beda perilaku dengan form di Overview.
9. **Transfer Between Wallets**: klik "Transfer Money" → wallet sumber otomatis terpilih → pilih wallet tujuan, isi jumlah & tanggal → klik Transfer.
   - **Baca catatan implementasi** di `TransferDialog.jsx` — karena tidak ada endpoint transfer khusus, fitur ini memakai 2× `POST /transactions` (kategori "Outgoing Transfer" & "Incoming Transfer" dari seeder Fase 1). Pastikan kedua kategori itu memang ada di database Anda.
10. **Test shortcut**: `Alt+N` buka Add Transaction, `Alt+T` buka Transfer Money, `Esc` menutup dialog.

---

## 4. Verifikasi Format & Detail Kecil

Cek ulang satu per satu poin ini — biasanya jadi poin penilaian detail:

| Item | Cara Cek |
|---|---|
| Document title per halaman | Lihat tab browser saat pindah halaman |
| Format currency | `formatCurrency(75000, 'IDR')` → harus `"IDR 75.000"` |
| Format tanggal | `formatDate('2025-06-07')` → harus `"Jun 7, 2025"` |
| Grouping tanggal transaksi | Buat 2+ transaksi di tanggal sama → tanggal cuma tampil sekali |
| Warna icon kategori | Expense = merah, Income = hijau |
| Prefix amount | Expense `-`, Income `+` |
| Shortcut keyboard | Test semua 4 kombinasi di Overview & Detail Wallet |
| Redirect guest/protected | Coba akses `/login` saat sudah login, dan `/` saat belum login |

---

## 5. Verifikasi Tidak Ada Endpoint Tambahan

- Cek ulang semua pemanggilan `api.get()` / `api.post()` / `api.put()` / `api.delete()` di seluruh halaman & dialog — pastikan **hanya memanggil 15 endpoint** yang sudah didefinisikan di Fase 1 (tidak ada endpoint baru dibuat di backend untuk kebutuhan frontend).
- Fitur seperti transfer sengaja "disiasati" dengan endpoint yang sudah ada (lihat Langkah D poin 9), bukan dengan membuat endpoint baru.

---

## 6. Testing Menyeluruh (Manual, End-to-End)

Lakukan alur berikut secara berurutan seperti pengguna asli:

1. Register akun baru → otomatis masuk ke Overview.
2. Logout → login kembali dengan akun yang sama.
3. Tambah 2 wallet dengan currency berbeda (misal IDR & USD).
4. Tambah beberapa transaksi (income & expense) di kedua wallet, dengan tanggal yang sengaja sama untuk menguji grouping.
5. Buka Detail Wallet salah satu → cek chart, filter bulan/tahun, edit nama, coba transfer ke wallet lain.
6. Hapus salah satu transaksi (double-click) → pastikan balance ter-update di Overview & Detail Wallet.
7. Terakhir, coba hapus salah satu wallet (kosongkan nama + Enter) → pastikan redirect balik ke Overview dan wallet hilang dari list.

---

## 7. Finalisasi & Pengumpulan

1. **Bersihkan folder sebelum zip**:
   ```bash
   rm -rf node_modules dist
   ```
2. **Susun folder submission**:
   ```
   XX_SERVER_MODULE_PHASE_2/
   ├── public/
   ├── src/
   ├── index.html
   ├── package.json
   ├── vite.config.js
   ├── tailwind.config.js
   ├── postcss.config.js
   └── .env.example      # jangan submit .env asli berisi kredensial
   ```
3. **Kecualikan folder** `vendor/` dan `node_modules/` saat zip (folder `vendor/` biasanya tidak relevan untuk frontend, tapi tetap ikuti instruksi soal).
4. Compress menjadi **`XX_SERVER_MODULE_PHASE_2.zip`**.
5. Submit ke halaman pengumpulan.

---

## Referensi Cepat: Pemetaan Fitur → File

| Fitur | File Utama |
|---|---|
| Routing & guard | `App.jsx`, `routes/ProtectedRoute.jsx`, `routes/GuestRoute.jsx` |
| Auth (register/login/logout) | `context/AuthContext.jsx`, `pages/Register.jsx`, `pages/Login.jsx` |
| Overview | `pages/Overview.jsx` |
| Detail Wallet + chart | `pages/WalletDetail.jsx` |
| List transaksi & grouping tanggal | `components/TransactionList.jsx` |
| Format currency & tanggal | `utils/format.js` |
| Shortcut keyboard | `hooks/useShortcuts.js` |
| Dialog Add Wallet | `components/dialogs/AddWalletDialog.jsx` |
| Dialog Add Transaction | `components/dialogs/AddTransactionDialog.jsx` |
| Dialog Transfer Money | `components/dialogs/TransferDialog.jsx` |
| Dialog konfirmasi hapus | `components/dialogs/ConfirmDialog.jsx` |

---

*Jika ada bagian yang errornya susah dilacak (misalnya chart tidak muncul atau shortcut tidak jalan), share pesan error/screenshot-nya agar bisa ditelusuri bersama.*
