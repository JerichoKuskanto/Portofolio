# 01 — Reconnaissance Overview (Sanitized)

## 1. Tujuan Recon
Tahap rekognisi (reconnaissance) dilakukan untuk:
- Mengidentifikasi permukaan serangan (attack surface) aplikasi.
- Mengumpulkan informasi awal terkait struktur URL, endpoint, dan fungsionalitas.
- Menentukan potensi vektor serangan sebelum melakukan scanning dan validasi kerentanan.

Pengujian ini dilakukan sebagai bagian dari penetration testing berizin terhadap aplikasi web **Dibishop (sanitized)** dalam model **Black-box**, tanpa akses internal.

---

## 2. Ruang Lingkup Recon
Recon dilakukan hanya terhadap domain yang diberikan izin:

- `http://REDACTED-TARGET.duckdns.org`
- Ruang lingkup terbatas pada fungsi user & admin yang dapat diakses secara langsung oleh user biasa.

Submenu / endpoint yang diobservasi:

| Menu            | Endpoint (Sanitized)                                     | Admin | User |
|-----------------|------------------------------------------------------------|--------|------|
| Profile         | `/customer/my_account.php?edit_account`                   | ❌     | ✔️   |
| Pembelian       | `/cart.php`                                               | ❌     | ✔️   |
| Pembelian       | `/customer/confirm.php?order_id=*`                        | ❌     | ✔️   |
| Product Page    | `/poco-f7-pro`                                            | ❌     | ✔️   |

---

## 3. Teknik Recon yang Digunakan

### 3.1 Passive Recon
Tujuan: memahami struktur aplikasi tanpa melakukan interaksi agresif.

Aktivitas:
- Observasi alur navigasi pengguna
- Mengidentifikasi parameter GET yang sensitif (`order_id`, `coupon_code`, dll)
- Mempelajari pola URL dinamis
- Mengamati struktur role-based access (admin vs user)

Temuan awal:
- Banyak endpoint menggunakan parameter yang dapat dimodifikasi user (indikasi potensi IDOR).
- Tidak ada obfuscation pada nilai harga produk (indikasi insecure design).
- Form input tidak memiliki validasi client-side yang ketat.

---

### 3.2 Active Recon (Non intrusive)
Aktivitas meliputi:
- Enumeration endpoint secara manual
- Menguji behavior parameter URL
- Mengamati response server terhadap input tidak valid
- Menguji form-field menggunakan input baseline (angka, simbol, string panjang)

Contoh parameter yang merespons input abnormal:
- `product_price`
- `order_id`
- `email`
- `confirmation_code`
- `coupon_code`

---

## 4. Ringkasan Hasil Recon
Recon mengidentifikasi potensi area risiko berikut:

| Risiko | Indikasi | Dampak Potensial |
|--------|----------|------------------|
| Manipulasi Harga | Parameter harga dapat dimodifikasi dari client | Fraud / pembelian ilegal |
| IDOR | Parameter `order_id` dapat ditebak | Pelanggaran data user lain |
| Weak Authentication | Tidak ada rate-limiting login | Brute force / credential stuffing |
| Unrestricted Upload | Upload tidak membatasi tipe file | File berbahaya masuk ke server |
| SQL Injection | Banyak endpoint error saat input abnormal | Akses database ilegal |
| XSS | Beberapa input tidak melakukan sanitasi | Script berjalan di browser user lain |

Informasi recon ini menjadi dasar untuk tahap scanning dan validasi kerentanan.
