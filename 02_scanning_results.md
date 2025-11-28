# 02 — Scanning & Enumeration (Sanitized)

Tahap scanning dilakukan setelah recon untuk memahami bagaimana aplikasi menangani input,
mengidentifikasi endpoint yang rentan, dan menentukan area yang perlu diuji lebih dalam pada
tahap eksploitasi. Semua proses dilakukan sesuai ROE dan hanya pada aset yang diizinkan.

---

## 1. Tujuan Scanning

Scanning bertujuan untuk:
- Mengidentifikasi endpoint yang menerima input user.
- Menguji bagaimana aplikasi memproses input abnormal.
- Menilai validasi input pada sisi server.
- Mengamati potensi kerentanan seperti SQL Injection, XSS, IDOR, dan insecure design.
- Melakukan enumerasi pada parameter kritis untuk eksploitasi lanjutan.

---

## 2. Tools & Teknik yang Digunakan

### 2.1 Automated/Assisted Tools
- **Burp Suite Community Edition** — intercept, repeater, param tampering.
- **Gobuster** — directory enumeration ringan.
- **WhatWeb / Wappalyzer** — identifikasi teknologi web.
- **SQLMap (lab-only)** — hanya digunakan untuk request yang sudah menunjukkan indikasi SQLi.

### 2.2 Manual Testing
- Manipulasi parameter GET/POST.
- Payload fuzzing untuk input handling.
- Low-rate authentication testing.
- Observasi respons server.

---

## 3. Endpoint dalam Scope

| Endpoint (Sanitized)                                | Metode Uji                     | Akses |
|-----------------------------------------------------|---------------------------------|-------|
| `/poco-f7-pro`                                      | Parameter tampering             | User  |
| `/cart.php`                                         | Form/input fuzzing              | User  |
| `/customer/confirm.php?order_id=*`                  | Parameter enumeration (IDOR)    | User  |
| `/customer_register.php`                            | Form tampering                  | User  |
| `/customer/my_account.php?edit_account`             | XSS & file upload tests         | User  |
| `/login`                                            | Rate-limit testing              | Public|
| `/admin_area/login`                                 | Rate-limit testing              | Public|

---

## 4. Hasil Scanning

### 4.1 Teknologi Web (Disamarkan)
- Server: Apache-like
- Backend: PHP
- Database: MySQL-like
- Auth system: Custom

---

### 4.2 Parameter Behavior

| Parameter               | Endpoint                                    | Observasi |
|-------------------------|----------------------------------------------|-----------|
| `product_price`         | `/poco-f7-pro`                               | Nilai dapat dimanipulasi klien → insecure design |
| `order_id`              | `/customer/confirm.php`                      | Dapat diubah dan menampilkan data lain → IDOR |
| `coupon_code`           | `/cart.php`                                  | Terkesan diproses langsung oleh DB |
| `email`                 | `/customer_register.php`                     | Input abnormal tidak divalidasi |
| `confirmation_code`     | `/customer/confirm.php`                      | Perilaku tidak stabil saat diberikan string abnormal |
| Profile fields          | `/edit_account`                              | Input teks disimpan tanpa filter → potensi XSS |
| File upload             | `/edit_account`                              | Menerima file tidak terbatas tipe |

---

### 4.3 Input Fuzzing

Payload baseline yang digunakan:

```plaintext
'
"
<>
<test>
';--
\<script>alert(1)\</script>
long_string_1024chars...(testing length limits)
```

**Hasil ringkas:**
- Beberapa endpoint tidak memvalidasi karakter spesial.
- Beberapa input tersimpan dan ditampilkan kembali tanpa encoding (potensi XSS).
- Parameter tertentu menunjukkan perilaku abnormal saat diberi payload panjang/asing.
- Tidak ada sanitasi input yang konsisten di seluruh modul.

---

### 4.4 Authentication & Rate-Limit Evaluation

Metodologi:
- Percobaan login berulang menggunakan Burp Repeater.
- Percobaan low-rate brute force (1 request/detik).
- Mengamati apakah terjadi lockout, delay, atau captcha.

Hasil:
- Tidak ada rate limiting.
- Tidak ada cooldown setelah percobaan gagal.
- Tidak ada mekanisme anti-bruteforce baik di login user maupun admin.
- Sistem rentan terhadap credential stuffing dan brute-force login.

---

### 4.5 File Upload Check

Metodologi:
- Upload berbagai jenis file: image valid, image renamed, file script, file teks.
- Observasi apakah server memvalidasi tipe file, MIME, konten, atau nama file.

Hasil:
- File upload menerima beragam jenis file tanpa pemeriksaan.
- Tidak ada pengecekan MIME atau magic bytes.
- Nama file tidak disanitasi.
- File tersimpan tanpa pembatasan tipe.

---

## 5. Kesimpulan Tahap Scanning

Dari hasil scanning, beberapa area yang perlu difokuskan pada tahap eksploitasi antara lain:
- **IDOR** pada endpoint dengan parameter ID.
- **Kurangnya validasi input**, yang membuka potensi:
  - SQL Injection,
  - Stored XSS,
  - Insecure Data Handling.
- **Login tanpa rate limit**, risiko besar bagi kredensial.
- **Unrestricted file upload**, yang dapat berkembang menjadi serangan lebih berat.
- **Insecure design** pada parameter harga/produk.

Tahap scanning ini memberikan dasar kuat untuk eksploitasi terarah pada fase berikutnya.

