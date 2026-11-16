# Paket Aturan Kasire

`rules.json` memperbarui variasi perintah untuk Pemrosesan Lokal Kasire. Ini JSON kecil, **bukan APK atau model AI**. Aturan bawaan aplikasi tetap tersedia saat offline atau unduhan gagal.

Aplikasi menampilkan sumber/versi aturan aktif, waktu pemeriksaan dan waktu pembaruan di Pengaturan → Cek Pembaruan Aturan. Paket baru menunggu transaksi/draft aktif selesai sebelum diaktifkan. Pemeriksaan otomatis paling sering enam jam sekali saat aplikasi dibuka/kembali aktif; tombol manual dapat memeriksa langsung.

## Mengajarkan variasi melalui ChatGPT

Jelaskan kalimat pengguna dan maksudnya. Susun perubahan `rules.json`, periksa benturan dengan aturan bawaan, uji, lalu naikkan `version` dan perbarui `publishedAt` sebelum commit/push. Jangan mengubah file APK atau release untuk pembaruan aturan.

Schema v1 mendukung alias kalimat utuh (`phrases`) yang tidak membedakan huruf besar/kecil, spasi berulang, atau tanda akhir . ! ?. Ini bukan skrip, SQL, atau bahasa pemrograman. Parameter dinamis barang/jumlah tetap ditangani parser bawaan.

Aksi yang didukung: CREATE_PRODUCT, CHECK_PRICE, OPEN_SCANNER, CLOSE_SCANNER, SHOW_HISTORY, SHOW_CATALOG, NEW_SALE, CLEAR_CHAT, REPORT_SALES, PAYMENT, NON_CASH, REPLY. SHOW_HISTORY/REPORT_SALES boleh memiliki `period`: TODAY, YESTERDAY, WEEK, LAST_WEEK, MONTH, LAST_MONTH. CHECK_PRICE boleh memiliki `query` nama barang. REPLY memerlukan field `reply` (1–600 karakter) untuk petunjuk umum; informasi harga/penjualan tetap harus dibaca dari database aplikasi. Aksi baru di luar daftar memerlukan dukungan aplikasi baru. Paket tidak boleh menyimpan harga katalog, menyelesaikan pembayaran, atau menulis database secara langsung.

`schemaVersion` saat ini 1. Aturan bawaan v1; paket pertama v2 menambah variasi dagangan kemarin, nyalain kamera, dan cari harga Yupi. Ukuran maksimum paket 256 KiB, versi harus meningkat; ID dan kalimat harus unik. Paket yang rusak, tidak kompatibel atau berbenturan ditolak dan aturan terakhir tetap dipakai.

Setiap versi adalah paket lengkap: pertahankan variasi lama yang masih ingin digunakan ketika menambahkan aturan baru.

PAYMENT menyiapkan permintaan uang diterima, NON_CASH menyiapkan pembayaran non-tunai; keduanya tidak menyelesaikan transaksi. REPORT_SALES boleh memakai `metric`: SUMMARY, FREQUENCY, REVENUE, PRODUCT. Metric PRODUCT memerlukan `query` nama produk. Aplikasi tetap menghitung hasil dari database lokal. Maksimum 1000 variasi kalimat dalam satu paket.
