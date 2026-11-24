# Laporan Analisis Keamanan ProcureFlow

Gunakan template ini untuk melaporkan hasil analisis source code dan verifikasi pada instance ProcureFlow lokal. Buat satu bagian temuan untuk setiap root cause yang unik. Jangan menghitung beberapa endpoint sebagai temuan terpisah jika semuanya berasal dari root cause yang sama.

Jangan menguji sistem di luar instance lab milik Anda. Jangan memasukkan credential nyata, token aktif di luar lab, atau data sensitif ke dalam laporan.

## 1. Identitas peserta

| Field                           | Isi peserta         |
| ------------------------------- | ------------------- |
| Nama                            |                     |
| Tim atau kelas                  |                     |
| Tanggal analisis                |                     |
| Durasi analisis                 |                     |
| Versi atau checksum ProcureFlow |                     |
| Sistem operasi                  |                     |
| Versi Docker                    |                     |
| AI agent                        | OpenCode / lainnya: |
| Model dan versi                 |                     |
| Konfigurasi penting             |                     |
| Akses Internet selama analisis  | Aktif / nonaktif    |
| Lokasi transcript AI            |                     |

## 2. Validasi lingkungan lokal

Catat bahwa artifact yang dianalisis dapat dijalankan sebelum melakukan pengujian keamanan.

- Perintah start:
- URL lokal:
- Container yang aktif:
- Hasil health check:
- Akun atau role yang digunakan:
- Kendala lingkungan:

Lampirkan output singkat yang membuktikan aplikasi sehat. Jangan menempelkan seluruh build log jika tidak relevan.

## 3. Ringkasan hasil

| ID   | Judul temuan | Kategori atau CWE | Severity | Confidence | Status bukti runtime      | Bagian dari chain |
| ---- | ------------ | ----------------- | -------- | ---------- | ------------------------- | ----------------- |
| F-01 |              |                   |          |            | Terbukti / belum terbukti |                   |

Jumlah root cause unik:

Jumlah hipotesis yang ditolak atau false positive:

Ringkasan dampak tertinggi:

## 4. Metode analisis

Jelaskan secara singkat:

- cara AI agent digunakan untuk memetakan source;
- module, route, worker, atau data flow yang diperiksa;
- cara temuan source divalidasi pada runtime;
- batasan waktu, tool, akses, atau bagian source yang belum diperiksa.

## 5. Detail temuan

Salin bagian berikut untuk setiap root cause. Gunakan ID laporan Anda sendiri seperti `F-01`, `F-02`, dan seterusnya.

### F-XX: Judul temuan

#### Klasifikasi

| Field                     | Isi peserta                    |
| ------------------------- | ------------------------------ |
| Kategori atau CWE         |                                |
| Severity                  | Critical / High / Medium / Low |
| Confidence                | High / Medium / Low            |
| Actor atau role penyerang |                                |
| Tenant atau konteks objek |                                |
| Prasyarat                 |                                |
| Bagian dari chain         | Ya / tidak; jelaskan kaitannya |

#### Root cause

Jelaskan invariant atau pemeriksaan keamanan yang hilang, salah, atau diterapkan pada tahap yang tidak tepat. Bedakan root cause dari gejala dan endpoint yang terdampak.

#### Lokasi source

| Urutan | File dan baris | Fungsi atau module | Peran dalam aliran                     |
| -----: | -------------- | ------------------ | -------------------------------------- |
|      1 |                |                    | Source / entry point                   |
|      2 |                |                    | Validation / authorization / transform |
|      3 |                |                    | Storage / queue / sink                 |

Sertakan potongan kode minimum yang diperlukan. Jelaskan mengapa kode tersebut dapat dieksploitasi. Potongan kode tanpa penjelasan tidak dianggap sebagai evidence lengkap.

```text
Tempel potongan kode relevan di sini.
```

#### Source-to-sink atau state-transition trace

Tuliskan aliran secara berurutan. Contoh format:

```text
Input actor
  -> route
  -> authorization decision
  -> repository atau storage
  -> worker atau renderer
  -> security impact
```

Untuk business logic, tuliskan state awal, action, state setelah action, dan invariant yang dilanggar.

#### Langkah reproduksi lokal

Tuliskan langkah yang dapat diulang dari deterministic seed atau state awal.

1. Persiapan akun, role, tenant, dan data:
2. Request atau tindakan pertama:
3. Perubahan state yang terjadi:
4. Request atau tindakan berikutnya:
5. Hasil aktual:
6. Hasil yang seharusnya:

Sertakan request, response, screenshot, DOM marker, log, atau artifact yang relevan. Hapus session cookie atau token dari evidence yang tidak memerlukannya.

#### Evidence runtime

- Nama file evidence:
- Waktu pengujian:
- Request atau tindakan:
- Response atau perubahan state:
- Marker keberhasilan:
- Cara mengulang evidence:

#### Dampak

Jelaskan data atau aksi yang dapat diakses actor, tenant yang terdampak, dan batas dampak yang sudah terbukti. Jangan mengklaim dampak yang belum diverifikasi.

#### Rekomendasi perbaikan

Jelaskan pemeriksaan atau perubahan desain yang harus dilakukan dan module tempat invariant seharusnya ditegakkan.

#### Regression test yang diusulkan

Tuliskan kondisi awal, action, dan hasil aman yang diharapkan. Test harus gagal pada source rentan dan lulus setelah perbaikan.

#### Batasan dan alternatif penjelasan

Tuliskan bagian yang belum dibuktikan, asumsi yang dipakai, dan kondisi yang dapat membuat temuan menjadi false positive.

## 6. Attack chain

Isi bagian ini jika dua atau lebih root cause dapat digabungkan.

```text
Actor awal
  -> temuan pertama
  -> capability atau data baru
  -> temuan berikutnya
  -> dampak akhir yang terbukti
```

Jelaskan mengapa setiap link diperlukan. Bedakan capability yang benar-benar diperoleh dari capability yang hanya diasumsikan.

## 7. Hipotesis yang ditolak dan false positive

Bagian ini digunakan untuk menunjukkan bahwa peserta memverifikasi hasil AI agent sebelum melaporkannya sebagai kerentanan.

| ID   | Lokasi atau dugaan awal | Alasan terlihat rentan | Pemeriksaan yang dilakukan | Alasan ditolak |
| ---- | ----------------------- | ---------------------- | -------------------------- | -------------- |
| R-01 |                         |                        |                            |                |

## 8. Catatan penggunaan AI

Ringkas bagaimana peserta mengarahkan dan memeriksa AI agent:

- prompt atau strategi awal;
- pertanyaan lanjutan yang membantu menemukan aliran lintas module;
- klaim AI yang ditolak setelah pemeriksaan source atau runtime;
- perubahan pendekatan setelah mendapatkan evidence;
- bagian laporan yang diverifikasi secara manual.

Lampirkan transcript sesuai aturan kelas. Jangan mengubah transcript untuk menghilangkan kegagalan analisis karena proses verifikasi juga menjadi bagian penilaian.

## 9. Daftar evidence

| ID   | Nama file | Jenis                                            | Temuan terkait | Deskripsi |
| ---- | --------- | ------------------------------------------------ | -------------- | --------- |
| E-01 |           | Screenshot / request / response / log / artifact |                |           |

## 10. Pernyataan peserta

Saya menyatakan bahwa pengujian dilakukan hanya terhadap instance ProcureFlow lokal yang diberikan untuk training. Seluruh klaim dalam laporan dibatasi pada evidence yang dicantumkan.

- Nama:
- Tanggal:
