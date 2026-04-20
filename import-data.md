# Panduan Import Data Master SISADIK

Dokumen ini menjelaskan alur dan urutan import data master ke sistem SISADIK melalui fitur Import Excel.

---

## Urutan Import Data

> [!IMPORTANT]
> Urutan import **WAJIB** diikuti karena ada ketergantungan antar tabel (foreign key).
> Jika urutan salah, import akan gagal karena data referensi belum tersedia.

```
┌─────────────────────────────────────────────────────────┐
│               ALUR IMPORT DATA MASTER                   │
│                                                         │
│  TAHAP 1 (Paralel - tidak saling bergantung)            │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐           │
│  │ ① JURUSAN │  │ ② MAPEL   │  │ ③ GURU    │           │
│  │  (Master) │  │  (Master) │  │  (Master) │           │
│  └─────┬─────┘  └───────────┘  └───────────┘           │
│        │                                                │
│  TAHAP 2 (Bergantung pada Jurusan)                      │
│        ▼                                                │
│  ┌───────────┐                                          │
│  │ ④ ROMBEL  │ ← FK: jurusanId                         │
│  └─────┬─────┘                                          │
│        │                                                │
│  TAHAP 3 (Bergantung pada Jurusan & Rombel)             │
│        ▼                                                │
│  ┌───────────┐                                          │
│  │ ⑤ SISWA   │ ← FK: rombelId, jurusanId               │
│  └───────────┘                                          │
└─────────────────────────────────────────────────────────┘
```

| Tahap | Data | Dependency | Akses Menu |
|-------|------|-----------|------------|
| 1 | **Jurusan** | Tidak ada | Dashboard → Jurusan → Import |
| 1 | **Mapel** | Tidak ada | Dashboard → Mapel → Import |
| 1 | **Guru** | Tidak ada | Dashboard → Guru → Import |
| 2 | **Rombel** | Jurusan | Dashboard → Rombel → Import |
| 3 | **Siswa** | Rombel, Jurusan | Dashboard → Siswa → Import |

> [!TIP]
> Tahap 1 (Jurusan, Mapel, Guru) bisa diimport dalam urutan bebas karena ketiganya tidak saling bergantung.

---

## Detail Per Entitas

### ① Jurusan (Tahap 1)

**API Endpoint:** `POST /api/jurusan/import`  
**Template:** `GET /api/jurusan/import` → `template_jurusan.xlsx`  
**Role:** ADMIN

| Kolom | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `Kode_Jurusan` | String | ✅ | Kode unik jurusan (contoh: `TKJ`, `RPL`) |
| `Nama Jurusan` | String | ✅ | Nama lengkap jurusan |
| `Jenjang` | String | ✅ | Jenjang pendidikan (`SMA`, `SMK`, dll) |

**Contoh data:**

| Kode_Jurusan | Nama Jurusan | Jenjang |
|-------------|-------------|---------|
| TKJ | Teknik Komputer dan Jaringan | SMK |
| RPL | Rekayasa Perangkat Lunak | SMK |
| AKL | Akuntansi dan Keuangan Lembaga | SMK |

---

### ② Mapel (Tahap 1)

**API Endpoint:** `POST /api/mapel/import`  
**Template:** `GET /api/mapel/import` → `template-mapel.xlsx`  
**Role:** ADMIN, GURU

| Kolom | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `Kode_Mapel` | String | ✅ | Kode unik mata pelajaran (contoh: `MAT001`) |
| `Nama Mata Pelajaran` | String | ✅ | Nama lengkap mata pelajaran |

**Contoh data:**

| Kode_Mapel | Nama Mata Pelajaran |
|-----------|-------------------|
| MAT001 | Matematika |
| BIN001 | Bahasa Indonesia |
| IPA001 | Ilmu Pengetahuan Alam |

---

### ③ Guru (Tahap 1)

**API Endpoint:** `POST /api/guru/import`  
**Template:** `GET /api/guru/import` → `template-guru.xlsx`  
**Role:** ADMIN

> [!NOTE]
> Import guru **otomatis membuat akun User** dengan role `GURU` untuk setiap guru yang diimport.

| Kolom | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `NIP` | String | ✅ | Nomor Induk Pegawai (unik) |
| `Nama` | String | ✅ | Nama lengkap guru |
| `Username` | String | ✅ | Username untuk login (min 3, max 20 karakter) |
| `Password` | String | ✅ | Password untuk login (min 6 karakter) |
| `Email` | String | ❌ | Email guru (opsional, jika kosong akan auto-generate) |
| `No WA` | String | ❌ | Nomor WhatsApp guru |

**Contoh data:**

| NIP | Nama | Username | Password | Email | No WA |
|-----|------|----------|----------|-------|-------|
| 196812345678901234 | Budi Santoso, S.Pd. | budi123 | password123 | budi@sekolah.com | 081234567890 |

**Auto-generate email:** Jika email kosong → `{username}@sekolah.local`

---

### ④ Rombel (Tahap 2)

**API Endpoint:** `POST /api/rombel/import`  
**Template:** `GET /api/rombel/import` → `template-rombel.xlsx`  
**Role:** ADMIN

> [!WARNING]
> Kolom `Jurusan` mereferensi data Jurusan yang sudah diimport di Tahap 1.
> Bisa diisi dengan **Kode Jurusan** atau **Nama Jurusan** (case-insensitive).

| Kolom | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `Nama Rombel` | String | ✅ | Nama rombongan belajar (contoh: `X RPL 1`) |
| `Tingkat` | String | ✅ | Tingkat kelas (`X`, `XI`, `XII`) |
| `Tahun Pelajaran` | String | ✅ | Tahun pelajaran (contoh: `2024/2025`) |
| `Jurusan` | String | ❌ | Kode atau nama jurusan yang sudah ada |

**Unique constraint:** Kombinasi `Nama Rombel` + `Tahun Pelajaran` harus unik.

**Contoh data:**

| Nama Rombel | Tingkat | Tahun Pelajaran | Jurusan |
|------------|---------|----------------|---------|
| X RPL 1 | X | 2024/2025 | Rekayasa Perangkat Lunak |
| X TKJ 1 | X | 2024/2025 | TKJ |
| XI RPL 1 | XI | 2024/2025 | RPL |

---

### ⑤ Siswa (Tahap 3)

**API Endpoint:** `POST /api/siswa/import`  
**Template:** `GET /api/siswa/import?tahunPelajaran=2024/2025` → `template-siswa.xlsx`  
**Role:** ADMIN, GURU

> [!WARNING]
> Kolom `Nama Rombel` mereferensi data Rombel yang sudah diimport di Tahap 2.
> Rombel dicari berdasarkan kombinasi `Nama Rombel` + `Tahun Pelajaran` yang dipilih saat import.

> [!NOTE]
> Import siswa **otomatis membuat akun User** dengan role `SISWA` dan menghubungkan ke Rombel & Jurusan.

| Kolom | Tipe | Wajib | Keterangan |
|-------|------|-------|------------|
| `NISN` | String | ✅ | Nomor Induk Siswa Nasional (unik) |
| `NIS` | String | ✅ | Nomor Induk Siswa (unik) |
| `Nama Rombel` | String | ✅ | Nama rombel yang sudah ada di sistem |
| `Nama` | String | ✅ | Nama lengkap siswa |
| `JK` | Enum | ✅ | Jenis kelamin: `L` atau `P` |
| `Agama` | String | ✅ | Agama siswa |
| `Username` | String | ✅ | Username untuk login (min 3, max 20 karakter) |
| `Password` | String | ✅ | Password untuk login (min 6 karakter) |
| `Email` | String | ❌ | Email siswa |
| `No WA Siswa` | String | ❌ | Nomor WhatsApp siswa |
| `No WA Orang Tua` | String | ❌ | Nomor WhatsApp orang tua |

**Parameter tambahan:** `tahunPelajaran` — digunakan untuk mencari rombel yang sesuai.

**Contoh data:**

| NISN | NIS | Nama Rombel | Nama | JK | Agama | Username | Password | Email | No WA Siswa | No WA Orang Tua |
|------|-----|------------|------|----|----|----------|----------|-------|-------------|----------------|
| 0012345678 | 12345 | X RPL 1 | Ahmad Siswa | L | Islam | ahmad123 | password123 | ahmad@sekolah.com | 081234567890 | 081234567891 |

---

## Checklist Import

Gunakan checklist berikut untuk memastikan seluruh data master sudah diimport dengan benar:

- [ ] **Setting Sekolah** sudah dikonfigurasi (nama sekolah, tahun pelajaran, dll)
- [ ] **Jurusan** — semua jurusan/program keahlian sudah diimport
- [ ] **Mapel** — semua mata pelajaran sudah diimport
- [ ] **Guru** — semua guru sudah diimport (pastikan akun login sudah benar)
- [ ] **Rombel** — semua rombongan belajar sudah diimport (cek kolom jurusan cocok)
- [ ] **Siswa** — semua siswa sudah diimport (cek kolom rombel cocok)

---

## Troubleshooting

| Error | Penyebab | Solusi |
|-------|----------|--------|
| `Jurusan "X" tidak ditemukan` | Import rombel sebelum jurusan | Import jurusan terlebih dahulu |
| `Rombel "X" tidak ditemukan untuk tahun Y` | Import siswa sebelum rombel, atau tahun pelajaran salah | Import rombel dulu & pastikan tahun pelajaran cocok |
| `NISN/NIS sudah ada` | Data duplikat | Cek dan hapus data duplikat di Excel |
| `Username sudah ada` | Username sudah dipakai user lain | Gunakan username yang berbeda |
| `File harus berformat Excel` | Upload file selain `.xlsx`/`.xls` | Gunakan format Excel yang benar |
| `Hanya admin yang dapat mengimpor` | Role user bukan ADMIN | Login sebagai ADMIN |
