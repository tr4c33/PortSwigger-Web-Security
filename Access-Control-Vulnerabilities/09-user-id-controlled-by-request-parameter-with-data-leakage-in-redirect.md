# [PortSwigger Academy] Access Control - Lab #9

## 📌 Detail Lab
* **Nama Tantangan:** User ID controlled by request parameter with data leakage in redirect
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities (IDOR / Information Disclosure)
* **Tujuan Lab:** Mencuri API key milik pengguna `carlos` dengan memanfaatkan kebocoran data (*data leakage*) yang terdapat pada *body response* HTTP 302 Redirect.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memvalidasi apakah pengguna yang sedang login berhak melihat halaman profil yang diminta (`/my-account?id=[username]`). Jika terjadi ketidakcocokan sesi, server backend akan memicu *redirect* ke halaman login.

Celah keamanan terjadi karena **logika eksekusi program di sisi server tidak dihentikan secara langsung setelah header `Location: /login` ditulis** (misalnya lupa menggunakan fungsi `return` atau `exit()` dalam skrip PHP/Framework). Akibatnya, server tetap memproses kueri database dan merender seluruh isi *body response* HTML profil pengguna target sebelum mengirimkan paket respons HTTP 302 tersebut ke klien.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Rekognisi & Pemicuan Redirect
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Masuk ke halaman **My account** (`/my-account?id=wiener`).
3. Tangkap request tersebut di Burp Suite, lalu kirim ke **Burp Repeater**.
4. Ubah nilai parameter `id` pada query string dari `wiener` menjadi `carlos`.

---

### Eksploitasi & Pengamatan Response Body
1. Kirim request `GET /my-account?id=carlos` melalui Burp Repeater.
2. Perhatikan bagian header respons dari server:
   * **Status Code:** `HTTP/2 302 Found`
   * **Header:** `Location: /login`
3. *Scroll* ke bawah menuju bagian **Response Body** dari paket HTTP 302 tersebut.
4. Di dalam kode HTML *body* (yang biasanya diabaikan dan tidak dirennder oleh browser karena otomatis melakukan redirect), terdapat data profil lengkap milik `carlos`.

Request HTTP final & Potongan Respons di Burp Repeater:
```http
GET /my-account?id=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=[SESSION_WIENER]

HTTP/2 302 Found
Location: /login
Content-Type: text/html; charset=utf-8

<div>Your API Key is: 9x8a7b6c5d4e3f2a1b0c</div>
```

### Eksekusi & Pengiriman Solusi
1. Salin nilai API Key milik carlos yang bocor di dalam response body tersebut.
2. Kembali ke halaman utama aplikasi web pada browser.
3. Klik tombol Submit solution, lalu tempelkan API Key tersebut.

Hasil Akhir: Meskipun status HTTP memberikan perintah 302 Redirect, kegagalan server dalam menghentikan render halaman membocorkan data sensitif pengguna. Lab dinyatakan selesai (Solved).