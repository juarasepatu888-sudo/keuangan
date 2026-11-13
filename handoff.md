# Handoff Project: Kas Juara Sepatu 👟

Dokumen acuan status dan riwayat pengembangan aplikasi **Kas Juara Sepatu (Laporan Juara)**.

---

## 📌 Ringkasan Proyek
- **Aplikasi:** Buku Kas Digital & Manajemen Stok Toko Sepatu (PWA, Mobile-First, Offline-First, Realtime Sync)
- **Repositori:** [juarasepatu888-sudo/keuangan](https://github.com/juarasepatu888-sudo/keuangan) (Branch: `main`)
- **Supabase Project:** `carolcjpbogxwdvqruzg` (Tabel: `app_state`, Key: `kas_juara_main`)
- **Google Spreadsheet Aktif:** [`17Z74OlTkIuWo54yffiu3nYPdv5jCm-v4Nl_M9eWixJ0`](https://docs.google.com/spreadsheets/d/17Z74OlTkIuWo54yffiu3nYPdv5jCm-v4Nl_M9eWixJ0/edit) (Judul: `kasjuarasepatu`)
- **Service Account Google:** `sheet-syc-keuangan@keuangan-juara-sepatu.iam.gserviceaccount.com` (File kunci: `src/config/service-account.json`)

---

## 🛠️ Tech Stack
- **Frontend:** Vanilla JavaScript (ES Modules), HTML5, Tailwind CSS
- **Build Tool:** Vite 6.x
- **Database Cloud:** Supabase (PostgreSQL + Realtime Channels)
- **Penyimpanan Lokal:** `localStorage` (untuk caching offline & base64 foto barang)
- **PWA:** `vite-plugin-pwa` (Workbox Service Worker)
- **Barcode Scanner:** `html5-qrcode`

---

## 🕒 Log Pembaruan Terkini (12 - 14 November 2026)

### 1. Peningkatan Batas Input Jumlah Pasang Belanja & Transaksi (14 Nov 2026 - Commit `dcfb35d`)
- **Masalah:** Pada form halaman **Belanja** (`tambah-barang-masuk.js`), input *Jumlah Pasang* dibatasi maksimal `999` pasang (`max="999"` dan `Math.min(999, ...)`). Toko membutuhkan pencatatan belanja grosir/partai hingga ribuan pasang (contoh: 1.100 pasang). Selain itu, lebar kotak angka sebelumnya sempit (`w-10` = 40px) sehingga angka ribuan terpotong.
- **Perbaikan:**
  - Atribut input diubah ke `min="1" max="999999"`.
  - Logika event listener tombol `+` dan input manual diubah dari `Math.min(999, ...)` menjadi `Math.min(999999, ...)`.
  - Tata letak diubah menjadi `flex-1 min-w-0` agar kotak input melebar otomatis dan angka ribuan hingga ratusan ribu tampil rapi.
  - Perubahan serupa diterapkan pada form **Transaksi Kas Masuk** (`tambah-transaksi.js`) agar mendukung transaksi penjualan partai/grosir dalam jumlah besar.

---

### 2. Penjelasan & Logika Laporan HPP Sepatu (Modal + Ongkir) (14 Nov 2026)
- **Konteks:** Pada halaman Laporan Keuangan, card *HPP RATA-RATA / PSG* menampilkan angka **Rp 319.671 / psg**.
- **Rumus & Sumber Perhitungan:**
  $$\text{HPP Rata-rata} = \frac{\text{Total Biaya Belanja Sepatu} + \text{Total Ongkir Belanja}}{\text{Total Pasang Sepatu}}$$
- **Detail Komponen:**
  1. **Total Belanja Sepatu (Rp 295.661.000):**
     - Faktur Stok `SUPLIERSEPATU-898` (1.100 psg × Rp 160.000): **Rp 176.000.000**
     - Kas Keluar Nota `QTY1100TH01` (*SEPATU BOOTS*): **Rp 119.661.000**
     - Dijumlahkan karena nomor nota/faktur berbeda sehingga sistem mendeteksi "2 Faktur Belanja".
  2. **Total Ongkir Belanja (Rp 55.977.000):**
     - Tiket Mustafa 1: Rp 4.328.000
     - Operasional 1 Tiket: Rp 5.000.000
     - Ongkir Lory: Rp 42.250.000
     - Packing Rolly TH: Rp 4.399.000
  3. **Total Biaya Modal Riil:**
     $$\text{Rp } 295.661.000 + \text{Rp } 55.977.000 = \text{Rp } 351.638.000$$
  4. **Eksekusi:**
     $$\frac{\text{Rp } 351.638.000}{1.100 \text{ pasang}} = \text{Rp } 319.670,909... \rightarrow \mathbf{Rp\ 319.671\ \text{/psg}}$$
- **Rekomendasi Operasional:** Nomor nota Kas Keluar (`QTY1100TH01`) dan nomor faktur Belanja Stok (`SUPLIERSEPATU-898`) disamakan agar sistem otomatis mengaitkannya sebagai satu kesatuan faktur dan tidak menghitung dobel nominal belanja.

---

### 3. Perbaikan Barcode Dropdown pada Fitur "+ Barang" Buku Transaksi (12 Nov 2026 - Commit `4f55864`)
- **Masalah:** Pada halaman Buku Transaksi (`transaksi-list.js`), saat menekan tombol `+ Barang` untuk menambah item pada transaksi pembeli yang sudah ada, dropdown *"Pilih Sepatu dari Stok / Barcode"* tidak memunculkan barcode yang sudah dibeli di transaksi tersebut.
- **Penyebab:** Fungsi `populateModalStockDropdown` sempat menghitung `usedInCurrentTrx` dan menguranginya lagi dari `sisaStock`. Akibatnya, barang yang seluruh stoknya sedang ada di transaksi tersebut terhitung `sisaTersedia = 0` sehingga disembunyikan.
- **Perbaikan:** Menghapus pengurangan ganda `usedInCurrentTrx`. Fungsi kini langsung menggunakan hasil `store.getAvailableProducts(addItemTargetTrxId)` yang sudah mengecualikan transaksi target secara benar. Barcode dari transaksi tersebut tetap muncul dan dapat ditambahkan kembali selama stok gudang mencukupi.

---

### 4. Perbaikan Default ID Google Spreadsheet & Sinkronisasi (12 Nov 2026 - Commit `4f55864`)
- **Masalah:** Indikasi data Google Spreadsheet kosong karena fallback default di `store.js` baris 2669 & 2689 sempat mengarah ke spreadsheet lama yang berstatus permission error (`1iTev6j8z2GBzEGSz1Ui9fEMtYjEs30yQnOM8LfhI53Y`).
- **Perbaikan:**
  - Fallback default ID spreadsheet di `src/store/store.js` diperbarui ke spreadsheet aktif toko: `17Z74OlTkIuWo54yffiu3nYPdv5jCm-v4Nl_M9eWixJ0`.
  - Verifikasi ke Google Sheets API memastikan data transaksi dan dashboard lengkap tersimpan di spreadsheet `kasjuarasepatu`.
  - File kunci Service Account (`src/config/service-account.json`) dipastikan aktif untuk email `sheet-syc-keuangan@keuangan-juara-sepatu.iam.gserviceaccount.com`.

---

### 5. Fleksibilitas Barcode Sama pada Penjualan Kas Masuk (12 Nov 2026 - Commit `2eabac7` & `5bc33b6`)
- **Kebutuhan:** Dalam 1 transaksi, pembeli dapat membeli lebih dari 1 pasang sepatu dengan barcode yang sama (misal 1 barcode mewakili 10 pasang stok sepatu).
- **Perbaikan:**
  - Mengizinkan barcode yang sama diinput berulang atau diisi kuantitas > 1 dalam satu transaksi pembeli.
  - Validasi stok real-time menghitung akumulasi total pasang yang dipilih untuk barcode tersebut dan memastikan tidak melebihi sisa stok fisik di toko.

---

## 📂 Struktur Modul Kunci
| Berkas | Fungsi Utama |
|---|---|
| `src/pages/tambah-barang-masuk.js` | Form Belanja / Pasok Stok Sepatu (Qty ribuan, status Pre-Paid, scan barcode kamera). |
| `src/pages/tambah-transaksi.js` | Form Transaksi Kas Masuk & Kas Keluar (Multi-item penjualan, Qty ribuan, kategori biaya). |
| `src/pages/transaksi-list.js` | Buku Kas Harian (Filter, Verifikasi Mutasi, Fitur Cepat `+ Barang`, Pencarian Barcode). |
| `src/pages/barang-masuk-list.js` | Daftar Stok & Belanja (Tarik Google Sheets, Stock Opname, Gabung Faktur). |
| `src/pages/laporan.js` | Laporan Keuangan, Omset, Sisa Stok Realtime, dan Laporan HPP (Modal + Ongkir). |
| `src/store/store.js` | State management tunggal, kalkulasi HPP (`getHppReports`), laporan stok real-time, sinkronisasi Supabase & Google Sheets. |
| `src/services/googleSheets.js` | Service sinkronisasi 2 arah dengan Google Sheets API via Google Service Account JWT. |
| `src/config/service-account.json` | Kunci otentikasi Google Cloud Service Account. |

---

## 🚀 Prosedur Build & Verifikasi
Sebelum melakukan deployment, selalu jalankan langkah berikut:
```bash
# 1. Jalankan build
npm run build

# 2. Hapus folder dist setelah build terverifikasi (agar repositori bersih)
Remove-Item -Recurse -Force dist

# 3. Commit dan push ke branch main
git add .
git commit -m "pesan perubahan"
git push origin main
```

---
*Dokumen diperbarui terakhir: 14 November 2026*
