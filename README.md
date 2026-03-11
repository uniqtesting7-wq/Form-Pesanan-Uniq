# 📦 UNIQ CELL — Order Management System

> Aplikasi form pemesan digital berbasis web untuk manajemen order pemasangan CCTV.
> Sinkron real-time antar perangkat via Firebase, responsive di semua ukuran layar.

---

## 🌐 Live Demo

🔗 **https://form-pesanan-uniq.vercel.app**

---

## ✨ Fitur Utama

| Fitur | Keterangan |
|---|---|
| 📱 Multi-Pelanggan | Kelola banyak pelanggan sekaligus via sistem tab |
| ☁️ Firebase Sync | Sinkron real-time ke semua perangkat otomatis |
| 💾 Auto-Save | Setiap perubahan tersimpan otomatis (debounce 1 detik) |
| 💚 Kirim WhatsApp | Ringkasan order langsung dikirim ke nomor WA tujuan |
| 📊 Google Sheets | Export data order ke spreadsheet via Apps Script |
| 🖨️ Cetak / PDF | Print-ready, elemen navigasi otomatis disembunyikan |
| 🎬 GSAP Preloader | Animasi loading screen saat pertama buka |
| 📱 Responsive | Nyaman di HP, tablet, maupun desktop |

---

## 🗂️ Struktur File

```
/
├── form-pemesan.html   # File utama (semua-dalam-satu: HTML + CSS + JS)
├── vercel.json         # Konfigurasi routing Vercel
└── README.md           # Dokumentasi ini
```

> Tidak ada dependencies yang perlu di-install. Semua library dimuat via CDN.

---

## 🔧 Library & CDN

| Library | Versi | Kegunaan |
|---|---|---|
| Bootstrap | 5.3.3 | Layout & komponen UI |
| Bootstrap Icons | 1.11.3 | Ikon tombol |
| Firebase Compat SDK | 10.12.0 | Koneksi Realtime Database |
| GSAP | 3.12.5 | Animasi preloader |

---

## 🔥 Konfigurasi Firebase

### Project Info
| Key | Value |
|---|---|
| Project ID | `form-pesanan-uniq` |
| Database URL | `https://form-pesanan-uniq-default-rtdb.firebaseio.com` |
| Auth Domain | `form-pesanan-uniq.firebaseapp.com` |
| Storage Bucket | `form-pesanan-uniq.firebasestorage.app` |

### Config (sudah di-hardcode di HTML)
```js
firebase.initializeApp({
  apiKey:            "AIzaSyBjTKyBkKdZ9d1p2XS2PrLMpHsvvqK0jYk",
  authDomain:        "form-pesanan-uniq.firebaseapp.com",
  databaseURL:       "https://form-pesanan-uniq-default-rtdb.firebaseio.com",
  projectId:         "form-pesanan-uniq",
  storageBucket:     "form-pesanan-uniq.firebasestorage.app",
  messagingSenderId: "507202084583",
  appId:             "1:507202084583:web:b097401aefce60bd03cc42"
});
```

### Rules Realtime Database
Agar data bisa dibaca & ditulis dari browser tanpa autentikasi:
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```
> ⚠️ Cocok untuk penggunaan internal/pribadi. Untuk produksi publik, gunakan autentikasi Firebase.

---

## 🗄️ Struktur Data Firebase

Semua data disimpan di satu path: **`appData`**

```
appData/
├── pelangganList[]          # Array daftar pelanggan (urutan tab)
│   ├── id: "abc123"         # ID unik (timestamp + random)
│   └── nama: "Ko Danny"     # Nama pelanggan (label tab)
│
└── data/
    └── [id pelanggan]/
        ├── pelanggan/
        │   ├── nama: "Ko Danny Toko Biru"
        │   ├── noOrder: "PT JL0001269"
        │   ├── noUpb: "UPB JL00000481"
        │   └── tanggal: "2026-03-11"
        │
        ├── material[]
        │   ├── { nama, qty, harga }
        │   └── ...
        │
        ├── aksesoris[]
        │   ├── { nama, qty, harga }
        │   └── ...
        │
        └── jasa/
            ├── desc: "Jasa Pemasangan + Setting"
            ├── qty: "16"
            └── harga: "200.000"
```

### Contoh Data Nyata di Firebase
```json
{
  "appData": {
    "pelangganList": [
      { "id": "abc123", "nama": "Ko Danny Toko Biru" },
      { "id": "def456", "nama": "Bu Sari Toko Merah" }
    ],
    "data": {
      "abc123": {
        "pelanggan": {
          "nama": "Ko Danny Toko Biru",
          "noOrder": "PT JL0001269",
          "noUpb": "UPB JL00000481",
          "tanggal": "2026-03-11"
        },
        "material": [
          { "nama": "Camera Analog Hikvison 16D0T", "qty": 14, "harga": 365000 },
          { "nama": "Hardisk 2TB Skyhawk", "qty": 1, "harga": 1965000 }
        ],
        "aksesoris": [
          { "nama": "Dc Male", "qty": 16, "harga": 3500 },
          { "nama": "Bnc Drat", "qty": 32, "harga": 5000 }
        ],
        "jasa": {
          "desc": "Jasa Pemasangan + Setting",
          "qty": "16",
          "harga": "200.000"
        }
      }
    }
  }
}
```

---

## ⚙️ Cara Kerja Sistem

### 1. Inisialisasi
```
Buka halaman
  → GSAP preloader animasi
  → Firebase listener aktif (db.ref("appData").on("value", ...))
  → Jika data ada → load ke form
  → Jika kosong  → buat 1 pelanggan default kosong
  → Preloader hilang, form muncul
```

### 2. Auto-Save Flow
```
User edit form
  → autoSave() dipanggil
  → simpanStateAktif() → allData[activeId] = ambilState()
  → renderTabs() (update nama tab)
  → hitungSemua() (hitung ulang total)
  → setTimeout 1 detik (debounce)
  → simpanKeFirebase() → dbRef.set({ pelangganList, data: allData })
  → Status: "Tersinkron real-time ☁️"
```

### 3. Multi-Pelanggan (Tab)
```
Klik + (tambah)
  → Modal input nama pelanggan
  → konfirmasiTambah()
  → Buat ID unik: uid() = timestamp36 + random
  → Buat formKosong(nama) → data baru KOSONG
  → Simpan ke pelangganList & allData
  → gantiTab(id baru) → isiForm(data kosong)
  → simpanKeFirebase()

Klik tab lain
  → simpanStateAktif() → simpan tab sekarang dulu
  → gantiTab(id baru)
  → isiForm(allData[id baru])

Klik × (hapus)
  → Konfirmasi
  → hapusPelanggan(id)
  → Pindah ke tab pertama
  → simpanKeFirebase()
```

### 4. Real-time Sync
```
Firebase Listener aktif terus selama halaman terbuka.
Setiap ada perubahan dari device lain:
  → isFromFirebase = true  (mencegah save loop)
  → isiForm(data baru)
  → isFromFirebase = false
```

---

## 📊 Google Sheets Integration

### Setup (sekali saja)
1. Buka [Google Sheets](https://sheets.google.com) → buat spreadsheet baru
2. Klik **Extensions → Apps Script**
3. Hapus kode default, paste script berikut:

```js
function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sh = ss.getSheetByName("Orders") || ss.insertSheet("Orders");
  const d  = JSON.parse(e.postData.contents);
  if (sh.getLastRow() === 0)
    sh.appendRow(["Waktu","Nama","No PT","No UPB","Tanggal","Barang","Qty","Harga","Subtotal","Kategori"]);
  const rows = [];
  d.material.forEach(i  => rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Material"]));
  d.aksesoris.forEach(i => rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Aksesoris"]));
  rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,d.jasaDesc,d.jasaQty,d.jasaHarga,d.jasaQty*d.jasaHarga,"Jasa"]);
  rows.forEach(r => sh.appendRow(r));
  return ContentService.createTextOutput(JSON.stringify({status:"ok"})).setMimeType(ContentService.MimeType.JSON);
}
```

4. Klik **Deploy → New deployment**
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy URL → paste di form (tombol 🟢 Sheets)

---

## 🚀 Deploy ke Vercel

### `vercel.json`
```json
{
  "rewrites": [
    { "source": "/", "destination": "/form-pemesan.html" }
  ]
}
```

### Langkah Deploy
1. Push file ke GitHub
2. Buka [vercel.com](https://vercel.com) → Import repository
3. Vercel otomatis deploy setiap `git push`

---

## 🏗️ Arsitektur JavaScript

### State Variables
| Variable | Type | Keterangan |
|---|---|---|
| `pelangganList` | `Array` | Daftar pelanggan & urutan tab |
| `allData` | `Object` | Semua data form per pelanggan |
| `activeId` | `String` | ID pelanggan yang sedang aktif |
| `dataMaterial` | `Array` | Baris material tab aktif |
| `dataAksesoris` | `Array` | Baris aksesoris tab aktif |
| `saveTimer` | `Timeout` | Debounce timer auto-save |
| `isFromFirebase` | `Boolean` | Flag mencegah save loop |

### Fungsi Utama
| Fungsi | Keterangan |
|---|---|
| `formKosong(nama)` | Buat object data kosong untuk pelanggan baru |
| `ambilState()` | Baca semua nilai dari DOM → object |
| `isiForm(s)` | Isi DOM dari object data |
| `simpanStateAktif()` | Simpan form aktif ke `allData` |
| `simpanKeFirebase()` | Upload `{pelangganList, data}` ke Firebase |
| `autoSave()` | Trigger save dengan debounce 1 detik |
| `renderTabs()` | Render ulang semua tab di header |
| `gantiTab(id)` | Pindah tab (simpan lama, load baru) |
| `konfirmasiTambah()` | Tambah pelanggan baru dengan form kosong |
| `hapusPelanggan(id)` | Hapus pelanggan + data dari Firebase |
| `renderTabel(bodyId, data)` | Render baris tabel material/aksesoris |
| `updateItem(...)` | Update satu cell tabel |
| `hitungSemua()` | Hitung ulang semua subtotal & grand total |
| `kirimWA()` | Format pesan & buka wa.me link |
| `kirimSheets()` | POST data ke Google Apps Script |

---

## 📱 Responsive Breakpoints

| Breakpoint | Perilaku |
|---|---|
| `< 768px` (HP) | Tombol jadi grid 2 kolom, tabel scroll horizontal, jasa stack vertikal |
| `768–991px` (Tablet) | Padding dikurangi, layout semi-compact |
| `≥ 992px` (Desktop) | Layout penuh, max-width 900px |

---

## 🎬 Animasi GSAP Preloader

Urutan animasi saat halaman pertama dibuka:

```
1. 📦 Icon  → bounce + rotation masuk
2. UNIQ     → slide up dari bawah
3. CELL     → slide up dari bawah
4. Tagline  → fade in
5. Bar      → loading bar 0% → 100% (1.4 detik)
6. Dots     → muncul bergantian + pulse
7. Semua    → fade out ke atas
8. Preloader → slide ke atas keluar
9. #app     → fade in
```

---

## 📋 Struktur Form

Setiap pelanggan memiliki form dengan 4 section:

1. **Informasi Pelanggan** — Nama, No. PT, No. UPB, Tanggal
2. **Material Utama** — Tabel dengan kolom: No, Nama Barang, Qty, Harga, Subtotal
3. **Aksesoris** — Tabel sama seperti Material Utama
4. **Jasa Pemasangan** — Deskripsi, Qty, Harga, Subtotal

**Grand Total** = Total Material + Total Aksesoris + Total Jasa

---

## 🔑 Generate ID Pelanggan

ID pelanggan dibuat unik menggunakan:
```js
const uid = () => Date.now().toString(36) + Math.random().toString(36).slice(2,5);
// contoh hasil: "lp3k2xabc"
```

---

*Dibuat untuk kebutuhan internal UNIQ CELL — Order Management System*
