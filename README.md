# Project Management (WPF, .NET 8)

Aplikasi desktop Windows untuk tim: data tersimpan di **Supabase** (paket gratis), akun **email + nickname + password**,
dan perubahan langsung terlihat oleh anggota lain secara **realtime**. Menggabungkan tiga konsep desain:

| Halaman | Konsep | Isi |
|---|---|---|
| **Ringkasan** (`Ctrl+1`) | D — Dashboard hybrid | Kartu statistik, timeline/Gantt per epic, Tugas Saya, beban kerja tim di sprint aktif |
| **Backlog & Sprint** (`Ctrl+2`) | B | Sprint aktif & direncanakan, backlog, seret issue antar-sprint, mulai/selesaikan sprint, riwayat |
| **Sprint Board** (`Ctrl+3`) | B — Sprint board | Issue sprint aktif per status + batas WIP, filter anggota/tipe, drag & drop kartu |
| **Daftar Issue** (`Ctrl+4`) | C — Fokus, keyboard-first | Daftar per status (semua / sprint aktif / backlog), pintasan J/K/S/P/A/C |
| Panel detail (kanan) | B | Status, penanggung jawab, prioritas, epic, tanggal, **sprint**, sub-tugas, komentar |
| **Epic** (`Ctrl+5`) | — | Daftar epic proyek: progres (story point selesai), rentang tanggal, jumlah issue; buat/ubah/hapus epic; *Lihat issue* membuka Daftar Issue terfilter |
| **Tim** (`Ctrl+6`) | — | Undang anggota lewat email akun, peran akses (owner/admin/member/viewer), status online, profil & kapasitas SP per sprint, beban di sprint aktif |
| **Burndown** | B | Kartu di Ringkasan + dialog lengkap (tombol di Board, riwayat sprint di Backlog, atau palet) |
| Palet perintah (`Ctrl+K`) | C | Navigasi, aksi issue, pindah sprint, beralih proyek, buat proyek/sprint, undang anggota, ganti bahasa |

### Tim & sinkron realtime (Supabase)

- **Setup sekali, gratis:** ikuti [docs/SUPABASE-SETUP.md](docs/SUPABASE-SETUP.md) (proyek Supabase Free, skema SQL,
  pengaturan login email). Tidak perlu hosting server aplikasi dan tidak perlu kartu kredit.
- **Akun:** *Buat akun* (email, nickname, password minimal 6 karakter) atau *Masuk*, langsung di aplikasi lewat
  Supabase Auth. Nickname menjadi nama yang dilihat rekan. Password tidak disimpan di komputer; sesi diingat dengan
  refresh token terenkripsi DPAPI (`%LocalAppData%\ProjectManagement\session.bin`).
- **Workspace:** buat workspace, lalu undang rekan lewat email akunnya. Rekan otomatis bergabung saat masuk/mendaftar dengan email itu.
  Satu akun bisa berada di beberapa workspace (*Ganti workspace* di menu akun kiri bawah).
- **Peran akses** dijaga oleh Row Level Security di database, bukan hanya di aplikasi:
  owner/admin (kelola anggota), member (ubah data), viewer (hanya baca, tampil lencana *Hanya lihat*).
- **Realtime:** perubahan rekan (issue, sprint, epic, komentar, anggota) muncul langsung tanpa muat ulang.
  Titik hijau pada avatar = sedang online; panel detail menampilkan siapa yang sedang membuka issue yang sama.
- **Simpan otomatis:** perubahan dikirim berkelompok ±0,7 detik setelah mengetik, hanya kolom yang berubah.
  Saat offline perubahan diantre dan dikirim ulang otomatis; status sinkron tampil di kiri bawah.
  Kolom yang sedang Anda ubah tidak ditimpa oleh perubahan rekan yang datang bersamaan.
- **SQL penyiapan database** tertanam di aplikasi: tombol *Salin SQL penyiapan* + *Buka SQL Editor* di layar Pengaturan server,
  otomatis muncul bila database belum disiapkan, dan di menu akun admin (*Salin SQL database*) untuk pembaruan skema.
- **Nomor issue** (`REA-12`) dibuat server secara atomik, jadi dua orang yang membuat issue bersamaan tidak bentrok.
- **Impor data lama:** saat membuat workspace, data versi offline (`workspace.json`) bisa diimpor sekali; file aslinya tidak diubah.

### Drag & drop

Di Sprint Board (kartu antar-kolom) dan halaman Backlog (issue antar-sprint), lewat `Controls/CardDrag.cs`:
- **Kartu terbawa:** salinan kartu melayang mengikuti kursor (miring 2,5°, membesar 4%, berbayang) dengan animasi "diangkat".
  Baris backlog yang lebar ditampilkan sebagai kartu ringkas (ikon, key, judul).
- Kartu asli menjadi samar sebagai penanda posisi asal. Kolom atau sprint tujuan menyala biru, dan kursor berubah menjadi tangan
  (atau tanda dilarang di luar area lepas).
- **Mendarat:** kartu muncul di tempat baru sambil memudar masuk dan sedikit memantul.
- Klik tanpa menyeret tetap membuka panel detail.

### Ekspor laporan (`Ctrl+E`)

Tombol **Export** di bilah atas, tombol di dialog Burndown, atau palet perintah. Tiga jenis:

| Jenis | Format | Isi |
|---|---|---|
| Laporan sprint | PDF / HTML | Tujuan, komitmen/selesai/sisa SP, grafik burndown, issue selesai & belum, rincian per penanggung jawab dan per epic |
| Laporan proyek | PDF / HTML | Progres, epic, riwayat sprint, burndown sprint aktif, beban kerja tim, issue terlambat |
| Daftar issue | CSV | Semua kolom issue (semua / sprint aktif / backlog), untuk Excel atau Google Sheets |

- **PDF (bawaan)** berukuran A4 dan dibuat langsung oleh aplikasi lewat **Microsoft Edge headless**
  (`Services/Reports/PdfExporter.cs`). Edge terpasang bawaan di Windows 10/11, jadi tidak perlu library tambahan;
  Google Chrome dipakai sebagai cadangan. Prosesnya tidak memunculkan jendela, memakai profil sementara yang terpisah
  dari Edge pengguna, dan dihentikan bila lebih dari 60 detik. Bila tidak ada Edge/Chrome, opsi PDF dinonaktifkan dan HTML tetap tersedia.
- **HTML** berupa satu file mandiri (tanpa internet, grafik SVG) yang bisa dikirim lewat email.
- CSV ditulis dalam UTF-8 dengan BOM, dan pemisah kolom mengikuti pengaturan wilayah Windows (`;` untuk Indonesia, `,` untuk US)
  agar langsung rapi di Excel. Tanggal memakai format `yyyy-mm-dd`, dan sel yang diawali `= + - @` diamankan dari formula injection.
- Teks laporan mengikuti bahasa aplikasi yang aktif.

### Epic

- Epic milik satu proyek, dengan nama (unik per proyek), deskripsi, dan warna.
- Hapus epic: issue-nya tetap ada, hanya kehilangan epic (ada konfirmasi).
- Panel detail: pilih epic atau *Tanpa epic*; tombol `+` membuat epic baru dan langsung memasangnya ke issue itu.
- Daftar Issue punya filter epic (*Semua epic*, tiap epic, *Tanpa epic*).
- Palet `Ctrl+K`: *Buat epic*, *Tampilkan issue di epic …*, *Pasang epic → …* untuk issue yang sedang dibuka.

### Tipe issue custom

- Selain tipe bawaan **Story, Bug, Task**, tim bisa membuat tipe sendiri (mis. *Improvement*, *Spike*) dengan nama dan warna.
  Tipe custom berlaku untuk semua proyek di workspace dan tersinkron realtime ke anggota lain.
- Panel detail: tombol `+` di samping *Type* membuat tipe baru dan langsung memasangnya; tombol pensil mengubah atau menghapus tipe custom.
- Menghapus tipe: issue yang memakainya kembali menjadi *Task* (dijaga juga oleh database).
- Filter tipe di Sprint Board dan kolom tipe di ekspor CSV ikut menampilkan tipe custom.
- Database lama: jalankan ulang SQL penyiapan (menu akun → **Salin SQL database**, tempel di SQL Editor Supabase) agar tabel `issue_types` dibuat.

### Multi-bahasa

- Bahasa bawaan **English**; tersedia juga **Bahasa Indonesia**. Ganti lewat pemilih bahasa di kiri bawah
  atau palet perintah. Berlaku langsung tanpa restart, dan pilihan disimpan di `settings.json`.
- Teks UI ada di `src/Rencana.App/Localization/en.json` dan `id.json` (kunci → teks, placeholder `{0}`).
  - XAML: `Text="{l:T Page.Overview}"` dengan `xmlns:l="clr-namespace:Rencana.Localization"`.
  - C#: `Loc.T("Page.Overview")` atau `Loc.F("Sprint.DaysLeft", 3)`.
  - Kunci yang tidak ada di bahasa aktif memakai teks English.
- **Menambah bahasa:** salin `en.json` menjadi misalnya `ja.json`, terjemahkan nilainya, lalu tambahkan
  `new LanguageOption("ja", "日本語")` di `Loc.Languages` (dan pemetaan budaya di `Loc.SetLanguage`).
- Format tanggal mengikuti bahasa (en-US / id-ID). Data yang diketik pengguna (judul issue, nama sprint) tidak diterjemahkan.

### Burndown

- Sisa(hari) = total story point di sprint − story point issue yang selesai sampai akhir hari itu.
  Garis ideal turun lurus dari total ke 0 di hari terakhir. Garis putus-putus oranye = hari ini.
- Waktu selesai dicatat otomatis saat issue pindah ke *Done* (`CompletedAt`). Bila dibuka kembali, waktunya dihapus.
- Saat sprint diselesaikan, total story point disimpan (`FinalTotalPoints`) agar burndown riwayat tetap benar
  walau issue yang belum selesai sudah dipindah ke sprint lain.

### Multi-proyek & sprint

- **Pemilih proyek** di kiri atas: beralih proyek, *Pengaturan proyek*, *Proyek baru*. Semua halaman hanya menampilkan data proyek aktif.
- Setiap proyek punya key sendiri (mis. `REA`, `KSR`) untuk nomor issue, warna, tanggal mulai/target, epic, dan sprint.
- Siklus sprint: **Direncanakan → Aktif → Selesai**. Hanya satu sprint aktif per proyek.
  - *Mulai sprint*: tanggal digeser ke hari ini bila dimulai lebih awal/terlambat (durasi tetap).
  - *Selesaikan sprint*: issue yang belum selesai dipindah ke sprint berikutnya, sprint baru, atau backlog; opsi langsung mulai sprint berikutnya.
- Issue baru dari board/tombol atas masuk sprint aktif; dari halaman Backlog masuk backlog.
- Anggota tim berlaku untuk semua proyek di workspace.

Tema **Light / Dark**: tombol di kiri bawah atau `Ctrl+Shift+L`. Pilihan disimpan; saat pertama kali dibuka, tema mengikuti Windows.

## Struktur

```
src/Rencana.App/
  App.xaml(.cs)            Startup: pengaturan → bahasa → tema → layar awal (login, workspace) → jendela utama
  Shell/                   MainWindow.xaml + MainViewModel (navigasi, pencarian, bahasa, overlay)
  Localization/            Loc.cs (layanan + markup {l:T}), en.json, id.json
  Controls/                BurndownChart (grafik digambar sendiri, ikut tema)
  Config/supabase.json     Alamat Supabase + anon key bawaan untuk installer tim (boleh kosong)
  Features/
    Start/                 StartWindow.xaml + StartViewModel.cs (server, masuk/buat akun, pilih/buat workspace, impor)                 EpicsView.xaml + EpicsViewModel.cs, EpicDialog + EpicEditorViewModel.cs
    Team/                  TeamView.xaml + TeamViewModel.cs, MemberDialog + MemberEditorViewModel.cs, InviteDialog
    Reports/               BurndownDialog + BurndownViewModel.cs, ExportDialog + ExportViewModel.cs
    Overview/              OverviewView.xaml + OverviewViewModel.cs
    Backlog/               BacklogView.xaml (+ drag & drop antar-sprint) + BacklogViewModel.cs
    Projects/              ProjectDialog.xaml + ProjectEditorViewModel.cs
    Sprints/               SprintDialog, CompleteSprintDialog (+ ViewModel masing-masing)
    Board/                 BoardView.xaml (+ drag & drop di .xaml.cs) + BoardViewModel.cs
    Issues/                IssueListView.xaml (+ pintasan keyboard) + IssueListViewModel.cs
    IssueDetail/           IssueDetailView.xaml + IssueDetailViewModel.cs
    CommandPalette/        CommandPaletteView.xaml + CommandPaletteViewModel.cs
  Models/                  Issue, Project, Sprint, Member, Epic, enum + label terlokalisasi
  Services/                Workspace (data, proyek aktif, operasi sprint, terima perubahan server),
                           Cloud/ (SupabaseAuth, SupabaseRest, RealtimeClient, CloudSync, CloudMapper,
                                   CloudService, LocalImport), Burndown, Reports/ (HtmlReport, CsvExporter, PdfExporter),
                           Migration (format file lama, untuk impor), AppSettings, LanguageService, ThemeService
  Themes/                  Light.xaml, Dark.xaml (warna), Styles.xaml (gaya kontrol)
  Assets/                  app.ico (ikon aplikasi, 16–256 px) + app-256.png
supabase/migrations/       Skema database + RLS + realtime (jalankan di SQL Editor Supabase)
tests/Rencana.CloudTests/  Uji lapisan cloud tanpa jaringan (server palsu)
docs/SUPABASE-SETUP.md     Panduan setup gratis
installer/                 Proyek WiX v5 → .msi
tools/make-icon.ps1        Membuat ulang ikon aplikasi
build-installer.ps1        Publish + build installer
```

- Pola MVVM dengan [CommunityToolkit.Mvvm](https://www.nuget.org/packages/CommunityToolkit.Mvvm).
- Semua warna di XAML memakai `DynamicResource`, jadi tema berganti langsung tanpa restart.
  Untuk menambah warna, definisikan key yang sama di **Light.xaml dan Dark.xaml**.
- Data tim disimpan di Supabase. Di komputer hanya ada `%LocalAppData%\ProjectManagement\settings.json` (tema, bahasa,
  workspace terakhir), `session.bin` (login), dan `supabase.json` bila server diisi lewat layar awal.

## Menjalankan

Butuh .NET 8 SDK.

```bash
dotnet run --project src/Rencana.App
```

Uji lapisan cloud (masuk/daftar akun, pemetaan data, antrian sinkron, realtime, penggabungan data):

```bash
dotnet run --project tests/Rencana.CloudTests
```

Skema database diuji terpisah dengan PostgreSQL tertanam (aturan akses per peran, penomoran issue, undangan).

## Membuat installer

```bash
powershell -ExecutionPolicy Bypass -File .\build-installer.ps1 -Version 0.0.1
```

Hasilnya `artifacts\ProjectManagement-Setup-0.0.1-x64.msi`. Paket WiX diunduh otomatis dari NuGet.
Aplikasi dipublish self-contained (pengguna tidak perlu memasang .NET), dipasang ke
`C:\Program Files\Project Management`, dengan shortcut di Start Menu dan Desktop. Upgrade otomatis
mengganti versi lama selama `UpgradeCode` di `installer/Package.wxs` tidak diubah.
Versi saat ini **0.0.1**; naikkan `-Version` (dan `<Version>` di `Rencana.App.csproj`) saat rilis berikutnya.
Installer memakai `AllowDowngrades`, jadi versi ini juga menggantikan instalasi lama bernama "Rencana" 1.x.
Saat pertama dijalankan, folder data lama `%LocalAppData%\Rencana` dipindah otomatis ke `%LocalAppData%\ProjectManagement`
(login dan pengaturan tetap tersimpan). Nama di kode (namespace `Rencana`, folder `src/Rencana.App`) sengaja tidak diubah. Isi `src/Rencana.App/Config/supabase.json` lebih dulu agar
anggota tim tidak perlu mengetik alamat server.
