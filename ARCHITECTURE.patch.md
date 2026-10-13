# ARCHITECTURE.patch.md: perubahan untuk ARCHITECTURE.md

---

## [TAMBAH] Bagian 1, diagram Mermaid

Di subgraph `MOTIONLAYER`, tambahkan dua node dan satu tautan (jangan memakai id `M` atau `S` ganda):

```
LEN["lenis.ts (scroll halaman)"]
LD["Loader + appReady"]
GS --> LEN
LD --> R
```

(`GS` dan `R` adalah id node gsap-setup dan Reveal yang sudah ada. Sesuaikan jika id di diagrammu berbeda.)

---

## [TAMBAH] Bagian 2, baris tabel

| Keputusan | Pilihan | Alasan |
|---|---|---|
| Smooth scroll | Lenis (`lenis`, MIT), digerakkan oleh `gsap.ticker` | Satu detak untuk scroll dan ScrollTrigger, tanpa dua loop `requestAnimationFrame` yang saling mendahului |
| Parallax dan pin | GSAP ScrollTrigger (`gsap.matchMedia`) | Kondisi responsif dan pembersihan otomatis |

---

## [TAMBAH] Bagian 3, pohon direktori

Tambahkan berkas berikut (di dalam folder yang sudah ada):

```
src/motion/lenis.ts                 # instans dan siklus hidup Lenis
src/motion/appReady.ts              # promise kesiapan (font, gambar hero, loader)
src/components/Loader.tsx
src/components/ParallaxLayer.tsx
src/components/HorizontalSection.tsx
src/components/HeroPortrait.tsx
public/img/hero-portrait.webp       # hasil olahan, tanpa EXIF
```

---

## [GANTI] Bagian 4, tabel kepemilikan: tambah dua baris dan ubah satu

| Pemilik | Wilayah | Contoh |
|---|---|---|
| Lenis | **Scroll halaman** (posisi window). Tidak menganimasikan elemen | Smooth wheel, `scrollTo` untuk anchor dan kembali ke atas |
| GSAP | Hal yang bergantung pada posisi scroll, pin, parallax, pemecahan teks, `gsap.ticker` sebagai detak | Linimasa yang tergambar, parallax, jalur horizontal Proyek, judul per kata |
| Motion | (tetap) Elemen React masuk dan keluar, layout, gestur, nilai bergerak | Tilt kartu, tombol magnetik, kedalaman potret, dialog detail |

Aturan tambahan:
- Tidak ada yang lain yang boleh menggeser posisi window (`window.scrollTo`, `scrollIntoView` langsung, `scroll-behavior: smooth`). Semuanya lewat modul `lenis.ts` yang jatuh ke perilaku native saat Lenis mati.
- Bilah progres scroll membaca posisi scroll native (Lenis memperbarui posisi native). Kompatibilitasnya diuji, bukan diasumsikan.
- Elemen yang `transform`-nya digerakkan GSAP (jalur horizontal, lapis parallax) tidak boleh punya anak yang memakai animasi layout Motion.

---

## [TAMBAH] Bagian 5.6 `lenis.ts` dan 5.7 `appReady.ts`

### 5.6 `lenis.ts`

Sinkron dengan ScrollTrigger mengikuti README resmi Lenis (periksa ulang lewat Context7 karena API bisa berubah):

```ts
import Lenis from "lenis";
import { gsap, ScrollTrigger } from "./gsap-setup";

let lenis: Lenis | null = null;
let tick: ((time: number) => void) | null = null;

export function startLenis(): Lenis {
  if (lenis) return lenis;
  lenis = new Lenis({ autoRaf: false });          // detak dari gsap.ticker
  lenis.on("scroll", ScrollTrigger.update);
  tick = (time: number) => lenis?.raf(time * 1000); // ticker dalam detik, Lenis dalam milidetik
  gsap.ticker.add(tick);
  gsap.ticker.lagSmoothing(0);
  return lenis;
}

export function stopLenis(): void {
  if (!lenis) return;
  if (tick) gsap.ticker.remove(tick);
  lenis.off("scroll", ScrollTrigger.update);
  lenis.destroy();
  lenis = null;
  tick = null;
  ScrollTrigger.refresh();
}

export const getLenis = () => lenis;

export function scrollToTarget(el: Element | string, offset = 0) {
  if (lenis) lenis.scrollTo(el as HTMLElement, { offset });
  else (typeof el === "string" ? document.querySelector(el) : el)?.scrollIntoView();
}
```

Aturan:
- Hanya `governor` yang memanggil `startLenis` dan `stopLenis`: mulai jika `tier === "full"` dan pointer halus; hancurkan di kondisi lain, termasuk saat turun otomatis ke Hemat.
- Pembersihan wajib untuk React StrictMode (efek dijalankan dua kali di pengembangan): `startLenis` idempoten dan `stopLenis` aman dipanggil berulang.
- `lagSmoothing(0)` mengubah perilaku ticker GSAP secara global. Dicatat di sini agar tidak mengejutkan siapa pun. Dipasang hanya selama Lenis hidup (kembalikan ke bawaan di `stopLenis` bila terbukti perlu, diuji).
- Impor CSS Lenis sesuai dokumentasi (berkas CSS bawaan paket). Verifikasi nama berkasnya lewat Context7, jangan menebak.

### 5.7 `appReady.ts` dan Loader

```ts
// Gambaran, bukan kode final
export const appReady: Promise<void> = Promise.all([
  document.fonts.ready,
  heroImageDecoded(),          // img.decode() dengan batas waktu
]).then(() => {});
```

- `Loader` menunggu `appReady` dan batas minimum 0,6 detik, lalu keluar. Failsafe 3 detik: `Promise.race` dengan timer.
- Setelah keluar, `Loader` memancarkan `loader:done`. Timeline hero GSAP (R4) dan `ScrollTrigger.refresh()` dijalankan **setelah** itu.
- `sessionStorage` untuk penanda "sudah tampil" selalu dalam `try/catch`. Jika gagal, loader boleh tampil lagi (aman).
- Jangan menaruh logika kerja nyata (fetch, perhitungan berat) di balik loader. Loader hanya menunggu aset yang memang dibutuhkan hero.

---

## [GANTI] Bagian 8, ADR 5 dan tambahan

Ganti baris 5:

| 5 | Loader singkat (maks 1,5 detik, sekali per sesi, bisa dilewati, nyata) dan Lenis hanya di tingkat Penuh dengan kursor halus | Permintaan pemilik. Risiko: loader menunda konten dan LCP; smooth scroll mengubah rasa scroll dan bisa merusak aksesibilitas | Loader dan Lenis dibatasi dan punya jalur mati. Keduanya dihapus jika ukuran LCP atau uji aksesibilitas gagal |

Tambahkan:

| 8 | Lenis digerakkan `gsap.ticker`, bukan loop sendiri | Satu detak, ScrollTrigger tidak tertinggal dari posisi scroll | `lagSmoothing(0)` global, modul `lenis.ts` jadi satu-satunya pintu |
| 9 | Proyek horizontal hanya di desktop Penuh, tumpukan vertikal di tempat lain | Pin horizontal buruk di sentuh dan bagi keyboard | Dua tata letak untuk satu data, keduanya diuji |
| 10 | Potret diolah offline (cutout, mask, blur tepi), bukan lewat CSS runtime | `filter: blur` besar mahal, dan blend mode tidak bisa memisahkan langit dari subjek | Pemilik menyiapkan berkas, Cline hanya memasangnya |

---

## [GANTI] Bagian 9, tambah satu baris di tabel dependensi

| `lenis` | Smooth scroll | MIT (menurut halaman GitHub darkroom.engineering/lenis) | Dimuat hanya di tingkat Penuh (impor dinamis). Periksa versi terbaru dan `postinstall` sebelum memasang, sesuai `AGENT-CLINE.md` 3.5. Halaman GitHub menyebut versi 1.3.x, versi yang dipasang diverifikasi saat pemasangan |

Catatan bundel: karena Lenis hanya untuk tingkat Penuh, impor dinamis (`import("lenis")`) di dalam `startLenis` menjaga bundel awal HP tetap kecil. Jika diubah ke impor dinamis, `startLenis` menjadi `async`. Sesuaikan kode di 5.6.
