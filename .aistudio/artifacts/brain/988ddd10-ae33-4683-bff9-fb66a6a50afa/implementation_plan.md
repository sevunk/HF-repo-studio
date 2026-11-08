# HF Repo Studio — Time Machine & File Commander

Aplikasi web konsol pengembang berbasis klien untuk menginspeksi riwayat commit repositori Hugging Face, memulihkan file yang tertimpa atau terhapus dalam satu klik, serta menyalin atau memindahkan file berukuran besar lintas repositori secara *Server-Side* (0 GB kuota unduhan lokal).

## User Review & Critical Decisions

> [!IMPORTANT]
> Seluruh fungsi inti dan struktur komponen dari kode referensi yang dilampirkan (`App.tsx.txt`) dipertahankan 100% tanpa penyederhanaan pada logika API Hugging Face, ditambah fitur pelengkap dan kesiapan *deployment* yang telah Anda konfirmasi.

- **Keputusan Terkonfirmasi 1 — Fitur Pelengkap Produktivitas**: Mempertahankan 100% kode dan logika asli (`executeNdjsonCommit`, `fetchCommitFileMap`, `fetchPathsInfoDetailed`, `ensureRepoExists`, `parseHfInput`, *Time Machine*, dan *Repo-to-Repo Mover*), serta menambahkan fitur ekstra berupa tombol **Salin (Copy) Commit ID & Hash SHA-256** dengan indikator umpan balik instan serta tombol **Salin / Ekspor Log Operasi (.txt)** pada panel status *Restore* dan *Mover*.
- **Keputusan Terkonfirmasi 2 — Otomatisasi GitHub Pages**: Menambahkan konfigurasi path relatif (`base: './'`) pada bundel aplikasi sekaligus menyertakan workflow otomatisasi GitHub Actions agar aplikasi langsung ter-deploy ke GitHub Pages setiap kali ada perubahan pada branch utama.
- **Keputusan Terkonfirmasi 3 — Sakelar Bahasa Dwibahasa (`ID | EN`)**: Menambahkan tombol pengalih bahasa **`ID | EN`** di bagian header (tersimpan otomatis di `localStorage`) yang menerjemahkan seluruh teks antarmuka, judul tab, petunjuk formulir, kartu deteksi otomatis (`TERTIMPA` / `OVERWRITTEN`, `DIHAPUS` / `DELETED`), opsi *Collision Guard*, hingga pesan log eksekusi antara Bahasa Indonesia dan Bahasa Inggris secara instan.

---

## 1. Overview & Core Concept

- **What It Does**: Menyediakan pusat kendali terpadu bagi pemilik repositori Hugging Face (Model, Dataset, dan Space) dengan dua kapabilitas utama:
  1. **🕰️ HF Time Machine**: Memindai riwayat commit secara paralel, membandingkan perubahan antar-commit berdasarkan hash SHA-256 penuh dan pointer LFS/Xet, mendeteksi secara otomatis kejadian file **🟡 TERTIMPA** maupun **🔴 DIHAPUS**, serta mengeksekusi pemulihan (*1-Click Restore*) langsung di server Hugging Face.
  2. **🚀 Repo-to-Repo Mover & Renamer**: Memilih file secara visual dari repositori asal, menampilkan pratinjau pemetaan nama atau direktori tujuan secara *real-time*, melindungi file tujuan dengan *Collision Guard* (`STOP_BATALKAN`, `AUTO_RENAME`, `OVERWRITE_TIMPA`), menduplikasi pointer LFS/Xet tanpa mengunduh file besar, serta memverifikasi keberadaan seluruh file di repositori tujuan sebelum menghapus file asal pada mode *Move / Cut*.
- **Target Audience / Persona**: *Machine Learning Engineer*, peneliti AI, dan kurator model/dataset di Hugging Face yang mengelola model berukuran puluhan gigabyte (`.safetensors`, `.gguf`, `.bin`, `.parquet`) dan membutuhkan manajemen file serta pemulihan versi yang aman tanpa menguras kuota *bandwidth* lokal.
- **Key Value**: Mengeliminasi proses *download-upload* manual bergigabyte dengan memanfaatkan endpoint duplikasi pointer LFS/Xet dan transaksi *Atomic Commit* (`application/x-ndjson`), dilengkapi pengaman anti-timpa dan verifikasi dua tahap.

---

## 2. User Experience & Visual Design

- **Key User Flows**:
  1. **Autentikasi Terpadu & Pemuatan Repositori**: Pengguna memasukkan `HF_TOKEN` (dengan opsi visibilitas karakter dan penyimpanan opsional di `localStorage`). Sistem memanggil `/api/whoami-v2` dan secara paralel memuat daftar seluruh repositori *model*, *dataset*, dan *space* milik pengguna ke dalam *autocomplete datalist* serta *quick-select dropdown*.
  2. **Alur Inspeksi & Penyelamatan File (Time Machine)**:
     - Pengguna memilih repositori atau menempelkan URL Hugging Face, menentukan branch dan jumlah commit, lalu menekan tombol **Inspeksi Riwayat Commit**.
     - Sistem menampilkan kartu **Rekomendasi Penyelamatan Otomatis** di bagian atas jika mendeteksi file yang tertimpa atau terhapus antar-commit, lengkap dengan ukuran byte dan potongan hash versi utuh sebelumnya.
     - Pengguna menekan tombol **Siapkan Restore Versi Ini** atau **Pulihkan Ini** pada tabel file commit untuk mengisi formulir *Panel Eksekusi Pemulihan* secara otomatis, lalu menjalankan pemulihan *Server-Side* dan mengekspor log hasilnya bila diperlukan.
  3. **Alur Transfer & Ganti Nama Lintas Repositori (Mover & Renamer)**:
     - Pengguna memuat daftar file dari repositori asal, menyaring dengan kotak pencarian instan, dan mencentang file yang diinginkan.
     - Panel kanan menampilkan **Pratinjau Pemetaan** (`src ---> dst`) secara langsung saat pengguna mengetik nama baru atau folder tujuan.
     - Pengguna memilih mode *Collision Guard* dan opsi *Move / Cut*, lalu mengeksekusi transfer. Sistem menyalin file, memverifikasi keberadaan 100% file di repositori tujuan, baru kemudian menghapus file asal jika opsi *Move* diaktifkan.

- **Visual Identity & Theme**:
  - *Aesthetic Direction*: Konsol utilitas pengembang (*Developer Utility Console*) berdensitas tinggi yang bersih, terstruktur, dan kontras tinggi, dilengkapi peralihan tema **Dark Mode** (default) dan **Light Mode** serta pengalih bahasa **`ID | EN`** secara instan di bilah navigasi atas.
  - *Color Palette & Mood*:
    - Kanvas Utama (60%): Slate gelap pekat (`bg-slate-950`, teks `text-slate-100`) pada mode gelap, dan putih bersih (`bg-slate-50`, teks `text-slate-900`) pada mode terang.
    - Permukaan Struktural (30%): Kartu panel `bg-slate-900` / `bg-white` dengan garis batas halus `border-slate-800` / `border-slate-200`.
    - Aksen & Semantik (10%): Amber (`amber-500` / `amber-400`) untuk aksi utama dan peringatan file tertimpa (`🟡 TERTIMPA`), Emerald (`emerald-500` / `emerald-400`) untuk indikator terhubung, status HEAD, dan eksekusi pemulihan, Indigo (`indigo-500` / `indigo-400`) untuk penanda pointer LFS dan pemilih file sumber, serta Crimson (`red-500` / `red-400`) untuk penanda file terhapus (`🔴 DIHAPUS`).
  - *Typography & Hierarchy*: Kombinasi *sans-serif* modern (`Plus Jakarta Sans`) untuk keterbacaan antarmuka dan navigasi, dipadukan dengan *monospace* (`JetBrains Mono` / `font-mono` dengan `tabular-nums`) untuk seluruh Commit ID, SHA-256 hash, ukuran byte, path file, dan log terminal.
  - *Component Styling & Layout*: Tata letak responsif dengan *sticky header* tiga zona, panel token terpadu di bagian atas, serta tata letak *split-view* proporsional (2/3 timeline commit di kiri + 1/3 panel aksi *sticky* di kanan pada *Time Machine*; 2 kolom sejajar *Source* dan *Target* pada *Mover*).
- **Interactive Feedback & Motion**: Indikator putar (*spinner*) pada tombol saat memindai API, *smooth scroll* otomatis ke panel eksekusi saat memilih file untuk dipulihkan, tombol salin cepat (*copy-to-clipboard*) dengan ikon centang konfirmasi selama 2 detik, serta log progres *real-time* baris demi baris.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1: Transfer Server-Side via LFS/Xet Duplicate & NDJSON Atomic Commit**
  - *Chosen Approach*: Memisahkan penanganan file besar (LFS/Xet) dan file teks kecil non-LFS. File LFS didaftarkan melalui endpoint `/lfs-files/duplicate` (lintas repositori) dan dikirim sebagai pointer `lfsFile` di dalam payload `application/x-ndjson`, sedangkan file kecil non-LFS diambil via `/resolve/` dan di-encode ke Base64 (`file`).
  - *Why*: Memungkinkan pemindahan model puluhan gigabyte dalam hitungan detik tanpa mengunduh bobot model ke peramban pengguna.
  - *Alternatives Considered*: Mengunduh dan mengunggah ulang seluruh blob melalui peramban ditolak karena lambat, memakan kuota *bandwidth* besar, dan berisiko gagal akibat batas memori peramban.

- **Decision 2: Verifikasi Pasca-Komit Sebelum Penghapusan Asal (Two-Phase Safe Move)**
  - *Chosen Approach*: Saat mode *Move / Cut* (`mvDeleteSource`) aktif, sistem terlebih dahulu mengeksekusi komit penyalinan ke repositori tujuan, lalu memanggil `fetchCommitFileMap` ulang pada repositori tujuan untuk memastikan seluruh file benar-benar terdaftar di pohon komit tujuan sebelum mengeksekusi komit `deletedFile` pada repositori asal.
  - *Why*: Menjamin keamanan data 100% agar tidak ada risiko kehilangan file apabila komit di repositori tujuan mengalami kendala parsial.
  - *Alternatives Considered*: Menghapus file asal secara bersamaan tanpa verifikasi pohon tujuan ditolak karena berisiko merusak repositori asal bila terjadi kegagalan jaringan.

- **Decision 3: Fitur Ekstra Non-Intrusif (Copy Hash, Export Log & Sakelar Bahasa `ID | EN`)**
  - *Chosen Approach*: Menambahkan tombol salin (*copy*) pada badge Commit ID dan kolom Hash di tabel file, tombol **Salin Log** dan **Unduh Log (.txt)** pada panel status *Time Machine* dan *Repo-to-Repo Mover*, serta kamus lokalisasi terpusat (`ID` & `EN`) dengan tombol segmented **`ID | EN`** di header.
  - *Why*: Memudahkan pengguna lokal maupun internasional memahami setiap opsi pengaman (*Collision Guard*, *Restore Conflict Mode*) serta mendokumentasikan riwayat penyelamatan atau menyalin SHA-256 hash secara presisi tanpa mengubah alur kerja utama dari kode referensi.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

- **Architecture & Component Diagram**:

```
┌────────────────────────────────────────────────────────────────────────────┐
│                   HF Repo Studio (Client-Side React SPA)                   │
├────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Header & Unified Auth Bar                                            │  │
│  │ • Token Input (Show/Hide, localStorage Sync, Logout/Clear)           │  │
│  │ • GET /api/whoami-v2 + Parallel Fetch (models, datasets, spaces)     │  │
│  │ • Autocomplete <datalist> & Quick-Select Repo Dropdowns              │  │
│  └──────────────────────────────────┬───────────────────────────────────┘  │
│                                     │                                      │
│            ┌────────────────────────┴────────────────────────┐             │
│            ▼                                                 ▼             │
│  ┌───────────────────────────────────┐   ┌──────────────────────────────┐  │
│  │ Tab 1: 🕰️ HF Time Machine         │   │ Tab 2: 🚀 Repo-to-Repo Mover │  │
│  ├───────────────────────────────────┤   ├──────────────────────────────┤  │
│  │ • Commit Inspector Form           │   │ • Source Repo File Loader    │  │
│  │ • Parallel Snapshot Fetcher       │   │ • Search & Multi-Checkbox UI │  │
│  │ • Automatic Diff & Rescue Engine  │   │ • Live Path Mapping Preview  │  │
│  │   (🟢 Added, 🟡 Overwritten,      │   │ • Target Repo Auto-Creator   │  │
│  │    🔴 Deleted + 1-Click Rescue)   │   │ • Anti-Overwrite Guard       │  │
│  │ • Commit Timeline & File Table    │   │   (STOP / AUTO_RENAME /      │  │
│  │   (+ Copy Commit ID & SHA Hash)   │   │    OVERWRITE)                │  │
│  │ • Sticky Restore Execution Panel  │   │ • Post-Copy Verification &   │  │
│  │   (+ Copy/Download Restore Logs)  │   │   Source Deletion (Move/Cut) │  │
│  └─────────────────┬─────────────────┘   └──────────────┬───────────────┘  │
│                    └────────────────┬───────────────────┘                  │
│                                     ▼                                      │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │ Core Hugging Face Hub REST API Engine                                │  │
│  │ • parseHfInput()          : URL / hf:// / repo_id Normalizer         │  │
│  │ • fetchCommitFileMap()    : GET /api/{type}s/{id}/tree/{rev}         │  │
│  │ • fetchPathsInfoDetailed(): POST /api/{type}s/{id}/paths-info/{rev}  │  │
│  │ • ensureRepoExists()      : GET Check + POST /api/repos/create       │  │
│  │ • executeNdjsonCommit()   : POST /lfs-files/duplicate +              │  │
│  │                             POST /commit/{rev} (x-ndjson)            │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────┘
```

- **Data Model & State**:
  - `HfUserInfo` & `HfRepoSummary[]`: Menyimpan profil pengguna aktif dari `whoami-v2` dan daftar seluruh repositori milik pengguna (`id`, `type`, `private`).
  - `HfFileItem`: Representasi metadata file pada suatu commit (`path`, `size`, `hash` 10 karakter, `fullHash` SHA-256/OID penuh, `isLfs`, `lfsOid`, `lfsSize`, `xetHash`).
  - `HfCommitInfo` & `RescueHint`: Menyimpan metadata commit (termasuk konversi waktu otomatis ke `Asia/Jakarta` WIB), peta file per commit, kalkulasi diff terhadap commit sebelumnya (`added`, `deleted`, `overwritten`), serta daftar rekomendasi penyelamatan otomatis.
- **Interactive Component & State Mapping**:
  - **Validasi & Normalisasi Input (`parseHfInput`)**: Menerima input berupa URL lengkap (`https://huggingface.co/datasets/user/repo/resolve/main/data.parquet`), URI `hf://`, maupun `username/repo`, lalu mengekstrak `repoId`, `repoType`, `revision`, dan `subpath` secara otomatis ke dalam state formulir.
  - **Deteksi Diff & 1-Click Pre-fill (`handleInspectCommits` & `handleSelectForRestore`)**: Membandingkan `fullHash` setiap file antara commit `[idx]` dan `[idx + 1]`. Klik pada tombol pemulihan langsung mengisi `restoreCommitId` dan `restoreSrcPath`, lalu menggulir layar ke `#restore-action-panel`.
  - **Kalkulasi Pratinjau Pemetaan (`plannedMoveMappings`)**: `useMemo` reaktif yang menghitung pasangan `{ src, dst }` secara instan berdasarkan daftar checkbox terpilih (`mvSelectedPaths`), input subpath/wildcard (`*`), dan spesifikasi nama/folder baru (`mvDstNewName`), termasuk mempertahankan ekstensi file asli secara cerdas.
  - **Utilitas Salin & Ekspor Log (`copyToClipboard` & `downloadLogsAsTxt`)**: Menyalin hash/commit ID atau seluruh baris log eksekusi ke *clipboard* dengan indikator status visual, serta mengunduh riwayat log sebagai file `.txt` bernama kontekstual.
  - **Sistem Dwibahasa Reaktif (`lang: 'id' | 'en'`)**: Menyimpan preferensi bahasa di `localStorage` (`hf_studio_lang`), menyediakan tombol pengalih **`ID | EN`** di sebelah tombol tema pada header, dan memetakan seluruh teks UI, placeholder, label tombol, pesan error, serta log progres ke bahasa yang dipilih pengguna.
