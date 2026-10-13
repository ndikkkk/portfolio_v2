# AGENT-CLINE.patch.md: perubahan untuk AGENT-CLINE.md

---

## [GANTI] Bagian 2, daftar membaca

Tambahkan satu kalimat di akhir bagian: "Berkas `*.patch.md` yang sudah digabung ke dokumen utama tidak perlu dibaca lagi. Jika masih ada berkas patch yang belum digabung, baca dan laporkan ke pemilik, jangan memilih diam-diam."

---

## [GANTI] Bagian 3.5, tiga baris

Ganti baris terakhir ("Anime.js dan React Spring: tidak dipasang") dan tambahkan:

- Anime.js dan React Spring: tidak dipasang (lihat `DESIGN.md` bagian 1).
- Lenis (paket `lenis`, MIT): **disetujui pemilik** sebagai pustaka runtime ketiga. Periksa versi, lisensi, ukuran, dan skrip `postinstall` sebelum memasang, laporkan, lalu pasang. Impor dinamis di `lenis.ts`.
- Pustaka scroll atau animasi lain (Locomotive, ScrollMagic, Framer Motion lama, dan sejenisnya): tidak dipasang.
- Alat penghapus latar atau penyunting foto: **bukan tugasmu.** Jangan mengunggah foto pemilik ke layanan mana pun. Kamu hanya memasang berkas hasil olahan yang diserahkan pemilik.

---

## [GANTI] Bagian 4, aturan 9 dan tambahan aturan 12 sampai 17

Ganti aturan 9:

9. Loader dan Lenis hanya sesuai batas di `DESIGN.md` 3.1 dan 4.3. Lenis hanya dimulai dan dihentikan oleh `governor` lewat `lenis.ts`, digerakkan `gsap.ticker`. Tidak ada `scroll-behavior: smooth`, tidak ada `window.scrollTo` langsung, tidak ada pustaka scroll kedua.

Tambahkan:

12. Loader: nyata (menunggu font dan gambar hero), 0,6 sampai 1,5 detik, sekali per sesi, bisa dilewati, failsafe 3 detik, tidak ada di reduced motion. Jangan menambah animasi di loader di luar `DESIGN.md` 3.1.
13. Parallax: hanya lapis dekoratif, maks 3 terlihat, maks 80 px, hanya `transform`, lewat `gsap.matchMedia`. Teks bacaan tidak di-parallax.
14. Section horizontal: hanya Proyek, hanya `(min-width: 1024px) and (hover: hover) and (pointer: fine)` di tingkat Penuh. Fokus keyboard wajib menggulirkan kartu ke layar. Tidak ada animasi layout Motion di dalam jalur yang `transform`-nya digerakkan GSAP.
15. Hover, micro-interaction, dan reveal: hanya dari katalog `DESIGN.md` 4.6. Mau menambah satu? Tulis tugasnya, pemiliknya, dan dampak anggarannya, lalu tanya pemilik.
16. Potret: jangan memproses, mengubah, atau mengunggah foto pemilik. Pasang berkas yang diserahkan. Verifikasi dengan `exiftool` bahwa tidak ada GPS dan laporkan hasilnya. Jika berkas belum ada, pakai gambar sementara dan katakan itu sementara.
17. Setiap kali menambah efek yang terhubung scroll, jalankan `ScrollTrigger.refresh()` setelah `appReady` dan uji kebocoran (`ScrollTrigger.getAll()` kembali nol setelah unmount atau pindah tingkat).

Ganti aturan 10 dengan: "Anggaran di `DESIGN.md` bagian 6 (versi yang sudah diperbarui) tidak boleh dilampaui tanpa persetujuan. Hitung ulang setelah menambah efek, jangan mengasumsikan."

Aturan 11 (jangan menyatakan "halus" dari tangkapan layar) berlaku lebih keras sekarang. Kehalusan scroll Lenis, rasa loader, dan gerak parallax hanya bisa dinilai pemilik di perangkat nyata. Laporkan angka (fps, LCP, TBT), bukan kesan.

---

## [GANTI] Bagian 5.2, langkah 4

Pemeriksaan wajib per fase ditambah: pengecekan jumlah `ScrollTrigger` dan instans Lenis (kait pengujian pengembangan), dan pengecekan scroll horizontal di 360 px.

---

## [TAMBAH] Bagian 6, baris tabel

| Godaan | Yang benar |
|---|---|
| Menyalakan Lenis di HP "supaya sama" | Mati di sentuh dan Hemat |
| Membuat loader lebih lama supaya "terlihat mewah" | Maks 1,5 detik, ukur LCP |
| Menambah satu section horizontal lagi | Hanya Proyek |
| Memberi `blur()` CSS pada foto besar | Blur dibuat di berkas gambar |
| Membiarkan `scroll-behavior: smooth` di samping Lenis | Dilarang, satu pemilik scroll |
| Menyatakan scroll "halus" dari tangkapan layar | Laporkan angka, serahkan penilaian ke pemilik |
| Mengunggah foto pemilik ke penghapus latar daring | Bukan tugasmu, foto diolah pemilik |

---

## [GANTI] Bagian 9, prompt pembuka

Tambahkan di akhir prompt:

```
Dokumen sudah diperbarui untuk loader, Lenis, parallax, section horizontal,
dan potret hero. Ada fase baru P3B. Baca bagian terkait di DESIGN.md
(3.1, 3.2, 4.3 sampai 4.6), ARCHITECTURE.md (5.6, 5.7), dan TRD.md (FR-16 sampai FR-24).
Jangan memasang lenis sebelum melaporkan versi, lisensi, dan ukurannya.
```

---

## Catatan untuk MCP-SETUP.md

Tidak ada perubahan konfigurasi. Satu tambahan di bagian 4 "Pemakaian per tugas": contoh pertanyaan Context7 untuk "Cara sinkron Lenis dengan GSAP ScrollTrigger di React, use context7" dan "ScrollTrigger pin horizontal dengan gsap.matchMedia, use context7". Context7 sudah menjadi sumber dokumentasi yang diwajibkan sebelum menulis kode Lenis atau ScrollTrigger.
