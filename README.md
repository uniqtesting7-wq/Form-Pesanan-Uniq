# 🔧 UNIQ CCTV AND NETWORK — Order Management System

> Aplikasi manajemen order & servis digital untuk UNIQ CCTV AND NETWORK.
> Multi-user dengan sistem login, data terpisah per akun, sinkron real-time via Firebase Realtime Database.
> Tidak ada build tool — semua library via CDN, langsung deploy ke Vercel.

---

## 🌐 Live Demo

🔗 **https://form-pesanan-uniq.vercel.app**

---

## 📁 Struktur File

```
/
├── login.html           # Halaman login & registrasi akun
├── form-pemesan.html    # Form Pemasangan — order material + jasa pasang CCTV
├── form-order.html      # Form Pemesanan — order barang murni (tanpa jasa pasang)
├── form-service.html    # Form Service / Garansi — tiket servis & retur ke supplier
├── po.html              # Purchase Order — manajemen & tracking PO ke supplier
├── admin.html           # Admin Panel — kelola semua user & data (role admin only)
├── setup-admin.html     # Setup akun admin — hapus setelah dipakai!
├── vercel.json          # Konfigurasi routing Vercel
└── README.md            # Dokumentasi ini
```

> Tidak ada `node_modules`, tidak ada `npm install`. Buka langsung di browser.

---

## 🧭 Navigasi Antar Halaman

Semua halaman (kecuali `login.html` dan `setup-admin.html`) memiliki sidebar navigasi yang sama:

| Menu | File | Keterangan |
|---|---|---|
| 🔧 Form Pemasangan | `form-pemesan.html` | Order material + aksesoris + jasa pasang |
| 🛒 Form Pemesanan | `form-order.html` | Order barang saja, tanpa jasa |
| 🛠️ Form Service | `form-service.html` | Tiket servis & garansi / retur supplier |
| 📋 Purchase Order | `po.html` | Buat & tracking PO ke supplier |
| 🛡️ Admin Panel | `admin.html` | Kelola user & lihat semua data (admin only) |

---

## 🔥 Konfigurasi Firebase

### Project Info

| Key | Value |
|---|---|
| Project ID | `form-pesanan-uniq` |
| Database URL | `https://form-pesanan-uniq-default-rtdb.firebaseio.com` |
| Auth Domain | `form-pesanan-uniq.firebaseapp.com` |

### Firebase Rules

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

> ⚠️ Rules ini cocok untuk penggunaan internal tertutup. Untuk deployment publik, tambahkan autentikasi Firebase Auth.

---

## 🗄️ Struktur Data Firebase

```
(root)
├── users/
│   ├── admin/
│   │   ├── nama:      "Administrator"
│   │   ├── username:  "admin"
│   │   ├── password:  "hash..."
│   │   ├── role:      "admin"
│   │   └── createdAt: "2026-01-01T00:00:00.000Z"
│   │
│   └── budi/
│       ├── nama:      "Budi Santoso"
│       ├── username:  "budi"
│       ├── password:  "hash..."
│       ├── role:      "user"
│       ├── createdAt: "2026-03-01T08:00:00.000Z"
│       │
│       ├── appData/            ← Data Form Pemasangan (per user, terpisah)
│       │   ├── pelangganList[] → [{ id, nama }]
│       │   └── data/
│       │       └── {pelangganId}/
│       │           ├── pelanggan: { nama, noOrderList[], noUpb, tanggal }
│       │           ├── material:  [{ nama, qty, harga }]
│       │           ├── aksesoris: [{ nama, qty, harga }]
│       │           └── jasa:      { desc, qty, harga }
│       │
│       └── orderData/          ← Data Form Pemesanan (per user, terpisah)
│           ├── pelangganList[] → [{ id, nama }]
│           └── data/
│               └── {pelangganId}/
│                   ├── pelanggan: { nama, noOrderList[], noUpb, tanggal }
│                   └── barang:    [{ nama, qty, harga }]
│
└── serviceTickets/             ← Data Form Service (SHARED — semua user lihat)
    └── {ticketId}/
        ├── id:         "ABC123XYZ"
        ├── ticketNo:   "SVC-260001"
        ├── nama:       "Ko Danny"
        ├── hp:         "08123456789"
        ├── barang:     "DVR Hikvision 8 Channel"
        ├── merk:       "DS-7208HQHI"
        ├── keluhan:    "Gambar blur, tidak merekam"
        ├── supplier:   "PT. Hikvision Indonesia"
        ├── estimasi:   "2026-03-20"
        ├── catatan:    "Cek power supply dulu"
        ├── status:     "masuk" | "proses" | "selesai" | "diambil"
        ├── createdAt:  "2026-03-12T08:00:00.000Z"
        ├── updatedAt:  "2026-03-13T10:30:00.000Z"
        └── createdBy:  "Budi Santoso"
```

### Perbedaan Scope Data

| Path | Scope | Penjelasan |
|---|---|---|
| `users/{u}/appData` | Per user | Setiap user hanya lihat data pemasangan miliknya sendiri |
| `users/{u}/orderData` | Per user | Setiap user hanya lihat data pemesanan miliknya sendiri |
| `serviceTickets` | Shared / global | Semua user bisa lihat & update tiket servis bersama |

---

## 📄 Halaman & Fitur Detail

### 🔧 Form Pemasangan (`form-pemesan.html`)

Form order untuk proyek pemasangan CCTV, mencakup material, aksesoris, dan jasa pasang.

**Fitur:**
- Tab multi-pelanggan (bisa buka beberapa pelanggan sekaligus)
- 3 tabel terpisah: Material Utama, Material Aksesoris, Jasa Pemasangan
- Grand total otomatis (material + aksesoris + jasa)
- Auto-save ke Firebase (debounce 2 detik)
- Export ringkasan ke WhatsApp
- Export ke Google Sheets via Apps Script
- Cetak / print

---

### 🛒 Form Pemesanan (`form-order.html`)

Form order barang murni tanpa jasa pemasangan — untuk pembelian stok atau pesanan pelanggan biasa.

**Fitur:**
- Tab multi-pelanggan
- 1 tabel Daftar Barang (nama, qty, harga satuan, subtotal)
- Info No. Order (bisa lebih dari satu) + No. UPB
- Grand total + ringkasan jumlah item
- Auto-save ke Firebase (debounce 2 detik)
- Export ke WhatsApp & Google Sheets
- Cetak / print

---

### 🛠️ Form Service / Garansi (`form-service.html`)

Form pencatatan tiket servis dan garansi barang — untuk pelanggan yang ingin meretur atau menitipkan barang untuk diservis/dikirim ke supplier.

**Fitur:**
- Buat tiket baru dengan nomor otomatis (`SVC-YYNNNNN`)
- Data tiket: nama & HP pelanggan, nama barang, merk/model, keluhan/deskripsi kerusakan, supplier tujuan retur, estimasi selesai, catatan internal
- **Tracking status 4 tahap** dengan stepper visual:
  - ⏳ **Masuk** — barang baru diterima dari pelanggan
  - 🔄 **Diproses** — barang sedang diperiksa atau dikirim ke supplier
  - ✅ **Selesai** — barang sudah selesai diservis/retur, siap diambil
  - 📦 **Diambil** — barang sudah diambil pelanggan
- Filter tiket per status + pencarian real-time
- Statistik ringkasan (total, per-status) di header
- **Notifikasi WA otomatis** ke pelanggan sesuai status terkini
- **Cetak struk penerimaan** (popup print-ready)
- Edit & hapus tiket
- Data tiket bersifat **shared** — semua user/staf bisa melihat dan mengupdate

---

### 📋 Purchase Order (`po.html`)

Manajemen Purchase Order ke supplier.

---

### 🛡️ Admin Panel (`admin.html`)

Hanya bisa diakses oleh user dengan `role: "admin"`.

**Fitur:**
- Lihat semua user terdaftar + tanggal daftar
- Lihat semua pelanggan per user beserta total nilai order
- Hapus user (akun + seluruh datanya)
- Hapus pelanggan individual dari akun user

---

## 🔐 Sistem Autentikasi

### Password Hashing

Password tidak disimpan plain text, menggunakan hash sederhana berbasis djb2:

```js
function hashPw(pw) {
  let h = 0;
  for (let i = 0; i < pw.length; i++)
    h = (Math.imul(31, h) + pw.charCodeAt(i)) | 0;
  return Math.abs(h).toString(36) + pw.length.toString(36);
}
```

### Sesi Login

Sesi disimpan di `sessionStorage` (hilang otomatis saat tab ditutup):

```js
sessionStorage.setItem("uniqUser", JSON.stringify({
  username: "budi",
  nama:     "Budi Santoso",
  role:     "user"  // atau "admin"
}));
```

### Auth Guard

Setiap halaman (kecuali login) memiliki guard di baris pertama script:

```js
const sessRaw = sessionStorage.getItem("uniqUser");
if (!sessRaw) { window.location.href = "login.html"; throw 0; }
const CURR_USER = JSON.parse(sessRaw);
```

---

## 🔧 Library & CDN

| Library | Versi | Kegunaan |
|---|---|---|
| Bootstrap | 5.3.3 | Layout & komponen UI |
| Bootstrap Icons | 1.11.3 | Ikon tombol & navigasi |
| Firebase Compat SDK | 10.12.0 | Realtime Database (sync, read, write) |
| Plus Jakarta Sans | — | Font utama (Google Fonts) |
| DM Mono | — | Font angka & kode (Google Fonts) |

---

## 📊 Google Sheets Integration

### Form Pemasangan

```js
function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sh = ss.getSheetByName("Orders") || ss.insertSheet("Orders");
  const d  = JSON.parse(e.postData.contents);
  if (sh.getLastRow() === 0)
    sh.appendRow(["Waktu","Nama","No Order","No UPB","Tanggal","Barang","Qty","Harga","Subtotal","Kategori"]);
  d.material.forEach(i  => sh.appendRow([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Material"]));
  d.aksesoris.forEach(i => sh.appendRow([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Aksesoris"]));
  sh.appendRow([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,d.jasaDesc,d.jasaQty,d.jasaHarga,d.jasaQty*d.jasaHarga,"Jasa"]);
  return ContentService.createTextOutput(JSON.stringify({status:"ok"})).setMimeType(ContentService.MimeType.JSON);
}
```

### Form Pemesanan

```js
function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sh = ss.getSheetByName("Pemesanan") || ss.insertSheet("Pemesanan");
  const d  = JSON.parse(e.postData.contents);
  if (sh.getLastRow() === 0)
    sh.appendRow(["Waktu","Nama","No Order","No UPB","Tanggal","Nama Barang","Qty","Harga","Subtotal"]);
  d.barang.forEach(i => {
    sh.appendRow([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga]);
  });
  return ContentService.createTextOutput(JSON.stringify({status:"ok"})).setMimeType(ContentService.MimeType.JSON);
}
```

**Cara deploy:**
1. Buka Google Sheets → **Extensions → Apps Script**
2. Paste script → **Deploy → Web app**
3. Execute as: `Me` | Access: `Anyone`
4. Copy URL → paste di modal Sheets dalam form

---

## 🚀 Deploy ke Vercel

### `vercel.json`

```json
{
  "rewrites": [
    { "source": "/", "destination": "/login.html" }
  ]
}
```

### Langkah Deploy

```bash
git add .
git commit -m "update"
git push
```

Vercel otomatis redeploy setiap `git push` ke branch utama.

---

## 🔑 Setup Akun Admin (Sekali Saja)

1. Buka `https://form-pesanan-uniq.vercel.app/setup-admin.html`
2. Isi password untuk akun admin → klik **Buat Akun Admin**
3. Username admin: `admin` (tidak bisa dipakai untuk registrasi biasa)
4. **Hapus `setup-admin.html` dari repository setelah selesai!**

---

## 📱 Responsive Design

| Breakpoint | Perilaku |
|---|---|
| `< 992px` | Sidebar tersembunyi, muncul via hamburger menu di topbar |
| `< 480px` | Beberapa kolom tabel disembunyikan, label tombol disembunyikan |
| `≥ 992px` | Layout penuh dengan sidebar fixed di kiri |

---

*UNIQ CCTV AND NETWORK — Order Management System · Sistem Internal*
