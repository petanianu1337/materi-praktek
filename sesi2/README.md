# Materi Assessment Sesi 2

Sesi 2 menggunakan `ProcureFlow.zip`. Jangan membuka atau menganalisis paket sebelum instruktur memulai assessment.

## Verifikasi paket

Dari folder `sesi2`, jalankan:

```bash
sha256sum -c SHA256SUMS.txt
unzip -t ProcureFlow.zip
```

Kedua pemeriksaan harus berhasil sebelum paket digunakan. Laporkan perbedaan checksum kepada instruktur dan jangan melanjutkan dengan paket yang berbeda.

## Memulai assessment

```bash
unzip ProcureFlow.zip
cd ProcureFlow
docker compose up --build -d
docker compose ps
```

Baca `ProcureFlow/README.md` dan `ProcureFlow/ASSESSMENT_RULES.md` sebelum menganalisis source. Aplikasi hanya boleh dijalankan secara lokal sesuai konfigurasi paket.

## Laporan dan submission

- Gunakan `REPORT_TEMPLATE.docx` atau `REPORT_TEMPLATE.md` sebagai satu laporan utama.
- Salin `SUBMISSION_TEMPLATE/` ke directory kerja di luar source ProcureFlow.
- Ikuti `PROSEDUR_SUBMISSION.md` untuk penamaan evidence, transcript, environment, dan ZIP final.
- Jangan memasukkan source ProcureFlow, `.env`, session cookie, token, API key, atau credential aktif ke submission.

Kanal upload, deadline, dan batas ukuran ditetapkan instruktur sebelum assessment dimulai.
