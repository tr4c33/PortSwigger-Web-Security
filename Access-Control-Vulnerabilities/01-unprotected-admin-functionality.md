# [PortSwigger Academy] Access Control - Lab #1

## 📌 Detail Lab
* **Nama Tantangan:** Unprotected admin functionality
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Menemukan panel admin yang tidak terlindungi, lalu menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi web menyediakan antarmuka administratif untuk mengelola pengguna. Namun, aplikasi ini mengalami celah **Unprotected Admin Functionality** karena pengembang hanya menyembunyikan tautan panel admin dari antarmuka pengguna biasa tanpa menerapkan pemeriksaan otorisasi (*Access Control Check*) di sisi server backend.

Siapa saja yang berhasil menemukan lokasi URL panel admin dapat langsung mengakses fitur administratif tanpa perlu login sebagai admin.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Rekognisi & Menemukan URL Tersembunyi
1. Buka aplikasi web di browser.
2. Periksa file `robots.txt` aplikasi dengan mengakses jalur `/robots.txt` pada URL browser:
   * **URL:** `https://0a990082031abfe882a44c4d00f70093.web-security-academy.net/robots.txt`
3. Di dalam file `robots.txt`, ditemukan aturan yang menyembunyikan direktori sensitif dari mesin pencari:
   ```text
   User-agent: *
   Disallow: /administrator-panel
   ```

### Akses Panel Admin & Eksekusi Target
Buka jalur direktori yang ditemukan tersebut langsung di browser:

1. URL: https://0a990082031abfe882a44c4d00f70093.web-security-academy.net/administrator-panel

2. Halaman panel administratif terbuka secara penuh tanpa meminta login atau verifikasi hak akses.

3. Cari akun carlos pada daftar pengguna, lalu klik tombol Delete.

Request HTTP yang dikirimkan oleh browser saat menghapus user:
```
GET /administrator-panel/delete?username=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
```

### Hasil Eksploitasi
Server menerima perintah tersebut dan merespons dengan status HTTP 302 Found, lalu menghapus akun carlos dari database.

Hasil Akhir: Akun carlos sukses dihapus dan lab dinyatakan selesai (Solved).