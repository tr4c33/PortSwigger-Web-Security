# [PortSwigger Academy] Access Control - Lab #3

## 📌 Detail Lab
* **Nama Tantangan:** User role controlled by request parameter
* **Tingkat Kesulitan:** APPRENTICE
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Mengeksploitasi parameter kontrol peran (*role parameter*) yang dikendalikan oleh pengguna untuk mengakses panel admin dan menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi mengidentifikasi hak akses pengguna (apakah seorang *administrator* atau bukan) berdasarkan nilai yang dikirimkan melalui parameter penyimpanan klien, seperti cookie `Admin=false` atau parameter tersembunyi pada request.

Celah keamanan terjadi karena pengembang **mengandalkan parameter sisi klien (*client-controlled parameter*) untuk mengambil keputusan keamanan di sisi server (*authorization decision*)** tanpa melakukan validasi terhadap kredensial atau *session state* internal database. Penyerang dapat dengan mudah memodifikasi parameter tersebut untuk melakukan eskalasi hak istimewa (*privilege escalation*).

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Analisis Sesi & Cookie saat Login
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Tangkap lalu lintas HTTP menggunakan Burp Suite atau periksa melalui penyimpanan browser (*Storage/Cookies*).
3. Perhatikan sekumpulan cookie yang diset oleh server setelah login berhasil. Anda akan menemukan cookie dengan nama indikasi peran, contohnya:
   * `Admin=false` atau `role=user`

---

### Manipulasi Parameter Peran (Privilege Escalation)
1. Kirim request halaman utama atau profil akun ke **Burp Repeater**.
2. Ubah nilai parameter kontrol peran tersebut dari status `false` menjadi `true`:
   * **Cookie Asli:** `Admin=false`
   * **Cookie Modifikasi:** `Admin=true`

Request HTTP final di Burp Repeater:
```http
GET /my-account HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=xyz123...; Admin=true
```

### Verifikasi Akses & Eksekusi Target

1. Kirim request tersebut melalui Burp Repeater. Perhatikan respons dari server: jika di halaman akun muncul tautan tambahan seperti "Admin panel", itu berarti eskalasi hak akses berhasil.
2. Akses endpoint panel admin secara langsung:

    URL: /admin

3. Cari akun carlos pada daftar pengguna, lalu klik tombol Delete.

Request penghapusan user target:
```
GET /admin/delete?username=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=xyz123...; Admin=true
```

### Hasil Eksploitasi

Server menerima cookie manipulasi Admin=true dan langsung memberikan hak istimewa administratif kepada sesi kita tanpa validasi lanjutan. Akun carlos berhasil dihapus dari sistem.

Hasil Akhir: Celah kontrol akses akibat parameter peran yang dapat dimodifikasi pengguna berhasil dieksploitasi. Lab dinyatakan selesai (Solved).