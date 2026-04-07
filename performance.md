Tentu, ini adalah representasi struktur data JSON berdasarkan schema yang Anda buat. JSON ini mensimulasikan satu siklus penuh untuk seorang karyawan, mulai dari penetapan target MBO hingga penilaian CA di akhir tahun.

### **1. Appraisal Period & Schedule**
[cite_start]Data ini digunakan oleh Next.js untuk menentukan menu mana yang aktif (Open/Locked)[cite: 20, 21, 33, 34].

```json
{
  "period": {
    "id": 1,
    "year": 2026,
    "is_active": true,
    "schedules": [
      { "stage": "SETTING", "start": "2026-01-01", "end": "2026-01-31", "is_open": false },
      { "stage": "MID_YEAR", "start": "2026-06-01", "end": "2026-06-30", "is_open": true },
      { "stage": "ADJUST_PLAN", "start": "2026-07-01", "end": "2026-07-31", "is_open": false },
      { "stage": "YEAR_END", "start": "2026-12-01", "end": "2026-12-31", "is_open": false }
    ]
  }
}
```

---

### **2. MBO (Management by Objective)**
[cite_start]Data ini mencakup detail target tim (30%) dan individu (70%) beserta progres evaluasinya[cite: 6].

```json
{
  "mbo_header": {
    "id": 101,
    "user_id": "USR-99",
    "employee_name": "Ari",
    "status": "MID_YEAR_SUBMITTED",
    "total_score": 85.5,
    "items": [
      {
        "category": "TEAM",
        "weight": 30.0,
        "objective": "Meningkatkan efisiensi sistem internal sebesar 20%",
        "measure_criteria": "Waktu response API < 200ms",
        "mid_year_achievement": "Response time saat ini rata-rata 250ms",
        "mid_year_score": 75,
        "year_end_score": null
      },
      {
        "category": "INDIVIDUAL",
        "weight": 70.0,
        "objective": "Implementasi modul Performance Management di Laravel & Next.js",
        "measure_criteria": "Modul deploy ke production tanpa major bug",
        "mid_year_achievement": "Schema DB dan API dasar selesai",
        "mid_year_score": 90,
        "year_end_score": null
      }
    ],
    "pdp": [
      {
        "activity": "Training Next.js Advanced",
        "target": "Sertifikasi completion",
        "remarks": "Selesai di bulan Maret"
      }
    ],
    "career_aspiration": {
      "aspirations": "Ingin memimpin tim engineering di tahun depan"
    }
  }
}
```

---

### **3. CA (Competency Appraisal)**
[cite_start]Data penilaian kompetensi yang mencakup aspek perilaku dan sikap kerja[cite: 56, 57, 58, 59].

```json
{
  "ca_header": {
    "id": 201,
    "user_id": "USR-99",
    "status": "PENDING",
    "final_average_score": 0,
    "appraiser_remark": "Menunggu penilaian Desember",
    "items": [
      { "criteria": "ATTENDANCE", "score": 95, "weight": 25.0, "comment": "Sangat disiplin" },
      { "criteria": "KNOWLEDGE", "score": 88, "weight": 25.0, "comment": "Menguasai framework dengan baik" },
      { "criteria": "WORK RESULT", "score": 85, "weight": 25.0, "comment": "Hasil kerja rapi" },
      { "criteria": "WORK ATTITUDE", "score": 90, "weight": 25.0, "comment": "Proaktif dalam tim" }
    ]
  }
}
```

---

### **4. Approval History**
[cite_start]Log untuk melihat siapa yang menyetujui dokumen tersebut[cite: 36, 46].

```json
{
  "approvals": [
    {
      "stage": "MID_YEAR",
      "approver_name": "Budi (Manager)",
      "status": "APPROVED",
      "comment": "Target sudah sesuai jalur, tingkatkan di semester 2",
      "approved_at": "2026-06-15T10:00:00Z"
    }
  ]
}
```

### **Catatan Implementasi di Next.js:**
* Gunakan `mbo_header.status` untuk mengatur *read-only* pada form. Jika status sudah `SUBMITTED`, maka `input` di frontend harus `disabled`.
* [cite_start]Gunakan `is_open` dari `schedules` untuk menyembunyikan atau menampilkan tombol "Edit" atau "Submit" di Dashboard[cite: 25, 35, 49].
