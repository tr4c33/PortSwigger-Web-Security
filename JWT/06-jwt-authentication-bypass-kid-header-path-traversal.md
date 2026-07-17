# [PortSwigger Academy] JWT - Lab #6

## 📌 Detail Lab
* **Nama Tantangan:** JWT authentication bypass via kid header path traversal
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** JSON Web Tokens (JWT) Attacks
* **Tujuan Lab:** Mengakses panel admin (`/admin`) dan menghapus user `carlos` dengan memanfaatkan celah Path Traversal pada parameter header `kid` untuk memaksa server menggunakan kunci verifikasi kosong.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Parameter header `kid` (Key ID) opsional digunakan untuk membantu server memilih kunci yang tepat dari sekumpulan kunci (*key store*) jika terdapat banyak kunci aktif. 

Celah keamanan terjadi karena aplikasi web langsung menggabungkan string parameter `kid` ke dalam jalur file lokal di server untuk memuat berkas kunci rahasia secara dinamis. Karena input tidak disanitasi dari karakter navigasi direktori (`../`), penyerang dapat membelokkan jalur pembacaan file menuju file sistem yang isinya dapat diprediksi atau kosong, seperti **`/dev/null`**. Ketika server membaca `/dev/null` sebagai kunci rahasia simetrisnya, server akan menggunakan string kosong (string dengan panjang nol) untuk memverifikasi tanda tangan JWT.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)
Karena kita akan memaksa server menggunakan kunci berukuran 0 byte via `/dev/null`, kita harus membuat kunci tiruan yang sama di lokal kita:


### Menangkap dan Memodifikasi Token Sesi
1. Login ke aplikasi menggunakan akun biasa (`wiener:peter`).
2. Masuk ke Developer Tools ambil JWT TOKEN dari storage.
3. Ubah komponen **Payload**: ganti `"sub": "wiener"` menjadi **`"sub": "administrator"`** di jwt_tool.

### Menyuntikkan Path Traversal pada Parameter `kid`
Ubah parameter `kid` pada bagian **Header** agar mundur keluar dari folder penyimpanan kunci bawaan server menuju file `/dev/null`:

* **Header Asli:** `"kid": "normal-key.pem"`
* **Header Manipulasi:** `"kid": "../../../../../../../dev/null"`

### Menandatangani Ulang Token (Signing)
Setelah Header dan Payload dimodifikasi, kita harus mengesahkan token menggunakan jwt_tool, payload akhir:

* **payload :** `python3 jwt_tool.py eyJraWQiOiIyMDYyMzI1NC1iY2QyLTQ1NDAtOGZjMC1lN2UyNjlmYmE4ZDYiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MzU0NDIwOCwic3ViIjoid2llbmVyIn0.z3tBU6SvolUWQ39s7Vc5eoPulxD233cgU1ThAQRKSRk -S hs256 -I -hc kid -hv ../../../../../../../dev/null -pc sub -pv administrator -k empty.txt`

Catatan -k untuk memasukkan sign kosong membuat key tiruan.

### Hasil Eksploitasi
Kirim request tersebut. Server backend akan memproses string "kid": "../../../../../../../dev/null", membuka file /dev/null, dan menggunakannya sebagai kunci verifikasi. Karena file tersebut kosong, server mengonfirmasi bahwa tanda tangan pada token kita (yang juga dibuat dengan kunci kosong) adalah VALID.

Halaman panel admin berhasil diakses (HTTP 200 OK). Selesaikan misi dengan mengirimkan perintah penghapusan user target:

* **Endpoint Eksekusi:** `/admin/delete?username=carlos`

Hasil Akhir: Akun carlos berhasil dihapus dari sistem dan lab dinyatakan selesai (Solved).