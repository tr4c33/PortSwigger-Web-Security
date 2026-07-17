# [PortSwigger Academy] JWT - Lab #2

## 📌 Detail Lab
* **Nama Tantangan:** JWT authentication bypass via flawed signature verification
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** JSON Web Tokens (JWT) Attacks
* **Tujuan Lab:** Mengakses panel admin (`/admin`) dan menghapus user `carlos` dengan memanfaatkan cacat verifikasi tanda tangan menggunakan algoritma `none`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menggunakan token JWT untuk mengelola sesi pengguna. Secara teori, server harus memverifikasi bahwa token ditandatangani dengan algoritma yang kuat (seperti `HS256` atau `RS256`) untuk mencegah manipulasi data.

Celah keamanan terjadi karena pustaka (*library*) JWT yang digunakan di backend secara membabi buta menerima parameter `"alg": "none"` yang dikirimkan di dalam komponen **Header**. Ketika nilai ini disuntikkan, mekanisme pertahanan server berasumsi bahwa token tersebut sengaja dibuat tanpa tanda tangan (*unsigned*), sehingga verifikasi integritas dilewati sepenuhnya.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Identifikasi Token Session
1. Login ke aplikasi menggunakan akun biasa (`wiener:peter`).
2. Ambil request halaman utama atau `/my-account` di Burp Suite, lalu kirim ke Burp Repeater.
3. Perhatikan cookie `session` yang berisi token JWT terenkripsi base64.

### Manipulasi Header dan Payload JWT
Buka tab **JWT Editor** atau lakukan modifikasi manual (decode-edit-reencode) terhadap token sesi tersebut:

1. **Ubah Header:** Ganti jenis algoritma dari `"RS256"` menjadi **`"none"`**.
2. **Ubah Payload:** Ganti nilai parameter `"sub"` dari `"wiener"` menjadi **`administrator`**.


### Penyesuaian Struktur Signature (Penting!)
Token JWT asli memiliki format `Header.Payload.Signature`. Karena kita menggunakan algoritma `none`, bagian ketiga (Signature) harus **dihapus**, tetapi **karakter titik kedua (`.`) wajib dipertahankan**.

* Format akhir token yang valid: `Header_Base64.Payload_Base64.` (berakhir dengan tanda titik).

Contoh request HTTP akhir yang dikirimkan melalui Burp Repeater:
```http
GET /admin HTTP/2
Host: 0a5c00e104b20cc8819d45e400bd0095.web-security-academy.net
Cookie: session=eyJraWQiOiJmMGE4NWM5Ni05MGM0LTQzMDctYTg4Zi05ZGVlY2MyYzk3MTUiLCJhbGciOiJub25lIn0%3d.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MzMzNjE0Nywic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
```

### Hasil Eksploitasi
Kirim request tersebut ke server. Karena backend mendecode "alg": "none", ia langsung membaca payload "sub": "administrator" tanpa memeriksa validitas data.

Halaman Panel Admin berhasil terbuka HTTP 200 OK. Langkah terakhir adalah mengeksekusi penghapusan user target dengan token manipulasi yang sama:

* **Endpoint Penghapusan:** `/admin/delete?username=carlos`

Hasil Akhir: Akun carlos berhasil dihapus dari sistem dan lab dinyatakan selesai (Solved).