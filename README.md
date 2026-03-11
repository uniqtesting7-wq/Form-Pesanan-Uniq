# 📦 UNIQ CELL — Order Management System

> Aplikasi form pemesan digital untuk manajemen order pemasangan CCTV.
> Multi-user dengan sistem login, data terpisah per akun, sinkron real-time via Firebase.

---

## 🌐 Live Demo

🔗 **https://form-pesanan-uniq.vercel.app**

---

## 📁 Struktur File

```
/
├── login.html          # Halaman login & registrasi
├── form-pemesan.html   # Halaman utama form order (butuh login)
├── setup-admin.html    # Setup akun admin — hapus setelah dipakai!
├── vercel.json         # Konfigurasi routing Vercel
└── README.md           # Dokumentasi ini
```

> Tidak ada build tool, tidak ada install. Semua library dimuat via CDN.

---

## 🔧 Library & CDN

| Library | Versi | Kegunaan |
|---|---|---|
| Bootstrap | 5.3.3 | Layout & komponen UI |
| Bootstrap Icons | 1.11.3 | Ikon tombol |
| Firebase Compat SDK | 10.12.0 | Realtime Database |
| GSAP | 3.12.5 | Animasi preloader |

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

> ⚠️ Cocok untuk penggunaan internal. Untuk produksi publik, tambahkan autentikasi Firebase.

---

## 🗄️ Struktur Data Firebase

Semua data tersimpan di root `users/`:

```
users/
├── admin/
│   ├── nama:      "Administrator"
│   ├── username:  "admin"
│   ├── password:  "hash..."
│   ├── role:      "admin"
│   └── createdAt: "2026-01-01T00:00:00.000Z"
│
├── budi/
│   ├── nama:      "Budi Santoso"
│   ├── username:  "budi"
│   ├── password:  "hash..."
│   ├── role:      "user"
│   ├── createdAt: "2026-03-01T08:00:00.000Z"
│   └── appData/
│       ├── pelangganList[]
│       │   ├── { id: "abc123", nama: "Ko Danny Toko Biru" }
│       │   └── { id: "def456", nama: "Bu Sari Apotek" }
│       └── data/
│           ├── abc123/
│           │   ├── pelanggan: { nama, noOrder, noUpb, tanggal }
│           │   ├── material:  [{ nama, qty, harga }, ...]
│           │   ├── aksesoris: [{ nama, qty, harga }, ...]
│           │   └── jasa:      { desc, qty, harga }
│           └── def456/
│               └── ...
│
└── sari/
    ├── ...info akun...
    └── appData/        ← data sari TERPISAH TOTAL dari budi
        └── ...
```

### Contoh Data `appData` Lengkap

```json
{
  "pelangganList": [
    { "id": "abc123", "nama": "Ko Danny Toko Biru" }
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
        { "nama": "Camera Hikvision 16D0T", "qty": 14, "harga": 365000 },
        { "nama": "Hardisk 2TB Skyhawk",    "qty": 1,  "harga": 1965000 }
      ],
      "aksesoris": [
        { "nama": "DC Male",  "qty": 16, "harga": 3500 },
        { "nama": "BNC Drat", "qty": 32, "harga": 5000 }
      ],
      "jasa": {
        "desc": "Jasa Pemasangan + Setting",
        "qty": "16",
        "harga": "200.000"
      }
    }
  }
}
```

---

## 🔐 Sistem Autentikasi

### Password Hashing

Password tidak disimpan plain text, menggunakan hash sederhana:

```js
function hashPw(pw) {
  let h = 0;
  for (let i = 0; i < pw.length; i++)
    h = (Math.imul(31, h) + pw.charCodeAt(i)) | 0;
  return Math.abs(h).toString(36) + pw.length.toString(36);
}
```

### Sesi Login

Sesi disimpan di `sessionStorage` (hilang otomatis saat tab/browser ditutup):

```js
// Login berhasil → simpan sesi
sessionStorage.setItem("uniqUser", JSON.stringify({
  username: "budi",
  nama: "Budi Santoso",
  role: "user"  // atau "admin"
}));

// Logout → hapus sesi
sessionStorage.removeItem("uniqUser");
```

### Auth Guard di `form-pemesan.html`

```js
const sessRaw   = sessionStorage.getItem("uniqUser");
if (!sessRaw) { window.location.href = "login.html"; throw 0; }
const CURR_USER = JSON.parse(sessRaw);
const IS_ADMIN  = CURR_USER.role === "admin";

// Path data hanya milik user ini
const dbRef = db.ref(`users/${CURR_USER.username}/appData`);
```

---

## ⚙️ Alur Sistem

### Login
```
Buka login.html
  → Cek sessionStorage → jika sudah login, langsung redirect ke form
  → Input username + password → doLogin()
  → db.ref("users/{username}").get()
  → Bandingkan hashPw(password) dengan data di Firebase
  → Cocok → simpan ke sessionStorage → redirect ke form-pemesan.html
  → Tidak cocok → tampilkan pesan error
```

### Registrasi
```
Klik tab "Daftar"
  → Input nama lengkap, username, password → doRegister()
  → Validasi: username min 3 karakter, hanya [a-z0-9_], tidak boleh "admin"
  → Cek ketersediaan username di Firebase
  → Jika tersedia → db.ref("users/{username}").set({ nama, hashPw, role:"user" })
  → Redirect ke tab Login
```

### Buka Form
```
form-pemesan.html dibuka
  → Auth guard → jika tidak ada sesi, redirect ke login.html
  → GSAP preloader animasi
  → Firebase listener aktif di path users/{username}/appData
  → Jika ada data → load ke form
  → Jika kosong → buat 1 tab pelanggan default kosong
  → Preloader keluar → form fade in
```

### Auto-Save
```
User mengetik / mengubah form
  → autoSave() dipanggil
  → simpanStateAktif() → allData[activeId] = ambilState()
  → renderTabs() — nama tab update dari field Nama Pelanggan
  → debounce 1 detik
  → simpanKeFirebase() → dbRef.set({ pelangganList, data: allData })
  → Status sync diperbarui di header
```

### Ganti Password
```
Klik tombol nama akun di header
  → Modal ganti password terbuka
  → Input password lama → diverifikasi ke Firebase lebih dulu
  → Input password baru + konfirmasi
  → Semua valid → db.ref("users/{username}/password").set(hashPw(baru))
  → Modal tutup otomatis
```

---

## 🛡️ Admin Panel

Hanya muncul jika `role === "admin"`. Tombol kuning 🟡 di header.

### Fitur

- Lihat semua user terdaftar beserta tanggal daftar
- Lihat semua pelanggan tiap user + breakdown total per pelanggan (Material / Aksesoris / Jasa)
- **Hapus User** — menghapus akun + seluruh data pelanggannya sekaligus
- **Hapus Pelanggan** — menghapus 1 pelanggan dari akun user, akun tetap ada

### Alur

```
Klik tombol Admin → loadAdminPanel()
  → db.ref("users").get() → loop semua user (skip "admin")
  → Untuk tiap user: db.ref("users/{u}/appData").get()
  → Hitung grand total semua pelanggan
  → Render kartu user + list pelanggan + tombol hapus

Hapus User
  → db.ref("users/{uname}").remove()
  → Refresh panel

Hapus Pelanggan
  → Ambil appData user → filter pelangganList → hapus dari data{}
  → db.ref("users/{uname}/appData").set(appData yang sudah difilter)
  → Row hilang dengan animasi fade (tanpa reload panel)
```

---

## 🏗️ Referensi Fungsi JavaScript

### `login.html`

| Fungsi | Keterangan |
|---|---|
| `doLogin()` | Verifikasi username + password ke Firebase, simpan sesi |
| `doRegister()` | Validasi input + buat akun baru di Firebase |
| `switchTab(tab)` | Toggle antara form Login dan Daftar |
| `hashPw(pw)` | Hash password sebelum disimpan atau dibandingkan |
| `togglePw(id, btn)` | Show / hide password input |
| `setLoading(btnId, bool, text)` | Toggle loading state tombol |
| `showAlert(msg, type)` | Tampilkan pesan error atau sukses |

### `form-pemesan.html`

| Fungsi | Keterangan |
|---|---|
| `doLogout()` | Hapus sesi, redirect ke login.html |
| `gantiPassword()` | Verifikasi pw lama → update pw baru di Firebase |
| `formKosong()` | Buat object data kosong untuk pelanggan baru |
| `ambilState()` | Baca semua nilai dari DOM → object |
| `isiForm(s)` | Isi DOM dari object data |
| `simpanStateAktif()` | Simpan form aktif ke `allData` |
| `simpanKeFirebase()` | Upload `{pelangganList, data}` ke Firebase |
| `autoSave()` | Trigger save dengan debounce 1 detik |
| `renderTabs()` | Render ulang semua tab di header |
| `gantiTab(id)` | Pindah tab — simpan yang lama, load yang baru |
| `konfirmasiTambah()` | Tambah pelanggan baru dengan form kosong |
| `hapusPelanggan(id)` | Hapus pelanggan dari data lokal + Firebase |
| `renderTabel(bodyId, data)` | Render baris tabel material / aksesoris |
| `updateItem(bodyId, idx, field, val)` | Update satu cell tabel + autoSave |
| `hitungSemua()` | Hitung ulang semua subtotal & grand total |
| `kirimWA()` | Format ringkasan order → buka wa.me link |
| `kirimSheets()` | POST data ke Google Apps Script |
| `loadAdminPanel()` | Muat semua data user ke modal admin |
| `adminHapusUser(uname, nama)` | Hapus user + seluruh datanya dari Firebase |
| `adminHapusPelanggan(uname, pelId, nama)` | Hapus 1 pelanggan dari akun user |

### State Variables

| Variable | Tipe | Keterangan |
|---|---|---|
| `CURR_USER` | Object | Data user login dari sessionStorage |
| `IS_ADMIN` | Boolean | `true` jika `role === "admin"` |
| `dbRef` | Firebase Ref | Path `users/{username}/appData` |
| `pelangganList` | Array | `[{ id, nama }]` — urutan tab |
| `allData` | Object | `{ [id]: { pelanggan, material, aksesoris, jasa } }` |
| `activeId` | String | ID pelanggan tab yang sedang aktif |
| `dataMaterial` | Array | Baris material tab aktif |
| `dataAksesoris` | Array | Baris aksesoris tab aktif |
| `saveTimer` | Timeout | Debounce timer auto-save |
| `isFromFirebase` | Boolean | Flag mencegah save loop saat terima update Firebase |

---

## 🎬 Animasi GSAP Preloader

Tampil setiap kali `form-pemesan.html` dibuka:

```
1. 📦 Icon   → bounce + rotation masuk
2. UNIQ      → slide up dari bawah
3. CELL      → slide up dari bawah
4. Tagline   → fade in
5. Bar       → loading bar 0% → 100%
6. 3 Dots    → muncul bergantian lalu pulse
7. Semua     → fade out ke atas
8. Preloader → slide ke atas keluar
9. #app      → fade in
```

---

## 📊 Google Sheets Integration

1. Buka Google Sheets → **Extensions → Apps Script**
2. Paste script berikut, deploy sebagai **Web App** (Execute as: Me, Access: Anyone):

```js
function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sh = ss.getSheetByName("Orders") || ss.insertSheet("Orders");
  const d  = JSON.parse(e.postData.contents);
  if (sh.getLastRow() === 0)
    sh.appendRow(["Waktu","Nama","No Order","No UPB","Tanggal","Barang","Qty","Harga","Subtotal","Kategori"]);
  const rows = [];
  d.material.forEach(i  => rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Material"]));
  d.aksesoris.forEach(i => rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,i.nama,i.qty,i.harga,i.qty*i.harga,"Aksesoris"]));
  rows.push([new Date(),d.nama,d.noOrder,d.noUpb,d.tanggal,d.jasaDesc,d.jasaQty,d.jasaHarga,d.jasaQty*d.jasaHarga,"Jasa"]);
  rows.forEach(r => sh.appendRow(r));
  return ContentService.createTextOutput(JSON.stringify({status:"ok"})).setMimeType(ContentService.MimeType.JSON);
}
```

3. Copy URL deployment → paste di form (tombol 🟢 Sheets)

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
3. Username admin: `admin` (fixed, tidak bisa dipakai untuk registrasi biasa)
4. **Hapus `setup-admin.html` dari GitHub setelah selesai!**

---

## 📱 Responsive Design

| Breakpoint | Perilaku |
|---|---|
| `< 768px` | Tombol header grid 2 kolom, tabel scroll horizontal, jasa stack vertikal |
| `768–991px` | Padding dikurangi, layout semi-compact |
| `≥ 992px` | Layout penuh, max-width 900px |

---

*UNIQ CELL Order Management System — Sistem Internal*
