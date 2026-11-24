# Prosedur Submission Final ProcureFlow

Dokumen ini mengatur struktur, penamaan, pemeriksaan, dan pengumpulan hasil assessment. Instruktur harus mengisi kanal pengumpulan dan batas ukuran sebelum presentasi Sesi 2.

## 1. Informasi yang ditetapkan instruktur

| Item | Nilai final |
| --- | --- |
| Kanal pengumpulan | `<UPLOAD-PORTAL / SFTP / LMS / LOKASI-LAIN>` |
| Alamat atau path | `<URL-ATAU-PATH>` |
| Batas waktu | `<TANGGAL DAN WAKTU, ASIA/JAKARTA>` |
| Batas ukuran ZIP | `<UKURAN-MAKSIMUM>` |
| Kebijakan keterlambatan | `<KEBIJAKAN>` |
| Kanal fallback | `<FALLBACK-JIKA-UPLOAD-GAGAL>` |
| Kontak kendala | `<KONTAK-INSTRUKTUR>` |

Jika submission menggunakan VPS, isi hostname, port, metode autentikasi, path upload, batas ukuran, dan akun peserta setelah server selesai disiapkan. Jangan menaruh private key atau password VPS di dokumen ini.

## 2. Nama folder peserta

Gunakan pola:

```text
<ID-PESERTA>_<NAMA-SINGKAT>_ProcureFlow/
```

Contoh:

```text
P017_BudiSantoso_ProcureFlow/
```

Aturan penamaan:

- gunakan ID peserta yang diberikan instruktur;
- gunakan huruf, angka, underscore, dan hyphen;
- jangan gunakan spasi, slash, tanda kutip, atau karakter khusus;
- nama folder di dalam ZIP harus sama dengan nama file ZIP;
- jangan memakai nama kelompok karena assessment dikerjakan individual.

## 3. Struktur submission

```text
<ID-PESERTA>_<NAMA-SINGKAT>_ProcureFlow/
├── LAPORAN.docx
├── evidence/
│   ├── EVIDENCE_INDEX.md
│   ├── E-001_<deskripsi-singkat>.<ext>
│   ├── E-002_<deskripsi-singkat>.<ext>
│   └── ...
├── transcript/
│   ├── TRANSCRIPT_INDEX.md
│   ├── T-001_<nama-session>.<ext>
│   └── ...
└── environment.txt
```

Peserta yang memakai template Markdown mengganti `LAPORAN.docx` dengan `LAPORAN.md`. Kumpulkan tepat satu laporan utama, bukan DOCX dan Markdown dengan isi yang berbeda.

Skeleton tersedia pada folder `SUBMISSION_TEMPLATE/`. Salin skeleton ke directory kerja di luar source ProcureFlow, lalu ganti nama folder sesuai ID peserta.

## 4. Isi setiap bagian

### 4.1 Laporan

Gunakan template yang terdapat di dalam `ProcureFlow.zip`.

Laporan harus memuat:

- identitas peserta;
- checksum atau release identifier ProcureFlow;
- environment dan versi tool;
- ringkasan root cause unik;
- detail source dan runtime evidence;
- attack chain jika setiap link terbukti;
- hipotesis yang ditolak atau false positive;
- catatan penggunaan AI;
- daftar evidence;
- limitation dan rekomendasi regression test.

Pastikan setiap evidence ID pada laporan mempunyai file atau record yang cocok di folder `evidence/`.

### 4.2 Evidence

Folder `evidence/` hanya berisi artifact yang dirujuk laporan, misalnya:

- screenshot PNG atau JPG;
- request dan response yang sudah disanitasi;
- browser snapshot atau DOM marker;
- potongan log aplikasi atau worker;
- output perintah yang relevan;
- artifact export dummy dari lab;
- catatan state sebelum dan sesudah tindakan.

Gunakan pola nama:

```text
E-<NOMOR>_<DESKRIPSI-SINGKAT>.<EXT>
```

Contoh:

```text
E-001_login-employee.png
E-002_request-own-object.txt
E-003_response-cross-tenant.txt
E-004_worker-log.txt
```

`EVIDENCE_INDEX.md` mencatat ID, nama file, jenis, finding terkait, waktu, provenance, hal yang dibuktikan, dan limitation.

Jangan menyertakan:

- session cookie;
- bearer token;
- API key;
- password di luar credential dummy lab;
- private key;
- data personal nyata;
- seluruh log container jika hanya beberapa baris yang relevan;
- file database atau volume Docker penuh;
- source ProcureFlow.

### 4.3 Transcript AI

Folder `transcript/` berisi transcript asli dari session AI yang digunakan dalam assessment.

Gunakan pola nama:

```text
T-<NOMOR>_<NAMA-SESSION>.<EXT>
```

Contoh:

```text
T-001_architecture-review.md
T-002_authorization-analysis.json
T-003_runtime-verification.txt
```

Transcript boleh dipisahkan per session. `TRANSCRIPT_INDEX.md` menjelaskan urutan, tool atau agent, waktu, dan finding yang terkait.

Peserta tidak boleh:

- menghapus respons AI yang salah;
- mengubah tool result;
- menyusun transcript baru yang tidak mencerminkan session asli;
- memasukkan token atau credential aktif;
- menyertakan percakapan yang tidak terkait assessment.

Redaksi secret diperbolehkan jika teks pengganti seperti `<REDACTED_SESSION>` mempertahankan konteks dan lokasi redaksi dicatat.

### 4.4 Environment

Isi `environment.txt` dengan:

- identitas peserta;
- waktu mulai dan selesai;
- release identifier dan SHA-256 `ProcureFlow.zip`;
- sistem operasi;
- versi Docker dan Docker Compose;
- versi OpenCode;
- provider, model, dan versi model;
- konfigurasi atau permission profile yang ditentukan instruktur;
- status akses Internet;
- URL dan port lokal;
- ringkasan container yang aktif;
- reset yang dilakukan;
- kendala environment dan intervensi instruktur.

Jangan menempelkan `.env`, credential, cookie, token, atau konfigurasi provider yang mengandung secret.

## 5. File yang dilarang masuk submission

- hasil ekstraksi `ProcureFlow.zip`;
- source aplikasi;
- `node_modules/`, image, container export, atau Docker volume;
- `.env`;
- cache OpenCode atau model;
- credential provider;
- cookie dan session database;
- file answer key atau materi instruktur;
- scanner database dan dependency cache;
- file yang tidak dirujuk laporan;
- artifact dari peserta lain.

## 6. Pemeriksaan sebelum membuat ZIP

- [ ] Nama folder mengikuti format.
- [ ] Hanya ada satu laporan utama.
- [ ] Laporan dapat dibuka.
- [ ] Identitas dan release identifier sudah diisi.
- [ ] Semua finding mempunyai source dan runtime evidence.
- [ ] Semua evidence ID memiliki file yang cocok.
- [ ] Semua file evidence tercantum pada `EVIDENCE_INDEX.md`.
- [ ] Transcript asli tersedia dan tercantum pada `TRANSCRIPT_INDEX.md`.
- [ ] `environment.txt` terisi.
- [ ] Cookie, token, API key, private key, dan data nyata sudah dihapus.
- [ ] Source ProcureFlow tidak ikut disalin.
- [ ] Tidak ada symlink atau shortcut ke file di luar folder submission.
- [ ] Ukuran submission berada di bawah batas instruktur.

## 7. Membuat ZIP final

Jalankan dari directory yang berisi folder submission:

```bash
zip -X -r P017_BudiSantoso_ProcureFlow.zip P017_BudiSantoso_ProcureFlow/
```

Ganti contoh nama dengan ID dan nama peserta sendiri.

Uji integritas ZIP:

```bash
unzip -t P017_BudiSantoso_ProcureFlow.zip
```

Hitung checksum submission:

```bash
sha256sum P017_BudiSantoso_ProcureFlow.zip
```

Simpan checksum yang muncul dan catat pada bukti pengumpulan.

## 8. Pengumpulan

1. Upload satu file ZIP final ke kanal yang ditentukan.
2. Tunggu konfirmasi upload selesai.
3. Catat timestamp, ukuran file, dan SHA-256.
4. Jangan mengubah ZIP setelah upload berhasil.
5. Jika upload gagal sebelum batas waktu, simpan screenshot atau log error dan gunakan kanal fallback.
6. Laporkan kegagalan kepada instruktur sebelum batas waktu jika memungkinkan.

Submission dianggap diterima setelah sistem atau instruktur memberikan receipt yang memuat nama file dan waktu.

## 9. Receipt peserta

Catat informasi berikut:

```text
Nama file ZIP:
Ukuran:
SHA-256:
Waktu upload:
Kanal:
Submission ID atau receipt:
Status: BERHASIL / FALLBACK / GAGAL
Catatan:
```

## 10. Pemeriksaan awal instruktur

Pemeriksaan ini memastikan kelengkapan file dan tidak menilai kebenaran finding.

- [ ] Nama ZIP dan folder root cocok.
- [ ] Timestamp berada dalam aturan.
- [ ] SHA-256 submission dicatat.
- [ ] ZIP lulus pemeriksaan integritas.
- [ ] Tidak ada absolute path, path traversal, atau symlink.
- [ ] Laporan dapat dibuka.
- [ ] Evidence index dan transcript index tersedia.
- [ ] File yang dirujuk laporan tersedia.
- [ ] `environment.txt` tersedia.
- [ ] Tidak ada source ProcureFlow atau file terlarang.
- [ ] Tidak ada secret aktif yang terlihat pada pemeriksaan awal.

Ekstrak submission hanya pada directory pemeriksaan yang terisolasi. Jangan menjalankan script, macro, executable, atau command yang berasal dari submission peserta.

## 11. Jika submission tidak lengkap

Instruktur mencatat kondisi apa adanya:

- file laporan tidak dapat dibuka;
- evidence yang dirujuk hilang;
- transcript tidak tersedia;
- ZIP rusak;
- nama peserta tidak jelas;
- submission melewati batas waktu;
- secret aktif ditemukan;
- source ProcureFlow ikut terkumpul.

Tindakan mengikuti kebijakan yang sudah diumumkan sebelum assessment. Jangan membuat kebijakan baru untuk satu peserta setelah deadline.
