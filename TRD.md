# TRD.md: Technical Requirements Document, Portofolio Futuristik

Dokumen persyaratan teknis. Setiap persyaratan punya ID agar bisa dilacak ke rencana (`IMPLEMENTATION-PLAN.md`) dan pengujian (`TESTING-GUIDE.md`). Angka target diberi tanda **(awal)** jika masih perlu dikalibrasi setelah pengukuran pertama.

## 1. Tujuan dan batasan

**Tujuan.** Situs portofolio pribadi satu halaman bertema futuristik, banyak animasi yang terkendali, termasuk latar dan teks yang mengikuti kursor, nyaman di HP dan laptop, dan aman untuk dipublikasikan.

**Bukan tujuan (fase 1).** Backend, login, formulir kontak, blog, mode terang, multi-bahasa, CMS, analitik pelacak.

**Pemangku kepentingan.** Pemilik portofolio (pengambil keputusan, penyedia konten), calon pemberi kerja atau pembimbing (pengunjung), agen Cline (pelaksana).

**Batasan.**
- Hosting gratis (Vercel Hobby). Tidak ada layanan server berbayar.
- Alat: Node.js LTS, Cline dengan MCP yang disetujui (`MCP-SETUP.md`).
- Seluruh konten pribadi dari pemilik. Tidak ada konten karangan.
- Komponen berlisensi atau berbayar pihak ketiga tidak disalin kecuali pemilik membelinya dan menyerahkannya sendiri.

**Asumsi.**
- Pengunjung utama memakai browser modern yang dirilis sejak awal 2023 (butuh dukungan `color-mix()`, `dvh`, `IntersectionObserver`). Browser yang lebih lama menerima tampilan cadangan tanpa efek.
- Pengunjung membuka dari HP sama banyaknya dengan dari laptop, jadi HP kelas menengah adalah perangkat acuan.

## 2. Persyaratan fungsional

| ID | Persyaratan | Kriteria penerimaan |
|---|---|---|
| FR-1 | Situs satu halaman dengan section: Hero, Tentang, Proyek, Keahlian, Pencapaian, Kontak, Footer, dan Navbar | Semua section ada dengan anchor yang berfungsi. Navbar menandai section aktif |
| FR-2 | Seluruh konten dibaca dari satu berkas data | Mengganti isi `profile.ts` mengubah situs tanpa mengedit komponen |
| FR-3 | Responsif dari 360 px sampai 1920 px | Tidak ada scroll horizontal. Teks tidak terpotong. Area sentuh minimal 44 x 44 px |
| FR-4 | Navigasi HP berupa menu yang bisa dibuka dan ditutup | Bisa dipakai keyboard, fokus terjebak di panel saat terbuka, Escape menutup |
| FR-5 | Animasi masuk saat scroll untuk section dan elemen utama | Terjadi sekali per elemen. Tidak ada animasi yang menyembunyikan konten permanen |
| FR-6 | Latar berlapis (L0 sampai L2 selalu, L3 opsional di hero) | Sesuai anggaran di `DESIGN.md` bagian 6. L3 punya cadangan statis |
| FR-7 | Efek mengikuti kursor: sorotan latar (C1), sorotan kartu (C2), pengikut kursor (C3), teks bereaksi (C4), tombol magnetik (C5) | Aktif hanya pada `(hover: hover) and (pointer: fine)`. Kursor asli tetap tampil. Tidak ada render ulang React per gerak kursor |
| FR-8 | Pengganti efek kursor di layar sentuh | Gradien ambient, riak sentuhan singkat, dan reveal berbasis scroll |
| FR-9 | Tiga tingkat efek (Penuh, Hemat, Mati) dengan kendali di footer | Pilihan tersimpan (jika penyimpanan tersedia) dan halaman benar tanpa penyimpanan |
| FR-10 | Penurunan kualitas otomatis dari ukuran frame nyata | Jika frame rata-rata terus lambat, tingkat turun ke Hemat dan efek berat berhenti |
| FR-11 | Mode `prefers-reduced-motion` | Tidak ada gerak non-esensial. Semua konten tetap terbaca dan tampil di keadaan akhir |
| FR-12 | Halaman detail proyek terbuka dalam halaman yang sama dengan animasi layout | Bisa ditutup dengan Escape dan tombol. Fokus kembali ke pemicu |
| FR-13 | Linimasa pencapaian yang menggambar dirinya mengikuti scroll | Animasi terhubung scroll native tanpa mengubah perilaku scroll |
| FR-14 | Tautan kontak langsung (email, LinkedIn, GitHub, WhatsApp jika diizinkan) | Tautan keluar memakai `rel="noopener noreferrer"`. Protokol URL divalidasi |
| FR-15 | Metadata SEO dan berbagi | `<html lang="id">`, judul dan deskripsi spesifik, Open Graph 1200 x 630, `robots.txt`, `sitemap.xml` |

## 3. Persyaratan non-fungsional

### 3.1 Performa

| ID | Persyaratan | Target |
|---|---|---|
| NFR-P1 | Largest Contentful Paint (HP, jaringan 4G terukur) | Sampai 2,5 detik (ambang "baik" Google) |
| NFR-P2 | Interaction to Next Paint | Sampai 200 ms (ambang "baik") |
| NFR-P3 | Cumulative Layout Shift | Sampai 0,1 (ambang "baik"). Semua gambar punya dimensi, tidak ada geseran akibat animasi masuk |
| NFR-P4 | Lighthouse Performance, mode HP | 90 atau lebih pada tingkat Hemat. 80 atau lebih pada tingkat Penuh **(awal)**. Latar WebGL hampir pasti menurunkan skor, jadi target Penuh lebih longgar dan harus diukur, bukan diasumsikan |
| NFR-P5 | Kelancaran animasi | Sekitar 50 fps atau lebih pada HP kelas menengah **(awal)** saat efek aktif. Di bawah itu, governor menurunkan efek |
| NFR-P6 | Ukuran JavaScript awal (gzip, tanpa chunk malas) | Sampai sekitar 150 KB **(awal)**. Chunk L3 dimuat malas |
| NFR-P7 | Gambar | WebP atau AVIF, tidak lebih dari 200 KB per gambar tanpa alasan |
| NFR-P8 | Tidak ada pekerjaan di luar layar | Loop render dan animasi berulang berhenti saat di luar layar atau tab tersembunyi |

Angka ukuran dan fps bergantung pada pilihan komponen akhir. Kalibrasi setelah fase pondasi dan fase hero (lihat rencana), lalu perbarui dokumen ini, jangan mengabaikan targetnya diam-diam.

### 3.2 Aksesibilitas (WCAG 2.1 AA)

| ID | Persyaratan |
|---|---|
| NFR-A1 | Kontras teks minimal 4,5:1 (3:1 untuk teks besar dan komponen UI). Palet di `DESIGN.md` sudah dihitung |
| NFR-A2 | Fokus keyboard tampak jelas dan urutan Tab logis. Ada tautan lewati ke konten |
| NFR-A3 | Semantik HTML benar, satu `h1`, `alt` pada gambar bermakna |
| NFR-A4 | Teks yang dipecah per huruf tetap terbaca pembaca layar (`aria-label` pada induk, potongan `aria-hidden`) |
| NFR-A5 | `prefers-reduced-motion` dihormati di CSS dan JavaScript, termasuk GSAP dan loop WebGL |
| NFR-A6 | Konten bergerak lebih dari 5 detik bisa dijeda |
| NFR-A7 | Informasi tidak disampaikan hanya lewat warna atau gerak |

### 3.3 Kompatibilitas

- Browser: versi terbaru dan satu versi sebelumnya dari Chrome, Edge, Firefox, dan Safari (termasuk Safari iOS). Pengujian otomatis hanya mencakup Chromium, Firefox, dan WebKit di desktop. WebKit Playwright bukan Safari iOS, jadi pemeriksaan manual di iPhone tetap diperlukan.
- Perangkat: lebar 360 sampai 1920 px, sentuh dan kursor.
- Tanpa WebGL, tanpa JavaScript untuk efek: konten tetap tampil (cadangan statis).

### 3.4 Keamanan dan privasi

Seluruh aturan ada di `SECURITY.md` dan mengikat. Ringkasan persyaratan:

| ID | Persyaratan |
|---|---|
| NFR-S1 | Tidak ada rahasia di repo atau bundel. Tidak ada variabel `VITE_*` berisi rahasia |
| NFR-S2 | Header keamanan dan CSP aktif di Vercel, diverifikasi dengan `curl -I` |
| NFR-S3 | `npm audit` tanpa temuan high atau critical yang bisa diperbaiki |
| NFR-S4 | Tidak ada skrip pihak ketiga tanpa keputusan pemilik. Font di-host sendiri |
| NFR-S5 | Foto bebas EXIF. Tidak ada data pribadi yang tidak diinginkan. Tidak ada data penelitian pribadi orang lain |
| NFR-S6 | Dependensi baru melalui pemeriksaan di `AGENT-CLINE.md` dan dicatat di `ARCHITECTURE.md` bagian 9 |

### 3.5 Kemudahan pemeliharaan

| ID | Persyaratan |
|---|---|
| NFR-M1 | TypeScript ketat, tanpa kesalahan tipe dan lint |
| NFR-M2 | Token warna dan gerak satu sumber (`globals.css`, `tokens.ts`). Tidak ada angka durasi atau warna liar di komponen |
| NFR-M3 | Aturan kepemilikan animasi di `ARCHITECTURE.md` bagian 4 dipatuhi |
| NFR-M4 | Setiap komponen React Bits yang disalin ditandai komentar sumber dan lisensi, dan dicatat di tabel dependensi |
| NFR-M5 | Pembersihan efek (listener, `requestAnimationFrame`, konteks GSAP, konteks WebGL) saat komponen dilepas |

## 4. Persyaratan animasi (ringkasan terikat)

Rincian di `DESIGN.md` bagian 4 sampai 7. Persyaratan yang bisa diuji:

| ID | Persyaratan |
|---|---|
| AN-1 | Hanya `transform`, `opacity`, dan `filter` secukupnya yang dianimasikan |
| AN-2 | Durasi sesuai token (`tokens.ts`). Tidak ada animasi satu kali di atas 1,2 detik |
| AN-3 | Anggaran animasi (maks animasi tanpa henti, maks L3, maks efek kursor tier 2, maks kaca) terpenuhi |
| AN-4 | Satu elemen hanya dimiliki satu pustaka animasi |
| AN-5 | Efek kursor tidak memicu render ulang React per gerak kursor |
| AN-6 | Efek berat berhenti di luar layar dan saat tab tersembunyi |
| AN-7 | Tidak ada tiga atau lebih pustaka animasi runtime selain yang disetujui |

## 5. Risiko

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Terlalu banyak animasi (overanimate) | Situs terasa berat dan melelahkan, merusak keterbacaan | Anggaran mengikat, tiga pertanyaan kewajaran, kendali tingkat |
| Efek berat membuat HP lag | Pengunjung HP pergi | Governor, tingkat Hemat default di HP, L3 hanya di hero |
| Efek kursor tidak ada di layar sentuh | Pengunjung HP melihat situs lebih sederhana | Pengganti di FR-8, dan dokumentasikan bahwa ini disengaja |
| Dua pustaka saling menimpa animasi | Gerak patah yang sulit dilacak | Aturan kepemilikan |
| Lisensi komponen | Masalah hukum atau reputasi | Hanya sumber gratis berlisensi jelas, catat per komponen, jangan menyalin yang berbayar |
| Rantai pasokan paket npm | Kode berbahaya masuk | Daftar dependensi disetujui, `npm audit`, periksa setiap paket baru |
| Agen mengarang konten | Klaim palsu di portofolio | Aturan tanpa karangan, placeholder, pemilik menyetujui konten |
| Prompt injection lewat web atau MCP | Agen melanggar aturan | `SECURITY.md` bagian 1, persetujuan manual perintah |
| Tampilan futuristik mengorbankan kontras | Gagal aksesibilitas | Palet sudah dihitung dan diuji ulang saat berubah |
| Heuristik perangkat salah | Efek berat jalan di perangkat lemah | Ukuran frame nyata sebagai penentu, kendali manual |

## 6. Pertanyaan terbuka untuk pemilik

1. Nama, peran, dan satu kalimat tentang kamu.
2. Tiga proyek terbaik: judul, ringkasan satu kalimat, tag teknologi, tautan demo dan repo.
3. Sertifikat atau lomba yang nyata, dengan tautan yang bisa diverifikasi.
4. Kontak mana yang boleh dipublikasikan (email, LinkedIn, GitHub, WhatsApp).
5. Apakah ingin latar WebGL (L3) di hero, atau cukup L0 sampai L2. L3 menambah ukuran dan beban.
6. Apakah ingin efek kursor tier 3 (fluida atau jejak). Defaultnya tidak.
7. Apa maksudmu dengan arc-ui.com. Situsnya tidak bisa dibaca dari sesi ini.

## 7. Kriteria penerimaan akhir

Situs dianggap selesai jika:
- Semua FR terpenuhi dan lulus pengujian terkait (`TESTING-GUIDE.md` bagian 11).
- NFR-P diukur dan angkanya dilaporkan, bukan diklaim. Target yang meleset dilaporkan beserta alasan, bukan disembunyikan.
- NFR-A dan NFR-S lulus.
- Anggaran animasi diverifikasi.
- Pemilik memeriksa situs di HP sungguhan dan menyetujui konten.
