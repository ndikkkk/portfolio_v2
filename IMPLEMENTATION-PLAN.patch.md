# IMPLEMENTATION-PLAN.patch.md: perubahan untuk IMPLEMENTATION-PLAN.md

---

## [GANTI] Bagian 1, tabel gambaran fase (ganti seluruh tabel)

| Fase | Isi | Usaha | Keluaran yang bisa dilihat |
|---|---|---|---|
| P0 | Persiapan | S | Proyek kosong yang ber-build, repo Git |
| P1 | Pondasi | M | Halaman dengan token warna, font, kerangka section |
| P2 | Lapisan gerak | M | Modul governor, pointer, Reveal, EffectBoundary yang teruji |
| P3 | Navbar dan Hero | M | Hero dengan latar L1 dan L2 serta sorotan C1. Pengukuran dasar |
| P3B | Loader, Lenis, potret hero | M | Loader nyata, smooth scroll yang bisa mati, potret berperlakuan. Pengukuran dengan dan tanpa |
| P4 | Efek kursor dan micro-interaction | M | C3, C4, C5, kedalaman potret, H dan M dari katalog, pengganti sentuh |
| P5 | Section isi | L | Tentang, Proyek (termasuk horizontal), Keahlian, Pencapaian, Kontak, Footer, parallax |
| P6 | Latar L3 (opsional) | M | Latar WebGL di hero dengan cadangan, atau keputusan tanpa L3 |
| P7 | Poles | M | Aksesibilitas, SEO, performa, anggaran terverifikasi |
| P8 | Uji, keamanan, deploy | M | Laporan pengujian, preview Vercel, produksi |

Catatan: fase P3B sengaja memisahkan tiga hal yang mengubah rasa halaman (loader, scroll, potret) dari efek kursor, supaya dampaknya terhadap performa bisa diukur satu per satu.

---

## [TAMBAH] Bagian 2, fase P3B (sisipkan setelah P3)

### P3B: Loader, Lenis, potret hero (M)

Prasyarat: P3 selesai dan angka dasar di `docs/baseline.md`. Berkas potret hasil olahan dari pemilik (lihat `DESIGN.md` 3.2). Jika belum ada, pakai gambar sementara abu-abu dengan rasio sama dan lanjut.

Tugas:
1. Periksa dokumentasi terbaru Lenis dan `@gsap/react` lewat Context7, lapor versi, lisensi, dan ukuran sebelum `npm install lenis` (aturan `AGENT-CLINE.md` 3.5).
2. `src/motion/lenis.ts` (`startLenis`, `stopLenis`, `scrollToTarget`) dan sambungkan ke `governor` (hanya Penuh dan kursor halus). Impor dinamis agar tidak masuk bundel awal.
3. Anchor navbar dan "Lewati ke konten" lewat `scrollToTarget`. Panel menu dan dialog memanggil `lenis.stop()` dan `start()`.
4. `appReady.ts` dan `Loader.tsx` sesuai `DESIGN.md` 3.1: nyata, 0,6 sampai 1,5 detik, sekali per sesi, bisa dilewati, failsafe 3 detik, tanpa loader di Mati.
5. `HeroPortrait.tsx` memasang `public/img/hero-portrait.webp` dengan `width`, `height`, `fetchpriority="high"`, dan mask CSS statis. Kedalaman kursor ditunda ke P4.
6. Orkestrasi: timeline hero (R4) mulai setelah `loader:done`.
7. **Pengukuran perbandingan:** Lighthouse HP (LCP, TBT, CLS) untuk empat kondisi: tanpa loader dan tanpa Lenis, dengan loader saja, dengan Lenis saja, dengan keduanya. Catat di `docs/p3b-measure.md`.

Kriteria keluar:
- Loader terbukti maks 1,5 detik dan hanya sekali per sesi, tidak ada di reduced motion dan Mati.
- Lenis hidup hanya di Penuh dengan kursor halus. Terbukti mati di emulasi sentuh, di Hemat, dan di reduced motion. Anchor, keyboard scroll, dan dialog berfungsi.
- NFR-P9 dan NFR-P10 terpenuhi atau pemilik menyetujui penyimpangan tertulis berdasarkan angka.
- Potret tanpa CLS, kontras judul terukur, `exiftool` tidak menampilkan GPS.

**Gerbang persetujuan:** berhenti. Pemilik menilai rasa scroll dan loader di perangkat nyata. Jika terasa mengganggu, loader atau Lenis dipersingkat atau dihapus sebelum lanjut.

---

## [GANTI] Bagian 2, P4 (tambahan tugas)

Tambahkan di daftar tugas P4:
6. Kedalaman potret hero (Motion, `MotionValue` dari pointer): geser maks 12 px, miring maks 3 derajat, hanya Penuh dan kursor halus.
7. Katalog hover H1 sampai H6 dan micro-interaction M1 sampai M7 (`DESIGN.md` 4.6). Tiap efek dicatat dengan pemiliknya. Tidak menambah di luar katalog.

Kriteria keluar ditambah: anggaran `DESIGN.md` bagian 6 diisi ulang dengan angka (tier 2 maks 2, termasuk kedalaman potret).

---

## [GANTI] Bagian 2, P5 (tambahan tugas)

Tambahkan di daftar tugas P5:
- Parallax (`ParallaxLayer`) pada lapis dekoratif: maks 3 terlihat, maks 80 px, hanya Penuh, dibungkus `gsap.matchMedia`.
- `HorizontalSection` untuk Proyek: pin, jalur horizontal, bilah progres, fokus keyboard menggulirkan kartu, tumpukan vertikal di luar syarat. Dialog detail via portal pada mode horizontal.
- Animasi masuk R1 sampai R3 dipasang lewat `Reveal` dan SplitText.

Kriteria keluar ditambah:
- Tidak ada ruang kosong setelah pin lepas di lima lebar.
- Tab melewati semua kartu proyek dan tiap kartu terlihat saat fokus.
- `ScrollTrigger.getAll()` kembali nol setelah unmount (uji kebocoran).

**Gerbang persetujuan:** pemilik menyetujui section horizontal sebelum P6.

---

## [TAMBAH] Bagian 3, aturan lintas fase

- Setelah menambah efek apa pun, jalankan ulang pengukuran dasar dan bandingkan dengan P3 dan P3B. Efek yang membuat angka melewati target dilaporkan, bukan disembunyikan.
- Urutan penghematan jika anggaran terlampaui (hapus dari atas): parallax, kedalaman potret, section horizontal (jadi vertikal), Lenis, loader. Reveal, hover, dan micro-interaction murah dan dipertahankan paling akhir.

---

## [TAMBAH] Bagian 4, risiko jadwal

- Pembuatan potret (cutout dan olahan) adalah pekerjaan pemilik dan bisa menjadi penghambat. P3B jalan dengan gambar sementara.
- Section horizontal memakan waktu pengujian paling banyak (fokus, ruang kosong, lima lebar). Beri anggaran waktu tambahan di P5.
