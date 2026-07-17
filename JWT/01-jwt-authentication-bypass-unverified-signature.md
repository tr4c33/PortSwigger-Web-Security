# [PortSwigger Academy] JWT - Lab #1

## 📌 Detail Lab
* **Nama Tantangan:** JWT authentication bypass via unverified signature
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** JSON Web Tokens (JWT) Attacks
* **Tujuan Lab:** Mengakses panel admin (`/admin`) dan menghapus user `carlos` dengan cara memodifikasi token session JWT tanpa memverifikasi signature.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menggunakan token JWT yang disimpan di dalam cookie session untuk mengidentifikasi pengguna yang login. Struktur JWT terdiri dari tiga bagian yang dipisahkan oleh titik (`.`): `Header.Payload.Signature`.

Celah keamanan terjadi karena komponen *backend* aplikasi hanya melakukan *decode* terhadap bagian *payload* untuk membaca data pengguna (seperti `"sub": "username"`), tetapi **lupa memverifikasi keabsahan Signature** menggunakan kunci rahasia (*secret key*). Karena validasi tanda tangan ini dilewati, aplikasi mempercayai data apa pun yang dikirimkan oleh pengguna, sehingga membuka celah bagi serangan *Privilege Escalation*.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Identifikasi Token Session
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Tangkap request saat mengakses halaman akun (`/my-account`) menggunakan Burp Suite, lalu kirim ke Burp Repeater.
3. Perhatikan bagian cookie `session`. Nilainya adalah sebuah token JWT yang valid.

### Modifikasi Payload JWT (Tampering)
Gunakan tab **JWT Editor** bawaan di Burp Suite  untuk mengubah data token:

1. Arahkan ke bagian **Payload** teks JWT yang telah di-decode.
2. Ubah nilai parameter `"sub"` dari `"wiener"` menjadi **`administrator`**.
3. **Biarkan bagian Signature apa adanya** (jangan diubah atau ditandatangani ulang, karena server tidak akan memeriksanya).


### Pengiriman Payload & Akses Admin
Ubah baris URL request pada Burp Repeater untuk mengakses endpoint administratif dan kirim token yang sudah dimodifikasi:

* **Request URL:** `/admin`
* **Cookie Header:** `session=[JWT_YANG_SUDAH_DI_UBAH]`

Request HTTP final di Burp Repeater:
```http
GET /admin HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=eyJraWQiOiI5ODg2OTRlOS1kZDk0LTQxZDUtOWVkZC1lOWRiMDE0ODg5MjYiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MzMzMDc2OSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.[Signature_Lama]
```

### Hasil Eksekusi
Server merespons dengan status HTTP 200 OK dan menampilkan halaman panel admin (menandakan bypass sukses). Cari tautan untuk menghapus user carlos, lalu kirim request tersebut menggunakan token manipulasi yang sama:
* **Target Akhir:** `/admin/delete?username=carlos`
Hasil Akhir: Server mengeksekusi penghapusan user carlos tanpa penolakan. Kerentanan unverified signature terbukti valid dan lab dinyatakan selesai (Solved).