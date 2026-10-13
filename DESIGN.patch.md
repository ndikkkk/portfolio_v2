# DESIGN.patch.md: perubahan untuk DESIGN.md

Cara pakai: tiap blok berlabel GANTI menimpa bagian bernomor yang sama di `DESIGN.md`. Blok TAMBAH disisipkan di tempat yang disebut. Bagian yang tidak disebut tidak berubah.

---

## [GANTI] Bagian 1, dua baris tabel, ditambah satu baris Lenis, dan paragraf alasan

Ganti baris GSAP dan tambahkan baris Lenis tepat di bawahnya:

| Referensi | Peran di proyek | Status |
|---|---|---|
| GSAP (gsap.com) | Animasi berbasis scroll (ScrollTrigger: parallax, pin, scrub), pemecahan teks (SplitText), timeline, ticker yang jadi satu-satunya detak (clock) untuk scroll | Dipakai. Gratis untuk penggunaan komersial di bawah lisensi standar GSAP |
| Lenis (paket `lenis`, MIT, darkroom.engineering) | Smooth scroll halaman. Hanya memperhalus gerak scroll dan menyediakan `scrollTo`. Tidak menganimasikan elemen | Dipakai, hanya di tingkat Penuh dan perangkat berkursor halus. Detail di bagian 4.3 |

Ganti paragraf "Alasan memilih Motion dan GSAP saja" dengan:

**Alasan memilih Motion, GSAP, dan Lenis saja:** tiga pustaka runtime adalah batas atas. Lenis tidak bersaing dengan dua lainnya karena tugasnya hanya scroll. Motion dan GSAP tetap tidak boleh menganimasikan `transform` pada elemen yang sama. Aturannya: **satu elemen, satu pemilik animasi.** Pembagiannya ada di `ARCHITECTURE.md` bagian 4.

---

## [GANTI] Bagian 3, butir Hero (butir 2) dan paragraf penutup "Tidak ada layar pemuatan"

Butir 2 menjadi:

2. **Hero**: label HUD kecil ("STATUS: tersedia untuk magang", jika sesuai fakta), judul besar dengan animasi masuk per kata, satu kalimat peran, dua tombol (satu magnetik), dan **potret pemilik** dengan perlakuan di bagian 3.2. Latar: lapis ambient (bagian 5).

Paragraf "Tidak ada layar pemuatan (preloader)..." dihapus dan diganti dengan bagian 3.1.

Butir 5 (Proyek) ditambah satu kalimat: "Di desktop tingkat Penuh, daftar proyek tampil sebagai section horizontal yang di-pin (bagian 4.5). Di HP dan tingkat Hemat atau Mati, daftar yang sama tampil sebagai tumpukan vertikal biasa."

### [TAMBAH] 3.1 Loader (layar pemuatan singkat)

Pemilik meminta loader. Ini mengubah keputusan awal "tanpa preloader". Risikonya nyata: loader menunda konten dan bisa memperburuk LCP. Karena itu loader dibatasi keras:

| Aturan | Nilai |
|---|---|
| Isi | Inisial atau nama pemilik (dari `profile.ts`) dan penghitung 0 sampai 100 |
| Tidak palsu | Penghitung mengikuti kesiapan nyata: `document.fonts.ready` dan gambar hero (`img.decode()`). Bukan timer pura-pura |
| Durasi | Minimum 600 ms, maksimum **1500 ms**. Failsafe: apa pun yang terjadi, loader lepas pada 3000 ms |
| Frekuensi | Sekali per sesi tab. Penanda di `sessionStorage` dalam `try/catch`. Kunjungan ulang atau muat ulang dalam sesi yang sama: tidak muncul |
| Bisa dilewati | Klik atau tombol apa pun menutupnya. Ada tombol "Lewati" yang bisa difokus |
| Konten | Halaman sudah ada di DOM di bawahnya (tidak `display: none`). Tanpa JavaScript, loader tidak ada karena ditambahkan oleh JavaScript |
| Tingkat | Penuh: tirai naik (`transform`, 600 ms). Hemat: pudar singkat 250 ms. Mati dan `prefers-reduced-motion`: tidak ada loader |
| Aksesibilitas | `role="status"` dengan teks "Memuat", dihapus dari DOM saat selesai, fokus tidak terjebak |
| Urutan | Satu orkestrator `appReady` (promise). Animasi masuk hero baru mulai setelah loader selesai. Animasi hero tidak jalan di belakang loader |

Uji kewajaran: jika angka LCP di HP naik lebih dari 300 ms dibanding tanpa loader (diukur, bukan ditebak), loader dipersingkat atau dihapus. Pemilik memutuskan berdasarkan angka itu.

### [TAMBAH] 3.2 Potret hero (foto pemilik, gaya gelap berkabut)

**Bisa dibuat, dengan satu syarat: foto perlu dipisahkan dari latarnya (cutout).** Foto yang dilampirkan berwarna, berlatar langit terang, pepohonan, dan tenda. Efek di contoh (potret hitam putih yang melebur ke latar gelap) tidak bisa dicapai hanya dengan CSS dari foto itu, karena langit yang terang akan tampak sebagai bercak abu-abu terang. Blend mode CSS tidak bisa memisahkan langit dari subjek. Tanpa cutout, hasilnya paling banter "foto abu-abu dengan tepi gradien", bukan tampilan contoh.

Risiko yang perlu diketahui sejak awal:
- Jaket puffer hitam akan menyatu dengan latar hampir hitam. Ini yang membuat tampilan contoh dramatis, tetapi siluet bahu bisa hilang. Naikkan sedikit abu-abu jaket (angkat bayangan) supaya masih terbaca.
- Topi dan rompi berwarna tan menjadi abu-abu sedang. Wajah harus tetap punya detail. Periksa wajah di layar HP, bukan hanya di laptop.
- Tepi rambut atau topi sering kasar pada penghapus latar otomatis. Periksa di zoom 200 persen.

Langkah pembuatan (dilakukan pemilik di luar kode, lalu hasilnya diserahkan ke Cline sebagai berkas statis):

1. Simpan foto asli di tempat aman. Kerjakan pada salinan.
2. Potong (crop) dari kepala sampai pinggang, wajah di sepertiga atas. Rasio sekitar 4:5.
3. Hapus latar. Pilihan: Photoshop atau GIMP (pilih subjek, haluskan tepi), atau alat lokal seperti paket Python `rembg` (nama dan cara pasang belum saya verifikasi di sesi ini, periksa dulu). Penghapus latar daring mengunggah wajahmu ke server pihak ketiga. Pertimbangkan dulu, dan pilih alat lokal bila ragu.
4. Ubah ke hitam putih (atau duotone tipis condong biru, sesuai `--accent-cyan`), naikkan kontras, dan naikkan titik hitam sedikit mendekati warna `--bg` agar bagian gelap melebur.
5. Efek "agak blur" seperti contoh dibuat **di berkas gambar, bukan di CSS**: tambahkan gradien pada layer mask di sisi kiri, kanan, dan bawah (feather sekitar 60 sampai 120 px pada gambar 1200 px), dan blur gaussian 2 sampai 4 px hanya pada tepi, bukan pada wajah. `filter: blur()` atau `backdrop-filter` pada gambar besar dihindari karena mahal di HP.
6. Ekspor WebP transparan, lebar sekitar 1200 px, kualitas 75 sampai 85. Ukuran berkas target di bawah 150 KB (target awal, kalibrasi di P3B). AVIF opsional.
7. **Hapus metadata (EXIF) termasuk koordinat GPS** sebelum berkas masuk repo. Verifikasi dengan `exiftool berkas.webp` yang harus tidak menampilkan GPS.

Pemasangan di halaman:
- Berkas: `public/img/hero-portrait.webp`. Tag `<img>` dengan `width` dan `height` eksplisit (cegah CLS), `fetchpriority="high"`, tanpa `loading="lazy"` (ini kandidat LCP), `alt` deskriptif.
- Tepi tambahan dari CSS statis saja: `mask-image: linear-gradient(to bottom, #000 65%, transparent)` sebagai jaring pengaman. Tidak dianimasikan.
- Kedalaman (tingkat Penuh, perangkat kursor halus): potret bergerak berlawanan arah kursor maksimal 12 px dan miring maksimal 3 derajat (3D). Lewat `MotionValue` dari pointer (aturan bagian 5.2). Satu pemilik: Motion.
- Lapis tambahan opsional (butuh cutout bersih): kata besar tipis atau outline di belakang potret, judul `h1` di depan. Judul tetap harus memenuhi kontras 4.5:1 di atas potret. Uji dengan alat kontras, bukan dengan mata.
- Foto adalah milikmu sehingga tidak ada masalah hak cipta. Catatan privasi di `SECURITY.md` bagian 10.7.

---

## [GANTI] Bagian 4.1, tambahkan token (di dalam `tokens.ts` yang sama)

Tambahkan di bawah `distance`:

```ts
export const loader = { min: 0.6, max: 1.5, failsafe: 3.0, exit: 0.6, exitLite: 0.25 } as const; // detik
export const parallax = { maxShift: 80, layers: 3 } as const;       // px, jumlah lapis terlihat
export const depth = { portraitShift: 12, portraitTilt: 3, perspective: 1000 } as const; // px, derajat, px
export const horizontal = { maxViewports: 3 } as const;             // panjang scroll pin, dalam tinggi layar
```

`distance` tidak berubah (`tilt` tetap 6 derajat untuk kartu, potret memakai `depth`).

---

## [TAMBAH] Bagian 4.3 sampai 4.6 (sisipkan setelah 4.2)

### 4.3 Smooth scroll (Lenis)

Pemilik meminta Lenis. Smooth scroll mengambil alih roda mouse. Risikonya: terasa lambat bagi sebagian orang, bentrok dengan kontainer scroll bersarang, dan tidak boleh menyala di HP. Aturan:

1. Aktif hanya di tingkat Penuh **dan** `(hover: hover) and (pointer: fine)`. Di sentuh, Hemat, dan Mati: scroll native penuh, instans Lenis dihancurkan (`destroy`).
2. Lenis menghormati `prefers-reduced-motion` secara bawaan (`respectReducedMotion`). Jangan menonaktifkan opsi itu.
3. Tidak ada `scroll-behavior: smooth` di CSS. Satu pemilik scroll halaman: Lenis.
4. Detak: Lenis tidak punya loop sendiri (`autoRaf: false`). Digerakkan oleh `gsap.ticker`. Rumus sinkron dengan ScrollTrigger ada di `ARCHITECTURE.md` bagian 5.6.
5. Panel menu HP, halaman detail proyek, dan dialog lain yang membuka lapisan: panggil `lenis.stop()` saat terbuka dan `lenis.start()` saat tertutup. Kontainer yang bisa di-scroll sendiri diberi atribut `data-lenis-prevent`.
6. Tautan anchor dan "Lewati ke konten" memakai `lenis.scrollTo(target, { offset: -tinggiNavbar })` saat Lenis aktif, dan perilaku bawaan saat tidak aktif. Fokus tetap dipindah ke target.
7. Keyboard (Page Down, Space, panah) dan pembaca layar tidak boleh rusak. Diuji (`TESTING-GUIDE.md` bagian 5.8).
8. Batas yang diketahui dari dokumentasi Lenis: dibatasi 60 fps di Safari, tidak berfungsi di atas iframe, `position: fixed` bisa tertinggal di Safari lama. Tidak ada iframe di situs ini.

### 4.4 Parallax

1. Pemilik: GSAP ScrollTrigger dengan `scrub`. Hanya `transform: translate3d` (sumbu Y, atau X di section horizontal). Tidak menganimasikan `top`, `margin`, atau `background-position`.
2. Pergeseran maksimal `parallax.maxShift` (80 px). Lapis parallax terlihat bersamaan maksimal `parallax.layers` (3). Tiap lapis punya `data-speed` kecil (misalnya 0.05 sampai 0.2).
3. Hanya pada elemen dekoratif (blob L2, bentuk, potret, gambar proyek). Teks bacaan panjang tidak boleh bergerak berlawanan dengan scroll.
4. Tingkat Penuh saja. Hemat dan Mati: posisi statis.
5. Pakai `gsap.matchMedia()` agar kondisi (lebar layar, reduced motion) mengaktifkan dan membersihkan animasi otomatis.

### 4.5 Section horizontal (parallax horizontal)

"Parallax horizontal" bisa berarti dua hal. Spesifikasi ini memilih yang paling aman: **satu section di-pin, lalu scroll vertikal menggeser jalur (track) kartu secara horizontal**, dengan lapis dekoratif di dalamnya bergerak pada kecepatan berbeda.

1. Hanya satu section horizontal di seluruh halaman: **Proyek**. Alasan: menambah satu lagi melipatgandakan panjang scroll yang terasa seperti jebakan.
2. Aktif hanya jika `(min-width: 1024px) and (hover: hover) and (pointer: fine)`, tingkat Penuh, dan bukan reduced motion. Selain itu: tumpukan vertikal biasa (isi dan urutan sama).
3. Panjang scroll pin maksimal `horizontal.maxViewports` kali tinggi layar. Gunakan `invalidateOnRefresh: true` dan hitung ulang setelah font dan gambar siap. Tidak boleh ada ruang kosong besar di bawah section setelah pin lepas.
4. **Fokus keyboard:** saat kartu di dalam jalur menerima fokus (Tab), halaman harus men-scroll sehingga kartu itu terlihat. Tanpa ini, pengguna keyboard kehilangan fokus di luar layar. Wajib diuji.
5. Tiap kartu punya pemilik animasi sendiri (tilt kartu di elemen dalam: Motion). Jalur dan pin dimiliki GSAP. Tidak ada animasi `transform` pada elemen yang sama.
6. Konflik yang diketahui: animasi layout Motion (halaman detail proyek, FR-12) tidak boleh berada di dalam induk yang `transform`-nya digerakkan GSAP. Di mode horizontal, detail proyek dibuka sebagai dialog di portal (di luar jalur) dengan animasi opacity dan scale dari Motion, tanpa `layoutId` bersama. Di mode tumpukan vertikal, animasi layout seperti di FR-12. Ini keputusan desain yang perlu disetujui pemilik (lihat `TRD.patch.md`, pertanyaan terbuka).
7. Indikator: bilah progres tipis dan penghitung "02 / 06". Tidak lebih.

### 4.6 Hover, micro-interaction, dan reveal masuk (katalog terbatas)

Pemilik meminta hover effect, micro-interaction, dan entrance reveal. Daftar di bawah adalah **seluruh** yang diizinkan. Efek di luar daftar perlu persetujuan dan perhitungan anggaran.

Hover (hanya `(hover: hover)`):

| ID | Efek | Pemilik |
|---|---|---|
| H1 | Garis bawah tautan menggambar dari kiri (`transform: scaleX`) | CSS |
| H2 | Tombol: isian menyapu dari bawah, teks berubah warna | CSS |
| H3 | Kartu: garis cahaya tepi mengikuti kursor (C2) | CSS |
| H4 | Gambar proyek zoom 1.04 dalam bingkai ber-`overflow: hidden` | CSS |
| H5 | Kartu proyek miring 3D, maks `distance.tilt` (6 derajat), satu kartu sekaligus | Motion |
| H6 | Ikon panah bergeser 4 px | CSS |

Micro-interaction:

| ID | Efek | Pemilik |
|---|---|---|
| M1 | Tekan tombol: skala 0.97 (`whileTap`) | Motion |
| M2 | Salin email: ikon berganti centang 1,5 detik, pesan ke `aria-live="polite"` | Motion |
| M3 | Indikator section aktif di navbar menggeser (layout) | Motion |
| M4 | Pilihan tingkat efek di footer: geser dan fokus jelas | CSS |
| M5 | Bilah progres scroll | GSAP atau CSS (bukan `useState`) |
| M6 | Cincin fokus keyboard beranimasi masuk 150 ms | CSS |
| M7 | Tombol kembali ke atas muncul setelah 1 layar, memakai `lenis.scrollTo` bila aktif | Motion |

Entrance reveal (menggantikan deskripsi reveal umum, tetap sekali per elemen):

| ID | Varian | Dipakai untuk | Pemilik |
|---|---|---|---|
| R1 | Naik dan memudar 24 px | Blok, kartu, gambar | Motion (`Reveal`) |
| R2 | Judul per kata dengan mask (muncul dari bawah garis) | `h2` tiap section | GSAP SplitText |
| R3 | Stagger item (0,08 detik) | Chip keahlian, daftar | Motion |
| R4 | Hero: urutan khusus setelah loader selesai | Hero | GSAP timeline |

Cursor-follow (permintaan "mouse cursor follow animation") sudah tercakup di C1, C3, dan C4 (bagian 5.2). Tidak ada efek kursor baru, kecuali pemilik memilih C6 (lihat bagian 5.2).

---

## [GANTI] Bagian 5.2, tambahkan catatan di bawah tabel C1 sampai C6

Contoh referensi (landing hitam putih) memakai kisi yang melengkung mengikuti kursor. Itu efek WebGL tier 3 (C6). Jika pemilik menginginkannya, cari komponen padanannya di katalog reactbits.dev (nama tepatnya belum saya verifikasi), laporkan nama, dependensi, dan ukurannya, dan gunakan sebagai satu-satunya tier 3 serta satu-satunya L3. Efek ini hanya di hero dan menumpuk dengan potret, jadi uji keterbacaan wajah. Bawaannya: **tidak dipakai** sampai pemilik memutuskan.

---

## [GANTI] Bagian 6, tabel anggaran (ganti seluruh tabel)

| Batas | Nilai |
|---|---|
| Animasi tanpa henti yang terlihat bersamaan | Maks 2 (L2 ambient, dan satu dari ticker atau L3) |
| Latar WebGL (L3) aktif | Maks 1, hanya hero |
| Efek kursor tier 2 aktif bersamaan | Maks 2 dari C3, C4, C5 (kedalaman potret dihitung sebagai satu efek tier 2) |
| Efek kursor tier 3 | Maks 1 per halaman |
| Teks beranimasi per section | Maks 2 efek berbeda |
| Elemen kaca (`backdrop-filter`) terlihat bersamaan | Maks 2 |
| Pustaka animasi runtime | Motion, GSAP, Lenis (+ pustaka khusus komponen React Bits yang disetujui) |
| Instans Lenis | 1 |
| Animasi terhubung scroll (scrub) aktif bersamaan | Maks 4 |
| Lapis parallax terlihat bersamaan | Maks 3 |
| Section yang di-pin | Maks 1 (Proyek) |
| Loader | Maks 1 per sesi, maks 1,5 detik |
| Animasi masuk saat scroll | Sekali per elemen |

---

## [GANTI] Bagian 7, tabel tingkat (ganti tabel)

| Tingkat | Isi | Default |
|---|---|---|
| Penuh | Semua efek dalam anggaran: loader, Lenis, parallax, section horizontal, kedalaman potret, C1 sampai C5 | Desktop dengan kursor halus |
| Hemat | L1, L2, reveal, loader pudar singkat, C1, C2. Tanpa Lenis, parallax, pin horizontal (jadi tumpukan vertikal), kedalaman potret, L3, C3 sampai C6 | HP, perangkat dengan inti CPU sedikit (indikasi, bukan kepastian), atau setelah penurunan otomatis |
| Mati | Tanpa gerak non-esensial, tanpa loader, tanpa Lenis, status akhir langsung tampil | `prefers-reduced-motion: reduce` atau pilihan pengguna |

Tambahan: penurunan otomatis ke Hemat juga harus **menghancurkan Lenis dan mematikan pin** (`ScrollTrigger.kill`/`matchMedia.revert`) sehingga scroll kembali native tanpa muat ulang halaman.

---

## [TAMBAH] Bagian 8, butir tambahan

- Loader: `role="status"`, bisa dilewati, tidak menjebak fokus, tidak muncul saat reduced motion.
- Lenis aktif tidak boleh merusak keyboard scroll, pembaca layar, dan zoom teks 200 persen.
- Section horizontal: fokus keyboard pada kartu di luar layar harus memicu scroll sampai kartu terlihat. Urutan Tab sama dengan urutan visual.
- Potret: `alt` bermakna ("Foto [nama] mengenakan topi dan rompi di alam terbuka", isi sendiri), bukan "foto".
- Reduced motion di JavaScript: selain GSAP dan loop render, **hancurkan Lenis dan lepas semua ScrollTrigger yang bergerak** (hitung ulang tata letak tanpa pin).

---

## [GANTI] Bagian 10, tambahkan kriteria

- Loader terbukti maksimal 1,5 detik dan hanya sekali per sesi (bukti di laporan uji).
- Lenis hanya aktif di Penuh dengan kursor halus, terbukti mati di sentuh, Hemat, Mati.
- Section horizontal tidak meninggalkan ruang kosong dan bisa dijelajahi dengan keyboard.
- Potret sudah melewati langkah 3.2 (cutout, tanpa EXIF GPS) dan kontras judul terukur.
