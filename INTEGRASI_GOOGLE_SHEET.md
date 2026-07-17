# 📊 Integrasi Data Modul Aktiviti dari Google Sheet

## Gambaran Umum
Aplikasi e-kokosmart mengintegrasikan data modul aktiviti unit beruniform dari Google Sheet menggunakan Google Apps Script.

---

## 🔧 Setup Google Apps Script

### Langkah 1: Buka Google Sheet Anda
1. Buka link Google Sheet: https://docs.google.com/spreadsheets/d/1xre2iGen2J3k7z9uGHjMzdTnD1wUUz5Ebn392Dk3ho0/edit
2. Klik **Extensions > Apps Script**

### Langkah 2: Salin Kode Google Apps Script
Salin dan paste kode berikut di **Apps Script Editor**:

```javascript
// ========== GOOGLE SHEET CONFIGURATION ==========
const SHEET_ID = '1xre2iGen2J3k7z9uGHjMzdTnD1wUUz5Ebn392Dk3ho0';
const ACTIVITIES_SHEET_NAME = 'Aktiviti'; // Nama sheet untuk aktiviti
const UNITS_SHEET_NAME = 'Unit'; // Nama sheet untuk unit
const TEACHERS_SHEET_NAME = 'Guru'; // Nama sheet untuk guru
const STUDENTS_SHEET_NAME = 'Pelajar'; // Nama sheet untuk pelajar

// ========== DATA RETRIEVAL FUNCTIONS ==========

// Ambil data aktiviti dari Google Sheet
function getActivities() {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(ACTIVITIES_SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const activities = [];
  for (let i = 1; i < data.length; i++) { // Mulai dari baris 2 (skip header)
    const row = data[i];
    if (row[0]) { // Pastikan nama aktiviti ada
      activities.push({
        name: row[0],
        date: row[1],
        venue: row[2],
        teacher: row[3],
        status: row[4]
      });
    }
  }
  
  return activities;
}

// Ambil data unit dengan aktiviti mereka
function getUnitActivities() {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(UNITS_SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const unitActivities = [];
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    if (row[0]) {
      unitActivities.push({
        name: row[0],
        icon: row[1] || '🎖️',
        activities: row[2] ? JSON.parse(row[2]) : []
      });
    }
  }
  
  return unitActivities;
}

// Ambil data guru
function getTeachers() {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(TEACHERS_SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const teachers = [];
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    if (row[0]) {
      teachers.push({
        name: row[0],
        id: row[1],
        pos: row[2],
        unit: row[3],
        phone: row[4]
      });
    }
  }
  
  return teachers;
}

// Ambil data pelajar
function getStudents() {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(STUDENTS_SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const students = [];
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    if (row[0]) {
      students.push({
        name: row[0],
        class: row[1],
        gender: row[2],
        unit: row[3],
        attendance: row[4]
      });
    }
  }
  
  return students;
}

// Ambil data unit
function getUnits() {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(UNITS_SHEET_NAME);
  const data = sheet.getDataRange().getValues();
  
  const units = [];
  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    if (row[0]) {
      units.push({
        name: row[0],
        advisor: row[1],
        asst: row[2],
        schedule: row[3],
        students: row[4] ? parseInt(row[4]) : 0
      });
    }
  }
  
  return units;
}

// ========== WEB APP HANDLER ==========

function doGet(e) {
  const action = e.parameter.action;
  
  let response;
  switch(action) {
    case 'getActivities':
      response = getActivities();
      break;
    case 'getUnitActivities':
      response = getUnitActivities();
      break;
    case 'getTeachers':
      response = getTeachers();
      break;
    case 'getStudents':
      response = getStudents();
      break;
    case 'getUnits':
      response = getUnits();
      break;
    default:
      response = { error: 'Action tidak dikenali' };
  }
  
  return ContentService.createTextOutput(JSON.stringify(response))
    .setMimeType(ContentService.MimeType.JSON);
}

// ========== CORS SUPPORT ==========
function doOptions(e) {
  return HtmlService.createHtmlOutput('OK');
}
```

### Langkah 3: Deploy Google Apps Script
1. Klik **Deploy > New Deployment**
2. Pilih tipe: **Web App**
3. Execute as: Pilih akun Anda
4. Who has access: **Anyone**
5. Klik **Deploy**
6. **Copy URL yang ditampilkan** — ini adalah endpoint Anda

---

## 📝 Format Google Sheet

### Sheet: "Aktiviti" (Columns: A-E)
| Nama Aktiviti | Tarikh | Lokasi | Guru | Status |
|---|---|---|---|---|
| Perkhemahan Pengakap 2026 | 22 Ogos 2026 | Padang Sekolah | En. Amirul Hakim | Akan Datang |

### Sheet: "Unit" (Columns: A-D)
| Nama Unit | Ikon | Aktiviti JSON | ... |
|---|---|---|---|
| Kadet Remaja Sekolah (KRS) | 🎖️ | [{"title":"Latihan Perbarisan","date":"Setiap Sabtu","status":"Aktif"}] | ... |

### Sheet: "Guru" (Columns: A-E)
| Nama | ID Staf | Jawatan | Unit | Telefon |
|---|---|---|---|---|
| Zulkifli Osman | GR-0231 | Guru Penasihat | Kadet Remaja Sekolah | 019-8801234 |

### Sheet: "Pelajar" (Columns: A-E)
| Nama | Kelas | Jantina | Unit | Kehadiran |
|---|---|---|---|---|
| Ahmad Bin Ismail | 4 Cemerlang | Lelaki | Pengakap | 96% |

---

## 🔌 Update index.html

Ganti endpoint GAS_ENDPOINT di baris 408 dengan URL yang Anda dapat dari deployment:

```javascript
const GAS_ENDPOINT = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
```

---

## ✅ Testing

1. Buka aplikasi di browser
2. Login dengan email dan password
3. Lihat di **Console** (F12 > Console) pesan:
   - ✅ Data guru dimuat
   - ✅ Data pelajar dimuat  
   - ✅ Data unit dimuat

---

## 🔐 Keamanan

- Endpoint Google Apps Script dapat diakses publik (untuk kemudahan)
- Untuk production, tambahkan authentication
- Pertimbangkan menggunakan Firebase Authentication

---

## 📞 Support

Jika ada error:
1. Cek console browser (F12)
2. Verifikasi nama sheet di Google Sheet
3. Pastikan kolom data sesuai urutan

