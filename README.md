# Materi Praktik Agentic Security

Repository ini berisi file yang digunakan peserta pada pelatihan analisis source code dan web vulnerability dengan OpenCode. Gunakan materi hanya pada target lab dan waktu pengujian yang ditetapkan instruktur.

## Persiapan perangkat

Pasang toolchain dari repository [prektek-lab](https://github.com/petanianu1337/prektek-lab) sebelum pelatihan. Installer mendukung Ubuntu 24.04 dan Kali Linux x86_64 serta menyiapkan OpenCode, RedteamAgent, VulnVerify Skills, Trail of Bits Skills terpilih, Xalgorix, dan Playwright MCP.

Jangan memasukkan target ke agent atau scanner sebelum menerima scope, akun dummy, batas request, dan stop condition dari instruktur.

## Isi repository

```text
materi-praktek/
├── sesi1/
│   ├── README.md
│   ├── CONTOH_LAPORAN_VULNVERIFY.md
│   └── CONTOH_LAPORAN_VULNVERIFY.pdf
└── sesi2/
    ├── README.md
    ├── ProcureFlow.zip
    ├── SHA256SUMS.txt
    ├── ASSESSMENT_RULES.md
    ├── REPORT_TEMPLATE.docx
    ├── REPORT_TEMPLATE.md
    ├── PROSEDUR_SUBMISSION.md
    └── SUBMISSION_TEMPLATE/
```

## Urutan penggunaan

1. Sebelum kelas, selesaikan instalasi dari repository `prektek-lab`.
2. Pada Sesi 1, gunakan contoh laporan untuk mempelajari exact claim, evidence, kontrol, dan verdict VulnVerify.
3. Gunakan hanya target Sesi 1 yang disebutkan instruktur.
4. Jangan membuka atau menganalisis `ProcureFlow.zip` sebelum instruktur memulai Sesi 2.
5. Pada Sesi 2, verifikasi checksum paket, baca aturan assessment, jalankan ProcureFlow secara lokal, lalu isi laporan dan struktur submission.

## Batasan distribusi

Repository ini tidak memuat slide presentasi, answer key, lokasi vulnerability, runbook instruktur, credential VPS, atau file development materi. `ProcureFlow.zip` adalah paket source peserta dan sengaja berisi aplikasi rentan. Jalankan hanya pada perangkat atau VM lab milik peserta.
