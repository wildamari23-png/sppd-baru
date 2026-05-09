Dokumentasi Teknis Aplikasi e-SPPD Puskesmas
Dokumen ini berisi panduan arsitektur, struktur kode, dan sistem kerja aplikasi e-SPPD. Gunakan dokumen ini sebagai "Konteks" saat Anda menggunakan AI lain (seperti ChatGPT, Claude, atau Gemini) untuk memperbarui atau memodifikasi aplikasi.
1. Arsitektur Sistem (Tech Stack)
Aplikasi ini dibangun menggunakan arsitektur Serverless Single-Page Application (SPA) yang di-host langsung di Google Apps Script (GAS).
•	Frontend: HTML5, Vue.js 3 (Global Build / CDN - Options API), Tailwind CSS (CDN) untuk styling.
•	Backend: Google Apps Script (GAS).
•	Database: Google Sheets (Diakses melalui GAS Backend).
•	Library Tambahan: * jsPDF & jsPDF-AutoTable: Untuk generate dokumen PDF (SPT, SPD, Kwitansi, Laporan) di sisi klien.
o	SweetAlert2: Untuk notifikasi / pop-up interaktif.
o	FontAwesome: Untuk ikon.
2. Struktur File
Aplikasi ini secara harfiah hanya terdiri dari 2 file utama di Google Apps Script:
A. Code.gs (Backend)
Bertugas sebagai jembatan antara Frontend dan Google Sheets.
•	doGet(e): Fungsi bawaan GAS untuk melayani halaman HTML (Index.html).
•	loadAllData(): Membaca seluruh data dari tab Sheet ('Pegawai', 'Rekening', dll) dan mengubah baris-baris Excel menjadi array of objects JSON. Data JSON kompleks (seperti array pengikut) di-parse kembali. Pengaturan (Settings) yang di-chunk (dipotong) akan digabung ulang di sini.
•	saveDatabase(dbPayload, settingsPayload): Menerima JSON dari frontend. Menimpa (overwrite) isi sheet yang bersangkutan dengan data terbaru. Fitur krusial: Memotong data gambar/logo (Chunking 45.000 karakter) untuk mencegah error limit 50k character per cell dari Google Sheets.
B. Index.html (Frontend)
Berisi seluruh antarmuka (UI) dan logika bisnis CRUD. File ini menampung Vue Instance di dalam <script>.
•	State (data()):
o	currentView: Mengontrol menu apa yang sedang aktif (login, dashboard, sppd, pegawai, dll).
o	db: Objek utama database lokal browser yang strukturnya mereplikasi tab di Google Sheets. (Berisi array pegawai, rekening, lokasi, shs, sppd).
o	settings: Objek untuk menyimpan konfigurasi kop surat, logo instansi (Base64), dan logo aplikasi.
o	isGAS: Boolean flag. Bernilai true jika dijalankan di URL Apps Script, false jika dibuka sebagai file HTML lokal biasa.
•	Lifecycle (mounted()): Saat aplikasi dimuat, ia akan mengecek isGAS. Jika true, ia akan memanggil google.script.run.loadAllData(). Jika gagal/tidak di GAS, ia akan memuat dari localStorage.
•	Penyimpanan (saveDB()): Setiap aksi Create, Update, Delete akan memodifikasi objek db di Vue, lalu memanggil saveDB(). Fungsi ini menyimpan ke localStorage (backup) DAN mengirimkannya ke GAS melalui google.script.run.saveDatabase().
3. Skema Database (Google Sheets)
Tabel di-generate secara dinamis jika tidak ditemukan. Format kolomnya otomatis menyesuaikan properti Object di Vue.js.
1.	Sheet Pegawai: id, nip, nama, pangkat, golongan, jabatan.
2.	Sheet Rekening: id, kode, kegiatan, subKegiatan, objek, sumberDana.
3.	Sheet Shs: id, kriteria, uangHarian, penginapan.
4.	Sheet Lokasi: id, tujuan, jarak, transport.
5.	Sheet Sppd: id, nomor, rekeningId, maksud, lokasiId, tglBerangkat, tglKembali, lamaHari, pegawaiUtama, pengikut (Array JSON Stringified), laporanHasil, createdAt.
6.	Sheet Settings: Hanya menggunakan Kolom A. Berisi String JSON dari objek settings. Baris A1, A2, A3 digunakan bersusun jika string sangat panjang (untuk gambar Base64 logo).
4. Logika Pembuatan PDF (Modul jsPDF)
Terdapat di method generatePDF(sppdData) di Vue.js.
•	Menggunakan ukuran kertas f4 (215.9 x 330.2 mm) atau sesuai settings.paperSize.
•	drawKopSurat(): Fungsi helper yang digunakan berulang di setiap halaman baru untuk menggambar logo (Base64) dengan doc.addImage() dan teks dinamis (Instansi, Dinas, Alamat).
•	doc.text(x, y): Penulisan teks menggunakan koordinat X (kiri-kanan) dan Y (atas-bawah). Nilai y selalu ditambah (y += 5) setiap baris baru.
•	doc.autoTable(): Digunakan untuk menggambar tabel grid pada dokumen SPD (halaman 2) dan Kwitansi (halaman 3).
•	doc.splitTextToSize(): Digunakan secara luas untuk memecah teks paragraf panjang (seperti Laporan atau Maksud) menjadi array baris-baris pendek agar tidak melebihi lebar margin kanan.
