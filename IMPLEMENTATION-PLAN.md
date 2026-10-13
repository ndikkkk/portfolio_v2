# IMPLEMENTATION-PLAN.md: Rencana Implementasi Portofolio Futuristik

Rencana kerja bertahap untuk agen Cline dan pemilik. Persyaratan ada di `TRD.md`, desain di `DESIGN.md`, struktur di `ARCHITECTURE.md`, pengujian di `TESTING-GUIDE.md`, aturan agen di `AGENT-CLINE.md`.

## 0. Cara memakai rencana ini

- Kerjakan **satu fase pada satu waktu.** Di akhir tiap fase agen berhenti, menjalankan pemeriksaan, melaporkan hasil, lalu menunggu persetujuan pemilik. Ini bukan formalitas: tiap fase menghasilkan sesuatu yang bisa dilihat dan diuji.
- Usaha diberi label relatif (S kecil, M sedang, L besar), bukan jam. Lama kerja nyata bergantung pada kecepatan model dan seberapa sering pemilik meninjau.
- Setelah tiap fase: `git add`, commit dengan pesan jelas, dan beri tag (`phase-1`, `phase-2`, dan seterusnya) agar mudah kembali jika fase berikutnya merusak sesuatu.
- Setiap fase punya **kriteria keluar.** Fase tidak dianggap selesai sebelum semuanya terpenuhi dan dilaporkan dengan bukti (keluaran perintah, angka, tangkapan layar), bukan klaim.
- Efek berat dan komponen React Bits dipilih bersama pemilik. Agen mengusulkan, memberi nama, dependensi, dan ukuran, lalu menunggu persetujuan.

## 1. Gambaran fase

| Fase | Isi | Usaha | Keluaran yang bisa dilihat |
|---|---|---|---|
| P0 | Persiapan | S | Proyek kosong yang ber-build, repo Git |
| P1 | Pondasi | M | Halaman dengan token warna, font, kerangka section |
| P2 | Lapisan gerak | M | Modul governor, pointer, Reveal, EffectBoundary yang teruji |
| P3 | Navbar dan Hero | M | Hero dengan latar L1 dan L2 serta sorotan C1. Pengukuran dasar |
| P4 | Efek kursor | M | C3, C4, C5, dan pengganti sentuh |
| P5 | Section isi | L | Tentang, Proyek, Keahlian, Pencapaian, Kontak, Footer |
| P6 | Latar L3 (opsional) | M | Latar WebGL di hero dengan cadangan, atau keputusan tanpa L3 |
| P7 | Poles | M | Aksesibilitas, SEO, performa, anggaran terverifikasi |
| P8 | Uji, keamanan, deploy | M | Laporan pengujian, preview Vercel, produksi |

## 2. Rincian fase

### P0: Persiapan (S)

Tugas:
1. Buat folder proyek baru (misalnya `E:\portofolio-futuristik`) dan taruh dokumen: `TRD.md`, `DESIGN.md`, `ARCHITECTURE.md`, `IMPLEMENTATION-PLAN.md`, `TESTING-GUIDE.md`, `MCP-SETUP.md`, `AGENT-CLINE.md`, `SECURITY.md`. Taruh `AGENT-CLINE.md` juga di `.clinerules/agent.md`.
2. `git init`. Buat `.gitignore` dan `.env.example` sesuai `SECURITY.md` bagian 2.
3. Pastikan MCP Context7, shadcn, dan Playwright hijau (`MCP-SETUP.md`).
4. Catat versi Node dan npm.
5. Buat proyek Vite: `npm create vite@latest . -- --template react-ts`. Karena folder tidak kosong, pilih "Ignore files and continue". Lalu `npm install`.

Kriteria keluar: `npm run build` sukses, `npm run dev` menampilkan halaman bawaan, `git status` bersih setelah commit pertama, tidak ada file rahasia.

Prompt untuk Cline:
```
Baca semua dokumen di folder ini sesuai urutan di AGENT-CLINE.md. Kerjakan fase P0
di IMPLEMENTATION-PLAN.md saja. Jangan lanjut ke P1. Laporkan keluaran build dan git status.
```

### P1: Pondasi (M)

Tugas:
1. Pasang Tailwind CSS untuk Vite dan inisialisasi shadcn menurut panduan resmi masing-masing (periksa dokumentasi terbaru lewat Context7, jangan menebak perintah).
2. Pasang font self-host (Sora variabel, JetBrains Mono variabel). Verifikasi nama paket di registry sebelum memasang.
3. Tulis `globals.css`: token warna dari `DESIGN.md` bagian 2.1, token tipografi, gaya dasar, blok `prefers-reduced-motion`, gaya fokus.
4. Tulis `src/motion/tokens.ts` persis menurut `DESIGN.md` bagian 4.1.
5. Tulis `src/data/profile.ts` dengan placeholder.
6. Tulis kerangka section kosong dengan `id` anchor, tautan lewati ke konten, dan struktur semantik.
7. Tulis `vercel.json` dari `SECURITY.md` bagian 3.
8. Atur lint dan pengecekan tipe yang ketat.

Kriteria keluar: build dan lint bersih, tidak ada scroll horizontal di 360 px (periksa dengan Playwright), token warna cocok dengan dokumen, kontras terverifikasi ulang untuk pasangan yang dipakai, font termuat dari domain sendiri (tanpa permintaan ke domain lain).

### P2: Lapisan gerak (M)

Tugas (lihat `ARCHITECTURE.md` bagian 5):
1. Pasang `motion`, `gsap`, `@gsap/react`.
2. Tulis `governor.ts`, dengan fungsi pemilihan tingkat yang murni dan terpisah dari pendengar browser.
3. Tulis `PointerProvider.tsx`, hooks, `Reveal.tsx`, `EffectBoundary.tsx`, `gsap-setup.ts`, dan `EffectToggle`.
4. Tulis uji unit untuk logika governor (tabel masukan dan keluaran) dan fungsi murni lainnya.

Kriteria keluar:
- Uji unit lulus.
- Bukti bahwa menggerakkan kursor tidak memicu render ulang React (misalnya penghitung render atau React Profiler yang menunjukkan nol render ulang komponen akibat gerak kursor).
- `prefers-reduced-motion` memaksa tingkat Mati, terbukti oleh uji.
- `EffectBoundary` menampilkan cadangan saat komponen sengaja dibuat melempar error.

### P3: Navbar dan Hero (M)

Tugas:
1. Navbar pil melayang, indikator section aktif, bilah progres scroll, menu HP dengan focus trap.
2. Hero: label HUD, judul dengan animasi masuk (GSAP SplitText atau pemecahan manual dengan `aria-label` pada induk), kalimat peran, dua tombol.
3. Latar L1 (grid dan noise statis) dan L2 (blob ambient, hanya `transform`, berhenti di luar layar).
4. Efek C1 (sorotan latar).
5. **Pengukuran dasar:** Lighthouse mode HP, ukuran bundel JS awal, CLS, dan rekaman performa singkat.

Kriteria keluar: hero tampil benar di lima lebar (`TESTING-GUIDE.md` bagian 3), angka pengukuran dasar dicatat di `docs/baseline.md`, dan **target di `TRD.md` bagian 3.1 dikalibrasi ulang** dengan angka nyata (pemilik menyetujui perubahan target).

### P4: Efek kursor (M)

Tugas:
1. C3 pengikut kursor (cincin, spring, membesar di `[data-cursor]`), kursor asli tetap tampil.
2. C4 teks bereaksi terhadap kursor pada judul hero (maks 40 huruf, posisi dihitung ulang saat resize dan scroll).
3. C5 tombol magnetik (maks dua).
4. C2 sorotan lokal pada kartu (siapkan utilitasnya, dipakai di P5).
5. Pengganti sentuh: riak singkat saat disentuh, dan kartu menyala saat masuk layar.

Kriteria keluar: semua efek hanya aktif pada kursor halus dan tingkat Penuh, mati saat reduced motion, tidak ada render ulang React per gerak kursor, anggaran `DESIGN.md` bagian 6 terpenuhi (tabel diisi bukti), uji emulasi sentuh menunjukkan tidak ada efek kursor.

### P5: Section isi (L)

Kerjakan per section dan tunjukkan ke pemilik secara bertahap. Konten dari pemilik atau placeholder.

1. Tentang (bento grid dengan C2).
2. Proyek (kartu dengan garis cahaya, kemiringan maks 6 derajat di kursor halus, halaman detail dengan animasi layout, fokus kembali ke pemicu, Escape menutup).
3. Keahlian (chip berkelompok, stagger).
4. Pencapaian (linimasa dengan garis tergambar lewat ScrollTrigger).
5. Kontak (CTA, tautan, validasi protokol URL).
6. Ticker (maks dua baris di seluruh halaman, bisa dijeda).
7. Footer (tautan, `EffectToggle`).

Kriteria keluar: semua FR-1 sampai FR-5, FR-12 sampai FR-14 terpenuhi, tidak ada scroll horizontal, anggaran animasi masih terpenuhi setelah semua section ada (jumlah efek dihitung ulang, bukan diasumsikan).

### P6: Latar L3 opsional (M)

Hanya jika pemilik menjawab ya di `TRD.md` bagian 6 nomor 5.

Tugas:
1. Telusuri katalog Backgrounds di reactbits.dev. Usulkan **dua sampai tiga kandidat** ke pemilik: nama, dependensi tambahan, ukuran, dan tangkapan layar.
2. Setelah pemilik memilih, pasang lewat CLI (`npx shadcn@latest add @react-bits/<Nama>-TS-TW`, perintah dari halaman instalasi mereka). Jika namespace tidak dikenali, ikuti petunjuk instalasi manual di halaman itu.
3. Bungkus dengan `EffectBoundary` (tier 3, lazy, jeda saat di luar layar, cadangan statis, batas DPR 1,5).
4. Catat di tabel `ARCHITECTURE.md` bagian 9, tambahkan komentar sumber dan lisensi di berkas komponen.
5. **Ukur dampaknya:** selisih skor Lighthouse, ukuran chunk, dan fps sebelum dan sesudah.

Kriteria keluar: pemilik menyetujui berdasarkan angka. Jika dampaknya terlalu besar, L3 dibuang dan latar L2 dipertahankan. Itu hasil yang sah.

### P7: Poles (M)

Tugas: audit aksesibilitas (kontras, fokus, reduced motion, pembaca layar untuk teks terpecah), SEO (judul, deskripsi, Open Graph, `robots.txt`, `sitemap.xml`), optimasi gambar dan font (preload yang perlu), mengurangi efek yang tidak lulus tiga pertanyaan kewajaran di `DESIGN.md` bagian 6, dan membersihkan kode.

Kriteria keluar: `TESTING-GUIDE.md` bagian 6 (aksesibilitas) dan bagian 7 (performa) lulus atau penyimpangannya dicatat beserta alasan.

### P8: Uji, keamanan, deploy (M)

Tugas:
1. Jalankan seluruh pengujian di `TESTING-GUIDE.md`, tulis `docs/test-report.md`.
2. Jalankan daftar periksa `SECURITY.md` bagian 7. `npm audit`, pemindai rahasia, header (`curl -I`).
3. Dorong ke repo GitHub **privat**. Hubungkan ke Vercel dengan izin terbatas pada repo ini.
4. Deploy ke preview. Pemilik memeriksa di HP sungguhan (Android dan iPhone jika ada).
5. Setelah disetujui, promosikan ke produksi. Ganti repo menjadi publik hanya setelah semua poin `SECURITY.md` bagian 7 dicentang.

Kriteria keluar: laporan pengujian lengkap, semua FR dan NFR terpenuhi atau penyimpangannya disetujui pemilik tertulis, situs di produksi, header terverifikasi.

## 3. Aturan lintas fase

- **Gerbang persetujuan:** perubahan di luar fase aktif, penambahan dependensi, penambahan efek di atas anggaran, dan perintah yang mengubah sesuatu di luar folder proyek selalu menunggu persetujuan.
- **Pemeriksaan wajib di akhir tiap fase:** `npm run build`, `npm run lint`, pengecekan tipe, uji yang ada, dan pengecekan scroll horizontal di 360 px.
- **Bukti, bukan klaim:** laporan fase menyertakan keluaran perintah dan angka. Hal yang belum diuji dinyatakan belum diuji.
- **Konten:** tidak mengarang. Placeholder `{{ISI_SENDIRI}}` sampai pemilik mengisi.
- **Rencana mundur:** kembali ke tag fase sebelumnya (`git checkout phase-N`) jika fase baru merusak sesuatu dan perbaikannya tidak jelas.

## 4. Risiko jadwal

| Risiko | Antisipasi |
|---|---|
| Pemilik belum menyiapkan konten | Lanjutkan dengan placeholder, tetapi jangan menganggap P5 selesai sebelum konten asli masuk |
| Efek terlalu berat setelah P3 | Kalibrasi target, turunkan efek, atau buang L3 |
| Komponen React Bits berubah nama atau varian | Verifikasi di situs resmi saat memasang, jangan mengandalkan nama dari dokumen ini |
| MCP shadcn gagal (timeout) | Pakai CLI `npx shadcn@latest` langsung di terminal |
| Konteks percakapan Cline penuh pada tugas panjang | Mulai tugas baru per fase. Dokumen dan tag Git menyimpan keadaan, bukan riwayat chat |
