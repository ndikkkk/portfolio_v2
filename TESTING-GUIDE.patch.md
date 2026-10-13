# TESTING-GUIDE.patch.md: perubahan untuk TESTING-GUIDE.md

Semua tambahan masuk di bagian 5 (setelah 5.6), kecuali yang disebut lain. Selektor `data-testid` yang dipakai di bawah harus dibuat oleh agen: `loader`, `loader-skip`, `hero-portrait`, `projects-track`, `projects-section`, `project-card`.

---

## [TAMBAH] 5.7 Loader

| Uji | Cara | Lulus jika |
|---|---|---|
| Sekali per sesi | Muat halaman, tunggu loader hilang, muat ulang di tab sama | Loader tidak muncul pada muat ulang. Muncul lagi di konteks baru (sesi baru) |
| Batas waktu | Ukur dari navigasi sampai `[data-testid=loader]` hilang | Maks 1,5 detik pada jaringan normal. Dengan jaringan dilambatkan (throttle) tetap hilang paling lambat 3 detik (failsafe) |
| Bisa dilewati | Klik atau tekan Escape saat loader tampil | Hilang dalam 300 ms dan fokus tidak terjebak |
| Reduced motion | `page.emulateMedia({ reducedMotion: "reduce" })` | Loader tidak ada dan konten tampil di keadaan akhir |
| Tanpa JavaScript | Nonaktifkan JS | Konten terbaca, tanpa loader menggantung |
| Penyimpanan diblokir | Stub `sessionStorage` agar melempar error | Halaman tetap benar, loader tampil paling banyak sekali |
| Urutan | Periksa bahwa timeline hero baru mulai setelah `loader:done` | Tidak ada animasi hero di belakang loader (rekaman Performance) |
| Dampak LCP | Lighthouse HP dengan dan tanpa loader, dua kali masing-masing | Selisih LCP maks 300 ms (NFR-P9). Catat angka |

## [TAMBAH] 5.8 Lenis dan perilaku scroll

| Uji | Cara | Lulus jika |
|---|---|---|
| Hidup di Penuh dengan mouse | Proyek desktop kursor halus | Elemen `html` mendapat kelas Lenis (periksa nama kelas dari dokumentasi) dan posisi scroll berubah bertahap, bukan loncat |
| Mati di sentuh | Konteks dengan `hasTouch` dan `isMobile` | Kelas Lenis tidak ada, scroll native |
| Mati di Hemat dan Mati | Pilih tingkat lewat footer | Instans dihancurkan tanpa muat ulang. Posisi scroll tidak melompat saat pergantian |
| Mati di reduced motion | Emulasi | Tidak ada smoothing |
| Anchor | Klik tautan navbar | Berhenti di target dengan selisih maks 2 px setelah settle, memperhitungkan tinggi navbar. Fokus pindah ke section |
| Keyboard | Tekan Page Down, Space, End, panah | Halaman bergerak, tidak macet |
| Dialog | Buka panel menu atau detail proyek lalu coba scroll roda di belakang | Latar tidak ikut bergerak. Setelah ditutup scroll normal kembali |
| Kontainer scroll sendiri | Isi yang punya scroll dalam | Berfungsi (atribut `data-lenis-prevent`) |
| Zoom 200 persen | Perbesar teks | Tidak ada scroll yang tersangkut |
| Kebocoran | Mount dan unmount berulang lewat pergantian tingkat | Satu instans, dan `gsap.ticker` tidak menumpuk callback (periksa lewat kait pengujian khusus pengembangan) |

## [TAMBAH] 5.9 Parallax dan section horizontal

| Uji | Cara | Lulus jika |
|---|---|---|
| Anggaran parallax | Hitung lapis `ParallaxLayer` terlihat saat scroll | Maks 3. Pergeseran maks 80 px (ukur `transform`) |
| Hanya transform | Rekaman Performance | Tidak ada layout berulang saat scroll |
| Mati di Hemat | Ganti tingkat | Lapis kembali ke posisi awal, tanpa sisa `transform` |
| Horizontal hanya di desktop Penuh | Lebar 1280 (Penuh), 390 (sentuh), 1280 dengan Hemat | Pin di yang pertama saja. Dua lainnya tumpukan vertikal dengan isi sama |
| Tanpa ruang kosong | Scroll melewati section lalu ambil tangkapan | Tidak ada celah kosong lebih dari sekitar 100 px setelah pin lepas, di lima lebar |
| Fokus keyboard | Tab ke kartu proyek ke-3 sampai ke-6 | `boundingBox` kartu fokus berada di dalam viewport |
| Scroll horizontal halaman | `scrollWidth <= innerWidth` di 360 px | Tidak ada scroll horizontal halaman |
| Refresh setelah font | Muat dengan font dilambatkan | Posisi pin benar setelah `appReady` (tidak loncat) |
| Kebocoran | Pindah tingkat Penuh ke Hemat | `ScrollTrigger.getAll().length === 0` untuk pin, tidak ada pin tersisa |
| Detail proyek | Buka dan tutup detail di mode horizontal dan vertikal | Escape menutup, fokus kembali ke pemicu, tanpa kedipan posisi |

## [TAMBAH] 5.10 Potret hero

| Uji | Cara | Lulus jika |
|---|---|---|
| Berkas bersih | `exiftool public/img/hero-portrait.webp` (lewat terminal) | Tidak ada GPS dan tidak ada nama perangkat yang tidak diinginkan |
| Ukuran | `ls -l` | Di bawah target (150 KB awal) |
| CLS | Lighthouse | Potret tidak menggeser tata letak (`width` dan `height` ada) |
| Kontras | Alat kontras pada judul di atas potret, di tiga titik | Minimal 4.5:1 |
| Tepi | Tangkapan di 200 persen zoom | Tidak ada halo putih atau tepi bergerigi (periksa mata, tidak otomatis) |
| HP | Perangkat nyata | Wajah terbaca, siluet bahu tidak hilang |
| Kedalaman | Gerakkan kursor | Geser maks 12 px, miring maks 3 derajat. Mati di sentuh, Hemat, Mati |

Catatan batas alat: Playwright MCP hanya menghasilkan tangkapan statis, jadi "terlihat bagus" dan "terasa halus" tetap dinilai pemilik.

---

## [GANTI] Bagian 5.1, tabel anggaran: tambahkan baris pemeriksaan

| Item | Cara periksa | Batas |
|---|---|---|
| Instans Lenis | Kait pengujian pengembangan | 1 |
| Animasi scrub aktif | `ScrollTrigger.getAll()` yang terlihat | Maks 4 |
| Pin | `ScrollTrigger.getAll().filter(t => t.pin)` | Maks 1 |
| Lapis parallax terlihat | Hitung elemen | Maks 3 |
| Loader | Pengukuran 5.7 | Maks 1,5 detik, sekali per sesi |

---

## [GANTI] Bagian 7, performa: tambahkan

- Jalankan matriks empat kondisi (tanpa, loader, Lenis, keduanya) di P3B dan ulangi di P7 dengan seluruh efek.
- Rekam Performance 10 detik saat scroll di desktop Penuh dengan semua efek. Catat frame persentil 95 dan task panjang (NFR-P13, NFR-P10).
- Ulangi di HP nyata pada tingkat Hemat. Di Hemat tidak boleh ada Lenis, pin, atau parallax (periksa DOM dan jumlah ScrollTrigger).

---

## [TAMBAH] Bagian 11, baris keterlacakan

| Persyaratan | Pengujian |
|---|---|
| FR-16, AN-12, NFR-P9 | 5.7 |
| FR-17, AN-8, AN-9, AN-10, NFR-P10, NFR-P11 | 5.8 |
| FR-18, FR-19, AN-11, AN-13 | 5.9 |
| FR-20, FR-21, FR-22 | 5.2, 5.9 |
| FR-23, NFR-P12 | 5.10 |
| FR-24 | 5.8, 5.9 |
