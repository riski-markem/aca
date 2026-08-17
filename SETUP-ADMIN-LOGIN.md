# Setup Login Admin & Catatan Update Terbaru

## 1. Setup Login Admin (Firebase Auth, WAJIB sebelum admin.html dipakai)

Panel admin sekarang pakai akun Firebase Auth asli (username + password),
BUKAN lagi PIN yang ditulis di kode.

Firebase Auth secara teknis tetap butuh format email, tapi kamu TIDAK
perlu pakai email asli/Gmail. Cukup pilih username pendek (misal `riski`
atau `admin`), lalu di Firebase Console kamu daftarkan sebagai
`riski@admin.mastulung.app` -- sama seperti nomor WA resto yang
"disamarkan" jadi email di resto.html. Saat login di admin.html, kamu
cukup ketik `riski`, bukan email lengkapnya.

### Langkahnya:
1. Buka https://console.firebase.google.com → project `mastulungmasgombong`
2. Authentication → Sign-in method → pastikan provider **Email/Password** aktif
3. Authentication → Users → **Add user**
   - Email: `<username_pilihanmu>@admin.mastulung.app` (contoh: `riski@admin.mastulung.app`)
   - Password: buat yang kuat, jangan dipakai di tempat lain
4. Setelah user dibuat, **salin UID**-nya (kolom "User UID")
5. Buka Realtime Database → data → tambahkan manual:
   ```
   admins/
     <UID_YANG_TADI_DISALIN>: true
   ```
   ⚠️ Wajib -- tanpa ini, login admin.html akan selalu ditolak walau
   username/password benar.
6. Login ke admin.html pakai username & password dari langkah 3. Sesi
   otomatis tersimpan di browser (tidak perlu login ulang tiap buka),
   sampai kamu klik "Keluar".

### Database Rules (opsional tapi disarankan)
Contoh dasar -- **jangan copy-paste mentah kalau kamu sudah punya rules
custom**, gabungkan dengan yang sudah ada:
```json
{
  "rules": {
    "admins": {
      ".read": "auth != null && root.child('admins').child(auth.uid).exists()",
      ".write": false
    }
  }
}
```
`admins` sengaja `.write: false` -- supaya tidak bisa diubah dari
aplikasi manapun, hanya lewat Firebase Console langsung.

---

## 2. Biaya Layanan untuk Order Non-Makanan (BARU)

Sebelumnya, biaya layanan Rp 1.000/order cuma dibebankan ke driver
(dipotong dari saldo saat ambil order). Sekarang disamakan dengan pola
yang sudah ada di order makanan: **customer juga bayar Rp 1.000 biaya
layanan** untuk order Antar Jemput, Kirim Barang, Titip Belanja, Bantu
Bersih, dan Jasa Unik -- dibayar cash langsung ke driver, lalu otomatis
dipotong dari saldo driver saat order ditandai selesai (jadi bukan
beban driver sendirian, sama seperti order makanan).

Yang berubah:
- **index.html**: form order & estimasi harga sekarang menampilkan
  baris "Biaya Layanan Rp 1.000" untuk semua layanan non-makanan.
  Untuk Bantu Bersih/Jasa Unik (yang harganya nego, tidak ada estimasi
  otomatis), catatan biaya layanan muncul statis di info tarif.
- **driver.html**: kartu order sekarang mengingatkan driver untuk
  menagih customer termasuk biaya layanan ini, dan `selesaikanOrderanDriver`
  otomatis memotong saldo driver Rp 1.000 saat order (non-makanan)
  ditandai selesai.
- **admin.html**: perhitungan "Total Pemasukan Admin" sudah disesuaikan
  supaya ikut menghitung biaya layanan dari order non-makanan ini, tidak
  cuma dari order makanan.

Tidak ada langkah setup tambahan yang diperlukan untuk fitur ini --
begitu file baru di-upload, langsung aktif untuk order baru. Order lama
(yang sudah ada di database sebelum update ini) tidak terpengaruh.

---

## 3. Ringkasan Perubahan Lain
- `admin.html`: tombol **✕ Tolak** untuk pendaftar driver/resto yang
  masih `pending` (alasan opsional, tersimpan di `alasan_tolak`).
- `admin.html`: tab **Pantau Orderan** sekarang punya pencarian + filter
  status, dan tombol intervensi untuk order yang macet (**🛑 Batalkan
  Paksa** / **✅ Tandai Selesai Paksa**).
- `admin.html`: peringatan visual saldo driver minus.
- `driver.html` & `resto.html`: status baru `ditolak` sekarang diblokir
  saat login/sesi, sama seperti `pending`/`suspend`.
