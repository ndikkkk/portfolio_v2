# SECURITY.patch.md: perubahan untuk SECURITY.md

Semua tambahan masuk di bagian 10.

---

## [GANTI] Bagian 10.5, lisensi pustaka animasi (tambah baris)

- Lenis (`lenis`): MIT menurut halaman GitHub darkroom.engineering/lenis. Baca berkas `LICENSE` di repo mereka sendiri. Saya tidak memberi nasihat hukum.
- Tetap: GSAP (lisensi standar gratis, larangan utamanya terkait pembuatan pembangun animasi visual pesaing), Motion (MIT), React Bits (menurut halaman GitHub: MIT dengan Commons Clause).

---

## [TAMBAH] 10.6 tambahan (ketahanan)

- `sessionStorage` untuk penanda loader selalu dalam `try/catch`. Jika gagal, loader boleh tampil lagi.
- Failsafe loader 3 detik adalah kontrol ketahanan: gambar hero yang gagal dimuat tidak boleh membuat situs terkunci di balik loader.
- Lenis dan ScrollTrigger harus dihancurkan saat pindah tingkat efek. Pendengar scroll yang tertinggal menumpuk dan memperlambat halaman (bentuk penolakan layanan oleh diri sendiri).

## [TAMBAH] 10.7 Foto pribadi di situs publik

1. **Metadata:** hapus EXIF sebelum berkas masuk repo, terutama koordinat GPS, tanggal, dan model perangkat. Verifikasi dengan `exiftool` dan lihat hasilnya sendiri. Foto outdoor dari ponsel hampir selalu membawa GPS. Git menyimpan riwayat selamanya, jadi hapus **sebelum** commit pertama yang memuat berkas itu. Jika terlanjur ter-commit, membersihkan riwayat butuh langkah khusus dan jangan dilakukan oleh agen tanpa persetujuan.
2. **Penghapus latar daring:** layanan semacam itu mengunggah wajahmu ke server pihak ketiga dan syarat penyimpanannya berbeda-beda. Pilih alat lokal bila ragu. Agen tidak mengunggah foto ke layanan mana pun.
3. **Foto wajah di situs publik tidak bisa ditarik kembali.** Foto bisa diunduh, diindeks mesin pencari, dan dipakai ulang oleh pihak lain (termasuk untuk akun palsu). Mitigasi: resolusi sekitar 1200 px (cukup untuk layar, kurang untuk cetak), tanpa nama berkas yang memuat nama lengkap dan kota, dan keputusan sadar dari pemilik bahwa foto itu boleh publik. Pemilik memutuskan, bukan agen.
4. Foto adalah milik pemilik sehingga tidak ada masalah hak cipta. Foto orang lain di latar harus dihapus lewat cutout, atau diburamkan, dan jangan mempublikasikan wajah orang lain tanpa izin.
5. Berkas hasil olahan dan berkas asli disimpan terpisah. Berkas asli (yang masih memuat EXIF) tidak masuk repo. Tambahkan jalur berkas asli ke `.gitignore` bila disimpan di folder proyek.

## [TAMBAH] 10.8 Daftar periksa tambahan (bagian 7)

- [ ] `exiftool` pada semua gambar di `public/` tidak menampilkan GPS
- [ ] Tidak ada berkas foto asli di repo (`git log --all -- <jalur>` kosong)
- [ ] `lenis` dipasang setelah pemeriksaan versi, lisensi, dan `postinstall`
- [ ] Tidak ada pustaka scroll tambahan di `package.json`
- [ ] Loader punya failsafe dan `try/catch` pada penyimpanan
- [ ] CSP tidak dilonggarkan untuk loader, Lenis, atau potret (semuanya berasal dari origin sendiri)
