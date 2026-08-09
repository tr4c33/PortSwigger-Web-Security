# [PortSwigger Academy] Access Control - Lab #5

## 📌 Detail Lab
* **Nama Tantangan:** URL-based access control can be circumvented
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Mengakali aturan kontrol akses berbasis URL (*URL-based Access Control*) dengan menyuntikkan header HTTP kustom, mengakses panel admin, lalu menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Sistem menggunakan komponen WAF/Reverse Proxy di bagian depan untuk memblokir seluruh request yang menuju ke jalur sensitif seperti `/admin` (*Access Denied / 403 Forbidden*).

Celah keamanan terjadi karena server backend di latar belakang dikonfigurasi untuk menerima header pembelokan URL seperti **`X-Original-URL`** atau **`X-Override-URL`**. Jika kita mengirim request HTTP ke URL yang diizinkan (misalnya `/`), tetapi menyisipkan header `X-Original-URL: /admin`, lapisan depan hanya memeriksa jalur `/` (lolos filter), sementara server backend membaca header kustom tersebut dan mengeksekusi fungsi `/admin`.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Deteksi Pemblokiran & Bypassing URL Filter
1. Akses halaman utama aplikasi web di browser.
2. Tangkap request `GET /` menggunakan Burp Suite, lalu kirim ke **Burp Repeater**.
3. Coba ubah path URL request menjadi `GET /admin`. Server akan mengembalikan respon **`403 Forbidden`** (membuktikan adanya aturan filter di lapisan depan).
4. Kembalikan path URL menjadi `GET /`, lalu tambahkan header kustom berikut pada request HTTP:
   ```http
   X-Original-URL: /admin
   ```
5. Kirim request tersebut. Server merespons dengan status HTTP 200 OK dan menampilkan antarmuka Panel Admin, membuktikan bahwa aturan pemblokiran URL berhasil dilewati (bypassed).

### Eksekusi Perintah Administratif
Setelah berhasil memuat panel admin, kita perlu mengeksekusi fungsi penghapusan pengguna (/admin/delete?username=carlos).

1. Beralih ke Burp Repeater.
2. Tetapkan request utama ke URL aman GET /?username=carlos .
3. Arahkan perintah aksi ke dalam header X-Original-URL:

Request HTTP final di Burp Repeater:
```http
GET /?username=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
X-Original-URL: /admin/delete
Cookie: session=xyz123...
```

### Hasil Eksploitasi

Kirim request tersebut ke server. Lapisan depan memeriksa kueri / yang dianggap aman, lalu memteruskannya ke backend. Backend membaca X-Original-URL: /admin/delete beserta parameter username=carlos dan mengeksekusi penghapusan akun.

Hasil Akhir: Akun carlos berhasil dihapus dan lab dinyatakan selesai (Solved).