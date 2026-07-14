# Guideline: Override Appraiser 1 & 2 (Department Approver)

Cara mengisi appraiser 1 dan appraiser 2 secara **override per-karyawan** di modul
Performance Management. Override dipakai saat seorang karyawan tidak ter-cover oleh
group-rule (kombinasi department + division + employee_level_group) yang ada.

## Konsep singkat

Appraiser di-resolve **live tiap request** oleh `AppraiserResolver::approverAtLevel`
(`otto-be/app/Services/AppraiserResolver.php`). Urutan prioritas:

1. **Override per-user** (`target_user_id = karyawan`, `level`) — menang duluan.
2. **Group-rule divisi-spesifik** (`department_id` + `level` + `employee_level_group` + `division_id`).
3. **Group-rule fallback** (sama, tapi `division_id = NULL`).

Karena live, override langsung efektif di fetch MBO berikutnya — **tidak perlu**
regenerate / resubmit MBO. Appraiser override otomatis melihat MBO karyawan di list
approval-nya.

## GOTCHA UTAMA — override wajib lengkap 2 level

`hasOverride()` mengecek **ada tidaknya row override APAPUN** untuk karyawan:

```php
if ($override) return $override;          // level ini ada override → pakai
if ($this->hasOverride($employee)) return null;  // ada override level LAIN → group-rule DIMATIKAN
```

Artinya: begitu satu karyawan punya minimal satu row override (level manapun),
**semua group-rule untuk karyawan itu diabaikan**. Jadi:

- Set override level 1 SAJA → appraiser 1 = override, appraiser 2 = **null**
  (walau group-rule level 2 sebelumnya ada).
- **Selalu isi appraiser 1 DAN appraiser 2 bersamaan** saat override, kecuali memang
  sengaja ingin karyawan itu tanpa appraiser 2.

## Cara 1 — Bulk override (disarankan)

Set appraiser 1 + 2 yang sama untuk satu atau banyak karyawan sekaligus. Field
`department_id`, `division_id`, `employee_level_group` **auto-derive** dari tiap
karyawan — tidak perlu diisi.

**Endpoint:** `POST /api/performance-management/department-approvers/bulk-override`

```json
{
  "target_user_ids": ["<user_id_karyawan_1>", "<user_id_karyawan_2>"],
  "appraiser_1_id": "<user_id_appraiser_1>",
  "appraiser_2_id": "<user_id_appraiser_2>"
}
```

Aturan:

- `target_user_ids` — wajib, array, minimal 1 karyawan.
- Minimal salah satu dari `appraiser_1_id` / `appraiser_2_id` wajib.
- **Additive** — level dengan appraiser `null` TIDAK disentuh (tidak dibuat, tidak
  dihapus). Untuk isi dua level, kirim dua-duanya (lihat gotcha di atas).
- Karyawan tanpa `employee_level_group` di-skip dan dilaporkan di `summary.skipped`.

Respon: `{ "message": "...", "summary": { "created": n, "updated": n, "skipped": [...] } }`

## Cara 2 — Single create (satu row, satu level)

**Endpoint:** `POST /api/performance-management/department-approvers`

```json
{
  "target_user_id": "<user_id_karyawan>",
  "user_id": "<user_id_appraiser>",
  "level": 1,
  "division_id": "<division_id>"
}
```

Aturan validasi (`StoreDepartmentApproverRequest`):

- `user_id` (appraiser) — wajib, harus ada di `users`.
- `level` — wajib, `1` atau `2`.
- `target_user_id` — untuk override. Jika diisi, `department_id`,
  `division_id`, `employee_level_group` **tidak wajib** (diturunkan dari karyawan).
- `department_id` + `employee_level_group` hanya wajib untuk **group-rule**
  (saat `target_user_id` kosong).

Untuk override 2 level → kirim 2 request (level 1 dan level 2) atau pakai bulk.

## Nilai `employee_level_group` (untuk group-rule, bukan override)

`STAFF`, `TEAM_HEAD`, `MANAGER`, `SUPERVISOR`, `DEPARTMENT_SECTION_HEAD`,
`DIVISION_HEAD`, `DEPARTMENT_HEAD`, `DIRECTOR`.

## Verifikasi setelah set

Cek hasil resolve:

```
GET /api/performance-management/mbo/{mboHeaderId}
```

Lihat field `first_appraiser` dan `second_appraiser` — harus terisi sesuai override.
Atau list row override:

```
GET /api/performance-management/department-approvers?target_user_id=<user_id>
```

## Menghapus override

`DELETE /api/performance-management/department-approvers/{id}`

Setelah semua row override karyawan dihapus, `hasOverride` kembali `false` dan
group-rule aktif lagi.

---

# Step-by-step via UI (Portal)

Halaman: **Dashboard → Performance Management → Department Approvers**
(`/dashboard/performance-management/department-approvers`).

## A. Membuat override (Appraiser 1 & 2) untuk karyawan

1. Klik tombol **"Add Approver"** (kanan atas tabel).
2. Di field **"Assignment Type"**, pilih **"Specific employee (override)"**.
   (Default-nya "Group rule" — wajib diganti ke Specific.)
3. **"Specific Employees (appraised)"** — ketik nama / employee ID karyawan yang
   di-appraise, klik hasilnya. Bisa pilih **banyak karyawan** sekaligus (muncul
   sebagai chip; klik ✕ untuk hapus). Semua karyawan terpilih akan dapat appraiser
   1 & 2 yang sama.
4. **"Appraiser 1 (First — MBO + CA)"** — cari & pilih user appraiser 1.
5. **"Appraiser 2 (Final — MBO only)"** — cari & pilih user appraiser 2.
   > Isi **kedua-duanya**. Lihat gotcha di atas: begitu override dibuat, group-rule
   > untuk karyawan itu mati total; level yang tak diisi jadi **null**.
6. Klik **"Assign"**. Muncul toast "Appraisers saved successfully".

Efek langsung: fetch MBO karyawan berikutnya, `first_appraiser` & `second_appraiser`
terisi; appraiser melihat MBO di list approval-nya. Tak perlu regen MBO.

## B. Mengubah / melengkapi override yang sudah ada

1. Di tabel, cari baris karyawan — ditandai badge kuning **"Specific: <nama>"**.
2. Klik ikon **pensil (Edit)** di baris itu.
3. Modal terbuka dengan Appraiser 1 & 2 saat ini ter-load. Ubah:
   - Ganti appraiser → cari user baru.
   - **Kosongkan** appraiser (klik ✕) → level itu **dihapus** saat disimpan.
4. Klik **"Update"**.

> Edit mode meng-upsert kedua level untuk satu karyawan. Mengosongkan appraiser 1
> menghapus row level 1 → appraiser 1 kembali null (group-rule tetap TIDAK aktif
> selama masih ada row override level lain).

## C. Menghapus override (kembali ke group-rule)

1. Di baris "Specific: <nama>", klik ikon **hapus (trash)**.
2. Konfirmasi di dialog "Remove Approver".
3. Ulangi untuk level 1 dan level 2 (dua baris terpisah). Setelah **semua** row
   override karyawan hilang, group-rule normal aktif lagi.

## D. Bulk lewat Import Excel (banyak karyawan)

Untuk volume besar, pakai **"Import"** (kanan atas):

1. Klik **"Export"** dulu untuk ambil template / lihat format kolom, atau pakai
   template dari endpoint template.
2. Isi baris override, upload via **"Import"**.
3. Format kolom mengikuti `DepartmentApproverTemplateExport` — set kolom target
   employee + appraiser + level.

## Verifikasi via UI

- Baris karyawan muncul dengan badge **"Specific: <nama>"** dan level yang benar.
- Buka MBO karyawan (Performance Management → MBO) — kolom appraiser terisi.
