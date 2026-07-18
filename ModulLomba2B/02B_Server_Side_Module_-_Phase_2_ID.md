# MODUL SISI SERVER

## DAFTAR ISI

Modul ini terdiri dari file-file berikut:

1. SERVER MODULE PHASE 2.docx
2. SERVER MODULE PHASE 2.pdf
3. SERVER MODULE PHASE 2 MEDIA.zip

## PENDAHULUAN

Klien telah meminta Anda untuk membangun sistem backend dan aplikasi web frontend untuk sebuah **Aplikasi Manajemen Keuangan** bernama **PintarMenabung**. Sistem ini harus memungkinkan pengguna untuk mendaftar, mengelola dompet (wallet) mereka, mencatat pemasukan dan pengeluaran, serta melihat laporan keuangan dasar.

## Konfigurasi API

**API URL:** `http://XX.api-server-module.lksn.id/` *(ganti XX dengan nomor komputer Anda)*

## GAMBARAN UMUM PROYEK

Proyek ini terdiri dari dua fase utama: Pada Fase 1, Anda diharuskan mengembangkan sebuah **REST API** menggunakan **Framework Laravel**, dengan mengimplementasikan fitur-fitur inti seperti autentikasi pengguna, manajemen dompet, pencatatan transaksi, dan pelaporan keuangan. Pada Fase 2, Anda akan membangun sebuah **aplikasi web frontend** yang terintegrasi dengan API, menggunakan **React** atau **Vue** sebagai framework pilihan Anda.

Untuk mendukung pekerjaan Anda, telah disediakan file media berupa Postman Collection dan Environment, Dump Basis Data (.sql), dan Framework Scaffolding.

---

# FASE 2 - Aplikasi Web Frontend

## Halaman & Fitur

| Halaman | Fitur |
|---|---|
| **General (Umum)** | • Judul dokumen (document title) harus mencerminkan halaman yang sedang aktif (contoh: "Login", "Register", "Overview").<br>• Setelah berhasil login, tampilkan tombol Logout di navbar.<br>• Pengguna dapat logout dengan mengklik tombol Logout di navbar. |
| **Register** | **Path:** `/register`<br><br>• Pengguna dapat mendaftar menggunakan field berikut:<br>&nbsp;&nbsp;○ Full Name (Nama Lengkap)<br>&nbsp;&nbsp;○ Email<br>&nbsp;&nbsp;○ Password<br>• Tampilkan pesan error yang spesifik berdasarkan field input yang terkait ketika registrasi gagal.<br>• Setelah registrasi berhasil, pengguna diarahkan (redirect) ke halaman Overview.<br>• Jika pengguna yang sudah login mengakses halaman ini, mereka akan otomatis diarahkan ke halaman Overview. |
| **Login** | **Path:** `/login`<br><br>• Pengguna dapat login menggunakan kredensial yang sudah ada (email dan password).<br>• Tampilkan pesan error yang spesifik ketika login gagal, berdasarkan field input yang terkait.<br>• Setelah login berhasil, pengguna diarahkan ke halaman Overview.<br>• Jika pengguna yang sudah login mengakses halaman ini, mereka akan otomatis diarahkan ke halaman Overview. |
| **Overview Page** | **Path:** `/`<br>**Akses:** Hanya dapat diakses oleh pengguna yang sudah login<br><br>**Informasi yang Ditampilkan**<br>• **User Wallets**<br>&nbsp;&nbsp;○ Tampilkan nama dan saldo (balance) terkini setiap wallet<br><br>• **Recent Transactions** (dari semua wallet)<br>&nbsp;&nbsp;○ **Transaction Date**<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Format: mudah dibaca manusia / human-readable (contoh: "Jun 7, 2025")<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Tanggal hanya ditampilkan **sekali per grup**; sembunyikan jika sama dengan tanggal transaksi sebelumnya.<br>&nbsp;&nbsp;○ **Category Icon**<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Merah untuk expense (pengeluaran)<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Hijau untuk income (pemasukan)<br>&nbsp;&nbsp;○ **Category Name**<br>&nbsp;&nbsp;○ **Wallet Name**<br>&nbsp;&nbsp;○ **Amount**<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Diberi awalan `-` untuk expense<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Diberi awalan `+` untuk income<br>&nbsp;&nbsp;&nbsp;&nbsp;▪ Ditampilkan dalam format mata uang (contoh: "IDR 75.000" atau "USD 12")<br>&nbsp;&nbsp;○ **Note** (jika ada)<br><br>**Actions (Aksi)**<br>• **Add Wallet**<br>&nbsp;&nbsp;○ Klik tombol "+" di bawah bagian Balance<br>&nbsp;&nbsp;○ Isi form wallet lalu klik "Save"<br>• **Add Transaction**<br>&nbsp;&nbsp;○ Klik tombol "Add Transaction"<br>&nbsp;&nbsp;○ Isi form transaksi lalu klik "Save"<br>• **Delete Transaction**<br>&nbsp;&nbsp;○ Klik dua kali (double-click) pada item transaksi<br>&nbsp;&nbsp;○ Konfirmasi penghapusan pada dialog popup |
| **Detail Wallet** | **Path:** `/wallets/:walletId`<br>**Akses:** Hanya dapat diakses oleh pengguna yang sudah login<br><br>**Navigation (Navigasi)**<br>• Pengguna dapat mengakses halaman ini dengan mengklik item wallet pada halaman Overview.<br><br>**Informasi yang Ditampilkan**<br>• **Wallet Information**<br>&nbsp;&nbsp;○ Tampilkan nama wallet dan saldo (balance) terkini.<br>• **Transactions**<br>&nbsp;&nbsp;○ Tampilkan transaksi wallet berdasarkan bulan dan tahun yang dipilih.<br>&nbsp;&nbsp;○ Rentang pilihan tahun: dari 2015 hingga 2030.<br>&nbsp;&nbsp;○ Mengacu pada halaman Overview untuk detail/atribut transaksi.<br>• **Summary of expense and income per month**<br>&nbsp;&nbsp;○ Ditampilkan dalam bentuk doughnut chart<br>&nbsp;&nbsp;○ Data harus diambil (fetch) dari API<br>&nbsp;&nbsp;○ Label harus terlihat di bagian bawah (bottom)<br><br>**Actions (Aksi)**<br>• **Edit Wallet Name**<br>&nbsp;&nbsp;○ Klik Nama Wallet.<br>&nbsp;&nbsp;○ Ganti nama wallet.<br>&nbsp;&nbsp;○ Tekan "Enter" untuk menyimpan.<br>• **Delete Wallet**<br>&nbsp;&nbsp;○ Klik Nama Wallet.<br>&nbsp;&nbsp;○ Kosongkan form.<br>&nbsp;&nbsp;○ Tekan "Enter" untuk menampilkan dialog konfirmasi lalu klik "Ok" untuk menghapus.<br>• **Add Transaction**<br>&nbsp;&nbsp;○ Klik tombol "Add Transaction".<br>&nbsp;&nbsp;○ Field wallet pada form harus otomatis terpilih (auto-selected) berdasarkan wallet yang sedang aktif.<br>&nbsp;&nbsp;○ Isi form lalu klik "Save".<br>• **Delete Transaction**<br>&nbsp;&nbsp;○ Klik dua kali (double-click) pada item transaksi<br>&nbsp;&nbsp;○ Konfirmasi penghapusan pada dialog popup<br>• **Transfer Between Wallets**<br>&nbsp;&nbsp;○ Klik tombol "Transfer Money".<br>&nbsp;&nbsp;○ Wallet sumber (source wallet) harus otomatis terpilih berdasarkan wallet yang sedang aktif.<br>&nbsp;&nbsp;○ Isi form lalu klik "Transfer". |

## Shortcuts (Pintasan Keyboard)

| Keymap | Description (Deskripsi) |
|---|---|
| `Alt + W` | Membuka form "Add Wallet" |
| `Alt + N` | Membuka form "Add Transaction" |
| `Alt + T` | Membuka form "Transfer Money" |
| `Esc` | Menutup form |

## FILE MEDIA

- **HTML Template** – digunakan sebagai layout dasar untuk frontend.
- **Utility Scripts** – fungsi bantuan (helper) untuk mata uang, format tanggal, dsb.
- **Contoh kode Chart-js** untuk React dan Vue.

## INSTRUKSI

- Anda tidak diperbolehkan membuat endpoint tambahan.
- Anda dapat menambahkan field tambahan pada response body jika diperlukan, **pastikan semua field yang wajib tetap disertakan dan struktur response tetap benar**.

## STRUKTUR PROYEK & PENGUMPULAN

1. Buat folder root bernama **XX_SERVER_MODULE_PHASE_2**, di mana **XX** adalah nomor komputer Anda.
2. Di dalam folder root, susun file-file Anda sebagai berikut:
   - *XX_SERVER_MODULE_PHASE_2/*
     - *public/*
     - *src/*
     - *…*
     - *package.json*
3. Kompres (zip) folder root menjadi satu file:
   **XX_SERVER_MODULE_PHASE_2.zip**
4. Kecualikan folder berikut saat melakukan zip:
   - `vendor/`
   - `node_modules/`
5. Kumpulkan **XX_SERVER_MODULE_PHASE_2.zip** ke halaman pengumpulan (submission page).

---

*Tanggal: 06.02.2026 | LKS2026_WEBTECH_KISI_KISI_EN | Versi: 1.0 | © webtechindonesia*
*Kisi-kisi LKS 2026 SMA/MA/SMK/MAK/Sederajat*
