# Traceable Tests Skill

Dokumentasi lengkap skill `traceable-tests`: metodologi pengujian 3-layer (**TS → TC → RTM**) untuk konsistensi struktur dan coverage tracking.

Instruksi yang dibaca agent ada di [`SKILL.md`](SKILL.md). Dokumen ini adalah pasangan manusianya — memuat hal yang sama plus rasional, aturan, checklist, dan metrik di baliknya.

- **Tipe:** Methodology/Process skill — language-agnostic, tidak terikat framework maupun agent
- **Format artefak:** Dokumentasi markdown + panduan struktur kode test
- **Portabilitas:** Berjalan tanpa modifikasi di semua agent pendukung [Agent Skills](https://agentskills.io)

## Daftar Isi

[Ikhtisar](#ikhtisar) · [Kapan Dipakai](#kapan-dipakai) · [Alur Kerja](#alur-kerja) · [Aturan Lintas Fase](#aturan-lintas-fase) · [7 Dimensi Coverage](#7-dimensi-coverage) · [Aturan Generate Test Case](#aturan-generate-test-case) · [Contoh Derivasi Coverage](#contoh-derivasi-coverage) · [Artefak Dokumentasi](#artefak-dokumentasi) · [Struktur Kode Test](#struktur-kode-test) · [Konvensi Penamaan](#konvensi-penamaan) · [Mode Audit](#mode-audit--gap-detection) · [Checklist Agent](#checklist-eksekusi-agent) · [Batasan & Anti-Pattern](#batasan--anti-pattern) · [Kriteria Keberhasilan](#kriteria-keberhasilan) · [Roadmap](#roadmap) · [Contoh Lengkap](#contoh-lengkap-autentikasi) · [Isi Paket](#isi-paket-skill) · [Instalasi](#instalasi) · [Lisensi](#lisensi)

## Ikhtisar

Skill ini meng-encode workflow pengujian 3-layer (TS → TC → RTM) dengan fokus pada konsistensi struktur dan pelacakan coverage. Agent dapat:

1. **Generate Test Scenario** — dari analisis fitur/domain
2. **Derive Test Cases** — dari TS dengan pendekatan fail-first
3. **Audit test existing** — parse, petakan ke struktur, laporkan gap
4. **Generate RTM** — requirement traceability untuk kebutuhan audit

Semua keluarannya adalah **dokumentasi dan struktur**, bukan eksekusi. Skill ini tidak menjalankan test dan tidak menulis kode produksi untuk meloloskan test.

## Kapan Dipakai

Invoke skill ini saat:

- User meminta "write test", "buat test case", "testing untuk fitur X"
- User meminta "audit test", "cek coverage", "review test completeness"
- User menyebut **TS**, **TC**, atau **RTM** secara eksplisit
- User menyediakan requirement doc dan meminta strategi pengujian
- User memanggil skill langsung: `/traceable-tests` atau "gunakan traceable-tests"

Pemicu yang sama juga tertulis di field `description` pada frontmatter `SKILL.md`, karena agent membaca field itulah untuk memutuskan relevansi skill.

**Jangan dipakai saat:**

- User hanya ingin menjalankan test yang sudah ada (eksekusi, bukan perancangan)
- User meminta mock/stub generation saja — di luar cakupan
- User meminta performance/load testing — domain berbeda

## Alur Kerja

Tiga fase, dengan **dua gerbang persetujuan** supaya hasilnya tetap di bawah kendali user.

| Fase | Input | Proses | Output | Gerbang |
|:---:|---|---|---|:---:|
| **1. Test Scenario** | Deskripsi fitur, requirement doc, atau kode existing | Analisis batas & tanggung jawab fitur, abstraksi jadi skenario | Daftar `TS-{DOMAIN}-{NN}` + deskripsi | Ya |
| **2. Test Case** | TS yang sudah disetujui | Derivasi TC fail-first dengan dimensi lengkap | Tabel TC: ID, langkah, expected result | Ya |
| **3. RTM & Audit** | Daftar TS + TC (+ kode test opsional) | Pemetaan requirement dan audit coverage | RTM + laporan gap | — |

### Fase 1 — Test Scenario Generation

1. Analisis batas dan tanggung jawab fitur
2. Abstraksi menjadi skenario tingkat tinggi
3. Buat ID: `TS-{DOMAIN}-{NUM}` (contoh: `TS-PRODUCT-01`, `TS-AUTH-01`)
4. Tulis deskripsi skenario berupa pernyataan cakupan yang luas

```
TS-PRODUCT-01: Kelola produk - Mengambil, menambah, edit, hapus data produk
TS-AUTH-01: Autentikasi - Proses pendaftaran akun baru, masuk ke sistem, dan keluar dari sistem
```

**Gerbang:** user meninjau daftar TS sebelum derivasi TC dimulai.

### Fase 2 — Test Case Derivation (Fail-First)

Untuk setiap TS yang disetujui:

1. Identifikasi dimensi uji (happy/error/security)
2. Generate TC dengan urutan **fail-first**: negative case dulu, happy path terakhir
3. Susun setiap TC dengan struktur:

   | Elemen | Isi |
   |---|---|
   | Scenario ID | TS induk |
   | Test Case ID | `TC-{FEATURE}-{NUM}` (contoh: `TC-LOGIN-01`) |
   | Scenario Description | Kondisi yang diuji |
   | Test Steps | Kondisi awal + langkah bernomor |
   | Expected Result | Status code + pesan eksplisit |

**Gerbang:** user meninjau kelengkapan TC sebelum artefak dibuat.

**Urutan fail-first:**

1. Empty/null cases (`TC-XXX-01` s.d. `0N`)
2. Invalid format cases
3. Boundary cases
4. Not found cases
5. Unauthorized cases
6. Forbidden cases
7. Happy path cases (terakhir)

### Fase 3 — RTM & Coverage Audit

1. Petakan requirements → TS → TC
2. Generate tabel RTM
3. Mode audit: parse test existing, identifikasi TC yang hilang, laporkan gap

**Tanpa gerbang:** ini artefak final.

## Aturan Lintas Fase

- **Coverage wajib:** 7 dimensi per grup TC
- **Fail-first:** negative/edge case dinomori lebih dulu, happy path terakhir
- **Traceability:** setiap TC harus tertaut ke TS induknya
- **Pesan error spesifik:** hindari "error occurred", gunakan pesan validasi konkret
- **Data uji konkret:** pakai contoh eksplisit (`"Valid123!"`, `"mahasiswa@mail.com"`)

## 7 Dimensi Coverage

Setiap Test Scenario wajib menurunkan Test Case yang mencakup seluruh dimensi berikut.

| No | Dimensi | Deskripsi | Contoh |
|:---:|---|---|---|
| 1 | Happy path | Normal flow yang menghasilkan sukses | Login kredensial valid → 200 + token |
| 2 | Empty/null input | Setiap required field dikosongkan | Email kosong → 400, "email wajib diisi" |
| 3 | Invalid format | Setiap formatted field dengan format salah | `email='invalid'` → 400 |
| 4 | Boundary cases | Min/max length, min/max value, edge value | Password < 8 karakter; nama 256 karakter |
| 5 | Not found | Operasi read/update/delete atas resource yang tidak ada | Update produk ID tak dikenal → 404 |
| 6 | Unauthorized | Tanpa auth token/session | Request tanpa token → 401 |
| 7 | Forbidden | Auth valid tapi role/permission tidak cukup | Role viewer mencoba hapus → 403 |

Checklist per TS:

```
TS-{ID} Coverage Checklist:
☐ Happy path: normal flow berhasil
☐ Empty fields: semua required field tested kosong
☐ Invalid format: semua formatted field tested format salah
☐ Boundary: min/max length/value per constraint
☐ Not found: ID/resource yang tidak exist
☐ Unauthorized: request tanpa credentials
☐ Forbidden: request dengan role yang tidak berhak
```

## Aturan Generate Test Case

**Rule 1 — Fail-First Ordering.** Urutan TC dalam satu TS mengikuti barisan pada [Fase 2](#fase-2--test-case-derivation-fail-first). Tidak ada pengecualian tanpa justifikasi.

**Rule 2 — Struktur Test Steps.** Setiap TC harus memuat kondisi awal, langkah arrange, langkah act, dan expected result eksplisit:

```
Test Steps:
- Kondisi awal: [precondition state]
- Step 1: [Arrange] Siapkan data dengan [kondisi spesifik]
- Step 2: [Act] Kirim/eksekusi [action] ke sistem

Expected Result:
[HTTP status/error code] - [kategori error spesifik].
[Pesan validasi/user-facing message].
```

**Rule 3 — Spesifik, bukan generik.**

| Aspek | Benar | Salah |
|---|---|---|
| Status code | `400`, `401`, `403`, `404` | "gagal" |
| Pesan error | "email wajib diisi" | "validation failed" |
| Data uji | `"Valid123!"`, `"mahasiswa@mail.com"` | "some value" |

**Rule 4 — Konsistensi penamaan.** Lihat [Konvensi Penamaan](#konvensi-penamaan); singkatan fitur harus konsisten per domain (LOGIN, AUTH, PROFILE, REG).

## Contoh Derivasi Coverage

**Input:** `TS-PRODUCT-01: Kelola produk - Mengambil, menambah, edit, hapus data produk`

**Generated TC (fail-first order):**

```
TC-PRODUCT-01: Tambah produk dengan nama kosong - 400, nama produk wajib diisi
TC-PRODUCT-02: Tambah produk dengan harga kosong - 400, harga wajib diisi
TC-PRODUCT-03: Tambah produk dengan format harga invalid (string) - 400, harga harus berupa angka
TC-PRODUCT-04: Tambah produk dengan nama terlalu panjang (256 char) - 400, nama maksimal 255 karakter
TC-PRODUCT-05: Tambah produk dengan harga negatif - 400, harga harus lebih besar dari 0
TC-PRODUCT-06: Edit produk dengan ID tidak ditemukan - 404, produk tidak ditemukan
TC-PRODUCT-07: Hapus produk tanpa autentikasi - 401, unauthorized
TC-PRODUCT-08: Hapus produk dengan role viewer - 403, akses ditolak
TC-PRODUCT-09: Tambah produk dengan data valid - 201, produk berhasil ditambahkan
TC-PRODUCT-10: Ambil list produk - 200, return list produk
```

**Verifikasi coverage:**

| Dimensi | TC |
|---|---|
| Empty | TC-01, TC-02 |
| Invalid format | TC-03 |
| Boundary | TC-04, TC-05 |
| Not found | TC-06 |
| Unauthorized | TC-07 |
| Forbidden | TC-08 |
| Happy path | TC-09, TC-10 |

## Artefak Dokumentasi

Organisasi berkas dipisah per jenis artefak:

```
docs/testing/
├── scenarios/
│   └── TS-{DOMAIN}-scenarios.md
├── cases/
│   └── TC-{FEATURE}-cases.md
└── traceability/
    └── RTM-{feature}.md
```

<details>
<summary><b>Format Test Scenario</b></summary>

```markdown
## Test Scenarios

| Scenario ID | Scenario Description |
|-------------|---------------------|
| TS-AUTH-01  | Autentikasi: Proses pendaftaran akun baru, masuk ke sistem, dan keluar dari sistem |
| TS-AUTH-02  | Kelola Profil: Mengubah data profil pengguna seperti nama, email, dan foto profil |
```

</details>

<details>
<summary><b>Format Test Case</b></summary>

```markdown
## Test Cases: TS-AUTH-01 (Autentikasi)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data login dengan mengosongkan kolom email dan mengisi kolom kata sandi dengan "Valid123!".<br>2. Kirim data login ke sistem. | Permintaan ditolak (400 - data tidak valid). Sistem menampilkan pesan bahwa email wajib diisi. |
```

</details>

<details>
<summary><b>Format RTM</b></summary>

```markdown
## Requirements Traceability Matrix

| Scenario ID | Requirements Description | Test Case IDs |
|-------------|-------------------------|---------------|
| TS-AUTH-01 (Autentikasi) | Pengguna dapat login ke sistem | TC-LOGIN-01 s.d. TC-LOGIN-10, TC-INT-LOGIN-001 s.d. TC-INT-LOGIN-024 |
```

</details>

## Struktur Kode Test

**Prinsip:** struktur bersarang — TS sebagai blok luar, TC sebagai test di dalamnya. Urutan test di dalam suite mengikuti fail-first.

**Pola pseudocode (framework-agnostic):**

```
TEST_SUITE([TS-ID]: [TS-DESCRIPTION]) {

  TEST_CASE([TC-ID]: [CONDITION] - [EXPECTED_RESULT]) {
    // Arrange (kondisi awal + setup)
    // Act (action/execution)
    // Assert (verify expected result)
  }

  TEST_CASE([TC-ID]: [CONDITION] - [EXPECTED_RESULT]) {
    // implementation
  }
}
```

<details>
<summary><b>Contoh Jest/Vitest</b></summary>

```javascript
describe('TS-AUTH-01: Autentikasi - Proses pendaftaran, login, dan logout', () => {

  it('TC-LOGIN-01: Login dengan email kosong - Permintaan ditolak (400), sistem tampilkan pesan email wajib diisi', () => {
    // Arrange: siapkan data dengan email kosong
    const loginData = { email: '', password: 'Valid123!' };

    // Act: kirim request
    const response = login(loginData);

    // Assert
    expect(response.status).toBe(400);
    expect(response.message).toContain('email wajib diisi');
  });

  it('TC-LOGIN-02: Login dengan password kosong - Permintaan ditolak (400), sistem tampilkan pesan password wajib diisi', () => {
    // implementation
  });

  // ... more negative cases ...

  it('TC-LOGIN-09: Login dengan data valid - Berhasil (200), return auth token', () => {
    // happy path - last
  });
});
```

</details>

**Pemetaan sintaks per framework:**

| Framework | Suite | Test | Assertion | Catatan |
|---|---|---|---|---|
| Jest/Vitest | `describe('[TS-ID]: [desc]', () => {})` | `it('[TC-ID]: [condition] - [expected]', () => {})` | `expect()` | Standar |
| Pytest | `class Test{Feature}:` | `def test_{tc_id}_{condition}(self):` | `assert` | Grouping per class |
| Go | `func Test{Feature}(t *testing.T) {}` | `t.Run("[TC-ID]: ...", func(t *testing.T) {})` | `if` / `t.Error` | Subtest |
| .NET xUnit | `public class {Feature}Tests {}` | `[Fact] public void {TcId}_{condition}() {}` | `Assert.*` | Berbasis atribut |
| Ruby RSpec | `describe '[TS-ID]: [desc]' do` | `it '[TC-ID]: [condition] - [expected]' do` | `expect().to` | Gaya BDD |

**Nama:** suite memakai `[TS-ID]: [TS-Description]`, test memakai `[TC-ID]: [kondisi singkat] - [ringkasan expected result]`. Jaga agar tetap terbaca: maksimal ~80 karakter.

## Konvensi Penamaan

| Artefak | Pola | Contoh |
|---|---|---|
| Test Scenario | `TS-{DOMAIN}-{2 digit}` | `TS-AUTH-01`, `TS-PRODUCT-01` |
| Test Case unit | `TC-{FEATURE}-{2 digit}` | `TC-LOGIN-01`, `TC-AUTH-15` |
| Test Case integrasi | `TC-INT-{FEATURE}-{3 digit}` | `TC-INT-LOGIN-001` |

Singkatan nama fitur harus konsisten per domain (LOGIN, AUTH, PROFILE, REG, dan seterusnya).

## Mode Audit & Gap Detection

### Kapan mode audit aktif

- User bertanya "audit test coverage", "cek gap testing", "review test completeness"
- User menyediakan test file existing untuk dianalisis
- User menyediakan dokumentasi TS/TC tapi belum ada kode test

### Proses audit

**Step 1 — Parse & Extract.** Parse test file, ekstrak struktur: nama suite (identifikasi TS-ID bila ada), nama test (identifikasi TC-ID bila ada), hitung jumlah test per suite. Bila nama test tidak memuat TS/TC ID, lakukan pencocokan fuzzy berdasarkan deskripsi.

**Step 2 — Coverage Mapping.** Untuk setiap TS (dari dokumen atau hasil inferensi): petakan TC yang ada ke dimensi coverage, kategorikan test (happy/empty/invalid/boundary/notfound/unauthorized/forbidden), identifikasi dimensi yang hilang.

**Step 3 — Gap Detection.** Susun laporan gap:

```
Coverage Gap Report

TS-PRODUCT-01: Kelola produk
✓ Happy path: 2 tests found
✓ Empty fields: 2 tests found
✓ Invalid format: 1 test found
✗ Boundary cases: MISSING
✗ Not found: MISSING
✓ Unauthorized: 1 tests found
✗ Forbidden: MISSING

Recommended additions:
- TC-PRODUCT-04: Boundary - nama produk max length
- TC-PRODUCT-05: Boundary - harga negatif/zero
- TC-PRODUCT-06: Not found - edit/delete produk tidak exist
- TC-PRODUCT-08: Forbidden - operasi tanpa role yang tepat
```

**Step 4 — RTM Validation.** Bila dokumen RTM ada, silangkan: apakah semua TC-ID di RTM punya implementasi? Laporkan TC *orphaned* (ada di RTM, belum diimplementasikan) dan test *undocumented* (ada di kode, tidak ada di RTM).

<details>
<summary><b>Format laporan audit lengkap</b></summary>

```markdown
## Test Coverage Audit Report
**Date:** {timestamp}
**Scope:** {feature/domain}

### Summary
- Total Test Scenarios: X
- Total Test Cases documented: Y
- Total Test Cases implemented: Z
- Coverage completeness: W%

### Gap Analysis

#### TS-{ID}: {Description}
**Coverage status:**
- ✓ Happy path (N tests)
- ✓ Empty/null (N tests)
- ✗ Invalid format (MISSING)
- ✗ Boundary (MISSING)
- ✓ Not found (N tests)
- ✗ Unauthorized (MISSING)
- ✗ Forbidden (MISSING)

**Missing test cases:**
- TC-{ID}: {suggested description}

#### RTM Discrepancies
**Orphaned (documented but not implemented):**
- TC-LOGIN-05

**Undocumented (implemented but not in RTM):**
- Unnamed test: "should reject invalid email format"

### Recommendations
1. Priority high: implement missing security tests (unauthorized/forbidden)
2. Priority medium: add boundary cases for {fields}
3. Update RTM: add TC-{new-ids} for undocumented tests
```

</details>

### Remediation Mode

Setelah audit, agent menawarkan:

1. Generate dokumentasi TC yang hilang untuk test yang belum terdokumentasi?
2. Generate stub kode test untuk coverage yang hilang?
3. Update RTM dengan TC-ID yang baru ditemukan?

User memilih, agent mengeksekusi.

## Checklist Eksekusi Agent

<details>
<summary><b>Mode 1 — Generate test baru</b></summary>

```
☐ 1. Analyze input (requirements/code/feature description)
☐ 2. Generate Test Scenarios
    ☐ 2a. Extract domain boundaries
    ☐ 2b. Create TS-ID + description per domain
    ☐ 2c. Present TS list to user
    ☐ 2d. GATE: Wait for user approval
☐ 3. Derive Test Cases (per approved TS)
    ☐ 3a. Apply coverage matrix (7 dimensions)
    ☐ 3b. Generate TC in fail-first order
    ☐ 3c. Write TC with: ID | condition | test steps | expected result
    ☐ 3d. Present TC table to user
    ☐ 3e. GATE: Wait for user approval
☐ 4. Generate artifacts
    ☐ 4a. Write TS doc to docs/testing/scenarios/
    ☐ 4b. Write TC doc to docs/testing/cases/
    ☐ 4c. Generate test code structure (describe/it with TS/TC IDs)
    ☐ 4d. Generate RTM to docs/testing/traceability/
☐ 5. Verify completeness
    ☐ 5a. Check all 7 coverage dimensions present per TS
    ☐ 5b. Check fail-first ordering in test code
    ☐ 5c. Check TS/TC ID consistency across docs + code
```

</details>

<details>
<summary><b>Mode 2 — Audit test existing</b></summary>

```
☐ 1. Collect inputs
    ☐ 1a. Parse test files (extract TS/TC structure)
    ☐ 1b. Read existing TS/TC/RTM docs (if any)
☐ 2. Map coverage
    ☐ 2a. Categorize existing tests by dimension
    ☐ 2b. Identify missing dimensions per TS
☐ 3. Generate gap report
    ☐ 3a. Summary stats (total TS/TC, coverage %)
    ☐ 3b. Per-TS gap analysis
    ☐ 3c. RTM discrepancies (orphaned + undocumented)
    ☐ 3d. Prioritized recommendations
☐ 4. Offer remediation
    ☐ 4a. Generate missing TC docs?
    ☐ 4b. Generate test code stubs?
    ☐ 4c. Update RTM?
```

</details>

## Batasan & Anti-Pattern

**Wajib dipatuhi:**

- Coverage 7 dimensi per TS
- Fail-first ordering: negative case sebelum happy path
- Traceability: setiap TC tertaut ke TS induk
- Pesan error spesifik, bukan "validation failed"
- Kepatuhan gerbang: persetujuan user sebelum artefak dibuat

**Wajib dihindari:**

- Melewati dimensi coverage tanpa justifikasi
- Nama test generik tanpa TC-ID
- Urutan happy path di depan
- Membuat kode test tanpa artefak dokumentasi
- Melanjutkan ke fase berikutnya tanpa persetujuan user

**Di luar cakupan:**

- Eksekusi/penjalanan test case
- Menulis kode produksi agar test lolos (fase implementasi TDD)
- Performance/load testing
- Detail alur otomasi E2E UI
- Mock/stub generation

## Kriteria Keberhasilan

Skill dinilai berhasil bila:

1. **Konsistensi** — TC dari sesi berbeda mengikuti struktur dan penamaan yang sama
2. **Coverage** — setiap TS menurunkan TC yang mencakup 7 dimensi wajib
3. **Traceability** — pemetaan jelas dari requirements → TS → TC → kode test
4. **Fail-first** — kode test konsisten menempatkan negative case sebelum happy path
5. **Akurasi audit** — deteksi gap benar-benar menemukan dimensi yang hilang
6. **Portabilitas** — skill berjalan tanpa modifikasi di agent apa pun yang mendukung Agent Skills
7. **Validasi user** — gerbang memastikan persetujuan sebelum artefak dibuat

**Metrik kualitas:**

| Metrik | Definisi |
|---|---|
| Coverage completeness | % TS dengan seluruh 7 dimensi tercakup |
| Traceability ratio | % test terimplementasi dengan TC-ID valid yang terpetakan ke TS |
| Ordering compliance | % suite test yang mengikuti urutan fail-first |
| Documentation sync | % kode test yang punya dokumentasi TC pendamping |

## Roadmap

Di luar cakupan v1, kandidat pengembangan berikutnya:

- Visual coverage dashboard
- Integrasi test runner untuk deteksi gap otomatis
- Prioritisasi test case berbasis risiko
- Panduan property-based testing
- Integrasi mutation testing
- Template dimensi coverage kustom per domain

## Contoh Lengkap: Autentikasi

### Fase 1 — Test Scenarios

| Scenario ID | Scenario Description |
|-------------|---------------------|
| TS-AUTH-01 | Login - Autentikasi pengguna dengan email dan password |
| TS-AUTH-02 | Registrasi - Pendaftaran akun pengguna baru |
| TS-AUTH-03 | Logout - Keluar dari sesi aktif |

### Fase 2 — Test Cases (TS-AUTH-01)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong | 1. Siapkan data: email='', password='Valid123!'<br>2. POST /auth/login | 400 - email wajib diisi |
| TS-AUTH-01 | TC-LOGIN-02 | Login dengan password kosong | 1. Siapkan data: email='user@mail.com', password=''<br>2. POST /auth/login | 400 - password wajib diisi |
| TS-AUTH-01 | TC-LOGIN-03 | Login dengan format email salah | 1. Siapkan data: email='invalid', password='Valid123!'<br>2. POST /auth/login | 400 - format email tidak valid |
| TS-AUTH-01 | TC-LOGIN-04 | Login dengan password < 8 karakter | 1. Siapkan data: email='user@mail.com', password='123'<br>2. POST /auth/login | 400 - password minimal 8 karakter |
| TS-AUTH-01 | TC-LOGIN-05 | Login dengan email tidak terdaftar | 1. Siapkan data: email='notfound@mail.com', password='Valid123!'<br>2. POST /auth/login | 404 - akun tidak ditemukan |
| TS-AUTH-01 | TC-LOGIN-06 | Login tanpa auth header | 1. Siapkan request tanpa header Authorization<br>2. POST /auth/login | 401 - unauthorized |
| TS-AUTH-01 | TC-LOGIN-07 | Login dengan akun suspended | 1. Siapkan data: email milik akun yang suspended<br>2. POST /auth/login | 403 - akun dinonaktifkan |
| TS-AUTH-01 | TC-LOGIN-08 | Login dengan kredensial valid | 1. Siapkan data: email='user@mail.com', password='Valid123!'<br>2. POST /auth/login | 200 - return JWT token |

Urutan tersebut sudah fail-first: empty → invalid format → boundary → not found → unauthorized → forbidden → happy path.

### Fase 3 — RTM

| Scenario ID | Requirements Description | Test Case IDs |
|-------------|-------------------------|---------------|
| TS-AUTH-01 | Pengguna dapat login dengan email dan password | TC-LOGIN-01 s.d. TC-LOGIN-08 |

### Kode Test (Jest)

```javascript
describe('TS-AUTH-01: Login - Autentikasi dengan email dan password', () => {

  // Negative cases first (fail-first)
  it('TC-LOGIN-01: Login dengan email kosong - 400, email wajib diisi', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({ email: '', password: 'Valid123!' });

    expect(response.status).toBe(400);
    expect(response.body.message).toContain('email wajib diisi');
  });

  it('TC-LOGIN-02: Login dengan password kosong - 400, password wajib diisi', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({ email: 'user@mail.com', password: '' });

    expect(response.status).toBe(400);
    expect(response.body.message).toContain('password wajib diisi');
  });

  // ... negative cases lain ...

  // Happy path last
  it('TC-LOGIN-08: Login dengan kredensial valid - 200, return JWT token', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({ email: 'user@mail.com', password: 'Valid123!' });

    expect(response.status).toBe(200);
    expect(response.body.token).toBeDefined();
  });
});
```

Versi berkas nyatanya ada di [`examples/`](examples/) — `example-ts-scenarios.md`, `example-tc-cases.md`, dan `example-rtm.md`. Versi kosongnya ada di [`templates/`](templates/).

## Isi Paket Skill

```
traceable-tests/
├── SKILL.md              ← instruksi agent (wajib, ber-YAML frontmatter)
├── README.md             ← dokumen ini; pusat informasi untuk manusia
├── examples/             ← contoh dokumen TS, TC, RTM
│   ├── example-ts-scenarios.md
│   ├── example-tc-cases.md
│   └── example-rtm.md
└── templates/            ← template kosong dengan placeholder
    ├── ts-template.md
    ├── tc-template.md
    └── rtm-template.md
```

Seluruh isi skill ada di folder ini — tidak ada salinan per-agent di repo. Installer yang membuat link atau salinan ke direktori skill milik agent yang dipilih.

`SKILL.md` dan dokumen ini harus dijaga tetap sinkron: `SKILL.md` adalah instruksi yang dieksekusi agent, README ini penjelasan lengkapnya.

## Instalasi

Skill mengikuti standar Agent Skills, sehingga bisa dipasang ke agent mana pun (Claude Code, OpenCode, Codex, Cursor, Gemini CLI, GitHub Copilot, dll.) dengan satu perintah:

```bash
# Interaktif: pilih skill & agent tujuan
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests

# Non-interaktif ke semua agent yang terdeteksi
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests --all

# Global (tersedia di semua project) untuk agent tertentu
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests -g -a claude-code -a codex -a opencode

# Dari checkout lokal
npx skills add ./skills/traceable-tests
```

Tanpa instalasi, skill bisa dipakai sekali jalan:

```bash
npx skills use MuhammadAidil-dev/trace-test@traceable-tests
```

CLI memasang ke direktori skill milik masing-masing agent (mis. `.agents/skills/` untuk OpenCode, Codex, Cursor, Copilot; `.claude/skills/` untuk Claude Code), dengan symlink ke satu sumber di `skills/traceable-tests/`.

### Instalasi manual

Salin folder ini ke direktori skill agent tujuan — mis. `.agents/skills/traceable-tests/` (project) atau `~/.agents/skills/traceable-tests/` (global). Isi skill adalah markdown murni, sehingga berjalan sama di semua agent.

## Lisensi

Dirilis dengan **MIT License**.
