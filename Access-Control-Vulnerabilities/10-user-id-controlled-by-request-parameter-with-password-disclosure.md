# [PortSwigger Academy] Access Control - Lab #10

## 📌 Detail Lab
* **Nama Tantangan:** User ID controlled by request parameter with password disclosure
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities (IDOR / Sensitive Data Exposure)
* **Tujuan Lab:** Mencuri kata sandi administrator melalui celah IDOR pada elemen form profil, login sebagai administrator, lalu menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menampilkan fitur ubah kata sandi pada halaman profil pengguna (`/my-account?id=[username]`).

Celah keamanan terjadi akibat dua kombinasi kerentanan:
1. **Broken Object Level Authorization (IDOR):** Server tidak memeriksa apakah parameter `id` yang diminta di URL cocok dengan identitas sesi pengguna yang terautentikasi.
2. **Sensitive Data Disclosure in Client-Side Source Code:** Form pembaruan kata sandi mengisi atribut `value` pada tag input kata sandi secara otomatis (*pre-filled*) dengan password murni pengguna dari database (`<input type="password" name="password" value="[PLAINTEXT_PASSWORD]">`).

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Exploiting IDOR untuk Mengakses Halaman Administrator
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Masuk ke halaman **My account**, lalu tangkap request `GET /my-account?id=wiener` menggunakan Burp Suite dan kirim ke **Burp Repeater**.
3. Ubah nilai parameter `id` pada query string dari `wiener` menjadi `administrator`.

Request HTTP di Burp Repeater:
```http
GET /my-account?id=administrator HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=[SESSION_WIENER]
```

### Ekstraksi Kata Sandi Administrator
1. Kirim request tersebut melalui Burp Repeater.
2. Amati respons HTML yang dikembalikan oleh server (HTTP 200 OK).
3. Cari tag elemen <input> yang mengelola kata sandi pengguna pada kode HTML respons:
```html
<input required type=password name=password value='tslrlzddsqz8lyhchpkj'/>
```
4. Salin string kata sandi plaintext milik administrator tersebut dari dalam atribut value 

### Escalation & Penghapusan User Target
1. Kembali ke browser, lakukan Logout dari akun wiener.
2. Login kembali menggunakan kredensial administrator yang baru saja dicuri:

    * Username: administrator
    * Password: [PASSWORD_HASIL_EKSTRAKSI]

3. Setelah berhasil masuk ke panel akun administrator, buka Admin panel.
4. Cari akun carlos pada daftar pengguna, lalu klik Delete.

Hasil Akhir: Kata sandi administrator berhasil dicuri via IDOR pada atribut form value, memungkinkan pengambilalihan akun penuh (account takeover) dan penghapusan user target. Lab dinyatakan selesai (Solved).