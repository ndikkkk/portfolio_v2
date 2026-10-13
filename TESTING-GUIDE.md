# TESTING-GUIDE.md: Panduan Pengujian Portofolio Futuristik

Cara membuktikan bahwa situs memenuhi `TRD.md`. Fokus khusus: animasi, efek kursor, dan performa, karena di situlah proyek ini paling mudah rusak. Pengujian otomatis tidak menggantikan pemeriksaan di HP sungguhan.

## 1. Prinsip

1. **Bukti, bukan klaim.** Laporan berisi keluaran perintah, angka, atau tangkapan layar. "Sudah diuji" tanpa bukti tidak dihitung.
2. **Ukur sebelum dan sesudah** setiap efek berat. Tanpa pembanding, kita tidak tahu efek itu merusak atau tidak.
3. **Uji pada kondisi terburuk yang wajar:** HP kelas menengah dan CPU diperlambat, bukan hanya laptop pengembang.
4. **Pengujian otomatis hanya menangkap sebagian masalah.** Alat aksesibilitas otomatis tidak menemukan semua pelanggaran, dan WebKit di Playwright bukan Safari iOS.
5. **Menyatakan yang belum diuji** adalah bagian dari laporan yang jujur.

## 2. Alat dan persiapan

| Alat | Fungsi |
|---|---|
| `npm run build`, `npm run lint`, `tsc --noEmit` | Pemeriksaan statis |
| Vitest dan Testing Library | Uji unit dan komponen |
| Playwright Test (`@playwright/test`) | Uji e2e, responsif, aksesibilitas, reduced motion, emulasi sentuh |
| `@axe-core/playwright` | Audit aksesibilitas otomatis |
| Lighthouse | Skor performa dan aksesibilitas |
| DevTools (Performance, Rendering, Memory) | fps, paint, kebocoran |
| Playwright MCP (lewat Cline) | Agen melihat dan menangkap layar saat pengembangan. Berbeda dari Playwright Test dan hanya mengakses origin yang diizinkan |

Pasang alat uji sebagai `devDependencies`. Verifikasi nama paket dan perintah terbaru lewat Context7 sebelum memasang. Nama paket di sini diambil dari pengetahuan umum dan belum diverifikasi di sesi pembuatan dokumen ini.

Konfigurasi dasar Playwright (templat, sesuaikan dan jalankan untuk memastikan sintaksnya cocok dengan versi terpasang):

```ts
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  use: { baseURL: "http://localhost:4173" },
  webServer: {
    command: "npm run build && npm run preview -- --port 4173 --strictPort",
    url: "http://localhost:4173",
    reuseExistingServer: true,
  },
  projects: [
    { name: "chromium-desktop", use: { ...devices["Desktop Chrome"] } },
    { name: "firefox-desktop", use: { ...devices["Desktop Firefox"] } },
    { name: "webkit-desktop", use: { ...devices["Desktop Safari"] } },
    { name: "mobile-chrome", use: { ...devices["Pixel 5"] } },
    { name: "mobile-safari", use: { ...devices["iPhone 12"] } },
  ],
});
```

Pengujian dijalankan pada hasil build (`vite preview`, port 4173), karena itu yang akan dideploy. Pengembangan harian memakai `npm run dev` (port 5173).

Kait uji yang harus ada di kode (agen menambahkannya sejak P2):
- `governor` menulis `document.documentElement.dataset.effects = "full" | "lite" | "off"`.
- Pengikut kursor punya `data-testid="cursor-follower"` dan hanya ada di DOM saat aktif.
- Latar L3 punya `data-testid="bg-l3"`.

## 3. Responsif (lima lebar)

Lebar uji: 360, 390, 768, 1280, 1920 px.

Otomatis:

```ts
// tests/responsive.spec.ts
import { test, expect } from "@playwright/test";

for (const w of [360, 390, 768, 1280, 1920]) {
  test(`tanpa scroll horizontal di ${w}px`, async ({ page }) => {
    await page.setViewportSize({ width: w, height: 800 });
    await page.goto("/");
    const overflow = await page.evaluate(
      () => document.documentElement.scrollWidth - window.innerWidth
    );
    expect(overflow).toBeLessThanOrEqual(0);
  });
}
```

Manual (per lebar, dengan tangkapan layar):
- Teks tidak terpotong, tidak ada elemen saling menimpa.
- Tombol dan tautan bisa disentuh dengan nyaman (minimal 44 x 44 px).
- Menu HP: membuka, menutup (tombol dan Escape), fokus terjebak di dalam saat terbuka, fokus kembali ke tombol saat ditutup.
- Rotasi layar di HP tidak merusak tata letak.
- Hero tidak tertutup bilah alamat browser (uji `dvh`).

## 4. Uji unit dan komponen

Vitest menguji **logika**, bukan tampilan.

Governor (fungsi pemilihan tingkat, murni):

| Masukan | Hasil yang diharapkan |
|---|---|
| `prefers-reduced-motion: reduce`, pengguna memilih Penuh | `off` (preferensi sistem menang) |
| Pengguna memilih Mati | `off` |
| Pengguna memilih Hemat | `lite` |
| Otomatis, perangkat sentuh | `lite` |
| Otomatis, kursor halus, tanpa frame lambat | `full` |
| Otomatis, frame rata-rata lambat berkelanjutan | `lite`, dan tidak naik lagi dalam sesi yang sama |
| Penyimpanan tidak tersedia (melempar error) | Tidak crash, kembali ke otomatis |

Lainnya:
- Validasi protokol URL (hanya `https:`, `mailto:`, `tel:`). Uji `javascript:` ditolak.
- `Reveal`: pada tingkat `off`, konten langsung terlihat tanpa gerak.
- `EffectBoundary`: saat anak melempar error, cadangan tampil dan halaman tidak runtuh.
- Pemecahan teks per huruf: induk punya `aria-label` teks utuh, potongan huruf `aria-hidden`.

## 5. Pengujian animasi dan efek

### 5.1 Kepatuhan anggaran (manual dengan DevTools, `DESIGN.md` bagian 6)

Isi tabel ini di laporan, per section dan untuk halaman utuh:

| Batas | Nilai maks | Terukur | Lulus |
|---|---|---|---|
| Animasi tanpa henti terlihat bersamaan | 2 | | |
| Latar L3 aktif | 1 | | |
| Efek kursor tier 2 aktif bersamaan | 2 | | |
| Efek kursor tier 3 | 1 | | |
| Elemen `backdrop-filter` terlihat bersamaan | 2 | | |
| Pustaka animasi runtime | Motion, GSAP, ditambah yang disetujui | | |

Cara menghitung: DevTools, Rendering, aktifkan "Paint flashing" dan "FPS meter", scroll tiap section dan catat apa yang terus-menerus bergerak. Periksa juga animasi CSS yang berjalan lewat `document.getAnimations()` di konsol.

### 5.2 Efek kursor

Otomatis:

```ts
// tests/cursor.spec.ts
import { test, expect } from "@playwright/test";

test("pengikut kursor tidak ada di perangkat sentuh", async ({ browser }) => {
  const ctx = await browser.newContext({ hasTouch: true, isMobile: true, viewport: { width: 390, height: 844 } });
  const page = await ctx.newPage();
  await page.goto("/");
  await expect(page.getByTestId("cursor-follower")).toHaveCount(0);
  await ctx.close();
});

test("efek mati saat reduced motion", async ({ browser }) => {
  const ctx = await browser.newContext({ reducedMotion: "reduce" });
  const page = await ctx.newPage();
  await page.goto("/");
  await expect(page.locator("html")).toHaveAttribute("data-effects", "off");
  await expect(page.getByTestId("cursor-follower")).toHaveCount(0);
  await ctx.close();
});
```

Manual (desktop dengan mouse):
- Variabel `--mx` dan `--my` di elemen `<html>` ikut berubah saat mouse digerakkan (cek di DevTools, Elements).
- **Tidak ada render ulang React per gerak kursor.** Buka React DevTools Profiler, rekam sambil menggerakkan mouse selama 5 detik. Tidak boleh ada komponen yang dirender ulang karena gerak kursor.
- Kursor asli tetap tampil. Pengikut hanya tambahan.
- Pengikut membesar di atas elemen `[data-cursor]` dan kembali normal setelah keluar.
- Judul hero: hanya satu judul yang bereaksi, maksimal 40 huruf. Mengubah ukuran jendela atau scroll tidak membuat posisi huruf salah.
- Tombol magnetik maksimal dua, pergeseran tidak melebihi batas token.
- Tingkat Hemat atau Mati mematikan semua efek C3 sampai C6.

### 5.3 Reduced motion

Otomatis: uji di atas, ditambah pemeriksaan bahwa konten hero dan section terlihat (opasitas penuh) tanpa menunggu animasi.

Manual: aktifkan "Reduce motion" di sistem operasi, muat ulang, dan pastikan:
- Tidak ada gerak non-esensial (ticker berhenti, ambient berhenti, tidak ada efek kursor, tidak ada reveal bergerak).
- Semua informasi tetap lengkap dan terbaca.
- Loop WebGL tidak berjalan.

### 5.4 Pembersihan dan kebocoran

- Ganti tingkat efek Penuh, Mati, Penuh sebanyak 20 kali dari footer. Jumlah pendengar peristiwa di `window` tidak boleh terus bertambah (di konsol Chrome: `getEventListeners(window)`).
- Ambil heap snapshot sebelum dan sesudah siklus di atas. Memori tidak boleh naik terus.
- Buka dan tutup detail proyek 20 kali. Tidak ada animasi atau pengamat yang tertinggal.
- Pada `StrictMode` (pengembangan), efek dipasang dua kali. Pastikan pembersihan benar: tidak ada dua loop yang berjalan bersamaan.

### 5.5 Terlihat dan tersembunyi

- Scroll L3 keluar dari layar: loop render berhenti (cek di Performance, tidak ada frame WebGL).
- Pindah tab (tab tersembunyi): efek berhenti, dan lanjut saat kembali.

### 5.6 Kegagalan yang disengaja

- Nonaktifkan WebGL (Chromium: `--disable-gpu`, atau lewat flag DevTools) dan pastikan cadangan statis tampil tanpa error yang tidak tertangani.
- Perlambat CPU 6x di DevTools (Performance, CPU throttling). Pastikan governor menurunkan tingkat ke Hemat dalam beberapa detik.

## 6. Aksesibilitas

Otomatis:

```ts
// tests/a11y.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("tidak ada pelanggaran WCAG A dan AA yang terdeteksi axe", async ({ page }) => {
  await page.goto("/");
  const results = await new AxeBuilder({ page })
    .withTags(["wcag2a", "wcag2aa", "wcag21a", "wcag21aa"])
    .analyze();
  expect(results.violations).toEqual([]);
});
```

Jalankan juga setelah menggulir ke tiap section dan setelah membuka menu HP dan detail proyek. Axe tidak menangkap semuanya.

Manual:
- **Keyboard saja:** Tab melewati semua elemen interaktif dalam urutan logis, fokus selalu terlihat, tidak ada jebakan fokus di luar menu dan detail proyek, Escape menutup menu dan detail, tautan lewati ke konten berfungsi.
- **Kontras:** periksa pasangan warna baru dengan rumus WCAG. Teks biasa minimal 4,5:1, komponen UI dan indikator fokus minimal 3:1. Batas input memakai `--line-strong`.
- **Pembaca layar** (NVDA di Windows, VoiceOver di Mac/iPhone, TalkBack di Android): judul hero dibacakan sebagai teks utuh, bukan huruf per huruf. Section punya judul. Tombol punya nama.
- **Zoom 200 persen** dan ukuran teks besar: tidak ada konten terpotong.
- **Reduced motion:** lihat bagian 5.3.

## 7. Performa

Ukur di build produksi (`vite preview` atau preview Vercel), bukan dev server.

Lighthouse (mode HP, bawaan emulasi). Jalankan **minimal lima kali** dan catat median, karena skornya bervariasi. Jalankan di dua kondisi: tingkat Penuh dan tingkat Hemat. Contoh (periksa opsi terbaru dengan `npx lighthouse --help`):

```bash
npx lighthouse http://localhost:4173 --only-categories=performance,accessibility,best-practices,seo --output=html --output-path=./docs/lh-full.html
```

Catat: Performance, LCP, INP atau Total Blocking Time (INP tidak diukur Lighthouse lab, TBT adalah penggantinya), CLS, ukuran JavaScript awal (gzip), ukuran chunk malas.

Ukuran bundel: `npm run build` menampilkan ukuran berkas. Untuk rincian, pakai penganalisis bundel (verifikasi paketnya sebelum memasang). Catat apa saja yang masuk bundel awal.

Kelancaran (fps): di DevTools, Performance, CPU throttling 4x, rekam 5 detik sambil scroll dan menggerakkan kursor. Atau ukur cepat di konsol:

```js
(() => { let n = 0; const t0 = performance.now();
  function f() { n++; if (performance.now() - t0 < 5000) requestAnimationFrame(f);
    else console.log("fps rata-rata:", (n / 5).toFixed(1)); }
  requestAnimationFrame(f); })();
```

Catatan: fps dibatasi laju penyegaran layar. Hasilnya hanya sebagai indikasi, rekaman Performance yang menunjukkan task panjang dan paint yang mahal.

Tabel hasil (isi di laporan):

| Metrik | Target (`TRD.md`) | Hemat | Penuh |
|---|---|---|---|
| Lighthouse Performance | 90 (Hemat), 80 (Penuh) | | |
| LCP | Sampai 2,5 s | | |
| TBT atau INP | INP sampai 200 ms | | |
| CLS | Sampai 0,1 | | |
| JS awal (gzip) | Sekitar 150 KB | | |
| fps (HP, CPU 4x) | Sekitar 50 | | |

Pengujian regresi: setiap menambah efek berat, ukur ulang dan bandingkan dengan baseline di `docs/baseline.md`. Penurunan yang melewati target dilaporkan, bukan diabaikan.

## 8. Lintas browser dan perangkat nyata

Otomatis: proyek Playwright (Chromium, Firefox, WebKit, emulasi Pixel 5 dan iPhone 12) menjalankan uji responsif, aksesibilitas, dan efek.

Perangkat nyata (wajib sebelum produksi, dilakukan pemilik):

| Perangkat | Periksa |
|---|---|
| HP Android kelas menengah, Chrome | Scroll halus, menu, detail proyek, tingkat efek default (Hemat), tombol Efek di footer, tidak panas atau lag |
| iPhone, Safari | Sama seperti di atas, plus `dvh`, area aman, dan font |
| Laptop, Chrome | Semua efek tingkat Penuh |
| Laptop, Firefox atau Safari (jika ada) | Efek kursor dan latar |

Catat model perangkat, versi browser, dan temuan.

## 9. Keamanan

Dijalankan di preview Vercel, bukan lokal, karena header berasal dari `vercel.json`.

- `curl -I https://<preview-url>` memuat semua header di `SECURITY.md` bagian 3.
- Buka situs dan pantau konsol: tidak ada pelanggaran Content Security Policy. Otomatis dengan Playwright:

```ts
// tests/csp.spec.ts (jalankan terhadap URL preview)
import { test, expect } from "@playwright/test";

test("tidak ada pelanggaran CSP di konsol", async ({ page }) => {
  const violations: string[] = [];
  page.on("console", (m) => { if (m.text().includes("Content Security Policy")) violations.push(m.text()); });
  await page.goto(process.env.PREVIEW_URL ?? "/");
  await page.waitForLoadState("networkidle");
  expect(violations).toEqual([]);
});
```

- `npm audit` tanpa temuan high atau critical yang bisa diperbaiki.
- Pemindai rahasia (misalnya gitleaks) pada repo dan riwayat.
- Tidak ada permintaan jaringan ke domain pihak ketiga tak terduga (tab Network): font dan aset dari domain sendiri.
- Tautan keluar memakai `rel="noopener noreferrer"`.
- Latar WebGL dan komponen React Bits tidak membutuhkan perluasan CSP yang berlebihan (misalnya `unsafe-eval`). Jika ada komponen yang membutuhkannya, laporkan ke pemilik dan jangan melonggarkan CSP diam-diam.

## 10. Templat laporan (`docs/test-report.md`)

```
# Laporan Pengujian

Tanggal:  
Commit/tag:  
Dijalankan oleh:  
Lingkungan: (OS, browser, versi Node)

## Ringkasan
(lulus / lulus dengan catatan / gagal) dan tiga temuan terpenting.

## Hasil per bagian
| Bagian | Hasil | Bukti (tautan berkas, keluaran) | Catatan |
|---|---|---|---|

## Pengukuran performa
(tabel bagian 7)

## Kepatuhan anggaran animasi
(tabel bagian 5.1)

## Belum diuji
(jelaskan jujur)

## Penyimpangan dari TRD dan alasannya
```

## 11. Keterlacakan persyaratan ke pengujian

| Persyaratan | Diuji di |
|---|---|
| FR-1, FR-2 | Bagian 3 (manual), bagian 4 |
| FR-3 | Bagian 3 |
| FR-4 | Bagian 3 (manual), bagian 6 (keyboard) |
| FR-5, FR-13 | Bagian 5.1, 5.5 |
| FR-6 | Bagian 5.1, 5.5, 5.6 |
| FR-7, FR-8 | Bagian 5.2 |
| FR-9, FR-10 | Bagian 4 (governor), 5.4, 5.6 |
| FR-11 | Bagian 5.3 |
| FR-12 | Bagian 3 (manual), bagian 6 |
| FR-14, FR-15 | Bagian 9 dan pemeriksaan manual metadata |
| NFR-P1 sampai P8 | Bagian 7 |
| NFR-A1 sampai A7 | Bagian 6, 5.3 |
| NFR-S1 sampai S6 | Bagian 9 |
| NFR-M1 sampai M5 | Bagian 2 (lint, tipe), 5.4 |
| AN-1 sampai AN-7 | Bagian 5.1, 5.2, 5.4, 5.5 |

## 12. Daftar periksa rilis (ringkas)

- [ ] Build, lint, tipe, uji unit bersih.
- [ ] Uji responsif lulus di lima lebar.
- [ ] Axe tanpa pelanggaran, keyboard dan pembaca layar diperiksa manual.
- [ ] Reduced motion menghasilkan halaman lengkap tanpa gerak.
- [ ] Anggaran animasi terukur dan lulus.
- [ ] Tidak ada render ulang React per gerak kursor (Profiler).
- [ ] Tidak ada kebocoran setelah siklus ganti tingkat 20 kali.
- [ ] Angka performa Hemat dan Penuh dicatat, target tercapai atau penyimpangan disetujui.
- [ ] Header, CSP, `npm audit`, pemindai rahasia lulus di preview.
- [ ] Pemilik memeriksa di HP Android dan iPhone sungguhan.
- [ ] Laporan `docs/test-report.md` ditulis, termasuk bagian "Belum diuji".
