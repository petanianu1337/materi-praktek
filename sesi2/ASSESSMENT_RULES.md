# Aturan Assessment ProcureFlow

## Format

- Assessment dikerjakan secara individual selama empat jam.
- Peserta memakai versi OpenCode, model, system prompt, tool permission, dan budget yang ditentukan instruktur.
- Akses Internet dinonaktifkan selama assessment setelah image dan dependency lab tersedia.
- Peserta menyerahkan laporan berdasarkan `REPORT_TEMPLATE.docx` atau versi Markdown `REPORT_TEMPLATE.md`, beserta transcript AI sesuai arahan instruktur.
- Source aplikasi tidak boleh diubah. Rekomendasi patch dan regression test ditulis dalam laporan.

## Aktivitas yang diizinkan

Peserta boleh:

- membaca dan mencari seluruh source dalam paket peserta;
- menjalankan Docker Compose dan perintah lokal yang diperlukan aplikasi;
- menggunakan browser, `curl`, dan tool bawaan sistem untuk menguji instance sendiri;
- membaca log container;
- memeriksa data dummy untuk memahami state aplikasi;
- membuat file laporan dan evidence di luar source aplikasi.

Pembacaan database secara langsung boleh digunakan untuk memahami state, tetapi bukan bukti bahwa actor aplikasi dapat mengakses data tersebut. Dampak keamanan harus dibuktikan melalui interface aplikasi yang dapat dicapai actor.

Peserta tidak boleh:

- menguji host, container, atau alamat selain milik lab ProcureFlow;
- membuka akses aplikasi ke Internet atau jaringan bersama;
- mengubah source untuk membuat dugaan kerentanan dapat dieksploitasi;
- menggunakan credential atau data di luar deterministic seed;
- memasang scanner tambahan yang tidak disediakan instruktur;
- meminta jawaban dari peserta lain;
- menghapus atau mengubah transcript AI yang diwajibkan.

## Scope penilaian

Yang dinilai:

- root cause pada first-party source;
- authorization, tenant context, dan object access;
- stored data flow dan rendering context;
- background worker dan job processing;
- network destination handling;
- state transition dan business logic;
- hubungan antar-root cause dalam attack chain.

Yang tidak dihitung sebagai temuan:

- advisory dependency;
- password dan credential demo yang tercantum dalam README;
- HTTP lokal tanpa TLS;
- tidak adanya rate limiting;
- header generik yang tidak menghasilkan dampak tambahan;
- konfigurasi development yang diperlukan oleh lab;
- denial of service;
- beberapa endpoint yang berasal dari satu root cause yang sama.

## Standar evidence

Setiap temuan harus menyertakan:

1. actor, role, tenant, objek, dan prasyarat;
2. file, fungsi, serta baris source yang relevan;
3. source-to-sink atau state-transition trace;
4. langkah reproduksi dari deterministic seed;
5. request, response, screenshot, DOM marker, log, atau artifact yang menunjukkan hasil;
6. hasil yang seharusnya terjadi;
7. dampak yang dibatasi sesuai bukti;
8. rekomendasi perbaikan dan regression test;
9. confidence serta kondisi yang dapat membuat dugaan menjadi false positive.

Output AI tanpa verifikasi source dan runtime tidak memenuhi standar evidence.

## Penilaian

| Area                                      | Bobot |
| ----------------------------------------- | ----: |
| Temuan valid dan cakupan root cause       |   35% |
| Precision dan false-positive control      |   25% |
| Kualitas evidence source dan runtime      |   20% |
| Rekomendasi perbaikan dan regression test |   10% |
| Kejelasan laporan dan batasan analisis    |   10% |

Peserta tidak diberi penalti karena menolak hipotesis AI yang salah. Catat hipotesis tersebut pada bagian false positive untuk menunjukkan proses verifikasi.
