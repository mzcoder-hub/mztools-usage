Jika Anda memiliki **dua HDD (894.253 GB)** dan server digunakan untuk **program dan database SIMRS**, berikut adalah rekomendasi pembagian partisi tanpa RAID untuk mengoptimalkan performa dan penggunaan ruang.

---

### **1. Prinsip Pembagian Disk**
- **Disk 1**: Sistem operasi dan aplikasi.
- **Disk 2**: Database dan log untuk memisahkan beban baca/tulis disk.

---

### **2. Rekomendasi Partisi**
#### **Disk 1 (Sistem Operasi dan Program)**
- **Sistem Operasi, aplikasi, dan file sementara (temp)**.
- Alokasi ruang cukup untuk program SIMRS dan aplikasi lainnya.

| Partisi      | Ukuran       | Mount Point | Filesystem | Keterangan                              |
|--------------|--------------|-------------|------------|------------------------------------------|
| `/boot`      | 500 MB       | /boot       | Ext4       | Untuk file boot loader.                  |
| `Swap`       | 8 GB         | Swap        | Swap       | Untuk menggantikan/mendukung RAM.        |
| `/` (root)   | 50 GB        | /           | Ext4       | Sistem operasi utama dan aplikasi.       |
| `/home`      | 100 GB       | /home       | Ext4       | Untuk data pengguna/server administrator.|
| `/var`       | 200 GB       | /var        | Ext4       | Untuk file log, cache aplikasi.          |
| `/tmp`       | 10 GB        | /tmp        | Ext4/tmpfs | Untuk file sementara aplikasi.           |
| Sisa ruang   | 525 GB       | /srv        | Ext4       | Untuk file aplikasi SIMRS (jika perlu).  |

#### **Disk 2 (Database)**
- Fokus untuk menyimpan database dan file penting lainnya.

| Partisi      | Ukuran       | Mount Point | Filesystem | Keterangan                               |
|--------------|--------------|-------------|------------|------------------------------------------|
| `/db`        | 800 GB       | /db         | Ext4       | Untuk database SIMRS (e.g., MySQL/PostgreSQL). |
| Sisa ruang   | 94 GB        | /backup     | Ext4       | Untuk backup database secara lokal.      |

---

### **3. Penjelasan**
1. **Disk 1**:
   - Semua kebutuhan sistem operasi, aplikasi SIMRS, dan log akan diletakkan di disk ini.
   - Partisi `/var` diperbesar karena aplikasi SIMRS dan sistem sering menghasilkan banyak log.
   - `Swap` sebesar 8 GB cukup untuk mendukung memori virtual jika server memiliki RAM > 8 GB.

2. **Disk 2**:
   - Database SIMRS disimpan di partisi `/db` untuk memisahkan I/O operasi database dari sistem operasi, meningkatkan performa.
   - Backup database ditempatkan di partisi `/backup` untuk redundansi lokal (selain backup eksternal).

---

### **4. Alasan Menggunakan Konfigurasi Ini**
- **Memisahkan sistem operasi dari database**:
  - Database biasanya melakukan banyak operasi baca/tulis. Memisahkannya dari sistem operasi mencegah konflik I/O.
- **Ukuran fleksibel**:
  - Kapasitas besar disediakan untuk database karena biasanya data SIMRS berkembang pesat.
- **Backup lokal**:
  - Dengan partisi `/backup`, Anda dapat menyimpan snapshot database tanpa mengganggu partisi lainnya.

---

### **5. Konfigurasi Filesystem**
- Gunakan **Ext4** untuk stabilitas dan kompatibilitas.
- Anda juga bisa mempertimbangkan **XFS** untuk partisi database (`/db`) jika membutuhkan performa lebih baik untuk file besar.

---

### **6. Contoh Fstab untuk Mounting**
Untuk memastikan partisi ter-mount dengan benar:

```bash
UUID=<UUID_BOOT> /boot ext4 defaults 0 2
UUID=<UUID_SWAP> none swap sw 0 0
UUID=<UUID_ROOT> / ext4 defaults 0 1
UUID=<UUID_HOME> /home ext4 defaults 0 2
UUID=<UUID_VAR> /var ext4 defaults 0 2
UUID=<UUID_TMP> /tmp ext4 defaults 0 2
UUID=<UUID_SRV> /srv ext4 defaults 0 2
UUID=<UUID_DB> /db ext4 defaults 0 2
UUID=<UUID_BACKUP> /backup ext4 defaults 0 2
```

---

### **7. Tips Tambahan**
1. **Monitoring Disk Usage**:
   - Pasang alat seperti `iotop` atau `dstat` untuk memantau penggunaan disk secara real-time.
2. **Backup Eksternal**:
   - Pastikan ada mekanisme backup ke media eksternal atau cloud untuk menghindari kehilangan data.

Jika Anda memerlukan panduan lebih lanjut untuk implementasi, beri tahu saya!
