Dokumentasi RPT (Rencana Pengoperasian Trayek)
==============================================

Versi Dokumen: 1.0  
Tanggal: 17 September 2025

Dokumen ini berisi panduan lengkap untuk:

1.  Pengguna (User) – cara menggunakan modul RPT dari membuat sampai laporan.
2.  Developer – arsitektur, alur kode, model relasi, extensibility & best practice.

  

1\. Ringkasan (Overview)
------------------------

Modul RPT (Rencana Pengoperasian Trayek) digunakan untuk mengelola permohonan trayek kapal: pengajuan baru, perpanjangan, dan penambahan pelabuhan. Fitur utama:

   Input & manajemen data RPT per kapal
   Multi tipe permohonan: Baru, Perpanjangan, Penambahan Pelabuhan
   Upload dokumen & tanda tangan (signature) pegawai
   Persetujuan (approval) multi item & bulk approve
   Notifikasi (pemberitahuan) masa berlaku / status
   Cetak dokumen pengajuan & Crew List
   Laporan (filter periode, kapal, tipe) termasuk ekspor PDF & Excel

Terminologi Singkat:

Istilah

Arti

RPT

Rencana Pengoperasian Trayek

Request Type

Jenis permohonan (Baru / Perpanjangan / Penambahan Pelabuhan)

Parent RPT

RPT induk yang menjadi dasar perpanjangan / penambahan

Extension

Perpanjangan masa berlaku (Perpanjangan)

Addition

Penambahan pelabuhan (Penambahan Pelabuhan)

Ports

Daftar pelabuhan yang diajukan

Approval

Proses persetujuan internal sebelum final

Notification

Pemberitahuan (expired / akan jatuh tempo)

  

2\. Peran (User Roles) & Akses (contoh)
---------------------------------------

> Sesuaikan dengan permission di sistem (policy/gate/permission). Contoh umum:

   Operator Operasional: Membuat & mengedit pengajuan RPT (baru / perpanjangan / penambahan) sebelum disubmit.
   Approver / Supervisor: Melihat daftar permohonan & menyetujui / menolak.
   Manajemen: Mengakses laporan & cetak.
   Admin: Konfigurasi master data (Kapal, Pelabuhan, Departemen, dll.).

  

3\. Alur Kerja Pengguna (User Workflow)
---------------------------------------

 3.1 Membuat RPT Baru

1.  Menu: Operasional > RPT > Tambah (Create)
2.  Pilih Kapal (Ship). Sistem dapat menampilkan detail kapal via AJAX.
3.  Isi: Issue Date, Exp Date, Pelabuhan (pilih multi; bisa ketik pelabuhan baru – akan ditandai (baru)).
4.  Upload dokumen pendukung (jika tersedia).
5.  Simpan (Simpan = status Draft / Pending awal sesuai implementasi).
6.  (Opsional) Cetak / lihat detail untuk verifikasi.

Tips: Field pelabuhan menggunakan Select2 dengan taggable input – pastikan ejaan konsisten untuk menghindari duplikasi logis.

 3.2 Perpanjangan RPT (Extend)

1.  Buka RPT induk (Parent) yang masih berlaku / akan habis.
2.  Klik aksi “Perpanjangan”.
3.  Form otomatis memuat data dasar; atur tanggal Issue/Exp baru.
4.  Simpan – sistem membuat record child (request\type = Perpanjangan) dengan parent\rpt\id.

 3.3 Penambahan Pelabuhan (Addition)

1.  Buka RPT induk yang ingin ditambah pelabuhan.
2.  Klik “Penambahan Pelabuhan”.
3.  Pilih pelabuhan baru yang belum ada dalam daftar; bisa kombinasi existing + baru.
4.  Simpan – child record dengan request\type = Penambahan Pelabuhan tercipta.

 3.3.1 Penting: Akses Invoice & Dokumen di Halaman Detail

Halaman Detail RPT memiliki beberapa panel/riwayat: RPT Induk, Riwayat Perpanjangan, Riwayat Penambahan Pelabuhan. Setiap baris/riwayat biasanya punya tombol "Detail".

Gunakan pedoman berikut:

Kebutuhan

Dimana Klik Detail

Alasan

Lihat / Upload Invoice RPT Baru

Panel utama (RPT induk)

Terkait permohonan pertama

Lihat / Upload Invoice Perpanjangan

Riwayat Perpanjangan (baris perpanjangan yang sesuai)

Invoice melekat pada child perpanjangan tertentu

Lihat / Upload Invoice Penambahan Pelabuhan

Riwayat Penambahan Pelabuhan (baris addition yang sesuai)

Setiap penambahan bisa punya invoice berbeda

Lihat Dokumen Lain (TTD, PDF scan)

Masuk ke detail entri yang relevan

Menghindari salah lampirkan ke induk

Jika Anda mencari invoice tetapi membuka detail yang salah (misal: induk padahal invoice ada di perpanjangan), maka file tidak akan terlihat. Selalu pastikan konteks riwayat yang benar sebelum upload atau review.

 3.3.2 Aturan Satu RPT Baru per Kapal

Untuk setiap kapal hanya dibuat SATU RPT dengan tipe "Baru". Setelah itu:

   Perpanjangan masa berlaku dilakukan lewat fitur Perpanjangan (tidak buat RPT Baru lagi).
   Penambahan pelabuhan menggunakan fitur Penambahan Pelabuhan pada RPT induk.

Jika user terus membuat RPT baru untuk kapal yang sama, data menjadi terfragmentasi dan riwayat terputus. Edukasi user: "Kalau kapal sudah punya RPT Baru, lanjutkan dengan Perpanjangan atau Penambahan saja.".

 3.3.3 Upload Dokumen Sesuai Konteks

Saat berada di halaman detail:

   Pastikan Anda sedang membuka detail riwayat yang tepat sebelum upload (contoh: invoice perpanjangan -> buka detail perpanjangan itu, bukan induk).
   Jangan mengunggah invoice penambahan ke induk atau sebaliknya.
   Penamaan file disarankan mencakup tipe & tanggal (contoh: INVOICEPERPANJANGAN2025-09-17.pdf).

Kesalahan umum:

Kesalahan

Dampak

Solusi

Upload invoice perpanjangan di induk

Reviewer tidak menemukannya

Re-upload di detail perpanjangan benar

Membuat RPT Baru lagi untuk kapal yang sudah punya

Riwayat terpecah

Gunakan fitur Perpanjangan / Penambahan

Tidak klik detail riwayat addition sebelum upload

Dokumen "hilang" bagi user lain

Arahkan user ke detail addition yang sesuai

 3.4 Edit / Update

   Bisa dilakukan sebelum proses final approval.
   Perpanjangan & Penambahan menggunakan route update dengan query ?type=extend atau ?type=addition.

 3.5 Approval Individu

1.  Menu: RPT > Approval / Approve
2.  Filter menggunakan kolom (status, kapal, range tanggal bila ada).
3.  Klik tombol “Approve” atau “Reject” pada baris.
4.  Status berubah; notifikasi / side effect lain berjalan (lihat bagian developer).

 3.6 Bulk Approval

1.  Di view daftar, centang beberapa RPT (checkbox per baris). Tombol akan menampilkan jumlah terpilih.
2.  Klik “Approve Checked”.
3.  Konfirmasi swal -> sistem mengirim request ke endpoint batch (approveChecked).

 3.7 Notifikasi

   Menu: Pemberitahuan RPT.
   Tabel DataTables memuat notifikasi (kadaluarsa / peringatan).
   Dapat dihapus (AJAX DELETE) saat sudah ditindaklanjuti.

 3.8 Cetak Pengajuan (Print)

   Aksi “Print” menghasilkan tampilan print.blade.php (branding perusahaan & detail permohonan).
   Menyertakan tanda tangan jika tersedia (PegawaiSignature).

 3.9 Crew List

   Aksi “Generate Crew List” (route: rpt/{rpt}/generate-crew-list).
   Menampilkan modal / PDF berdasarkan crewlistprint.blade.php.

 3.10 Laporan (Reporting)

1.  Menu: Operasional > Laporan RPT.
2.  Filter: Periode (daterangepicker), Kapal (Select2 server-side), Tipe Permohonan.
3.  Tabel hasil menyorot baris tertentu (mis: baris ke-7 highlight di PDF – logic contoh di view pdf-rpt.blade.php).
4.  Ekspor: PDF / Excel (param: month, period, ship\id, request\type).

 3.11 FAQ Singkat

Pertanyaan

Jawaban

Mengapa pelabuhan baru muncul dengan label (baru)?

Karena input taggable Select2 – belum tersimpan dalam master port.

Bisa hapus child perpanjangan?

Ya, selama belum final/approved & tidak dipakai dependensi lain.

Bagaimana jika tanggal exp melewati kapasitas?

Sistem harus validasi di controller (cek implementasi).

Tidak muncul tanda tangan?

Pastikan record PegawaiSignature terkait pegawai pembuat / penanggung jawab tersedia.

Kenapa invoice perpanjangan tidak muncul di detail induk?

Karena invoice melekat pada child perpanjangan. Buka detail riwayat perpanjangan terkait.

Bolehkah buat dua RPT Baru untuk kapal yang sama?

Tidak direkomendasikan. Gunakan Perpanjangan / Penambahan untuk kelanjutan.

Upload invoice penambahan salah tempat, bagaimana?

Buka detail riwayat penambahan yang benar lalu upload ulang; hapus yang salah jika perlu.

  

4\. Panduan Developer (Developer Guide)
---------------------------------------

 4.1 Arsitektur Umum

Komponen utama:

   Controller Utama: RptController (CRUD, approval, notifications, print, crew list).
   Controller Laporan: Report\RptReportController (index + export PDF/Excel + server-side data fetch).
   Views: Folder resources/views/operasional/rpt/ & resources/views/operasional/report/rpt/.
   Model (indikatif): Rpt, RptPort, Port, MasterKapal, RptRequest (jika terpisah), PegawaiSignature, Pemberitahuan.
   Library: DataTables (AJAX), PhpSpreadsheet (Excel), DOMPDF / snappy (alias PDF), Datepicker, Select2.

 4.2 Routing Kunci

Lokasi: routes/operasional/web.php.  
Contoh:

    Route::resource('rpt', 'Operasional\RptController');
    Route::get('rpt/{rpt}/generate-crew-list', 'Operasional\RptController@generateCrewList')->name('rpt.generate-crew-list');
    // Laporan
    Route::get('laporan/rpt', 'Operasional\Report\RptReportController@index')->name('rpt.report.index');
    Route::get('laporan/rpt/export-excel', 'Operasional\Report\RptReportController@exportExcel')->name('rpt.report.excel');
    Route::get('laporan/rpt/export-pdf', 'Operasional\Report\RptReportController@exportPdf')->name('rpt.report.pdf');
    

Catatan: Sesuaikan dengan route sebenarnya bila berbeda; tambahkan naming route untuk konsistensi.

 4.3 Relasi Model (Konseptual)

    Rpt
        belongsTo ship (MasterKapal)
        hasMany ports (pivot: rptports)
        belongsTo parent (rptparentid)
        hasMany children (perpanjangan / penambahan)
        hasOne signature? (melalui PegawaiSignature -> pegawai)
    

Pastikan eager loading di query besar (with(['ship','ports','children'])) untuk menghindari N+1.

 4.4 Field & Atribut Utama (Indicative)

Kolom

Deskripsi

request\type

'Baru' / 'Perpanjangan' / 'Penambahan Pelabuhan'

issue\date

Tanggal diterbitkan

exp\date

Tanggal kadaluarsa

parent\rpt\id

Referensi induk (nullable)

status

Draft / Pending / Approved / Rejected (cek implementasi)

notes

Catatan internal

 4.5 Status Flow (State Machine)

Dari

Ke

Trigger

Side Effect

Draft

Pending

Submit awal (store)

Mungkin buat notifikasi awal

Pending

Approved

approve() / approveChecked()

Update tanggal, kirim pemberitahuan lanjutan

Pending

Rejected

reject()

Simpan alasan penolakan (jika ada)

Approved

(Extension) Child Baru

extend form

Child inherits context

 4.6 Request Type Handling

Controller memisahkan logika pembuatan child melalui method privat: newRptRequest, extendRptRequest, additionRptRequest. Parameter umum: request, ship, requestNumber.

 4.7 DataTables & AJAX

Setiap view list menggunakan inisialisasi DataTables dengan server-side processing (kolom: nomor, kapal, issue, exp, status, aksi, dsb.). Endpoint: method index() -> cabang if ($request->ajax()) memanggil helper privat dataTable($request) yang menyiapkan collection / query.

 4.8 Approval Batch

Method approveChecked(Request) menerima array ID -> loop update status; tambahkan transaksi DB jika perlu atomicity. Pertimbangkan locking jika concurrency tinggi.

 4.9 Notifikasi

Method notification() mempersiapkan DataTables dari model Pemberitahuan. notificationDestroy($id) menghapus entri -> front-end swal konfirmasi.  
CRON (lihat routes) memanggil CronController@checkrpt untuk membuat notifikasi kedaluwarsa / reminder.

 4.10 PDF & Export

   Print Pengajuan: view rpt/print.blade.php.
   Crew List: view rpt/crewlistprint.blade.php.
   Laporan PDF: operasional/report/rpt/pdf-rpt.blade.php (helper variable $data, highlight baris ke-7 contoh).
   Laporan Excel: Method exportExcel() menggunakan PhpSpreadsheet (mergeCells, styling header, border, alignment). Perhatikan kinerja jika dataset besar; bisa streaming writer.

 4.11 Performance Notes

   Tambahkan index DB pada kolom: requesttype, issuedate, expdate, parentrptid, shipid, status.
   Gunakan pagination server-side bila row > beberapa ribu.
   Batasi eager loading children (mungkin hanya jenis tertentu) pada laporan berat.

 4.12 Validasi

Pastikan di store() / update():

   issue\date < exp\date
   Pelabuhan unik per RPT (hindari duplikasi di pivot)
   Perpanjangan hanya diizinkan bila RPT induk Approved.
   Penambahan tidak memodifikasi list induk; hanya child extension (kecuali requirement lain).

 4.13 Security & Permissions

   Gunakan gate/permission (contoh: operasional-lihat-laporan-rpt).
   Validasi ID di approval untuk memastikan user hanya menyetujui yang diizinkan.
   Hindari mass assignment tanpa guarded / fillable.

 4.14 Error Handling

   Bungkus operasi kritikal (batch approve, create child) dalam DB::transaction().
   Logging: gunakan Log::error() bila terjadi exception; kirim respon JSON standar { success:false, message }.

 4.15 Testing (Direkomendasikan)

Unit / Feature:

   Membuat RPT baru (assert DB + status awal).
   Perpanjangan membuat child dengan parent\rpt\id benar.
   Approval mengubah status & mencatat waktu.
   Export Excel menghasilkan header sesuai.
   Notifikasi cron menambahkan entri untuk RPT expiring <= X hari.

 4.16 Extensibility Ideas

   Tambah role khusus auditor (read-only).
   Tambah filter advanced (range Exp Date, by Port).
   Integrasi digital signature (PKI) untuk print.
   Soft delete agar histori dapat dipulihkan.
   Event / Listener (e.g. RptApproved) untuk integrasi modul lain.

 4.17 Common Pitfalls

Masalah

Penyebab

Solusi

Duplicate pelabuhan

Input taggable tanpa normalisasi

Normalisasi nama (uppercase/trim) sebelum insert

N+1 query children

Tidak eager load

Tambahkan with('children')

Approval race condition

Multi approver klik bersamaan

Gunakan where status = Pending pada update & cek affected rows

File besar PDF lambat

Banyak image / style inline

Optimalkan logo, caching view

 4.18 Struktur View Ringkas

File

Fungsi

index.blade.php

List utama + bulk approve

create/edit.blade.php

Form RPT baru / edit

edit\extend\rpt.blade.php

Form perpanjangan

edit\addition\rpt.blade.php

Form penambahan pelabuhan

approval.blade.php / approve.blade.php

Layar approval (varian UI)

notification.blade.php

Daftar pemberitahuan

detail.blade.php

Tampilan detail & modal invoice/KSOP

print.blade.php

Cetak pengajuan

crew\list\print.blade.php

Cetak crew list

report/\

Laporan & export

 4.19 Pseudocode Ringkas Alur Store

    if (requesttype == 'Baru') newRptRequest();
    elseif (requesttype == 'Perpanjangan') extendRptRequest();
    elseif (requesttype == 'Penambahan Pelabuhan') additionRptRequest();
    attachPorts();
    if (needsNotification) createNotification();
    return JSON / redirect.
    

 4.20 Batch Approval Pseudocode

    ids = request->ids
    DB::transaction(function(){
        foreach ids as id:
            rpt = Rpt::lockForUpdate()->find(id)
            if rpt.status == 'Pending':
                 rpt.status = 'Approved'
                 rpt.approvedat = now()
                 rpt.save()
    });
    

  

5\. Laporan & Export
--------------------

Parameter input (query string):

Param

Deskripsi

Contoh

period

Range tanggal (issue\date) YYYY-MM-DD - YYYY-MM-DD

2025-09-01 - 2025-09-30

ship\id

ID kapal atau 'all'

12

request\type\[\]

Array tipe (multi)

\['Baru','Perpanjangan'\]

month

Label bulan untuk judul export

September 2025

Excel: Styling header (merge A1:H1, border thin semua kolom).  
PDF: Menggunakan view pdf-rpt.blade.php. Pastikan memory\limit cukup (optimalkan collection sebelum render).

  

6\. Saran Peningkatan (Backlog)
-------------------------------

   Refinement status (Submitted vs Pending vs Verified).
   Audit log (spatie/laravel-activitylog).
   Port master normalization service.
   Attachment management (versioning, file size limit display).
   Full-text search di catatan.
   Multi-level approval chain.
   Scheduler untuk auto-close expired tanpa extension.

  

7\. Lampiran (Appendix)
-----------------------

 7.1 Tabel Status (Detail)

Status

Deskripsi

Dapat Diedit

Aksi Lanjutan

Draft

Baru dibuat, belum diajukan

Ya

Submit -> Pending

Pending

Menunggu approval

Terbatas

Approve / Reject

Approved

Disetujui final

Tidak (kecuali admin)

Cetak / Extend / Addition

Rejected

Ditolak

Ya (bisa revisi & submit ulang)

Submit kembali

 7.2 Glossary

Istilah

Penjelasan

Crew List

Daftar awak kapal untuk satu permohonan / kapal

Signature

TTD elektronik tersimpan di tabel pegawai\signature

Highlight Row

Baris ke-7 diberi warna khusus pada PDF untuk kebutuhan internal

  

Dokumen ini dapat diperbarui seiring perubahan requirement. Harap review sebelum release besar.

> Disusun oleh: Tim Pengembang Operasional
