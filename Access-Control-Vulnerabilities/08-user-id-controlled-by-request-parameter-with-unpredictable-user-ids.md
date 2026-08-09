# [PortSwigger Academy] Access Control - Lab #8

## 📌 Detail Lab
* **Nama Tantangan:** User ID controlled by request parameter, with unpredictable user IDs
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities (IDOR / BOLA)
* **Tujuan Lab:** Menemukan GUID unik milik pengguna `carlos` yang bocor di halaman publik, lalu mengeksploitasi parameter `id` untuk mencuri API key milik `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi mengidentifikasi profil pengguna di halaman `/my-account?id=[GUID]` menggunakan UUID 128-bit yang acak untuk mencegah tebakan acak (*enumeration attack*).

Celah keamanan terjadi akibat dua kombinasi masalah:
1. **Information Disclosure:** Aplikasi mempublikasikan nilai GUID sensitif tersebut secara terbuka pada halaman postingan blog/komentar publik.
2. **Missing Object-Level Authorization (IDOR):** Server backend hanya memeriksa keberadaan GUID pada kueri tanpa memvalidasi apakah GUID yang diminta sesuai dengan cookie sesi pengguna yang sedang login.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Rekognisi & Pencarian GUID Target (Information Disclosure)
1. Akses halaman utama blog aplikasi web.
2. Cari postingan blog yang ditulis oleh pengguna **carlos** (atau komentar yang dibuat oleh `carlos`).
3. Klik nama penulis `carlos` atau periksa kode sumber (*source code*) HTML pada postingan tersebut.
4. Perhatikan URL profil penulis yang terbuka:
   * **URL:** `/blogs?authorId=882ef948-e8a0-4a87-87eb-1254bf3e5c93`
5. Catat nilai GUID unik milik `carlos` tersebut (misal: `882ef948-e8a0-4a87-87eb-1254bf3e5c93`).

---

### Eksploitasi IDOR
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Masuk ke halaman **My account**, lalu tangkap request tersebut di Burp Suite dan kirim ke **Burp Repeater**.
3. Ganti nilai parameter `id` milik `wiener` pada query string dengan GUID milik `carlos` yang didapatkan dari Langkah A.

Request HTTP final di Burp Repeater:
```http
GET /my-account?id=882ef948-e8a0-4a87-87eb-1254bf3e5c93 HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=[SESSION_WIENER]
```

### Eksekusi & Pengiriman Solusi
1. Kirim request tersebut melalui Burp Repeater.
2. Server mengembalikan respon HTTP 200 OK dan menampilkan halaman akun profil milik carlos.
3. Salin nilai API Key milik carlos yang tertera pada respons HTML.
4. Klik tombol Submit solution di header aplikasi web, lalu tempelkan API Key tersebut.

Hasil Akhir: Meskipun IDOR dilindungi oleh ID yang tidak dapat ditebak, kebocoran GUID di area publik memungkinkan akses tidak sah ke data sensitif. Lab dinyatakan selesai (Solved).