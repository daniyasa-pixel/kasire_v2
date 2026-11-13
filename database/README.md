# Database Referensi Produk Kasire

Repository publik: `daniyasa-pixel/kasire_v2`, folder `database/`.

## Sumber dan distribusi

- **Excel master**: `produk-master.xlsx` (diunggah/diperbarui admin); worksheet pertama **Produk**.
- **products.json**: hasil konversi dan validasi Excel yang dibaca aplikasi; bukan file yang diedit tangan.
- **version.json**: metadata versi untuk menghindari unduhan berulang.
- **template-produk.csv**: contoh struktur header; CSV hanya contoh, bukan sumber otomatis.

Aplikasi Kasire **read-only** terhadap GitHub. Produk toko, harga lokal, stok, transaksi, dan perubahan pengguna tetap disimpan lokal; aplikasi tidak mem-push data ke repo. Pengunduhan opsional dan kegagalan jaringan tidak boleh mengganggu transaksi atau memunculkan popup.

## Format Excel: worksheet Produk

Baris pertama tepat:
```csv
barcode,nama_produk,variasi,merek,kategori,satuan,harga_beli,harga_jual
```

| Kolom | Tipe | Aturan |
| --- | --- | --- |
| barcode | TEXT | Wajib, unik setelah trim, tidak diubah menjadi angka; digit awal 0 dipertahankan; untuk produk scan publik gunakan kode 1D yang valid |
| nama_produk | TEXT | Wajib; trim spasi |
| variasi | TEXT | Wajib, misalnya 85g / Rasa Ayam |
| merek | TEXT | Boleh kosong |
| kategori | TEXT | Boleh kosong; hanya saran |
| satuan | TEXT | Boleh kosong; hanya saran |
| harga_beli | INTEGER | Rupiah, >=0 atau kosong; hanya referensi |
| harga_jual | INTEGER | Rupiah, >=0 atau kosong; hanya referensi |

Barcode harus disimpan sebagai sel **Text** di Excel. Jangan gunakan formula, nilai error Excel, barcode duplikat, spasi pada barcode, atau format harga seperti "Rp 3.500". Jangan isi data pribadi/pelanggan. Template boleh memakai barcode contoh yang *bukan produk nyata*; jangan publikasikan contoh sebagai database aktif.

## Konversi publikasi

1. Admin edit dan unggah Excel.
2. Script/CI memvalidasi setiap baris. Jika ada duplikasi barcode, barcode tak valid, nama/variasi kosong, harga negatif atau bukan bilangan bulat: **gagalkan publikasi**; JSON aktif sebelumnya tetap aman.
3. Setelah valid, buat `products.json` secara deterministik dan `version.json` untuk versi yang sama; unggah sebagai satu commit atomik ke main.
4. Aplikasi memeriksa versi via HTTPS, mengunduh hanya saat berbeda, memvalidasi keseluruhan file (termasuk checksum SHA-256 dari metadata), lalu mengimpor secara transaksi ke tabel referensi lokal; bila gagal, gunakan data versi terakhir.

Contoh bentuk `products.json` (contoh bukan database aktif):
```json
{
  "schema_version": 1,
  "data_version": "2026-11-13.1",
  "products": [
    {
      "barcode": "8991234567890",
      "nama_produk": "Produk Contoh",
      "variasi": "85g",
      "merek": "Merek Contoh",
      "kategori": "Makanan",
      "satuan": "pcs",
      "harga_beli": 2800,
      "harga_jual": 3500
    }
  ]
}
```

Metadata `version.json`:
```json
{
  "schema_version": 1,
  "data_version": "2026-11-13.1",
  "products_file": "products.json",
  "sha256": "<SHA-256 hex dari byte products.json>",
  "product_count": 1
}
```

## Kebijakan merge aplikasi

- Simpan **referensi pusat** terpisah dari **produk toko**; jangan replace tabel transaksi ataupun produk toko.
- Cocokkan berdasarkan **barcode**. Tambahkan referensi barcode baru; perbarui metadata referensi barcode lama; jangan overwrite nama, kategori, satuan, harga beli/jual yang telah diubah pengguna di produk lokal.
- Produk lokal tanpa barcode pusat tetap aman; produk toko yang dibuat pengguna tidak pernah terhapus akibat update pusat.
- Bila barcode pusat tersedia dan produk belum dibuat di toko, tampilkan data sebagai saran/prefill saat scan, untuk dikonfirmasi; harga lokal diputuskan pemilik toko.
- Penghapusan baris dari Excel bukan perintah menghapus produk lokal.
- Pertahankan operasi offline penuh. Periksa pembaruan secara tidak mengganggu, bukan setiap scan.
- Hindari token GitHub berizin tulis dalam APK.

## Raw URLs sesudah file diterbitkan

- `https://raw.githubusercontent.com/daniyasa-pixel/kasire_v2/main/database/version.json`
- `https://raw.githubusercontent.com/daniyasa-pixel/kasire_v2/main/database/products.json`

**Status**: panduan/schema awal. File distribusi `products.json` dan `version.json` baru diterbitkan setelah ada master data valid dan alur konversi. Jangan arahkan aplikasi mengunduh `template-produk.csv`.
