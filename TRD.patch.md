# TRD.patch.md: perubahan untuk TRD.md

---

## [TAMBAH] Bagian 2, baris FR baru (setelah FR-15)

| ID | Persyaratan | Kriteria penerimaan |
|---|---|---|
| FR-16 | Loader singkat dengan nama atau inisial dan penghitung nyata | Maks 1,5 detik, sekali per sesi, bisa dilewati, tidak ada saat reduced motion atau tingkat Mati, failsafe 3 detik, konten sudah ada di DOM |
| FR-17 | Smooth scroll Lenis | Aktif hanya di Penuh dengan kursor halus. Mati di sentuh, Hemat, Mati, reduced motion. Anchor, keyboard scroll, dan dialog tetap benar |
| FR-18 | Parallax (GSAP ScrollTrigger) pada lapis dekoratif | Maks 3 lapis terlihat, pergeseran maks 80 px, hanya `transform`, mati di Hemat dan Mati |
| FR-19 | Section Proyek horizontal ter-pin di desktop Penuh | Tumpukan vertikal di tempat lain dengan isi sama. Fokus keyboard menggulirkan kartu ke layar. Tanpa ruang kosong setelah pin |
| FR-20 | Kedalaman 3D: tilt kartu proyek (maks 6 derajat) dan kedalaman potret hero (maks 12 px dan 3 derajat) | Hanya kursor halus dan Penuh. Tanpa render ulang React per gerak kursor |
| FR-21 | Katalog hover (H1 sampai H6) dan micro-interaction (M1 sampai M7) | Hanya yang ada di `DESIGN.md` 4.6. Tiap efek punya satu pemilik animasi |
| FR-22 | Reveal masuk bervariasi (R1 sampai R4) | Sekali per elemen. Hero mulai setelah loader selesai |
| FR-23 | Potret hero berperlakuan gelap berkabut | Berkas WebP hasil olahan tanpa EXIF GPS, `width` dan `height` eksplisit, tanpa CLS, kontras judul di atas potret minimal 4.5:1, `alt` bermakna |
| FR-24 | Pergantian tingkat efek tanpa muat ulang | Turun ke Hemat atau Mati menghancurkan Lenis dan melepas pin, scroll kembali native |

FR-5 dan FR-7 tetap berlaku. FR-7 tidak berubah: C1 sampai C5 sudah mencakup permintaan "mouse cursor follow".

---

## [GANTI] Bagian 3.1, tambahkan butir target (target awal, dikalibrasi di P3B)

| ID | Target |
|---|---|
| NFR-P9 | Loader tidak menambah LCP di HP lebih dari 300 ms dibanding tanpa loader (diukur dua kali, sebelum dan sesudah) |
| NFR-P10 | Lenis tidak menambah Total Blocking Time lebih dari 50 ms, diukur dengan dan tanpa Lenis |
| NFR-P11 | Lenis dimuat lewat impor dinamis dan tidak masuk bundel awal tingkat Hemat |
| NFR-P12 | Berkas potret di bawah 150 KB (target awal) |
| NFR-P13 | Selama scroll di desktop Penuh, frame terburuk persentil 95 di bawah sekitar 24 ms (kalibrasi dengan angka nyata) |

---

## [GANTI] Bagian 4, ubah AN-7 dan tambah AN-8 sampai AN-13

| ID | Persyaratan |
|---|---|
| AN-7 | Pustaka animasi runtime hanya Motion, GSAP, dan Lenis (ditambah pustaka khusus komponen React Bits yang disetujui) |
| AN-8 | Hanya satu instans Lenis, hanya dimulai dan dihentikan oleh `governor` |
| AN-9 | Lenis tidak aktif di sentuh, Hemat, Mati, dan reduced motion |
| AN-10 | Tidak ada `scroll-behavior: smooth` dan tidak ada `window.scrollTo` langsung di luar `lenis.ts` |
| AN-11 | Pin maksimal satu, parallax terlihat maksimal tiga, scrub bersamaan maksimal empat |
| AN-12 | Loader maks 1,5 detik, sekali per sesi, dengan failsafe |
| AN-13 | Setelah unmount atau pindah tingkat, jumlah `ScrollTrigger.getAll()` kembali ke nol dan instans Lenis dihancurkan |

---

## [TAMBAH] Bagian 5, risiko baru

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Loader memperburuk LCP dan terasa menghalangi | Skor performa dan kesan pertama turun | Batas 1,5 detik, sekali per sesi, ukur NFR-P9, bisa dihapus jika gagal |
| Smooth scroll terasa "berat", bentrok dengan kontainer scroll, atau merusak keyboard | Aksesibilitas dan kenyamanan | Hanya Penuh dan kursor halus, `respectReducedMotion` dibiarkan, `lenis.stop()` di dialog, uji keyboard |
| Pin horizontal merusak tata letak setelah font atau gambar dimuat | Ruang kosong atau loncatan | `invalidateOnRefresh`, hitung ulang setelah `appReady`, uji lima lebar |
| Animasi layout Motion di dalam jalur GSAP | Posisi salah, kedipan | Dialog detail lewat portal pada mode horizontal |
| Terlalu banyak efek menumpuk di hero (potret, loader, C1, C3, C4, L2, parallax) | Frame lambat di HP kelas menengah | Anggaran bagian 6, hero diuji sebagai satu kesatuan di P3B, tingkat Hemat |
| Potret hasil olahan: jaket hitam melebur dan siluet hilang, tepi cutout kasar | Tampilan tidak seperti contoh | Langkah 3.2, tinjau di HP, pemilik menyetujui berkas |
| Lisensi: Lenis MIT, GSAP standar | Rendah | Pemilik membaca teks lisensi sendiri |
| Foto wajah publik di situs | Privasi, penyalahgunaan gambar | `SECURITY.md` 10.7, pemilik memutuskan resolusi dan apakah dipublikasikan |

---

## [TAMBAH] Bagian 6, pertanyaan terbuka tambahan untuk pemilik

1. Teks loader: nama lengkap, inisial, atau tulisan lain?
2. Section horizontal: setuju hanya untuk Proyek? Berapa jumlah proyek (menentukan panjang scroll)?
3. Detail proyek di mode horizontal sebagai dialog di portal (tanpa animasi layout bersama): setuju?
4. Berkas potret hasil olahan (setelah langkah 3.2): sudah ada, atau ingin dipandu membuatnya dulu?
5. Efek kisi melengkung seperti contoh (WebGL tier 3, C6): dipakai atau tidak? Bawaannya tidak.
6. Teks `alt` potret dan apakah foto boleh dipublikasikan dalam resolusi sekitar 1200 px.

---

## [TAMBAH] Bagian 7, kriteria penerimaan tambahan

- FR-16 sampai FR-24 lulus dengan bukti.
- NFR-P9 sampai NFR-P13 terukur dan dicatat di `docs/test-report.md`. Angka yang gagal berarti efek dipersingkat atau dihapus, bukan target dilonggarkan diam-diam.
- Uji fisik: pemilik merasakan scroll di laptop dengan mouse sungguhan, di laptop dengan trackpad, dan di HP nyata.
