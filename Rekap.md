## ✅ **Rekap Diskusi Proyek HRGA System**

### **A. Ruang Lingkup Proyek**

Proyek ini mencakup 3 sub-sistem utama:

1. **Visitor Management System (VMS)**
2. **Meeting Room Booking**
3. **Stok Inventory Barang Umum (ATK, snack, dll)** – fokus utama yang belum memiliki vendor.

---

### **B. Fokus: Modul Stok Inventory GA**

#### **1. Kebutuhan Sistem**

* **Real-time monitoring** stok barang (awal, masuk, keluar).
* **Request barang** dari departemen lain ke GA (dengan notifikasi).
* **Waktu pemesanan dibatasi** (misalnya setiap Selasa jam 14.00–17.00).
* **Perencanaan kebutuhan triwulan** oleh masing-masing departemen.
* **Pengelolaan vendor dan harga**, karena satu item bisa memiliki banyak harga tergantung vendor.
* **Tracking penggunaan per user/departemen**.
* **Akumulasi pemakaian ke dalam nominal (Rp)**.
* **Pelabelan item** dengan kode purchase atau QR/Barcode untuk distribusi.
* **Output grafik & evaluasi**, antara lain:

  * Rata-rata pemakaian item per 3 bulan.
  * Rata-rata pemakaian tiap departemen.
  * Departemen pemakaian tertinggi/terendah (per item & total).

---

#### **2. Alur Flow Pengadaan Barang**

1. Departemen input **request**.
2. GA **verifikasi & tampilkan vendor + harga**.
3. Finance **melakukan approval harga**.
4. GA **melakukan pembelian**.
5. Barang **datang dan dicek kuantitas & kualitas**.
6. Jika **sesuai**, request ditandai **selesai**; jika **tidak**, status **pending**.
7. GA **update stok** di sistem.

---

### **C. Platform & Teknologi**

* Sistem dibangun **berbasis web**.
* **Pop-up notification tidak tersedia di browser**, karena butuh aplikasi yang berjalan di background.

  * Alternatif: **email notification** & **in-system notification**.
* **Desktop app memungkinkan pop-up**, tapi dianggap kurang efisien & umum digunakan.
* **Mobile app bisa push notification**, tapi tidak direncanakan untuk tahap awal.

---

### **D. Visitor Management System (VMS)**

#### **Fitur Utama**

1. QR Code Tamu
2. Buku Tamu Digital
3. Notifikasi Email
4. Validasi akses tamu sebelum entry

---

### **E. Meeting Room Booking**

* Sudah ada **calon vendor**, tidak dibahas detail.

---

### **F. Integrasi dan Output Sistem**

* Sistem **berdiri sendiri**, **tidak terintegrasi dengan Accounting**.
* Namun sistem mampu:

  * Menyimpan data pembelian (vendor, harga, qty)
  * **Generate report biaya pembelian** per bulan/triwulan.
  * **Export Excel** (mirip sistem fingerprint).
* **Target penyelesaian: September**.

---

### **G. Lanjutan & Tindak Lanjut**

* Dibutuhkan:

  * Estimasi waktu pengerjaan stok inventory system.
  * Desain sistem & portfolio untuk pitching ke pihak Korea.
  * Kemungkinan perlu tambahan tim untuk kolaborasi.
