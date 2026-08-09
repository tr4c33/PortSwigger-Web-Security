# [PortSwigger Academy] Access Control - Lab #2

## 📌 Detail Lab
* **Nama Tantangan:** Unprotected admin functionality with unpredictable URL
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Menemukan URL panel admin yang tersembunyi/tidak dapat ditebak melalui analisis sumber halaman (*source code*), lalu menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi mencoba menerapkan keamanan berbasis kerahasiaan (*Security through obscurity*) dengan membuat path URL panel admin yang acak dan tidak umum.

Celah keamanan terjadi karena:
1. **Information Disclosure:** Script sisi klien (JavaScript/HTML) memuat referensi variabel atau logika yang membocorkan path URL admin tersembunyi tersebut ke publik.
2. **Missing Otorisasi:** Backend server tidak memvalidasi peran (*role*) pengguna yang mengakses endpoint tersebut.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Analisis Kode Sumber (Source Code Review)
1. Buka halaman utama aplikasi web di browser.
2. Tampilkan kode sumber halaman (*View Page Source* atau tekan `Ctrl + U`).
3. Cari skrip JavaScript internal atau tag `<script>` di dalam HTML.
4. Temukan blok kode JavaScript yang menentukan kondisi logika pengguna (contoh):
   ```javascript
   
    if (isAdmin) {
        var topLinksTag = document.getElementsByClassName("top-links")[0];
        var adminPanelTag = document.createElement('a');
         adminPanelTag.setAttribute('href', '/admin-l5mnhf')
   }
   ```
5. Catat path unik panel admin yang ditemukan 

### Eksploitasi & Eksekusi Perintah
1. Salin path URL unik tersebut dan akses langsung melalui address bar browser:

        URL: web-security-academy.net/admin-l5mnhf

2. Halaman panel kontrol administratif terbuka penuh tanpa meminta otentikasi admin.
3. Cari akun carlos dari daftar pengguna, lalu klik tombol Delete.

Request HTTP yang dikirimkan ke server:
```
GET /admin-3g6y8a/delete?username=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
```

### Hasil Eksploitasi

Server memproses permintaan dan mengembalikan respons HTTP 302 OK. Akun carlos berhasil dihapus dari sistem.

Hasil Akhir: Pembocoran informasi path tersembunyi pada skrip klien berhasil dimanfaatkan untuk memicu kontrol akses yang tidak terlindungi. Lab dinyatakan selesai (Solved).