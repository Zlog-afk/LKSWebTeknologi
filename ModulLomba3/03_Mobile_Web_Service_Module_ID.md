# MODUL LAYANAN WEB MOBILE (MOBILE WEB SERVICE MODULE)

## DAFTAR ISI

Modul ini terdiri dari file-file berikut:

1. MOBILE WEB SERVICE MODULE.docx
2. MOBILE WEB SERVICE MODULE.pdf
3. MOBILE WEB SERVICE MODULE MEDIA.zip

## PENDAHULUAN

Klien telah meminta pengembangan sebuah aplikasi web yang cepat dan **mobile-first** bernama **ApaKabar**, sebuah **Platform Media Online**. Tujuan utamanya adalah memberikan pengalaman yang lancar dan mudah diakses, terutama untuk pengguna di perangkat mobile. Performa dan kemudahan akses menjadi prioritas utama, karena target audiens utamanya adalah pengguna mobile.

## GAMBARAN UMUM PROYEK

Tugas Anda adalah membangun sebuah **Aplikasi Web Mobile-first** menggunakan framework JavaScript pilihan Anda, baik **React** atau **Vue**. Sebagai alternatif, Anda juga dapat membangunnya menggunakan **Vanilla JavaScript (Native)** tanpa framework apapun. Sebuah **API yang sudah dibuat sebelumnya (pre-built)** disediakan untuk mendukung pengembangan Anda. Berikut adalah fitur-fitur inti yang harus ada:

- Dioptimalkan sepenuhnya hanya untuk layar mobile
- Halaman Home dengan berita yang dipersonalisasi berdasarkan kategori pilihan
- Halaman Discover dengan pencarian, pagination, dan filter kategori
- Halaman detail artikel yang menampilkan semua atribut dari artikel yang dipilih
- Fitur Bookmark untuk menyimpan artikel
- Mode tampilan Light, Dark, dan System

## TATA LETAK WEB MOBILE (MOBILE WEB LAYOUT)

**Ukuran Layar:** Lebar maksimum 480px (hanya untuk layar mobile).

Tata letak terdiri dari tiga bagian utama:

- **Header Bar** (Atas)
- **Main Content** (Dapat di-scroll)
- **Navigation Bar** (Bawah)

Header bar bersifat tetap (fixed) di bagian atas layar, menampilkan judul halaman yang sedang aktif dan tombol kembali (back button) opsional untuk navigasi.

Main content diposisikan di antara header bar dan navigation bar. Bagian ini harus dapat di-scroll (scrollable) ketika konten melebihi tinggi viewport.

Navigation bar bersifat tetap (fixed) di bagian bawah layar.

## NAVIGATION BAR

Terdapat 4 tombol pada navigation bar:

1. Home
2. Discover
3. Bookmark
4. Settings

## THE DATA API

**API Endpoint** disediakan di: `http://api-mobile-service.lksn.id`

API ini mencakup endpoint utama berikut:

- **Get all posts** (Ambil semua post)
  - `GET /api/v1/posts`
- **Get a post by slug** (Ambil satu post berdasarkan slug)
  - `GET /api/v1/posts/:slug`
- **Get all categories** (Ambil semua kategori)
  - `GET /api/v1/categories`

## QUERYING POSTS (MELAKUKAN QUERY PADA POST)

Endpoint `/api/v1/posts` mendukung beberapa parameter query opsional untuk memfilter dan melakukan pagination pada hasil:

- **page** — angka, minimal: 1, default: 1 (contoh: 5 untuk halaman ke-5)
- **per_page** — angka, minimal: 1, default: 20 (contoh: 50 untuk menampilkan 50 post)
- **order_by** — `latest` atau `popular`, default: `latest`
- **search** — string, default: kosong (`""`) untuk mereset pencarian
- **category** — slug kategori dipisahkan koma (contoh: `slug_1,slug_2`)

## HOME PAGE

Tombol pertama pada navigation bar membuka halaman Home.

### BREAKING NEWS

Bagian ini menyoroti artikel-artikel berita yang mendesak (urgent). Item breaking news ditampilkan dalam daftar yang dapat di-scroll secara horizontal dengan scroll snapping, memungkinkan pengguna untuk menggeser (swipe) antar cerita dengan lancar satu per satu.

### RECOMMENDATION LIST

Rekomendasi ini berupa artikel-artikel berdasarkan preferensi kategori yang dipilih pengguna, dan bertujuan untuk menampilkan konten yang sesuai dengan minat mereka.

## DISCOVER PAGE

Tombol kedua pada navigation bar membuka halaman Discover. Halaman ini memungkinkan pengguna untuk menjelajahi artikel berita.

### SEARCH

Pengguna dapat mencari artikel menggunakan kata kunci. Input pencarian dirancang untuk meningkatkan performa dan mengurangi pemanggilan API yang tidak perlu dengan menggunakan mekanisme **debounce**.

- Aplikasi menunggu hingga pengguna berhenti mengetik selama 0.5–1 detik sebelum mengirim request ke API.
- Artinya, pencarian **tidak** memicu request API pada setiap ketukan tombol (keystroke) atau saat mengklik tombol apapun.

### FILTER BY CATEGORY

Filter kategori memungkinkan pengguna untuk memfilter artikel berita berdasarkan kategori tertentu. Daftar kategori diambil secara dinamis dari API.

### INFINITE SCROLL

Ketika pengguna melakukan scroll hingga ke bawah, halaman web harus memuat data artikel selanjutnya dan menampilkan artikel tambahan kepada pengguna. Ini juga dikenal sebagai infinite scrolling.

Mohon sesuaikan (fine-tune) waktu dan kehalusan (smoothness) dari infinite scrolling ini. Pemuatan konten selanjutnya tidak boleh terlalu lambat, dan juga tidak boleh dimuat terlalu cepat/dini. Dengan kata lain, infinite scroll tidak boleh terasa lag (tersendat), dan tidak boleh memuat data terlalu banyak secara berlebihan. Pastikan pemuatan data sudah benar — artinya tidak boleh ada duplikasi tampilan data yang sama, dan tidak ada data yang terlewat untuk ditampilkan.

## ARTICLE DETAIL PAGE

Halaman Detail Artikel menampilkan konten lengkap dari artikel berita yang dipilih. Halaman ini menampilkan semua informasi relevan, meliputi:

- Nama kategori
- Judul
- Tanggal publikasi
- Nama penulis
- Jumlah kunjungan (visited count)
- Thumbnail atau gambar sampul (cover image)
- Konten lengkap atau isi teks (body text)
- Tags

Di bagian bawah halaman, harus terdapat sebuah bagian yang menampilkan 3 artikel berita terkait dari kategori yang sama dengan artikel saat ini, dan mengecualikan artikel yang sedang dilihat.

## BOOKMARK

Tombol ketiga pada navigation bar membuka tampilan Bookmark. Halaman ini menampilkan daftar artikel yang telah disimpan pengguna untuk dibaca nanti. Pengguna dapat menambah atau menghapus bookmark langsung dari halaman detail artikel atau dari tampilan daftar artikel.

## SETTINGS

Tombol keempat pada navigation bar membuka tampilan Settings.

### THEME MODE

Pengguna dapat memilih di antara tiga opsi tema: **light**, **dark**, dan **system** (mengikuti pengaturan light/dark pada perangkat).

### CATEGORY PREFERENCE

Pengguna dapat memilih atau membatalkan pilihan kategori berita favorit mereka. Preferensi ini akan digunakan untuk mempersonalisasi bagian Recommendation pada halaman Home.

### OFFLINE MODE

Ketika pengguna kehilangan koneksi internet saat menggunakan aplikasi, sebuah halaman offline khusus (`game.html`) akan ditampilkan untuk menjaga pengalaman pengguna. File HTML ini disediakan dalam file media.

## FILE MEDIA

- **Wireframes** – panduan visual untuk referensi layout dan UI
- **JavaScript Frameworks** – React/Vue (dengan setup Vite)
- **File HTML** untuk halaman offline

## INSTRUKSI UNTUK PESERTA

- Anda harus mengonfigurasi API base URL di satu tempat saja, seperti:
  - Menggunakan environment variable:
    - `VITE_API_URL=http://api-mobile-service.lksn.id` (di dalam file `.env`)
  - Atau mendefinisikan sebuah konstanta di dalam kode Anda:
    - `const apiUrl = 'http://api-mobile-service.lksn.id'`
- Mohon konfigurasikan **manifest.json** dan meta tag penting untuk sistem web mobile Android maupun iOS.
- Viewport juga harus dikonfigurasikan agar ramah tampilan (friendly) untuk web mobile.
- Mohon pastikan memiliki aksesibilitas yang baik. Kami akan menilai skor aksesibilitas **Chrome Lighthouse**.
- Mohon juga pastikan membuat kode JavaScript yang mudah dipelihara (easy-to-maintain).

## STRUKTUR PROYEK & PENGUMPULAN

1. Ganti nama folder proyek menjadi: **XX_MOBILE_WEB_SERVICE_MODULE**, di mana **XX** adalah nomor komputer Anda.
2. Jika Anda menggunakan Framework NodeJs, mohon letakkan **versi build** dari proyek Anda ke dalam **XX_MOBILE_WEB_SERVICE_MODULE**, sehingga proyek Anda dapat diakses mulai dari **index.html**.
3. Kompres (zip) folder root/proyek Anda (bukan versi build-nya) menjadi satu file:
   **XX_MOBILE_WEB_SERVICE_MODULE.zip**
   **Kecualikan `node_modules` saat melakukan zip pada proyek (jika ada).**
4. Unggah **XX_MOBILE_WEB_SERVICE_MODULE** dan **XX_MOBILE_WEB_SERVICE_MODULE.zip** ke server FTP di dalam folder **"submission"**.

---

*Tanggal: 06.02.2026 | LKS2026_WEBTECH_KISI_KISI_EN | Versi: 1.0 | © webtechindonesia*
*Kisi-kisi LKS 2026 SMA/MA/SMK/MAK/Sederajat*
