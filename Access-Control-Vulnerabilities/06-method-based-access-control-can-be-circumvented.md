# [PortSwigger Academy] Access Control - Lab #6

## 📌 Detail Lab
* **Nama Tantangan:** Method-based access control can be circumvented
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Mengakali pembatasan kontrol akses berbasis metode HTTP (*HTTP Method Tampering*) untuk menaikkan hak akses akun `wiener` menjadi Administrator.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi memiliki fitur administratif untuk mengubah peran pengguna (misalnya menaikkan akun biasa menjadi admin). Fungsi ini biasanya dikirimkan menggunakan metode `POST`.

Celah keamanan terjadi karena aturan otorisasi (*access control filter*) hanya dikonfigurasi untuk memeriksa hak akses pengguna saat request menggunakan metode **`POST`**. Namun, handler fungsi di backend menerima dan memproses data yang sama meskipun dikirim menggunakan metode **`GET`** (dengan parameter yang dipindahkan ke URL query string). Akibatnya, pengguna biasa dapat mengeksekusi fungsi admin hanya dengan mengubah metode request-nya.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Rekognisi Perilaku Otorisasi
1. Login menggunakan akun administrator yang diberikan (`administrator:admin`).
2. Masuk ke panel admin, lalu naikkan hak akses pengguna `wiener` menjadi admin.
3. Tangkap request HTTP pembaruan peran tersebut di Burp Suite, lalu kirim ke **Burp Repeater**.
4. Amati struktur request bawaan saat dilakukan oleh Admin:
   ```http
   POST /admin-roles HTTP/2
   Host: web-security-academy.net
   Cookie: session=[ADMIN_SESSION]
   Content-Type: application/x-www-form-urlencoded

   username=wiener&action=upgrade
   ```

### Pengujian Akses dengan Sesi User Biasa
1. Buka browser terpisah atau incognito window, lalu login sebagai user biasa (wiener:peter).
2. Salin nilai cookie sesi milik wiener.
3. Kembali ke Burp Repeater pada request POST /admin-roles tadi, ganti cookie admin dengan cookie sesi wiener.
4. Kirim request POST tersebut. Server mengembalikan respon 401 Unauthorized atau 403 Forbidden (membuktikan bahwa metode POST dilindungi kontrol akses).

### Eksploitasi via HTTP Method Tampering
1. Klik kanan pada request di Burp Repeater, lalu pilih Change request method (mengubah metode dari POST menjadi GET).
2. Burp Suite akan secara otomatis memindahkan body payload ke dalam URL query string:

    URL Baru: /admin-roles?username=wiener&action=upgrade

3. Pastikan cookie yang digunakan tetap milik user biasa (wiener).
4. Kirim request GET tersebut.

Request HTTP final di Burp Repeater:
```
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=[USER_WIENER_SESSION]
```

### Hasil Eksploitasi
Server merespons dengan status HTTP 302 Found. Karena filter otorisasi hanya memeriksa metode POST, request GET lolos dari pemeriksaan keamanan dan backend tetap memproses perintah upgrade untuk user wiener.

Akses administratif kini telah aktif dan lab dinyatakan selesai (Solved).
Hasil Akhir: Kontrol akses berbasis metode HTTP berhasil dilewati dengan mengubah metode POST menjadi GET