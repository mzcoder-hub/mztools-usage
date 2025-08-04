
## 🎯 **Tujuan Sistem**

Membangun **Stationery Stock System** untuk memantau dan mengontrol penggunaan ATK secara efisien dan transparan di lingkungan perusahaan, agar:

* Karyawan lebih **sadar** terhadap penggunaan ATK.
* Perusahaan dapat **menghemat biaya operasional** dengan pengendalian penggunaan.
* Admin dan manajemen dapat **mengambil keputusan berdasarkan data** penggunaan ATK.

---

## 🧩 **Fitur Utama Sistem**

### 1. 📦 **Manajemen Stok dan Pemasukan Barang**

* Klasifikasi barang berdasarkan:

  * **Jenis Barang** (Pulpen, Kertas, Map, dll)
  * **Satuan** (pcs, pack, rim)
  * **Harga satuan**
  * **Tanggal masuk (in)** dan **Tanggal keluar (out)** saat distribusi
* Input data stok bisa dilakukan oleh Admin Gudang ATK.
* Sistem otomatis menghitung **stok akhir** setiap barang berdasarkan transaksi masuk dan keluar.

### 2. 📊 **Laporan Penggunaan oleh Departemen**

* Statistik pemakaian berdasarkan:

  * Departemen mana yang **paling banyak menggunakan** barang (total rupiah)
  * Departemen mana yang **paling sedikit menggunakan** barang
  * **Ranking** pemakaian per departemen (descending dari tertinggi)
  * Item ATK yang **paling banyak digunakan di seluruh perusahaan**
  * Rata-rata pemakaian **per item per bulan**
* Bisa di-export dalam format PDF, Excel, atau ditampilkan dalam dashboard interaktif.

### 3. 🧑‍💼 **Laporan Penggunaan per Karyawan**

* Setiap permintaan ATK akan dicatat atas nama karyawan (menggunakan ID Karyawan).
* Admin atau atasan dapat melihat:

  * Karyawan **dengan total penggunaan ATK tertinggi** (berdasarkan nilai rupiah)
  * **Item apa** saja yang paling sering diminta oleh karyawan tersebut
  * **Ranking** penggunaan per karyawan dalam 1 departemen atau seluruh perusahaan

---

## 🧠 **Logika Tambahan Sistem**

* **Request flow**:

  1. Karyawan membuat permintaan ATK (via form online).
  2. Permintaan diverifikasi oleh supervisor.
  3. Admin gudang mencatat dan mencetak bukti serah terima.
  4. Stok berkurang dan tercatat sebagai “tanggal keluar” dengan nama peminta.

* **Threshold Notifikasi**:

  * Jika stok suatu barang mendekati batas minimum, sistem memberikan notifikasi ke Admin.

* **Histori Transaksi**:

  * Semua data keluar-masuk tercatat dengan histori lengkap untuk audit.

---

## 🖥️ **Komponen Sistem (Modul)**

| Modul            | Deskripsi                                                     |
| ---------------- | ------------------------------------------------------------- |
| Modul Barang     | CRUD untuk data ATK, jenis, satuan, harga                     |
| Modul Stok       | Input barang masuk dan keluar                                 |
| Modul Permintaan | Karyawan request ATK melalui sistem                           |
| Modul Approval   | Supervisor menyetujui atau menolak request                    |
| Modul Distribusi | Admin mencatat distribusi barang ke karyawan                  |
| Modul Laporan    | Laporan per departemen, per karyawan, per item                |
| Modul Notifikasi | Peringatan untuk stok minimum                                 |
| Modul Role Akses | Akses berdasarkan peran: Admin, Karyawan, Supervisor, Manager |

---

## 📈 **Contoh Output Laporan**

### 🔹 Departemen Usage Report:

| Departemen | Total Pemakaian (Rp) | Rata-rata Bulanan (Rp) | Peringkat |
| ---------- | -------------------- | ---------------------- | --------- |
| Finance    | 2.500.000            | 208.000                | 1         |
| HRD        | 1.200.000            | 100.000                | 2         |
| IT         | 850.000              | 71.000                 | 3         |

### 🔹 Karyawan Usage Report (HRD):

| Nama Karyawan | Total Penggunaan (Rp) | Item Terbanyak Digunakan | Ranking |
| ------------- | --------------------- | ------------------------ | ------- |
| Rani Kusuma   | 600.000               | Pulpen                   | 1       |
| Budi Hartono  | 350.000               | Kertas A4                | 2       |
