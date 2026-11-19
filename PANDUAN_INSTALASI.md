# PANDUAN INSTALASI - OBSERVA DIGITAL v2 (NETLIFY)

**Penting:** OBSERVA DIGITAL v2 **tidak memerlukan Google Sheet, Google Apps Script, atau Supabase**. Data disimpan di **Netlify Blobs site-wide store** bernama `observa-digital-v2`. Setiap penilaian disimpan terpisah. Penyimpanan site-wide bertahan ketika kode web diperbarui, selama memakai situs Netlify yang sama. Data tidak disimpan dalam file HTML, localStorage, atau browser observer.

## A. Isi paket

- `public/index.html`, `public/style.css`, `public/app.js`: website formulir observer dan area admin, responsif HP/laptop.
- `netlify/functions/api.mjs`: endpoint simpan data, login admin, analisis, backup dan PDF.
- `lib/core.mjs`: instrumen, validasi, statistik, rekomendasi berbasis skor dan catatan.
- `lib/pdf.mjs`: pembuat file PDF A4 yang dapat dipreview dan diunduh.
- `package.json`, `netlify.toml`, `.nvmrc`: konfigurasi Netlify.
- `tests/`: pengujian logika instrumen, perhitungan, serta PDF.

## B. Mengunggah kode ke Netlify - metode utama (GitHub)

> **Jangan langsung menyeret file HTML atau ZIP ini ke Netlify Drop**. Aplikasi memerlukan fungsi server Netlify dan pustaka `@netlify/blobs`. Deployment drag-and-drop biasa tidak menjalankan build; halaman mungkin tampil tetapi penyimpanan/login bisa gagal.

1. Ekstrak ZIP. Di GitHub, buat **repository baru** (sebaiknya **Private**). Unggah **isi folder** `OBSERVA_DIGITAL_NETLIFY_v2` sebagai isi repository, sehingga `package.json` dan `netlify.toml` ada di root.
2. Buka [Netlify](https://app.netlify.com/) → **Add new project → Import an existing project** → GitHub → pilih repository tersebut.
3. Netlify akan membaca `netlify.toml`. Pastikan **Build command** `npm run build`, **Publish directory** `public`, **Functions directory** `netlify/functions`.
4. **Sebelum digunakan publik**, tambahkan variabel lingkungan di **Project configuration → Environment variables** (atau bagian Environment variables yang tersedia pada UI Netlify):

| Nama | Contoh pengisian dan fungsinya |
|---|---|
| `ADMIN_PASSWORD` | Kata sandi hanya untuk admin. **Gunakan sandi kuat minimal 12-16 karakter**. Jangan gunakan `@123456` pada aplikasi publik. |
| `OBSERVER_ACCESS_CODE` | Kode yang dibagikan kepada observer (contoh: kode acak panjang yang tidak sama dengan sandi admin). |
| `SESSION_SECRET` | Nilai acak kuat minimal **32 karakter**, disarankan 64 karakter heksadesimal. Buat menggunakan `openssl rand -hex 32` pada terminal, atau generator kata sandi kriptografis tepercaya. |

5. Atur nilai **untuk Functions / seluruh scope** sesuai menu. Simpan lalu lakukan **Trigger deploy / Deploy site** ulang. Perubahan environment perlu deployment baru agar fungsi server menerima nilainya.
6. Buka URL `https://NAMA-SITUS.netlify.app/`. Halaman observer muncul tanpa login, tetapi **kode observer diwajibkan saat mengirim**. Area Admin memakai `ADMIN_PASSWORD`.
7. Uji menggunakan **data percobaan**, cek hasil masuk dashboard lalu **hapus isian percobaan lewat tombol Hapus** setelah cadangkan data jika diperlukan.

**Alternatif untuk yang terbiasa command-line:** instal Node.js 22 dan Netlify CLI, masuk ke direktori proyek, jalankan `npm install`, lalu `npx netlify login`, `npx netlify init` atau `npx netlify link`, lalu `npx netlify deploy --build --prod`. Pastikan variabel lingkungan sudah diatur pada situs Netlify yang benar. Metode Git lebih mudah untuk pembaruan rutin.

## C. Alur observer

1. Buka tautan publik OBSERVA DIGITAL; pilih **Isi Observasi**.
2. Isi sekolah, guru, mapel, kelas, tanggal, topik, **kode sesi** dan nama/peran observer.
3. Untuk **setiap** indikator I01-I12 pilih 1, 2, 3, 4, atau N/O. Ada **satu kolom catatan di bawah setiap indikator**. Catatan untuk skor 1, 2, dan N/O wajib minimal delapan karakter. Untuk skor 3/4 sangat dianjurkan menulis bukti spesifik.
4. Opsional: tulis bukti praktik umum, kekuatan, perbaikan, hambatan.
5. Isi kode observer dari admin. Klik **Kirim Observasi**. Simpan nomor tanda terima `OBS-XXXXXXXX` bila diperlukan.

**Penyamaan sesi sangat penting**: observer berbeda yang mengamati kelas dan waktu yang sama harus menggunakan sekolah, nama guru, mapel, kelas, tanggal, dan **kode sesi yang sama**. Contoh `VIIA-2026-11-19-P1`.

Sistem mencegah duplikat pengisian dari nama/peran observer yang sama pada satu sesi. Hanya admin yang dapat menghapus sebuah isian agar dapat dikirim ulang bila keliru.

## D. Cara membaca dashboard admin

- Masuk **Area Admin**, lalu login dengan `ADMIN_PASSWORD`.
- Dashboard berisi jumlah guru, sesi, total isian, skor keseluruhan, skor tiga aspek.
- Klik **Lihat analisis** pada guru untuk meninjau seluruh 12 indikator, skor gabungan, **catatan per indikator dari masing-masing observer**, kekuatan, prioritas perbaikan, rekomendasi, simpulan dan rincian sesi.
- **Preview PDF Rekap** menampilkan PDF gabungan seluruh guru. Di halaman detail, **Preview PDF** dan **Unduh PDF** menampilkan laporan per guru. PDF hanya bisa diminta jika sudah login admin.
- Gunakan **Unduh Backup JSON** berkala, misalnya setiap selesai satu siklus observasi. Data sekolah merupakan data internal; simpan backup dengan aman.

## E. Rumus olah data dan kategori

1. N/O = tidak ada skor; **tidak dihitung sebagai angka nol**.
2. Untuk setiap sesi dan indikator, rerata **Rekan Sejawat** dihitung sendiri, demikian juga **Observer Internal**.
3. Jika kedua kelompok menilai indikator yang sama, nilai akhir indikator sesi = `(rerata sejawat + rerata internal) / 2`. Bila hanya satu kelompok memiliki skor, gunakan kelompok tersebut.
4. Untuk setiap guru, nilai indikator = rerata nilai indikator dari **semua sesi** guru tersebut. Nilai setiap aspek = rerata empat indikatornya yang dapat diamati. Rerata guru = rerata semua indikator yang dapat diamati.
5. **Kategori internal**: 3,50-4,00 Sangat baik; 2,75-3,49 Baik; 2,00-2,74 Berkembang; 1,00-1,99 Perlu pendampingan.
6. Tiga indikator kuat = skor >= 3, diurutkan tertinggi. Tiga prioritas = skor < 3, diurutkan terendah. Rekomendasi dirumuskan dari butir rubrik, serta **kutipan catatan nyata observer** ditampilkan sebagai bukti. Kesimpulan bersifat **deskriptif**, bukan bukti dampak terhadap hasil belajar.

**Penting:** rekomendasi **berbasis aturan dan bukti catatan**, **bukan ringkasan AI atau analisis semantik otomatis** terhadap isi catatan bebas. Admin tetap perlu memvalidasi konteks bersama guru.

## F. Privasi, keamanan, dan batasan

- Website observer tidak memiliki endpoint pembaca data. Pengambilan hasil, catatan, backup, dan PDF membutuhkan **cookie sesi admin yang diperiksa di server** (HttpOnly, Secure, SameSite=Strict, 2 jam).
- Kode observer **berbeda** dengan sandi admin. Keduanya disimpan sebagai environment variables pada server, **tidak ditaruh pada kode publik**.
- Percobaan salah login dibatasi secara dasar (maksimal lima sebelum terkunci 15 menit per alamat IP). Ini **bukan pengganti** MFA atau sistem identitas organisasi untuk penyebaran berskala besar.
- Peran observer dipilih sendiri dari formulir. Jika pengesahan peran diperlukan, tambahkan daftar akun dan verifikasi terpusat pada versi mendatang.
- Semua data ada dalam Netlify Blobs. Fitur Blobs di akun Free mempunyai kuota/batas layanan yang dapat berubah sesuai kebijakan Netlify. Sebaiknya tidak menyimpan NIK, informasi kesehatan siswa, atau data sensitif di catatan.
- Paket dioptimalkan untuk penggunaan observasi skala sekolah/pelatihan (maksimal sekitar 3.000 penilaian yang dimuat per laporan). Di atas batas itu pertimbangkan database relasional dan pagination server.
- Saat pindah ke situs Netlify **baru**, Blobs **tidak ikut otomatis berpindah**. Unduh backup dari dashboard dan rencanakan migrasi sebelum mengganti situs.
- Aplikasi belum terpasang secara online sampai Bapak melakukan deploy dan mengatur environment variables. `npm test` memverifikasi fungsi analisis dan pembuat PDF secara lokal, tetapi hasil deploy perlu diuji di Netlify karena konteks Blobs ditentukan oleh platform.

## G. Jika ada masalah

| Gejala | Penyebab umum / tindakan |
|---|---|
| Halaman tampil, form tidak bisa mengirim | Biasanya fungsi server tidak ikut ter-deploy (drag-and-drop HTML saja) atau `OBSERVER_ACCESS_CODE` belum diisi. Deploy via Git dan cek Functions. |
| Pesan konfigurasi belum lengkap | Isi `ADMIN_PASSWORD`, `OBSERVER_ACCESS_CODE`, `SESSION_SECRET`, kemudian redeploy. |
| Login admin gagal | Pastikan sandi environment persis, tanpa spasi tambahan. Jika berubah, redeploy lalu login ulang. |
| Observer dianggap duplikat | Nama, peran, guru, kelas, mapel, tanggal dan kode sesi identik. Minta admin menghapus isian lama setelah backup jika perlu revisi. |
| Rekap skor guru terpisah menjadi dua | Ejaan sekolah/nama guru tidak konsisten. Gunakan satu format baku. |
| PDF kosong | Isi data observasi dulu, lalu login admin dan tekan Preview PDF. Jangan gunakan link file PDF tanpa sesi admin. |
| Data tidak muncul setelah pindah alamat | Pastikan tetap pada situs Netlify yang sama; Netlify Blobs bersifat per situs. |

**Dokumentasi teknis:** https://docs.netlify.com/build/data-and-storage/netlify-blobs/ dan https://docs.netlify.com/build/functions/get-started/.
