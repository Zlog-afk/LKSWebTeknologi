# LANGKAH PENGERJAAN — MOBILE WEB SERVICE MODULE

Panduan ini disusun berdasarkan `03_Mobile_Web_Service_Module_ID.md` dan paket
`MOBILE_WEB_SERVICE_MODULE_MEDIA.zip`. Ikuti urutan fase di bawah — setiap fase
menyebutkan file media mana yang dipakai sebagai referensi.

---

## 0 — Persiapan (± 15 menit)

1. Ekstrak `MOBILE_WEB_SERVICE_MODULE_MEDIA.zip`, cek isinya:
   `wireframes/`, `starter-react-vite/`, `starter-vue-vite/`, `offline/game.html`.
2. Buka kelima file di `wireframes/` — ini acuan tata letak wajib:
   Header fixed atas → Main content scrollable → Nav bar fixed bawah (4 tombol:
   Home, Discover, Bookmark, Settings). Lebar maksimum 480px.
3. **Putuskan framework**: React, Vue, atau Vanilla JS. Kalau pilih React/Vue,
   salin folder starter yang sesuai (`starter-react-vite` atau `starter-vue-vite`)
   ke folder kerja dan ganti namanya menjadi `XX_MOBILE_WEB_SERVICE_MODULE`
   (XX = nomor komputer Anda).
4. Jalankan starter untuk memastikan environment sudah benar:
   ```bash
   npm install
   npm run dev
   ```
5. Cek `.env` — pastikan `VITE_API_URL=http://api-mobile-service.lksn.id` sudah
   benar dan **hanya** dikonfigurasi di satu tempat itu (dibaca oleh
   `src/api/client.js`).

---

## 1 — Verifikasi API (± 15 menit)

Sebelum membangun UI, pastikan Anda paham bentuk data sebenarnya dari API
(field starter memakai asumsi seperti `slug`, `category`, `author`,
`published_at`, `visited_count`, `tags` — **cocokkan dengan response asli**,
sesuaikan `src/api/client.js` dan semua page bila nama field berbeda):

1. `GET /api/v1/categories` → cek struktur tiap kategori (nama, slug).
2. `GET /api/v1/posts` → cek struktur tiap post & bentuk pagination
   (apakah ada `meta.has_more`, `total_pages`, dsb — sesuaikan logika
   infinite scroll di `Discover` dengan bentuk response yang sebenarnya).
3. `GET /api/v1/posts/:slug` → cek field lengkap yang tersedia untuk halaman
   detail (pastikan semua atribut yang diminta modul tersedia: kategori,
   judul, tanggal, penulis, jumlah kunjungan, cover image, body, tags).

---

## 2 — Shell Aplikasi & Navigasi (± 30 menit)

Acuan: seluruh file di `wireframes/` (struktur header/content/nav yang sama
di semua halaman).

1. Pastikan `App.jsx`/`App.vue` sudah punya 4 tab (Home, Discover, Bookmark,
   Settings) + state untuk membuka halaman Article Detail.
2. Rapikan `Header` (judul halaman + tombol back opsional) dan `BottomNav`
   (4 tombol, indikator aktif) — sudah ada di starter, sesuaikan gaya visual.
3. Konfirmasi CSS layout (`src/styles/index.css`): header fixed top, nav fixed
   bottom, content di antaranya `overflow-y: auto`, max-width 480px.
4. Jalankan `npm run dev`, coba pindah antar 4 tab — pastikan tidak ada layout
   yang geser/patah di layar mobile (gunakan device toolbar di DevTools,
   pilih ukuran mobile).

---

## 3 — Halaman Home (± 45 menit)

Acuan: `wireframes/01_home.svg`.

1. **Breaking News**: hubungkan ke `GET /api/v1/posts` (mis. `order_by=popular`
   atau kriteria "urgent" sesuai API asli). Terapkan horizontal scroll dengan
   `scroll-snap-type: x mandatory` pada wrapper + `scroll-snap-align: start`
   pada tiap card (kelas `.breaking-news-scroller` / `.breaking-card` sudah
   disiapkan di CSS starter).
2. **Recommendation**: hubungkan ke `GET /api/v1/posts?category=...` memakai
   kategori pilihan pengguna. Kategori ini disimpan di halaman Settings
   (lihat Fase 6) — baca dari `localStorage` (`apakabar_category_preference`
   di starter) saat halaman Home dimuat.
3. Tap salah satu card → harus membuka Article Detail (`onOpenArticle`/
   `$emit('open-article', slug)` sudah tersambung di starter).

---

## 4 — Halaman Discover (± 60 menit)

Acuan: `wireframes/02_discover.svg`. Ini bagian paling teknis dari modul.

1. **Search dengan debounce**: input sudah terhubung ke `useDebounce` (React)
   / `useDebounce` composable (Vue) dengan delay 600ms. Uji: ketik cepat →
   pastikan network tab hanya menunjukkan **satu** request setelah berhenti
   mengetik 0.5–1 detik, bukan satu request per huruf.
2. **Filter kategori**: render chip dari `GET /api/v1/categories`. Multi-select
   sudah disiapkan (`selectedCategories`), dikirim sebagai
   `category=slug1,slug2` sesuai spesifikasi query API.
3. **Infinite scroll**:
   - Starter memakai `IntersectionObserver` pada elemen sentinel di akhir
     list — cek `rootMargin: '200px'` supaya tidak terasa lag (mulai load
     sebelum benar-benar mentok bawah) dan tidak juga terlalu dini.
   - **Wajib uji tidak ada duplikasi/kehilangan data**: reset `page=1` &
     kosongkan list setiap kali `search`/`category` berubah (sudah ada di
     starter) — jangan sampai halaman lama tercampur dengan hasil filter baru.
   - Pastikan tidak ada request infinite-scroll yang terpicu dobel saat
     `loading` sedang `true` (sudah dijaga di starter, jangan dihapus).
4. Fine-tune kecepatan/threshold sesuai feeling di device asli (bukan hanya
   emulator), karena ini yang dinilai ("tidak boleh lag, tidak boleh terlalu
   dini/berlebihan").

---

## 5 — Article Detail & Bookmark (± 45 menit)

Acuan: `wireframes/03_article_detail.svg` dan `wireframes/04_bookmark.svg`.

1. Tampilkan semua atribut wajib: kategori, judul, tanggal publikasi, penulis,
   jumlah kunjungan, cover image, isi lengkap, tags — semua sudah di-mapping
   di starter, tinggal disesuaikan dengan nama field API asli.
2. **Artikel terkait**: ambil `GET /api/v1/posts?category=<slug artikel ini>`,
   filter agar **mengecualikan artikel yang sedang dibuka**, ambil 3 pertama
   (logika ini sudah ada di starter — cek ulang bila field kategori API asli
   berbeda strukturnya).
3. **Bookmark**:
   - Toggle bookmark dari tombol ♥ di halaman detail (sudah ada).
   - Data bookmark disimpan di `localStorage` (`apakabar_bookmarks` di
     starter) — pastikan persist setelah reload halaman.
   - Halaman Bookmark menampilkan daftar tersimpan + tombol hapus (✕) —
     pastikan hapus juga bisa dilakukan langsung dari sini, bukan hanya dari
     detail artikel (sesuai instruksi modul).

---

## 6 — Settings (± 30 menit)

Acuan: `wireframes/05_settings.svg`.

1. **Theme Mode** (light/dark/system): `useTheme` sudah menerapkan
   `data-theme` ke `<html>` dan mendengarkan `prefers-color-scheme` saat mode
   `system` dipilih. Uji ketiga mode, termasuk ganti tema OS saat mode
   `system` aktif (harus otomatis berubah tanpa reload).
2. **Category Preference**: checkbox kategori tersimpan di `localStorage`
   (dipakai oleh Recommendation di Home — lihat Fase 3). Uji: ubah preferensi
   → balik ke Home → pastikan Recommendation berubah sesuai.
3. **Offline Mode**: lihat Fase 7.

---

## 7 — Offline Mode (± 20 menit)

Acuan: `offline/game.html`.

1. Salin `game.html` ke folder public/static project Anda (mis. `public/offline/game.html`
   di React/Vue-Vite, agar ikut ter-build apa adanya ke `dist/`).
2. Lengkapi bagian yang ditandai TODO di `main.jsx`/`main.js` atau `App.jsx`/`App.vue`:
   ketika event `offline` terpicu, arahkan pengguna ke halaman ini (redirect
   penuh, atau tampilkan via `<iframe src="/offline/game.html">` — pilih salah
   satu, starter sudah punya state `isOffline` sebagai titik awal).
3. Uji dengan mematikan network di DevTools (Network → Offline) — pastikan
   transisi ke halaman offline mulus, dan kembali normal saat online lagi.

---

## 8 — PWA, Meta Tag, & Aksesibilitas (± 30 menit)

1. **manifest.json**: sudah disiapkan di `public/manifest.json` kedua starter
   — ganti ikon placeholder di `public/icons/` dengan ikon final (192x192 &
   512x512 minimal).
2. **Meta tag mobile**: viewport, `theme-color`, `apple-mobile-web-app-*`
   sudah ada di `index.html` — jangan dihapus, sesuaikan warna/nama sesuai
   desain final Anda.
3. **Aksesibilitas** (dinilai pakai Chrome Lighthouse):
   - Pastikan semua tombol punya label yang jelas (`aria-label` sudah ada di
     beberapa tempat seperti back button, remove bookmark — lengkapi yang
     lain).
   - Kontras warna teks vs background cukup, terutama di dark mode.
   - Semua elemen interaktif bisa diakses dengan keyboard (fokus terlihat).
   - Jalankan Lighthouse (DevTools → Lighthouse → Accessibility) dan
     perbaiki temuan sebelum submit.

---

## 9 — Review Kode & Maintainability (± 15 menit)

1. Pastikan **tidak ada URL API hardcode** di luar `src/api/client.js` — cek
   dengan `grep -r "api-mobile-service" src/` (harus hanya muncul di `.env`).
2. Hapus semua `console.log` sisa debugging.
3. Pastikan komponen/halaman tidak terlalu panjang & terpisah rapi per
   tanggung jawab (sudah mengikuti pola starter: `components/`, `pages/`,
   `hooks/` atau `composables/`, `api/`).

---

## 10 — Build & Pengumpulan

1. `npm run build` → hasil ada di `dist/`.
2. Salin isi `dist/` ke folder `XX_MOBILE_WEB_SERVICE_MODULE` (folder ini
   harus bisa diakses mulai dari `index.html`).
3. Zip **folder proyek/root (bukan hasil build)** menjadi
   `XX_MOBILE_WEB_SERVICE_MODULE.zip`, **kecualikan `node_modules`**.
4. Upload `XX_MOBILE_WEB_SERVICE_MODULE` (versi build) dan
   `XX_MOBILE_WEB_SERVICE_MODULE.zip` (source project) ke folder
   `submission` di server FTP.

---

## Checklist Cepat Sebelum Submit

- [ ] Home: Breaking News (horizontal scroll-snap) + Recommendation (sesuai kategori pilihan)
- [ ] Discover: search debounce, filter kategori, infinite scroll halus tanpa duplikasi/kehilangan data
- [ ] Article Detail: semua atribut lengkap + 3 artikel terkait (exclude current)
- [ ] Bookmark: tambah/hapus dari detail maupun dari daftar list
- [ ] Settings: theme light/dark/system, category preference, catatan offline mode
- [ ] Offline: `game.html` tampil saat koneksi hilang
- [ ] manifest.json + meta tag mobile (Android & iOS) + viewport benar
- [ ] API base URL dikonfigurasi di satu tempat saja
- [ ] Skor Lighthouse Accessibility diperiksa & diperbaiki
- [ ] Folder & file zip dinamai sesuai format `XX_MOBILE_WEB_SERVICE_MODULE`
