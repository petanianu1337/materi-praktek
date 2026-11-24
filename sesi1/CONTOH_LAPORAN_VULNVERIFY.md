# Contoh Laporan Verifikasi Lima Finding dengan VulnVerify

> **DATA SINTETIS UNTUK PEMBELAJARAN.** Seluruh nama aplikasi, host, endpoint, akun, request, response, lokasi source, timestamp, dan hasil pada dokumen ini adalah contoh buatan. Dokumen ini bukan hasil pengujian terhadap LabVulnerDursGo, VulnLab, ProcureFlow, atau sistem nyata.

## 1. Tujuan dokumen

Dokumen ini menunjukkan bentuk laporan setelah finding yang sudah ada diperiksa dengan workflow:

```text
vulnerability-triage-brocards
  -> fp-check standard atau deep
  -> poc-builder bila diperlukan dan diizinkan
  -> fp-check final review
  -> Confirmed / False Positive / Inconclusive
```

VulnVerify bukan scanner. Lima item di bawah dimulai sebagai klaim yang harus diuji, bukan lima vulnerability yang otomatis benar. Variasi verdict sengaja digunakan agar peserta tidak memaksa seluruh alert menjadi `Confirmed`.

## 2. Konteks assessment sintetis

| Item | Nilai contoh |
| --- | --- |
| Aplikasi | `Example Procurement Lab` |
| Target | `http://lab.invalid` |
| Source revision | `training-synthetic-r1` |
| Waktu pengujian | `2026-11-22 09:00-11:30 Asia/Jakarta` |
| Actor | Akun dummy `user-a`, `user-b`, dan `admin-a` |
| Scope | Host sintetis, lima endpoint yang disebutkan, dan source latihan |
| Diizinkan | Request minimum, browser test, pembacaan source, data dummy |
| Dilarang | DoS, brute force, persistence, callback publik, host lain |
| Stop condition | Berhenti setelah outcome dan satu kontrol tercatat, atau saat aplikasi tidak stabil |

Host `.invalid` memang digunakan agar contoh tidak menunjuk target Internet nyata. Request dan response tidak boleh dijalankan tanpa mengganti seluruh konteks dengan lab yang benar-benar diizinkan.

## 3. Ringkasan hasil

| ID | Klaim awal | Route | Status PoC | Verdict |
| --- | --- | --- | --- | --- |
| VV-EX-01 | IDOR pada invoice milik user lain | Standard | Executed, demonstrated | **Confirmed** |
| VV-EX-02 | SQL injection pada pencarian vendor | Standard | Executed, refuted | **False Positive** |
| VV-EX-03 | Stored XSS pada catatan invoice | Standard | Executed, demonstrated | **Confirmed** |
| VV-EX-04 | SSRF melalui worker preview dokumen | Deep | Executed, demonstrated | **Confirmed** |
| VV-EX-05 | Dependency lama memungkinkan insecure deserialization | Deep | Blocked, unresolved | **Inconclusive** |

Severity tidak ditentukan dalam contoh ini. Verdict teknis menjawab apakah exact claim terbukti; severity adalah langkah penilaian risiko yang terpisah.

## 4. Legenda evidence

| Label | Arti |
| --- | --- |
| `REPORTED` | Berasal dari finding atau artifact input |
| `OBSERVED` | Diamati langsung selama verifikasi contoh |
| `DERIVED` | Disimpulkan secara traceable dari source atau konfigurasi yang dibaca |
| `HYPOTHESIZED` | Penjelasan yang belum dibuktikan |

---

## 5. VV-EX-01: Broken Object-Level Authorization pada Invoice

### A. Identitas dan exact claim

| Field | Nilai sintetis |
| --- | --- |
| Component | Invoice API |
| Endpoint | `GET /api/invoices/{invoiceId}` |
| Parameter | `invoiceId` |
| Actor | User terautentikasi `user-a` |
| Protected resource | Invoice `inv-b-200`, milik `user-b` |
| Prerequisite | Session valid milik `user-a`; ID invoice korban diketahui |
| Expected security rule | User hanya dapat membaca invoice miliknya atau invoice yang secara eksplisit dibagikan |
| Exact claim | `user-a` dapat mengganti ID invoice miliknya menjadi ID milik `user-b` dan menerima detail invoice korban |
| Claimed impact | Pelanggaran confidentiality lintas user |

### B. Intake dan triage tujuh brocards

| Brocard | Hasil | Alasan ringkas | Locator sintetis |
| --- | --- | --- | --- |
| Threat model | PASS | Actor, object owner, dan aturan akses jelas | `input/findings.md#VV-EX-01` |
| Capability baru | PASS | Actor diduga memperoleh data user lain | `input/findings.md#VV-EX-01-impact` |
| Actual usage | PASS | Endpoint dipakai aplikasi untuk melihat invoice | `synthetic-src/invoices.ts:41-53` |
| Standard behavior | PASS | Object lookup bukan alasan mengabaikan ownership | `policy/access-control.md#invoice` |
| Documented behavior | PASS | Dokumentasi menyatakan invoice bersifat private | `docs/invoices.md#visibility` |
| Cure proportionality | NEEDS-MORE-INFO | Biaya dan dampak deployment remediasi tidak disuplai; gap prioritas ini tidak membatalkan review teknis | `input/context.md#remediation` |
| Report independence | PASS | Klaim tetap dapat diuji tanpa bergantung pada label IDOR | `input/findings.md#VV-EX-01-evidence` |

**Keputusan triage:** `ACCEPT` untuk verifikasi teknis.

### C. Route dan data flow

**Route:** Standard. Klaim spesifik, satu endpoint, dua role setara, dan tidak memerlukan concurrency.

```text
attacker-controlled invoiceId
  -> route parameter
  -> invoice lookup
  -> response serializer
  -> detail invoice dikirim ke session user-a
```

### D. Evidence ledger

| ID | Provenance | Locator sintetis | Apa yang dibuktikan | Batasan |
| --- | --- | --- | --- | --- |
| E01-1 | REPORTED | `input/findings.md#VV-EX-01` | Klaim awal dan endpoint | Belum membuktikan runtime |
| E01-2 | DERIVED | `synthetic-src/invoices.ts:41-53` | Lookup memakai ID; guard hanya memeriksa login | Hanya revision contoh |
| E01-3 | OBSERVED | `evidence/VV-EX-01/positive.txt` | `user-a` dapat membaca invoice miliknya | Membuktikan setup, bukan bypass |
| E01-4 | OBSERVED | `evidence/VV-EX-01/cross-user.txt` | Session yang sama menerima invoice `user-b` | Hanya object yang diuji |
| E01-5 | OBSERVED | `evidence/VV-EX-01/missing-id.txt` | ID acak menghasilkan `404` | Kontrol pembanding terbatas |

### E. PoC dan kontrol

**Status:** planned -> created -> executed -> demonstrated.

Positive setup:

```http
GET /api/invoices/inv-a-100 HTTP/1.1
Host: lab.invalid
Cookie: session=<REDACTED_USER_A>
```

Observed: `200`, response berisi `ownerId: user-a` dan invoice milik actor.

Trigger:

```http
GET /api/invoices/inv-b-200 HTTP/1.1
Host: lab.invalid
Cookie: session=<REDACTED_USER_A>
```

Observed: `200`, response berisi `ownerId: user-b`, nama vendor dummy, dan total invoice.

Negative control:

```http
GET /api/invoices/inv-does-not-exist HTTP/1.1
Host: lab.invalid
Cookie: session=<REDACTED_USER_A>
```

Observed: `404`. Ini menunjukkan trigger memakai object yang valid, bukan halaman error yang salah dibaca sebagai data.

### F. Enam gates

| Gate | Status | Evidence | Rationale |
| --- | --- | --- | --- |
| Process | PASS | P01, E01-1 sampai E01-5 | Standard route, required reads, adversarial review, PoC substeps, dan kontrol tercatat |
| Reachability | PASS | E01-2, E01-4 | Actor mengontrol ID dan mencapai lookup |
| Real Impact | PASS | E01-4 | Data object milik user lain dikembalikan |
| PoC Validation | PASS | E01-3 sampai E01-5 | Setup valid, trigger berbeda, kontrol error tersedia |
| Math Bounds | N/A | E01-1 | Klaim bukan integer overflow atau bounds issue |
| Environment | PASS | E01-2, E01-4 | Revision, session, role, dan object cocok |

### G. Verdict

**Confirmed.** Evidence runtime menunjukkan user terautentikasi dapat membaca object user lain, dan source trace pada revision sintetis tidak menemukan pemeriksaan ownership sebelum response. Kesimpulan dibatasi pada endpoint dan revision yang diuji; belum membuktikan semua endpoint invoice terdampak.

**Remediasi dan regression test:** ikat query pada `invoiceId` dan principal/tenant yang aktif, lalu tambahkan test yang memastikan invoice sendiri `200`, invoice user lain `403` atau `404`, dan invoice tidak ada `404`.

---

## 6. VV-EX-02: Kandidat SQL Injection pada Pencarian Vendor

### A. Identitas dan exact claim

| Field | Nilai sintetis |
| --- | --- |
| Component | Vendor search API |
| Endpoint | `GET /api/vendors?q={query}` |
| Actor | User terautentikasi biasa |
| Prerequisite | Session valid |
| Expected security rule | Input pencarian tidak boleh mengubah struktur query SQL |
| Exact claim | Nilai `q` dengan karakter quote mengubah query dan memungkinkan bypass filter |
| Claimed impact | Pembacaan data vendor di luar hasil pencarian normal |

### B. Intake dan triage tujuh brocards

| Brocard | Hasil | Alasan ringkas | Locator sintetis |
| --- | --- | --- | --- |
| Threat model | PASS | Input, sink yang diduga, dan data target jelas | `input/scanner.json#/alerts/7` |
| Capability baru | NEEDS-MORE-INFO | Alert belum menunjukkan hasil di luar pencarian normal | `input/scanner.json#/alerts/7/evidence` |
| Actual usage | PASS | Parameter digunakan oleh fitur pencarian | `synthetic-src/vendors.ts:28-39` |
| Standard behavior | PASS | Quote literal boleh dicari; yang diuji adalah perubahan struktur SQL | `docs/vendor-search.md#query` |
| Documented behavior | PASS | Tidak ada fitur query language yang didokumentasikan | `docs/vendor-search.md` |
| Cure proportionality | NEEDS-MORE-INFO | Biaya remediasi belum relevan sebelum affectedness terbukti dan tidak disuplai | `input/context.md#remediation` |
| Report independence | NEEDS-MORE-INFO | Klaim awal hanya didasarkan pada pattern scanner | `input/scanner.json#/alerts/7` |

**Keputusan triage:** `ACCEPT` untuk focused verification. Alert cukup spesifik untuk diuji, tetapi belum membuktikan vulnerability.

### C. Route dan data flow

**Route:** Standard. Satu parameter dan satu query handler.

```text
q
  -> trim dan length check
  -> SQL placeholder $1
  -> database driver binding
  -> daftar vendor
```

### D. Evidence ledger

| ID | Provenance | Locator sintetis | Apa yang dibuktikan | Batasan |
| --- | --- | --- | --- | --- |
| E02-1 | REPORTED | `input/scanner.json#/alerts/7` | Scanner menandai string construction di sekitar query | Pattern bukan data-flow proof |
| E02-2 | DERIVED | `synthetic-src/vendors.ts:28-39` | Query memakai placeholder `$1` dan parameter terpisah | Berlaku pada revision contoh |
| E02-3 | OBSERVED | `evidence/VV-EX-02/normal.txt` | Query normal mengembalikan satu vendor | Setup valid |
| E02-4 | OBSERVED | `evidence/VV-EX-02/quote.txt` | Quote diperlakukan literal, hasil kosong | Tidak menguji semua encoding |
| E02-5 | OBSERVED | `evidence/VV-EX-02/boolean.txt` | Boolean-like input tidak memperluas hasil | Hanya payload minimum |
| E02-6 | OBSERVED | `evidence/VV-EX-02/db-log.txt` | Driver mengirim statement dan bind value terpisah | Log sintetis dari instance yang sama |

### E. PoC dan kontrol

**Status:** planned -> created -> executed -> refuted.

Positive setup: `q=Acme` menghasilkan satu record `Acme Demo Ltd`.

Claimed trigger: `q=%27` menghasilkan `200` dengan array kosong; tidak ada SQL error.

Boolean comparison:

- `q=nonexistent-value` -> array kosong;
- `q=nonexistent-value%27%20OR%20%271%27%3D%271` -> array kosong;
- jumlah hasil, status, dan bentuk response sama.

Negative control: request tanpa `q` ditolak `400`, sehingga handler dan validasi yang benar benar-benar terpakai.

Source dan log menunjukkan value dikirim melalui bind parameter, bukan digabung ke string query.

### F. Enam gates

| Gate | Status | Evidence | Rationale |
| --- | --- | --- | --- |
| Process | PASS | P02, E02-1 sampai E02-6 | Standard route, required reads, adversarial review, refutation, dan kontrol tercatat |
| Reachability | REFUTED | E02-2, E02-6 | Input mencapai nilai bind, bukan grammar SQL |
| Real Impact | REFUTED | E02-4, E02-5 | Tidak ada bypass, disclosure, atau perubahan query |
| PoC Validation | PASS | E02-3 sampai E02-5 | Setup dan perbandingan terkontrol dijalankan |
| Math Bounds | N/A | E02-1 | Klaim bukan bounds issue |
| Environment | PASS | E02-2, E02-6 | Source dan log berasal dari revision/instance yang sama |

### G. Verdict

**False Positive.** Evidence afirmatif membantah syarat penting klaim: input tidak masuk ke grammar SQL, tetapi terikat sebagai value parameter. Verdict ini hanya berlaku pada handler dan revision sintetis yang diuji; endpoint lain tidak ikut dinilai.

Tidak diperlukan remediasi security untuk klaim ini. Pertahankan parameterized query dan tambahkan regression test bila alert yang sama sering muncul.

---

## 7. VV-EX-03: Stored XSS pada Catatan Invoice

### A. Identitas dan exact claim

| Field | Nilai sintetis |
| --- | --- |
| Component | Invoice notes dan halaman review admin |
| Entry point | `POST /api/invoices/{id}/notes` |
| Render path | `GET /admin/invoices/{id}` |
| Actor | `user-a` membuat catatan; `admin-a` membuka review |
| Prerequisite | Kedua akun dummy valid; invoice berada dalam workflow review |
| Expected security rule | Teks user tidak boleh dieksekusi sebagai script pada origin aplikasi |
| Exact claim | Catatan disimpan lalu dirender sebagai HTML tanpa encoding sehingga JavaScript berjalan pada browser admin |
| Claimed impact | Eksekusi script pada origin aplikasi dalam session reviewer |

### B. Intake dan triage tujuh brocards

| Brocard | Hasil | Alasan ringkas | Locator sintetis |
| --- | --- | --- | --- |
| Threat model | PASS | Writer dan viewer berbeda; browser boundary jelas | `input/findings.md#VV-EX-03` |
| Capability baru | PASS | User diduga mengeksekusi script pada session reviewer | `input/findings.md#VV-EX-03-impact` |
| Actual usage | PASS | Admin membuka halaman review pada workflow normal | `docs/review-workflow.md#notes` |
| Standard behavior | PASS | Catatan didefinisikan sebagai plain text, bukan trusted HTML | `policy/content-types.md#invoice-note` |
| Documented behavior | PASS | Dokumentasi menyatakan note berupa plain text | `docs/invoices.md#notes` |
| Cure proportionality | NEEDS-MORE-INFO | Biaya dan compatibility impact output encoding tidak disuplai | `input/context.md#remediation` |
| Report independence | PASS | Delivery path dan observable effect dapat diuji | `input/findings.md#VV-EX-03-evidence` |

**Keputusan triage:** `ACCEPT`.

### C. Route dan data flow

**Route:** Standard dengan browser verification. Alurnya dua request tetapi tidak ambigu dan tidak memakai concurrency.

```text
note body dari user-a
  -> penyimpanan database
  -> template admin
  -> HTML parser browser admin-a
  -> observable DOM marker
```

### D. Evidence ledger

| ID | Provenance | Locator sintetis | Apa yang dibuktikan | Batasan |
| --- | --- | --- | --- | --- |
| E03-1 | REPORTED | `input/findings.md#VV-EX-03` | Klaim stored XSS dan alur writer/viewer | Belum membuktikan delivery |
| E03-2 | DERIVED | `synthetic-src/admin-view.ts:73` | Template memakai raw HTML helper | Hanya revision contoh |
| E03-3 | OBSERVED | `evidence/VV-EX-03/save.json` | Marker disimpan melalui form yang diklaim | Payload aman untuk lab |
| E03-4 | OBSERVED | `evidence/VV-EX-03/browser-snapshot.txt` | DOM marker muncul pada halaman admin | Tidak membuktikan dampak selain marker |
| E03-5 | OBSERVED | `evidence/VV-EX-03/console.txt` | JavaScript marker dieksekusi satu kali | Session/data sudah disanitasi |
| E03-6 | OBSERVED | `evidence/VV-EX-03/encoded-control.txt` | Catatan benign ditampilkan sebagai teks | Kontrol pembanding minimum |

### E. PoC dan kontrol

**Status:** planned -> created -> executed -> demonstrated.

Playwright MCP digunakan pada browser dan origin yang diizinkan. Trigger dimasukkan melalui form aplikasi, bukan melalui developer console.

Payload marker sintetis:

```html
<img src=x onerror="document.body.dataset.vvExample='executed'">
```

Expected: jika input dirender sebagai markup, browser menetapkan `data-vv-example="executed"`.

Observed: setelah `admin-a` membuka halaman review, DOM memiliki marker tersebut dan console mencatat satu event. Tidak ada pembacaan cookie atau pengiriman data keluar.

Positive setup: catatan teks `TRAINING-NOTE-123` tersimpan dan terlihat pada invoice yang benar.

Negative control: string `&lt;img src=x&gt;` tetap terlihat sebagai teks dan tidak membuat element `img`.

Stop condition terpenuhi setelah satu eksekusi marker dan satu negative control.

### F. Enam gates

| Gate | Status | Evidence | Rationale |
| --- | --- | --- | --- |
| Process | PASS | P03, E03-1 sampai E03-6 | Standard route, browser branch, adversarial review, dan kontrol tercatat |
| Reachability | PASS | E03-3 sampai E03-5 | Trigger melewati form, storage, dan render path |
| Real Impact | PASS | E03-5 | Script berjalan pada origin dan session reviewer |
| PoC Validation | PASS | E03-3 sampai E03-6 | Setup, trigger, observable marker, dan kontrol konsisten |
| Math Bounds | N/A | E03-1 | Bukan bounds issue |
| Environment | PASS | E03-2, E03-4 | Revision, origin, account, dan halaman cocok |

### G. Verdict

**Confirmed.** Payload yang dikirim melalui entry point aplikasi disimpan dan dieksekusi pada halaman reviewer. Kesimpulan membuktikan script execution dengan marker aman; laporan tidak mengklaim pencurian cookie, account takeover, atau impact lain yang tidak diuji.

**Remediasi dan regression test:** render note sebagai text dengan contextual output encoding, hindari raw HTML helper, dan tambahkan browser test yang memastikan payload menjadi text node serta marker tidak pernah terbentuk.

---

## 8. VV-EX-04: SSRF melalui Worker Preview Dokumen

### A. Identitas dan exact claim

| Field | Nilai sintetis |
| --- | --- |
| Component | Web API, preview queue, background worker |
| Entry point | `POST /api/previews` dengan field `sourceUrl` |
| Actor | User terautentikasi biasa |
| Protected resource | Internal metadata endpoint sintetis `http://metadata.internal.invalid/status` |
| Prerequisite | Worker aktif; job queue dapat diproses; host internal hanya dapat diakses worker |
| Expected security rule | User tidak boleh membuat worker mengambil resource dari private/internal network |
| Exact claim | Redirect dari URL eksternal yang diizinkan tidak divalidasi ulang, sehingga worker mengikuti redirect ke host internal dan menyimpan response sebagai preview |
| Claimed impact | Pembacaan response service internal melalui aplikasi |

### B. Intake dan triage tujuh brocards

| Brocard | Hasil | Alasan ringkas | Locator sintetis |
| --- | --- | --- | --- |
| Threat model | PASS | Actor, worker boundary, dan internal resource jelas | `input/findings.md#VV-EX-04` |
| Capability baru | PASS | Actor diduga memakai egress worker untuk membaca host internal | `input/findings.md#VV-EX-04-impact` |
| Actual usage | PASS | Preview URL adalah fitur yang digunakan | `docs/preview.md#workflow` |
| Standard behavior | PASS | Redirect HTTP standar tidak membenarkan akses private network | `policy/network-boundary.md#worker` |
| Documented behavior | PASS | Dokumentasi hanya mengizinkan public HTTPS document | `docs/preview.md#allowed-source` |
| Cure proportionality | NEEDS-MORE-INFO | Biaya perubahan redirect dan egress policy tidak disuplai | `input/context.md#remediation` |
| Report independence | PASS | Chain dapat diuji dengan redirect dan direct control | `input/findings.md#VV-EX-04-evidence` |

**Keputusan triage:** `ACCEPT`.

### C. Route dan data flow

**Route:** Deep. Klaim melintasi web API, queue, worker, DNS/redirect handling, dan internal service.

```text
sourceUrl dari user
  -> API hanya memeriksa URL awal
  -> queue job
  -> worker fetch
  -> redirect 302
  -> destination internal tanpa validasi ulang
  -> response internal disimpan
  -> preview dapat dibaca user
```

### D. Evidence ledger

| ID | Provenance | Locator sintetis | Apa yang dibuktikan | Batasan |
| --- | --- | --- | --- | --- |
| E04-1 | REPORTED | `input/findings.md#VV-EX-04` | Klaim redirect-based SSRF | Belum membuktikan worker path |
| E04-2 | DERIVED | `synthetic-src/preview-route.ts:31-48` | URL awal divalidasi sebelum enqueue | Tidak membuktikan network behavior |
| E04-3 | DERIVED | `synthetic-src/preview-worker.ts:62-84` | Client mengikuti redirect; tidak ada per-hop policy | Library behavior harus dikorelasikan runtime |
| E04-4 | OBSERVED | `evidence/VV-EX-04/public-control.txt` | Public document menghasilkan preview yang benar | Setup dan worker valid |
| E04-5 | OBSERVED | `evidence/VV-EX-04/direct-private.txt` | Direct private URL ditolak API | Membuktikan initial validation aktif |
| E04-6 | OBSERVED | `evidence/VV-EX-04/redirect-chain.txt` | URL publik -> 302 -> internal, lalu job selesai | Hanya chain sintetis |
| E04-7 | OBSERVED | `evidence/VV-EX-04/preview-result.txt` | Preview berisi marker khusus internal service | Marker tidak mengandung secret nyata |
| E04-8 | OBSERVED | `evidence/VV-EX-04/worker-log.txt` | Worker mencatat final destination internal | Log dari instance yang sama |

### E. PoC dan kontrol

**Status:** planned -> created -> executed -> demonstrated.

Positive setup: URL public dummy tanpa redirect menghasilkan preview `PUBLIC-DOCUMENT-MARKER`.

Direct negative control: pengiriman `http://metadata.internal.invalid/status` ditolak `400 private destination`.

Trigger: URL public dummy yang berada dalam scope membalas `302 Location: http://metadata.internal.invalid/status`.

Observed:

- API menerima URL awal dan membuat job;
- worker mengikuti redirect;
- hasil preview memuat `INTERNAL-STATUS-MARKER`;
- worker log menunjukkan final destination internal;
- tidak ada request ke Internet atau host di luar jaringan lab sintetis.

Stop condition terpenuhi setelah satu chain dan satu direct-private control.

### F. Enam gates

| Gate | Status | Evidence | Rationale |
| --- | --- | --- | --- |
| Process | PASS | P04, E04-1 sampai E04-8 | Deep route, dependent phases, required reads, adversarial review, dan kontrol dicatat |
| Reachability | PASS | E04-5 sampai E04-8 | Actor mengontrol URL awal dan redirect mencapai internal service |
| Real Impact | PASS | E04-7 | Response internal kembali melalui preview |
| PoC Validation | PASS | E04-4 sampai E04-8 | Setup, direct block, redirect trigger, output, dan log konsisten |
| Math Bounds | N/A | E04-1 | Bukan bounds issue |
| Environment | PASS | E04-2, E04-3, E04-8 | API, worker, network, dan revision cocok |

### G. Verdict

**Confirmed.** Evidence source dan runtime bersama-sama menunjukkan initial URL check dapat dilewati melalui redirect karena destination berikutnya tidak divalidasi ulang. Kesimpulan dibatasi pada preview worker dan redirect behavior yang diuji; laporan tidak mengklaim akses ke cloud metadata nyata atau jaringan produksi.

**Remediasi dan regression test:** resolve dan validasi destination pada setiap redirect hop, tolak private/link-local/loopback ranges setelah resolution, batasi egress worker, dan tambahkan test untuk direct URL, redirect satu hop, multi-hop, serta DNS result yang berubah.

---

## 9. VV-EX-05: Klaim Insecure Deserialization pada Dependency Historis

### A. Identitas dan exact claim

| Field | Nilai sintetis |
| --- | --- |
| Component | Import worker historis |
| Affected version yang dilaporkan | `Example Procurement Lab 0.8.0` |
| Version yang tersedia | `1.2.0` |
| Actor | User dengan izin mengunggah file import |
| Prerequisite | Build 0.8.0, dependency rentan, handler import aktif, payload mencapai deserializer |
| Expected security rule | File import tidak boleh memicu object construction atau code execution yang tidak diizinkan |
| Exact claim | Versi 0.8.0 memakai dependency deserialization rentan sehingga file import dapat memicu code execution pada worker |
| Claimed impact | Remote code execution pada worker |

### B. Intake dan triage tujuh brocards

| Brocard | Hasil | Alasan ringkas | Locator sintetis |
| --- | --- | --- | --- |
| Threat model | PASS | Actor, upload path, worker, dan claimed impact disebutkan | `input/findings.md#VV-EX-05` |
| Capability baru | NEEDS-MORE-INFO | Tidak ada execution evidence atau vulnerable call path | `input/findings.md#VV-EX-05-evidence` |
| Actual usage | NEEDS-MORE-INFO | Build historis dan handler tidak tersedia | `evidence/VV-EX-05/artifact-inventory.txt` |
| Standard behavior | PASS | Deserialization normal bukan bukti code execution | `input/advisory.json` |
| Documented behavior | NEEDS-MORE-INFO | Format import versi 0.8.0 tidak tersedia | `evidence/VV-EX-05/artifact-inventory.txt` |
| Cure proportionality | NEEDS-MORE-INFO | Upgrade mungkin benar, tetapi affectedness dan biaya deployment belum terbukti | `input/context.md#remediation` |
| Report independence | NEEDS-MORE-INFO | Klaim bertumpu pada advisory dan version string | `input/advisory.json` |

**Keputusan triage:** `NEEDS-MORE-INFO`, tetapi dilanjutkan ke deep review atas artifact yang tersedia.

### C. Route dan data flow

**Route:** Deep. Klaim berlaku pada historical version, dependency, dan worker path yang tidak dapat direkonstruksi secara lengkap.

```text
uploaded import file
  -> historical route UNKNOWN
  -> queue/worker UNKNOWN
  -> dependency API UNKNOWN
  -> deserialization gadget UNKNOWN
  -> claimed code execution UNKNOWN
```

### D. Evidence ledger

| ID | Provenance | Locator sintetis | Apa yang dibuktikan | Batasan |
| --- | --- | --- | --- | --- |
| E05-1 | REPORTED | `input/findings.md#VV-EX-05` | Exact historical claim | Tidak menyertakan request atau PoC |
| E05-2 | REPORTED | `input/advisory.json` | Dependency version tertentu memiliki advisory | Advisory tidak membuktikan aplikasi reachable |
| E05-3 | OBSERVED | `evidence/VV-EX-05/current-lock.txt` | Version 1.2.0 memakai dependency yang sudah diperbarui | Bukan build 0.8.0 |
| E05-4 | OBSERVED | `evidence/VV-EX-05/artifact-inventory.txt` | Image dan lockfile 0.8.0 tidak tersedia | Menjelaskan blocker |
| E05-5 | DERIVED | `synthetic-src-current/import-worker.ts:20-57` | Current path memakai parser berbeda | Tidak membuktikan historical path |
| E05-6 | HYPOTHESIZED | Catatan analyst | Build 0.8.0 mungkin memanggil vulnerable API | Belum ada source atau binary pendukung |

### E. PoC dan kontrol

**Status:** planned -> blocked -> unresolved.

PoC plan membutuhkan image atau source revision 0.8.0, lockfile yang cocok, format file import, dan akun yang dapat mencapai handler tersebut. Artifact itu tidak tersedia. Pengujian terhadap versi 1.2.0 tidak dilakukan sebagai pengganti karena hasilnya tidak dapat membuktikan atau membantah keadaan versi 0.8.0.

Positive setup: blocked karena handler historis tidak tersedia.

Negative control: tidak dijalankan karena tidak ada environment comparable. Current version bukan negative control yang sah untuk historical claim.

Tidak ada payload yang dieksekusi dan tidak ada target yang dihubungi.

### F. Enam gates

| Gate | Status | Evidence | Rationale |
| --- | --- | --- | --- |
| Process | PASS | P05, E05-1 sampai E05-6 | Deep route selesai sebatas evidence; blocker, skips, dan unknown dicatat tanpa mengarang eksekusi |
| Reachability | UNKNOWN | E05-1, E05-4 | Historical handler dan call path tidak tersedia |
| Real Impact | UNKNOWN | E05-2, E05-6 | Advisory ada, tetapi application impact belum terbukti |
| PoC Validation | UNKNOWN | E05-4 | Reproduksi tidak dapat dijalankan |
| Math Bounds | N/A | E05-1 | Bukan bounds issue |
| Environment | UNKNOWN | E05-3, E05-4 | Hanya current build yang tersedia |

### G. Verdict

**Inconclusive.** Tidak ada evidence cukup untuk mengonfirmasi reachability dan impact pada versi 0.8.0, tetapi juga tidak ada evidence afirmatif yang membantah klaim historis. Kegagalan memperoleh build bukan `False Positive`.

Evidence berikut yang dibutuhkan:

1. source, image, atau binary build 0.8.0 yang dapat diverifikasi;
2. lockfile atau SBOM dari build tersebut;
3. konfigurasi yang menunjukkan import worker aktif;
4. call path dari file upload ke API dependency yang terdampak;
5. reproduksi minimal dan kontrol pada environment terisolasi.

---

## 10. Process completeness record

Bagian ini adalah contoh ringkas receipt yang mendukung nilai `Process: PASS`. Pada laporan nyata, nama reference saja tidak cukup; agent harus benar-benar memuat isinya dan mencatat locator sesi atau tool result.

### 10.1 Reference-read receipt

| Process ID | Finding | Route | Reference bodies yang diperiksa pada contoh |
| --- | --- | --- | --- |
| P01 | VV-EX-01 | Standard | triage `SKILL.md`, `report-intake.md`, `brocards-detail.md`, fp-check `SKILL.md`, `validation-fidelity.md`, `bug-class-verification.md` bagian Logic Bugs, `standard-verification.md`, `false-positive-patterns.md`, `gate-reviews.md`, `evidence-templates.md`, poc-builder `SKILL.md` |
| P02 | VV-EX-02 | Standard | Daftar standard yang sama; bug class Injection dan evidence context source/runtime |
| P03 | VV-EX-03 | Standard + browser | Daftar standard yang sama; bug class Injection, poc-builder, dan `browser-verification.md` |
| P04 | VV-EX-04 | Deep | Seluruh daftar dasar; `deep-verification.md`, `data-flow-analysis.md`, `exploitability-analysis.md`, dan poc-builder |
| P05 | VV-EX-05 | Deep | Seluruh daftar deep; bug class Deserialization dan evidence context dependency/historical version |

### 10.2 Status substep PoC

| ID | 4.1 Sketch | 4.2 Executable | 4.3 Unit test | 4.4 Controls | 4.5 Review |
| --- | --- | --- | --- | --- | --- |
| VV-EX-01 | Complete | Executed melalui HTTP record sintetis | Skipped: runtime path dan source review cukup; source tidak diubah | Own-object setup dan cross-user trigger executed; invalid-ID hanya supplemental | Demonstrated pada object yang diuji |
| VV-EX-02 | Complete, termasuk titik block pada parameter binding | Executed melalui focused HTTP comparison | Skipped: tidak perlu menambah harness untuk refutation yang sudah lengkap | Normal query, quote, boolean comparison, dan missing-parameter control executed | Refuted pada handler yang diuji |
| VV-EX-03 | Complete | Executed melalui browser action sintetis | Skipped: browser path aktual lebih sesuai dan source tidak diubah | Benign note dan trigger melalui form executed | Demonstrated; impact dibatasi pada marker script |
| VV-EX-04 | Complete lintas API, queue, worker, dan internal service | Executed pada jaringan lab sintetis | Skipped: tidak ada isolated harness yang setara dengan network boundary | Public URL, direct-private block, dan redirect chain executed | Demonstrated untuk satu redirect chain |
| VV-EX-05 | Complete dengan node `UNKNOWN` | Blocked: build 0.8.0 tidak tersedia | Skipped: source dan harness historis tidak tersedia | Not executed; current build bukan kontrol yang comparable | Unresolved, tidak diperlakukan sebagai refutation |

### 10.3 Tujuh pertanyaan devil's advocate untuk standard route

| No. | VV-EX-01 | VV-EX-02 | VV-EX-03 |
| --- | --- | --- | --- |
| 1. Pattern bias | Tidak hanya dari pattern; cross-user response diamati | Alert awal memang pattern dan kemudian dibantah | Tidak hanya dari sink; delivery browser diamati |
| 2. Trusted data | Invoice ID dikontrol actor | `q` dikontrol actor tetapi hanya sebagai bound value | Note dikontrol `user-a` melalui form |
| 3. Math rigor | N/A, bukan bounds claim | N/A, bukan bounds claim | N/A, bukan bounds claim |
| 4. Defense-in-depth | Ownership adalah primary control | Parameter binding adalah primary control yang efektif | Output encoding adalah primary control pada context ini |
| 5. Hallucination check | Source dan role comparison selaras | Dangerous-looking alert tidak cukup; refutation dipertahankan | DOM marker dan console record selaras dengan claim sempit |
| 6. False-negative check | Kompleksitas rendah tidak memengaruhi keputusan | Klaim tidak diterima hanya karena scanner yakin | Dampak tidak dikecilkan karena payload dibuat aman |
| 7. Invented mitigation | Tidak mengasumsikan guard lain untuk endpoint yang diuji | Binding diverifikasi dari source dan log | Tidak mengasumsikan CSP atau sanitizer yang tidak terlihat |

### 10.4 Tiga belas pertanyaan devil's advocate untuk deep route

| No. | VV-EX-04 | VV-EX-05 |
| --- | --- | --- |
| 1. Penjelasan non-vulnerability | Redirect bisa aman jika tiap hop divalidasi, tetapi final internal destination diamati | Dependency dapat hadir tanpa affected path; kemungkinan ini belum terselesaikan |
| 2. Justifikasi developer | Initial URL check mungkin dimaksudkan cukup, tetapi tidak menjaga redirect hop | Current parser mungkin mengganti dependency lama, tetapi bukan evidence historis |
| 3. Context arsitektur hilang | Egress protection diperiksa pada instance contoh dan tidak mencegah outcome | Build, config, handler, dan worker historis hilang |
| 4. Pattern bias | Chain didukung output dan worker log, bukan pattern fetch semata | Advisory/version match tidak dianggap proof |
| 5. Validation mencegah? | Direct-private validation mencegah URL awal, tidak mencegah redirect | Tidak dapat ditentukan pada versi 0.8.0 |
| 6. Hidden constraints | Satu-hop chain tercatat; generalisasi multi-hop tidak dibuat | Constraints historical parser tidak diketahui |
| 7. Alternate flow | Klaim dibatasi pada preview worker yang diuji | Alternate safe/unsafe caller tetap unknown |
| 8. Environment protection | Proteksi initial URL nyata tetapi tidak mengeliminasi chain | Environment historical tidak tersedia |
| 9. Attacker control | Actor mengontrol URL awal; redirect fixture dalam scope menyediakan hop | Upload permission dilaporkan, tetapi reachability ke deserializer unknown |
| 10. Bounds | N/A, bukan bounds claim | N/A, bukan bounds claim |
| 11. Concurrency | Queue async dicatat; exploit tidak bergantung pada race | Worker lifecycle historical unknown, tetapi claim bukan race |
| 12. False-negative check | Chain tidak ditolak karena lintas komponen | Klaim tidak ditolak hanya karena build sulit diperoleh |
| 13. Invented mitigation | Tidak mengarang per-hop validation atau egress block | Tidak mengarang gadget, allowlist, atau loaded dependency |

### 10.5 False-positive checklist 1-13 dan 1a

| Item | Disposition pada lima contoh |
| --- | --- |
| 1. Full validation chain | Setiap finding memuat entry point sampai sink atau node `UNKNOWN` |
| 1a. Conditional logic | Initial URL block, SQL binding, ownership gap, rendering path, dan historical unknown dicatat terpisah |
| 2. Defensive patterns | Parameter binding VV-EX-02 dikenali efektif; initial URL check VV-EX-04 tidak digeneralisasi ke redirect |
| 3. Exploitable data path | `Confirmed` hanya diberikan saat path dan consequence ditunjukkan |
| 4. Data source context | Writer untuk ID, query, note, URL, dan import file disebutkan |
| 5. Bounds logic | Semua finding non-bounds diberi `N/A` dengan alasan |
| 6. TOCTOU | Tidak ada race claim; async queue VV-EX-04 tidak otomatis diperlakukan sebagai race |
| 7. API contract | SQL binding, browser context, HTTP redirect, dan dependency behavior diperiksa sesuai claim |
| 8. Internal versus external input | Database note ditelusuri ke writer; historical dependency reachability tetap unknown |
| 9. Pattern recognition | Scanner SQL dan advisory dependency tidak diterima sebagai proof |
| 10. Concurrent access | Tidak dibutuhkan untuk empat claim; historical worker state tetap unknown |
| 11. Real impact | Disclosure, script execution, SSRF disclosure, atau unresolved RCE dipisahkan |
| 12. Primary control | Ownership, query binding, output encoding, network policy, dan deserialization controls dinilai sesuai peran |
| 13. Rigorous application | Setiap gate, mismatch, skip, dan limitation memiliki record |

## 11. Pelajaran penilaian untuk peserta

Laporan yang baik bukan laporan dengan jumlah `Confirmed` paling banyak. Instruktur menilai apakah peserta:

- mempertahankan exact claim, actor, object, version, dan expected security rule;
- memisahkan `REPORTED`, `OBSERVED`, `DERIVED`, dan `HYPOTHESIZED`;
- tidak mengaku mengeksekusi PoC yang hanya dibuat atau direncanakan;
- menggunakan positive setup dan negative control yang comparable;
- tidak menganggap HTTP `200`, timeout, atau error sebagai bukti otomatis;
- memberikan evidence locator untuk setiap gate;
- menggunakan `False Positive` hanya jika ada refutation afirmatif;
- menggunakan `Inconclusive` ketika syarat penting masih `UNKNOWN`;
- membatasi impact pada apa yang benar-benar ditunjukkan evidence;
- menyebut evidence berikut yang diperlukan jika assessment belum selesai.

## 12. Prompt untuk meminta OpenCode mereview laporan peserta

```text
Use vulnerability-triage-brocards, fp-check, and poc-builder only where a
reproduction is necessary and authorized. Review each exact finding in this
report independently. Do not perform discovery or contact a target yet.
Check actor, role, object, version, expected security rule, evidence provenance,
execution status, positive and negative controls, and all six gates. Identify
unsupported observations, invented internal mechanisms, and environment
mismatches. Return exactly one technical verdict per claim: Confirmed, False
Positive, or Inconclusive, with evidence locators and the next evidence needed.
```

## 13. Referensi metodologi

- [VulnVerify Skills](https://github.com/roomkangali/vulnverify-skills)
- [`vulnerability-triage-brocards`](https://github.com/roomkangali/vulnverify-skills/blob/main/skills/vulnerability-triage-brocards/SKILL.md)
- [`fp-check`](https://github.com/roomkangali/vulnverify-skills/blob/main/skills/fp-check/SKILL.md)
- [`poc-builder`](https://github.com/roomkangali/vulnverify-skills/blob/main/skills/poc-builder/SKILL.md)

Contoh ini harus tetap berada di materi Sesi 1 dan tidak boleh dimasukkan ke `ProcureFlow.zip` untuk peserta Sesi 2.
