# [PortSwigger Academy] Access Control - Lab #7

## 📌 Detail Lab
* **Nama Tantangan:** User ID controlled by request parameter
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Access Control Vulnerabilities (IDOR / BOLA)
* **Tujuan Lab:** Mengeksploitasi celah *Insecure Direct Object Reference* (IDOR) pada parameter `id` halaman akun untuk melihat API key milik pengguna `carlos`, lalu mengirimkan kunci tersebut untuk menyelesaikan tantangan.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menampilkan informasi akun pengguna berdasarkan parameter query string `id` yang dikirimkan pada URL (misalnya `/my-account?id=wiener`).

Celah keamanan terjadi karena fungsi penanganan di sisi server **hanya mengandalkan nilai parameter `id` untuk menarik data pengguna dari database tanpa memverifikasi cookie sesi milik pengguna yang sedang login**. Akibatnya, setiap pengguna yang terautentikasi dapat mengganti nilai parameter `id` dengan nama pengguna lain untuk melihat data sensitif milik akun tersebut.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Rekognisi Parameter ID
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Masuk ke halaman **My account**.
3. Amati struktur URL pada browser:
   * **URL:** `https://[LAB-ID].web-security-academy.net/my-account?id=wiener`
4. Perhatikan bahwa halaman menampilkan informasi spesifik akun `wiener`, termasuk nilai **API Key**.

---

### Eksploitasi IDOR (Parameter Tampering)
1. Tangkap request halaman akun tersebut menggunakan Burp Suite, lalu kirim ke **Burp Repeater**.
2. Ubah parameter `id` pada query string dari `wiener` menjadi `carlos`.

Request HTTP final di Burp Repeater:
```http
GET /my-account?id=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=[SESSION_WIENER]
```

### Eksekusi & Pengiriman Solusi

1. Kirim request tersebut melalui Burp Repeater.
2. Server mengembalikan respon HTTP 200 OK dan menampilkan halaman profil milik akun carlos.
3. Salin nilai API Key milik carlos yang muncul pada respons HTML.
4. Kembali ke halaman utama aplikasi di browser, klik Submit solution, lalu tempelkan API Key yang berhasil didapatkan.

Hasil Akhir: Data sensitif milik pengguna lain berhasil diakses akibat ketiadaan pemeriksaan otorisasi berbasis objek (IDOR). Lab dinyatakan selesai (Solved).

