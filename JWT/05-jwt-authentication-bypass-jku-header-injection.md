# [PortSwigger Academy] JWT - Lab #5

## 📌 Detail Lab
* **Nama Tantangan:** JWT authentication bypass via jku header injection
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** JSON Web Tokens (JWT) Attacks
* **Tujuan Lab:** Mengakses panel admin (`/admin`) dan menghapus user `carlos` dengan cara menghosting kunci publik palsu di Exploit Server dan menyuntikkan tautannya ke dalam parameter header `jku`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Parameter header `jku` (JWK Set URL) digunakan oleh server untuk merujuk ke URL eksternal yang menyimpan kumpulan kunci publik (JWKS) untuk memverifikasi token. 

Celah keamanan terjadi karena mekanisme backend **mengambil berkas kunci dari URL mana pun yang disuplai oleh pengguna di dalam parameter `jku` tanpa melakukan verifikasi atau pembatasan origin URL (*Open Fetching*)**. Penyerang dapat memanfaatkan perilaku ini dengan mengarahkan server untuk mengunduh kunci publik palsu dari server yang dikendalikan penyerang, sehingga token manipulasi yang ditandatangani dengan kunci privat pasangannya akan dianggap sah.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Membuat Kunci RSA Baru di Burp Suite
1. Masuk ke tab **JWT Editor Keys** di Burp Suite.
2. Klik **New RSA Key** (2048 bits), lalu klik **Generate**.
3. Klik kanan pada kunci baru yang berhasil dibuat tersebut, lalu pilih **Copy Key as JWK**.
catatan ini bisa menggunakan jwt_tool 

### Menghitung dan Mengunggah Kunci Palsu ke Exploit Server
1. Buka **Exploit Server** yang disediakan oleh PortSwigger Lab.
2. Pada bagian **Body**, buat struktur JSON Web Key Set (JWKS) dengan format berikut dan tempelkan hasil salinan JWK dalam *array* `keys`:
```json
   {
       "keys": [
           {
               "kty": "RSA",
               "e": "AQAB",
               "kid": "b7f3...",
               "n": "v91b..."
           }
       ]
   }
```
Klik Store untuk menyimpan file tersebut. Catat URL exploit server contoh: https://exploit-0a1b008f03c4.exploit-server.net/exploit.

### Melakukan Serangan JKU Injection menggunakann menggunakan jwt_tool
1. Login ke aplikasi menggunakan akun biasa (wiener:peter).
2. Ambil session value nya di storage lewat inspect 
3. Masukkan payload ini:
* **Payload :** `python jwt_tool <JWT-TOKEN> -I -pc sub -pv administrator -hc kid -hv jwt_tool -X s -ju https://exploit-0a1b008f03c4.exploit-server.net/exploit`

### Hasil Eksploitasi
Saat menerima token ini, server backend akan melakukan HTTP request ke URL penyerang yang tertera di "jku", mengunduh kunci publik dari sana, lalu menggunakannya untuk memverifikasi tanda tangan. Karena tanda tangan tersebut cocok dengan kunci privat pasangannya, akses admin diberikan.

Panel administratif terbuka penuh (HTTP 200 OK). Kirim kueri terakhir untuk menghapus user target:

* **Endpoint Eksekusi:** `/admin/delete?username=carlos`

Hasil Akhir: Akun carlos sukses dihapus dan lab dinyatakan selesai (Solved).
