# DESIGN.md: Web Portofolio Futuristik dengan Animasi Terukur

Sumber kebenaran untuk tampilan, animasi, dan interaksi. Baca sebelum menulis kode. Urutan prioritas jika ada konflik: `SECURITY.md`, `TRD.md`, dokumen ini, lalu preferensi pribadi agen. Dokumen terkait: `ARCHITECTURE.md`, `IMPLEMENTATION-PLAN.md`, `TESTING-GUIDE.md`, `MCP-SETUP.md`, `AGENT-CLINE.md`.

## 0. Prinsip utama

1. **Banyak animasi, tetapi tiap animasi harus punya tugas.** Tugas yang sah: memberi umpan balik (tombol ditekan), menunjukkan hierarki (apa yang muncul dulu), menjaga kontinuitas (elemen berpindah, bukan hilang), atau satu momen mengesankan di hero. Animasi yang tidak bisa dijelaskan tugasnya dihapus. Itu yang membedakan "banyak animasi" dari "terlalu banyak animasi" (overanimate).
2. **Anggaran animasi (bagian 6) mengikat.** Menambah efek di atas anggaran harus lewat persetujuan pemilik.
3. **Konten pribadi tidak boleh dikarang.** Pakai placeholder `{{ISI_SENDIRI}}` dan tanya pemilik. Detail di `AGENT-CLINE.md`.
4. **Responsif dan hemat dulu.** Situs harus nyaman di HP kelas menengah, bukan hanya di laptop. Efek mahal punya jalur turun kualitas otomatis (bagian 7).
5. **Aksesibilitas tidak dinegosiasikan.** Kontras, fokus keyboard, `prefers-reduced-motion`, dan kursor asli tetap ada.

## 1. Referensi dan peran masing-masing

Sepuluh lebih referensi tidak boleh menjadi sepuluh dependensi. Tabel ini menetapkan mana yang dipasang, mana yang hanya inspirasi, dan mana yang tidak boleh disalin.

| Referensi | Peran di proyek | Status |
|---|---|---|
| reactbits.dev (gratis) | Sumber utama komponen teks animasi, latar, dan efek kursor. Dipasang sebagai kode sumber lewat CLI shadcn, jadi kode ada di repo | Dipakai, selektif |
| pro.reactbits.dev | Inspirasi | Berbayar. Jangan menyalin, men-scrape, atau meniru kodenya. Boleh dipakai hanya jika pemilik membelinya dan memberikan file sendiri |
| 21st.dev | Inspirasi komponen. Komunitas, lisensi tiap komponen berbeda | Periksa lisensi per komponen sebelum menyalin |
| GSAP (gsap.com) | Animasi berbasis scroll (ScrollTrigger), pemecahan teks (SplitText), timeline | Dipakai. Gratis untuk penggunaan komersial di bawah lisensi standar GSAP |
| Motion (motion.dev, MIT) | Animasi UI di React: mount/unmount, layout, gestur, nilai bergerak untuk kursor, spring | Dipakai, mesin utama untuk UI |
| Anime.js v4 | Referensi | Tidak dipasang. Fungsinya tumpang tindih dengan GSAP dan Motion |
| React Spring | Referensi | Tidak dipasang. Spring sudah ada di Motion |
| animos.app | Membuat video presentasi proyek (template gerak, ekspor MP4/WebM di browser) | Opsional, alat luar. Bukan pustaka kode. Syarat lisensi hasil ekspor tidak tertera di halaman, periksa sendiri |
| swishy.ai | Membuat aset animasi dengan AI (tipografi, grafis gerak) | Opsional, alat luar. Format ekspor dan lisensi belum jelas, periksa sendiri |
| arc-ui.com | Tidak diketahui | Halaman tidak bisa dibaca dari sesi ini (diblokir robots). **Jangan dipakai** sampai pemilik menjelaskan apa yang diinginkan darinya |
| useplanes.com | Inspirasi interaksi | Berbayar (satu kali, harga dan batas waktunya tertera di situs). Tidak dibeli. Jangan meniru kode atau tampilannya |
| neobrutalism.dev, ikhsan3adi.is-a.dev | Dari proyek sebelumnya | neobrutalism.dev tidak dipakai (gayanya berlawanan). ikhsan3adi hanya untuk urutan section |

**Alasan memilih Motion dan GSAP saja, bukan semuanya:** empat mesin animasi sekaligus menambah ukuran bundel, dan lebih parah, dua pustaka yang menganimasikan `transform` pada elemen yang sama saling menimpa dan menghasilkan lonjakan yang sulit dilacak. Aturannya: **satu elemen, satu pemilik animasi.** Elemen yang dimiliki GSAP tidak boleh disentuh Motion, dan sebaliknya. Pembagiannya ada di `ARCHITECTURE.md`.

## 2. Gaya visual

Arah: gelap, presisi, bernuansa antarmuka instrumen (HUD). Elemen khas: garis grid tipis, label monospace kecil, cahaya (glow) aksen yang terkendali, permukaan kaca tipis, dan latar yang bergerak pelan. Hindari: gradien pelangi, neon di semua tempat, dan efek blur besar yang memberatkan GPU.

Catatan kompromi: situs ini gelap saja (satu skema warna). Itu menyederhanakan kontras dan efek cahaya, tetapi kurang nyaman di ruangan sangat terang. Kalau pemilik ingin mode terang, itu fase 2.

### 2.1 Token warna

Rasio kontras dihitung dengan rumus WCAG 2.1. Semua aksen lolos sebagai teks di atas latar gelap.

| Token | Nilai | Catatan |
|---|---|---|
| `--bg` | `#07080C` | Latar halaman |
| `--surface-1` | `#0E1118` | Kartu dan panel |
| `--surface-2` | `#151A24` | Kartu terangkat, input |
| `--text` | `#E8ECF4` | Teks utama. 16,91 terhadap `--bg`, 14,72 terhadap `--surface-2` |
| `--muted` | `#9AA4B8` | Teks sekunder. 7,98 terhadap `--bg`, 6,95 terhadap `--surface-2` |
| `--accent-cyan` | `#22D3EE` | Aksen utama. 11,08 terhadap `--bg` |
| `--accent-violet` | `#A78BFA` | Aksen kedua. 7,36 |
| `--accent-lime` | `#BEF264` | Aksen status (sukses, "tersedia"). 15,32 |
| `--accent-pink` | `#F472B6` | Aksen sorotan jarang. 7,56 |
| `--accent-amber` | `#FBBF24` | Peringatan. 11,99 |
| `--danger` | `#FB7185` | Error. 7,44 |
| `--on-accent` | `#07080C` | Teks di atas isian aksen (tombol). 7,36 sampai 15,32 |
| `--line` | `#2A3142` | Garis dekoratif (kontras hanya 1,54, jadi **jangan** dipakai untuk batas kontrol) |
| `--line-strong` | `#5B6A8C` | Batas input, tombol, dan indikator fokus. 3,70 terhadap `--bg`, 3,22 terhadap `--surface-2` |

Aturan: komponen yang bisa diklik harus punya batas atau isian dengan kontras minimal 3:1 terhadap latarnya. Pakai `--line-strong` untuk itu. `--line` hanya untuk garis hias.

Gradien aksen (untuk teks judul atau garis): `linear-gradient(90deg, var(--accent-cyan), var(--accent-violet))`. Teks bergradien harus tetap berkontras cukup di kedua ujung (keduanya lolos).

Cahaya: `box-shadow: 0 0 24px -4px color-mix(in srgb, var(--accent-cyan) 45%, transparent)`. Batasi radius dan jumlah elemen yang memakainya; bayangan besar di banyak elemen memberatkan penggambaran.

### 2.2 Tipografi

- Judul dan teks: **Sora** (variabel, sumbu bobot), memungkinkan efek bobot huruf yang mengikuti kursor.
- Label HUD dan kode: **JetBrains Mono**, ukuran kecil, huruf kapital, jarak huruf lebar.
- Pasang lewat `@fontsource-variable` (self-host). Verifikasi nama paket di registry sebelum memasang; kalau tidak ada, berhenti dan laporkan.
- Ukuran fluid:

```css
h1 { font-size: clamp(2.5rem, 9vw, 6.5rem); line-height: 0.98; letter-spacing: -0.03em; }
h2 { font-size: clamp(1.75rem, 5vw, 3.5rem); line-height: 1.05; letter-spacing: -0.02em; }
body { font-size: clamp(1rem, 0.95rem + 0.25vw, 1.125rem); line-height: 1.65; }
.hud { font-family: var(--font-mono); font-size: 0.75rem; letter-spacing: 0.14em; text-transform: uppercase; }
```

- Teks isi minimal 16 px di HP.

### 2.3 Permukaan, garis, bentuk

- Radius: `--radius: 14px` untuk kartu, 999px untuk tombol pil. Pilih dan konsisten.
- Permukaan kaca: `background: color-mix(in srgb, var(--surface-1) 70%, transparent); backdrop-filter: blur(12px)`. `backdrop-filter` mahal: **paling banyak dua elemen kaca terlihat bersamaan** (navbar dan satu panel).
- Grid hias: pola garis 1 px `--line` dengan opasitas rendah, dibuat dengan CSS (`background-image`), statis, bukan elemen yang dianimasikan.
- Noise halus: satu lapisan gambar kecil yang diulang, opasitas 3 sampai 5 persen. Statis.

### 2.4 Tata letak dan responsif

Mobile-first. Breakpoint Tailwind: `sm` 640, `md` 768, `lg` 1024, `xl` 1280.

- Wadah `max-w-6xl`, padding `px-4 sm:px-6 lg:px-8`. Jarak section `py-16 md:py-28`.
- Grid proyek: 1 kolom, `md` 2 kolom, `lg` 3 kolom. Bento di bagian Tentang: 1 kolom di HP, 4 kolom di `lg`.
- Tidak boleh ada scroll horizontal di lebar 360 px. Periksa marquee, latar, dan gambar.
- Area sentuh minimal 44 x 44 px.
- Gunakan `dvh`, bukan `vh`, untuk tinggi layar penuh di HP. Hormati `env(safe-area-inset-*)`.
- Gaya hover dibungkus `@media (hover: hover) and (pointer: fine)`. Informasi tidak boleh hanya muncul saat hover.
- Gambar: `width` dan `height` eksplisit, `loading="lazy"` kecuali di atas lipatan, format WebP atau AVIF.
- `<meta name="viewport" content="width=device-width, initial-scale=1">`. Jangan mematikan zoom.

## 3. Struktur halaman

Satu halaman, anchor per section.

1. **Navbar**: pil melayang, kaca tipis, indikator section aktif, bilah progres scroll di bawahnya. Di HP: tombol menu membuka panel penuh dengan fokus terjebak di dalamnya (focus trap) dan tombol tutup yang jelas.
2. **Hero**: label HUD kecil ("STATUS: tersedia untuk magang", jika sesuai fakta), judul besar dengan animasi masuk per kata, satu kalimat peran, dua tombol (satu magnetik). Latar: lapis ambient (bagian 5).
3. **Ticker**: satu baris frasa bergerak, dua baris maksimal di seluruh halaman.
4. **Tentang**: bento grid. Tiap sel kartu dengan sorotan lokal yang mengikuti kursor.
5. **Proyek**: kartu dengan garis cahaya di tepi, kemiringan 3D kecil (maks 6 derajat) di perangkat berkursor, dan halaman detail yang terbuka dengan animasi layout (bukan pindah halaman).
6. **Keahlian**: chip dikelompokkan, masuk bertahap (stagger). Tidak ada bilah persentase kemampuan.
7. **Pencapaian**: linimasa vertikal. Garis menggambar dirinya mengikuti scroll (GSAP ScrollTrigger).
8. **Kontak**: CTA besar, tombol magnetik, tautan langsung (email, LinkedIn, GitHub, WhatsApp bila diizinkan). Tanpa formulir di fase 1 (lihat `SECURITY.md`).
9. **Footer**: tautan, hak cipta, catatan teknologi, kendali tingkat efek.

Tidak ada layar pemuatan (preloader). Preloader menunda konten dan hampir selalu dianggap mengganggu. Hero langsung tampil dengan animasi masuknya.

## 4. Sistem gerak

### 4.1 Token gerak

Satu sumber di `src/motion/tokens.ts`. Semua animasi memakainya, tidak ada angka durasi atau easing liar di komponen.

```ts
export const duration = { micro: 0.15, short: 0.3, base: 0.5, long: 0.8, hero: 1.1 } as const; // detik
export const easing = {
  out: [0.16, 1, 0.3, 1],        // masuk, melambat di akhir
  inOut: [0.65, 0, 0.35, 1],     // pindah posisi
  snap: [0.2, 0.9, 0.1, 1],      // umpan balik cepat
} as const;
export const spring = {
  soft:  { type: "spring", stiffness: 140, damping: 18, mass: 0.8 },
  firm:  { type: "spring", stiffness: 320, damping: 26, mass: 0.7 },
} as const;
export const stagger = { chars: 0.02, words: 0.06, items: 0.08 } as const;
export const distance = { reveal: 24, magnetic: 8, tilt: 6 } as const; // px, derajat
```

Untuk GSAP, petakan ke string ease setara (misalnya `power3.out`) dalam satu berkas pemetaan, jangan menyebarkannya.

### 4.2 Aturan gerak

- Animasikan hanya `transform` dan `opacity` (dan `filter` secukupnya). Jangan menganimasikan `width`, `height`, `top`, `left`, atau `margin` karena memicu perhitungan tata letak.
- Durasi: umpan balik 100 sampai 200 ms, reveal 400 sampai 700 ms, momen hero 800 sampai 1100 ms. Tidak ada animasi satu kali yang lebih dari 1,2 detik.
- Reveal saat scroll terjadi sekali per elemen, bukan setiap kali masuk layar.
- Animasi yang berulang tanpa henti (ambient, ticker) harus berhenti saat di luar layar (`IntersectionObserver`) dan saat tab tersembunyi.
- Jangan memakai `will-change` permanen. Pasang hanya saat animasi berjalan, lepas sesudahnya.
- Cleanup wajib: setiap listener, `requestAnimationFrame`, konteks GSAP (gunakan `useGSAP` atau `gsap.context` dan `revert`), dan konteks WebGL dibersihkan saat komponen dilepas.

## 5. Latar dan efek kursor

### 5.1 Lapis latar

| Lapis | Isi | Biaya | Aturan |
|---|---|---|---|
| L0 | Warna solid `--bg` | Nol | Selalu |
| L1 | Grid hias dan noise (CSS statis) | Sangat rendah | Selalu |
| L2 | Gradien ambient: 2 sampai 3 blob besar, bergerak pelan (20 sampai 40 detik) dengan `transform` | Rendah | Selalu, di-pause saat di luar layar |
| L3 | Latar WebGL atau shader dari React Bits (aurora, partikel, dan sejenisnya) | Tinggi | **Satu saja di layar**, hanya di hero, dimuat malas (`React.lazy`), punya cadangan statis |

Nama komponen React Bits berubah dari waktu ke waktu. Cari di katalog reactbits.dev (bagian Backgrounds), pilih yang paling ringan untuk hero, lalu **lapor ke pemilik** nama, dependensi, dan ukurannya sebelum memasang.

### 5.2 Efek mengikuti kursor

Pemilik meminta latar atau tulisan yang mengikuti kursor. Spesifikasinya:

| ID | Efek | Tier | Detail |
|---|---|---|---|
| C1 | Sorotan latar (spotlight) | 1 | Satu elemen tetap berisi gradien radial, digerakkan dengan `transform: translate3d` dari variabel `--mx`, `--my`. Jangan menganimasikan `background-position` atau ukuran gradien |
| C2 | Sorotan lokal pada kartu | 1 | Variabel `--x`, `--y` per kartu dihitung saat `pointermove` pada kartu itu. Hanya CSS |
| C3 | Pengikut kursor (cincin kecil) | 2 | Cincin 28 px dengan lag spring (`spring.soft`), membesar di atas elemen `[data-cursor]`. **Kursor asli tetap tampil**, pengikut hanya tambahan |
| C4 | Teks bereaksi terhadap kursor | 2 | Judul hero: tiap huruf bobotnya naik (sumbu bobot Sora) atau naik 2 sampai 4 px sesuai jarak ke kursor. Maksimal 40 huruf, hanya satu judul di seluruh halaman. Posisi huruf dihitung ulang saat resize dan scroll, bukan setiap frame |
| C5 | Tombol magnetik | 2 | Maksimal dua tombol di seluruh halaman. Pergeseran maksimal `distance.magnetic` |
| C6 | Efek fluida atau jejak WebGL | 3 | Opsional, berat. Hanya jika pemilik memintanya secara khusus. Satu efek tier 3 per halaman, mati di mode Hemat |

Aturan sistem kursor:

1. **Satu pendengar `pointermove`** di level jendela (pasif), memperbarui `--mx`, `--my` (piksel) dan `--mxp`, `--myp` (0 sampai 1) pada `document.documentElement` lewat satu `requestAnimationFrame`. Efek lain membaca variabel itu atau nilai bergerak (`MotionValue`) yang sama.
2. **Jangan memicu render ulang React pada setiap gerak kursor.** Pakai `MotionValue` atau variabel CSS, bukan `useState`.
3. Aktif hanya jika `(hover: hover) and (pointer: fine)`. Di perangkat sentuh, efek C1 sampai C6 tidak ada.
4. Mati jika `prefers-reduced-motion: reduce`.
5. Pengganti di layar sentuh (agar HP tidak terasa kosong): gradien ambient L2, kartu yang menyala saat masuk layar, riak kecil saat sentuhan (maks 400 ms), dan reveal berbasis scroll.

## 6. Anggaran animasi

Berlaku per layar (viewport) dan per halaman.

| Batas | Nilai |
|---|---|
| Animasi tanpa henti yang terlihat bersamaan | Maks 2 (L2 ambient, dan satu dari ticker atau L3) |
| Latar WebGL (L3) aktif | Maks 1, hanya hero |
| Efek kursor tier 2 aktif bersamaan | Maks 2 dari C3, C4, C5 |
| Efek kursor tier 3 | Maks 1 per halaman |
| Teks beranimasi per section | Maks 2 efek berbeda |
| Elemen kaca (`backdrop-filter`) terlihat bersamaan | Maks 2 |
| Pustaka animasi yang dipasang | Motion dan GSAP (+ pustaka khusus komponen React Bits yang disetujui) |
| Animasi masuk saat scroll | Sekali per elemen |

Uji kewajaran sebelum menambah efek, jawab tiga pertanyaan: (1) apa yang dikomunikasikan animasi ini, (2) apa yang hilang jika dihapus, (3) berapa milidetik frame yang ditambahkannya di HP kelas menengah. Kalau jawaban (2) adalah "tidak ada", hapus.

## 7. Performa dan penurunan kualitas otomatis

Tiga tingkat efek, diatur satu modul (`src/motion/governor.ts`, lihat `ARCHITECTURE.md`):

| Tingkat | Isi | Default |
|---|---|---|
| Penuh | Semua efek dalam anggaran | Desktop dengan kursor halus |
| Hemat | L1, L2, reveal, C1, C2. Tanpa L3, C3, C4, C5, C6 | HP, perangkat dengan inti CPU sedikit (indikasi, bukan kepastian), atau setelah penurunan otomatis |
| Mati | Tanpa gerak non-esensial, status akhir langsung tampil | `prefers-reduced-motion: reduce` atau pilihan pengguna |

Penurunan otomatis: ukur durasi frame selama detik pertama efek berat. Jika rata-rata frame lebih lambat dari sekitar 24 ms (di bawah kurang lebih 40 fps) berkelanjutan, turunkan ke Hemat dan hentikan efek berat. Heuristik perangkat (`hardwareConcurrency`, `deviceMemory`) tidak akurat dan tidak tersedia di semua browser, jadi ukuran frame nyata yang menentukan, bukan tebakan perangkat.

Kendali tampil di footer: "Efek: Penuh / Hemat / Mati". Pilihan disimpan di `localStorage` dengan `try/catch` dan halaman tetap benar tanpanya.

Batas tambahan: batasi `devicePixelRatio` canvas ke 1,5; hentikan loop render saat di luar layar atau tab tersembunyi; satu konteks WebGL maksimal.

## 8. Aksesibilitas

- Kontras: ikuti tabel 2.1.
- Fokus keyboard tampak jelas: `outline: 3px solid var(--accent-cyan); outline-offset: 3px`. Jangan `outline: none` tanpa pengganti.
- Semua elemen interaktif bisa dijangkau dengan Tab. Tautan "Lewati ke konten" di awal. Satu `h1`.
- Elemen semantik (`header`, `nav`, `main`, `section`, `footer`, `button`, `a`).
- Teks yang dipecah per huruf untuk animasi harus tetap terbaca pembaca layar: elemen induk punya `aria-label` berisi teks utuh dan potongan hurufnya `aria-hidden="true"`.
- Marquee yang bergerak lebih dari 5 detik bisa dijeda (tombol atau hover/fokus).
- Informasi tidak hanya disampaikan lewat warna atau gerak.
- Kursor asli tidak disembunyikan.
- `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation: none !important; transition-duration: 0.01ms !important; scroll-behavior: auto !important; }
}
```

  Di JavaScript, baca `matchMedia("(prefers-reduced-motion: reduce)")` dan masuk ke tingkat Mati, termasuk menghentikan GSAP dan loop render.

## 9. Konten

Semua konten pribadi di `src/data/profile.ts` (bentuknya di `ARCHITECTURE.md`). Tanyakan ke pemilik: nama, peran, tiga proyek terbaik dengan tautan, sertifikat atau lomba yang nyata, dan kontak yang boleh dipublikasikan. Tanpa itu, pakai `{{ISI_SENDIRI}}`. Jangan membuat proyek, testimoni, angka, atau sertifikat fiktif. Data penelitian pribadi orang lain tidak boleh tampil (lihat `SECURITY.md`).

## 10. Definisi selesai untuk desain

- Seluruh anggaran bagian 6 terpenuhi dan diverifikasi.
- Tiga tingkat efek bekerja dan bisa dipindah dari footer.
- Tidak ada scroll horizontal di 360 px, area sentuh memadai.
- Mode `prefers-reduced-motion` menghasilkan halaman yang lengkap dan terbaca tanpa gerak.
- Audit kontras dan fokus keyboard lulus (`TESTING-GUIDE.md`).
