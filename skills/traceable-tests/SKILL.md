---
name: traceable-tests
description: Traceable 3-layer testing workflow (Test Scenario -> Test Case -> Requirements Traceability Matrix) with 7 mandatory coverage dimensions and fail-first ordering. Use when the user asks to write or derive test cases, design a testing strategy, audit existing test coverage, review test completeness, or mentions TS/TC/RTM.
---

# Skill: traceable-tests

**Nama Skill:** `traceable-tests`  
**Tipe:** Methodology/Process skill (language-agnostic, agent-agnostic)  
**Platform:** Portable ke semua agent yang mendukung standar Agent Skills  
**Lokasi sumber:** `skills/traceable-tests/` (satu sumber kebenaran, tanpa salinan per-agent)  
**Format artefak:** Markdown documentation + test code structure guidance  
**Versi:** 1.1 (2026-10-27)

---

## 1. Deskripsi

Skill `traceable-tests` encode workflow pengujian 3-layer (TS → TC → RTM) dengan fokus konsistensi struktur dan coverage tracking. Agent dapat:

1. **Generate Test Scenario** dari analisis fitur/domain
2. **Derive Test Cases** dari TS dengan fail-first approach
3. **Audit test existing**: parse, map ke struktur, report gap
4. **Generate RTM** untuk requirement traceability

---

## 2. Trigger Conditions (Kapan Menggunakan Skill)

**Invoke skill ini saat:**
- User minta "write test", "buat test case", "testing untuk fitur X"
- User minta "audit test", "cek coverage", "review test completeness"
- User menyebutkan TS/TC/RTM secara eksplisit
- User menyediakan requirement doc + meminta testing strategy
- User invoke skill secara eksplisit: `/traceable-tests` atau "gunakan traceable-tests"

**SKILL INI BUKAN untuk:**
- Menjalankan/mengeksekusi test cases (test execution)
- Menulis production code untuk lolos test (TDD implementation phase)
- Skill HANYA membuat: traceable test documentation, test case structure, audit, dan konsistensi check

**Di luar cakupan (Out of Scope):**
- Framework-specific syntax generation (skill menyediakan struktur/prinsip, agent menyesuaikan ke stack)
- Performance/load testing
- E2E UI automation flow details
- Mock/stub generation specifics

---

## 3. Coverage Dimensions (Wajib)

Setiap Test Scenario harus derive Test Cases yang mencakup:

| No | Dimensi | Deskripsi |
|----|---------|-----------|
| 1 | Happy path | Normal flow yang menghasilkan sukses |
| 2 | Empty/null input | Setiap required field dikosongkan |
| 3 | Invalid format | Setiap formatted field (email, regex, type) dengan format salah |
| 4 | Boundary cases | min/max length, min/max value, edge values |
| 5 | Not found | Untuk operasi read/update/delete (resource tidak ada) |
| 6 | Unauthorized | Tanpa auth token/session |
| 7 | Forbidden | Auth valid tapi role/permission tidak cukup |

---

## 4. Workflow Arsitektur (Tiga Fase dengan Gate)

### Fase 1: Test Scenario Generation

**Input:** Feature/domain description, requirement doc, atau existing code

**Proses:**
1. Analyze feature boundary & responsibilities
2. Abstract menjadi high-level test scenarios
3. Generate TS ID: `TS-{DOMAIN}-{NUM}` (contoh: TS-PRODUCT-01, TS-AUTH-01)
4. Tulis scenario description (broad coverage statement)

**Output:** Test Scenario list dengan ID + description

**Gate:** User meninjau TS list sebelum derive TC

### Fase 2: Test Case Derivation (Fail-First)

**Input:** Approved Test Scenarios

**Proses per TS:**
1. Identify test dimensions (happy/error/security)
2. Generate TC dengan **fail-first order**: negative cases duluan, happy path terakhir
3. Struktur per TC:
   - Scenario ID
   - Test Case ID: `TC-{FEATURE}-{NUM}`
   - Scenario Description
   - Test Steps (sertakan: kondisi awal + numbered steps)
   - Expected Result (status code + message eksplisit)

**Output:** Test Case table/document

**Gate:** User meninjau TC completeness sebelum implementation

**Urutan Fail-First:**
1. Empty/null cases (TC-XXX-01 s.d. 0N)
2. Invalid format cases
3. Boundary cases
4. Not found cases
5. Unauthorized cases
6. Forbidden cases
7. Happy path cases (terakhir)

### Fase 3: RTM & Coverage Audit

**Input:** TS list + TC list (+ optional: existing test code)

**Proses:**
1. Map requirements → TS → TC IDs
2. Generate RTM table
3. Audit mode: parse existing tests, identify missing TC, report gap

**Output:** RTM document + gap report

**Tidak ada gate:** Final artifact

---

## 5. Aturan Cross-Cutting

- Coverage wajib per TC group: 7 dimensi (happy/empty/invalid/boundary/notfound/unauthorized/forbidden)
- Fail-first sequence diwajibkan: negative/edge cases ditangani dulu, happy path terakhir
- Traceability diwajibkan: setiap TC harus link ke parent TS
- Specific error messages: hindari generic "error occurred", gunakan pesan validasi spesifik
- Concrete test data: gunakan contoh eksplisit ("Valid123!", "mahasiswa@mail.com")

---

## 6. Output Architecture

### A. Dokumentasi Artefak (File Organization: Terpisah/Modular)

```
docs/testing/
├── scenarios/
│   └── TS-{DOMAIN}-scenarios.md
├── cases/
│   └── TC-{FEATURE}-cases.md
└── traceability/
    └── RTM-{feature}.md
```

**Test Scenario Format:**

```markdown
## Test Scenarios

| Scenario ID | Scenario Description |
|-------------|---------------------|
| TS-AUTH-01   | Autentikasi: Proses pendaftaran akun baru, masuk ke sistem, dan keluar dari sistem |
| TS-AUTH-02   | Kelola Profil: Mengubah data profil pengguna seperti nama, email, dan foto profil |
```

**Test Case Format:**

```markdown
## Test Cases: TS-AUTH-01 (Autentikasi)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data login dengan mengosongkan kolom email dan mengisi kata sandi "Valid123!".<br>2. Kirim data login ke sistem. | Permintaan ditolak (400 - data tidak valid). Sistem menampilkan pesan bahwa email wajib diisi. |
```

**RTM Format:**

```markdown
## Requirements Traceability Matrix

| Scenario ID | Requirements Description | Test Case IDs |
|-------------|-------------------------|---------------|
| TS-AUTH-01 (Autentikasi) | Pengguna dapat login ke sistem | TC-LOGIN-01 s.d. TC-LOGIN-10, TC-INT-LOGIN-001 s.d. TC-INT-LOGIN-024 |
```

### B. Test Code Structure

**Prinsip:** Nested structure dengan TS sebagai outer block, TC sebagai inner test

**Pseudo-code Pattern (Framework-Agnostic):**

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

**Contoh - Jest/Vitest:**

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

**Framework-Specific Mapping:**

| Framework | Suite syntax | Test syntax | Notes |
|-----------|-------------|-------------|-------|
| Jest/Vitest | `describe('[TS-ID]: [desc]', () => {})` | `it('[TC-ID]: [condition] - [expected]', () => {})` | Standard |
| Pytest | `class Test{Feature}:` | `def test_{tc_id}_{condition}(self):` | Class-based grouping |
| Go | `func Test{Feature}(t *testing.T) {}` | `t.Run("[TC-ID]: ...", func(t *testing.T) {})` | Subtests |
| .NET xUnit | `public class {Feature}Tests {}` | `[Fact] public void {TcId}_{condition}() {}` | Attribute-based |
| Ruby RSpec | `describe '[TS-ID]: [desc]' do` | `it '[TC-ID]: [condition] - [expected]' do` | BDD style |

**Naming Convention:**
- Suite name: `[TS-ID]: [TS-Description]`
- Test name: `[TC-ID]: [Brief condition] - [Expected result summary]`
- Keep concise: max ~80 chars untuk readability

**Fail-First Ordering in Code:**
Tests within suite harus diurutkan: negative cases duluan, happy path terakhir

---

## 7. Naming Conventions

- **TS ID:** `TS-{DOMAIN}-{2-digit-num}` (contoh: TS-AUTH-01, TS-PRODUCT-01)
- **TC ID (unit):** `TC-{FEATURE}-{2-digit-num}` (contoh: TC-LOGIN-01, TC-AUTH-15)
- **TC ID (integration):** `TC-INT-{FEATURE}-{3-digit-num}` (contoh: TC-INT-LOGIN-001)
- **Feature abbreviations:** Konsisten per domain (LOGIN, AUTH, PROFILE, REG, dll.)

---

## 8. Coverage Rules & Test Generation Logic

### Coverage Matrix per Test Scenario

Setiap TS harus derive TC yang mencakup:

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

### Aturan Generate Test Case

**Rule 1: Fail-First Ordering**
Sequence TC dalam 1 TS:
1. Empty/null cases (TC-XXX-01 s.d. 0N)
2. Invalid format cases
3. Boundary cases
4. Not found cases
5. Unauthorized cases
6. Forbidden cases
7. Happy path cases (terakhir)

**Rule 2: Test Steps Structure**
Setiap TC harus berisi:
```
Test Steps:
- Kondisi awal: [precondition state]
- Step 1: [Arrange] Siapkan data dengan [kondisi spesifik]
- Step 2: [Act] Kirim/eksekusi [action] ke sistem

Expected Result:
[HTTP status/error code] - [kategori error spesifik].
[Pesan validasi/user-facing message].
```

**Rule 3: Specific vs Generic**
- Status code: eksplisit (400, 401, 403, 404, 500)
- Error message: spesifik validation message, bukan generic "validation failed"
- Data values: contoh konkret dalam test steps ("Valid123!", "mahasiswa@mail.com")

**Rule 4: Naming Consistency**
TC ID pattern:
- Unit test: `TC-{FEATURE}-{NUM}`
- Integration test: `TC-INT-{FEATURE}-{NUM}`

Feature name abbreviation konsisten per domain.

### Contoh Coverage Derivation

**Input:** TS-PRODUCT-01: Kelola produk - Mengambil, menambah, edit, hapus data produk

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

**Coverage Verification:**
- ✓ Empty: TC-01, TC-02
- ✓ Invalid format: TC-03
- ✓ Boundary: TC-04, TC-05
- ✓ Not found: TC-06
- ✓ Unauthorized: TC-07
- ✓ Forbidden: TC-08
- ✓ Happy path: TC-09, TC-10

---

## 9. Audit Mode & Gap Detection

### Audit Triggers

Skill masuk audit mode saat:
- User bertanya "audit test coverage", "cek gap testing", "review test completeness"
- User menyediakan existing test files untuk dianalisis
- User menyediakan TS/TC documentation tapi belum ada test code

### Audit Process

**Step 1: Parse & Extract**
- Parse test files, extract structure:
  - Test suite names (identify TS-ID if present)
  - Test case names (identify TC-ID if present)
  - Count total tests per suite
- If no TS/TC ID in test names: attempt fuzzy matching based on description

**Step 2: Coverage Mapping**
For each Test Scenario (from doc or inferred):
- Map existing TC to coverage dimensions
- Categorize tests: happy/empty/invalid/boundary/notfound/unauthorized/forbidden
- Identify missing dimensions

**Step 3: Gap Detection**
Generate report:
```
Coverage Gap Report

TS-PRODUCT-01: Kelola produk
✓ Happy path: 2 tests found
✓ Empty fields: 2 tests found
✓ Invalid format: 1 test found
✗ Boundary cases: MISSING
✗ Not found: MISSING
✓ Unauthorized: 1 test found
✗ Forbidden: MISSING

Recommended additions:
- TC-PRODUCT-04: Boundary - nama produk max length
- TC-PRODUCT-05: Boundary - harga negatif/zero
- TC-PRODUCT-06: Not found - edit/delete produk tidak exist
- TC-PRODUCT-08: Forbidden - operasi tanpa role yang tepat
```

**Step 4: RTM Validation**
If RTM doc exists:
- Cross-check: all TC-IDs in RTM have test code implementation?
- Report orphaned TC (in RTM but no implementation)
- Report undocumented tests (test code exists but not in RTM)

### Audit Output Format

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
- TC-{ID}: {suggested description}

#### RTM Discrepancies
**Orphaned (documented but not implemented):**
- TC-LOGIN-05
- TC-AUTH-12

**Undocumented (implemented but not in RTM):**
- Unnamed test: "should reject invalid email format"

### Recommendations
1. Priority high: implement missing security tests (unauthorized/forbidden)
2. Priority medium: add boundary cases for {fields}
3. Update RTM: add TC-{new-ids} for undocumented tests
```

### Remediation Mode

After audit, agent offers:
1. "Generate missing TC documentation for undocumented tests?"
2. "Generate test code stubs for missing coverage?"
3. "Update RTM with newly discovered test IDs?"

User selects action, agent executes.

---

## 10. Agent Instructions & Execution Checklist

### Skill Invocation Rules

**When to invoke this skill:**
- User asks "write test", "buat test case", "testing untuk fitur X"
- User asks "audit test", "cek coverage", "review test completeness"
- User mentions TS/TC/RTM explicitly
- User provides requirement doc + asks for testing strategy
- User explicitly invokes: `/traceable-tests` or "gunakan traceable-tests"

**IMPORTANT: This skill is NOT for:**
- Test execution/running test cases
- Writing production code to make tests pass (TDD implementation)
- This skill ONLY creates: traceable test documentation, test case structure, audit, and consistency checks

**When NOT to invoke:**
- User only asks to run existing tests (execution, not generation)
- User asks for mock/stub generation only (out of scope)
- User asks for performance/load testing (different domain)

### Mode 1: Generate New Tests

```
☐ 1. Analyze input (requirements/code/feature description)
☐ 2. Generate Test Scenarios
    ☐ 2a. Extract domain boundaries
    ☐ 2b. Create TS-ID + description per domain
    ☐ 2c. Present TS list to user
    ☐ 2d. GATE: Tunggu persetujuan user
☐ 3. Derive Test Cases (per approved TS)
    ☐ 3a. Apply coverage matrix (7 dimensi)
    ☐ 3b. Generate TC dalam fail-first order
    ☐ 3c. Tulis TC: ID | condition | test steps | expected result
    ☐ 3d. Present TC table to user
    ☐ 3e. GATE: Tunggu persetujuan user
☐ 4. Generate artifacts
    ☐ 4a. Tulis TS doc ke docs/testing/scenarios/
    ☐ 4b. Tulis TC doc ke docs/testing/cases/
    ☐ 4c. Generate test code structure (describe/it dengan TS/TC IDs)
    ☐ 4d. Generate RTM ke docs/testing/traceability/
☐ 5. Verify completeness
    ☐ 5a. Cek semua 7 coverage dimensions per TS
    ☐ 5b. Cek fail-first ordering dalam test code
    ☐ 5c. Cek TS/TC ID consistency di docs + code
```

### Mode 2: Audit Existing Tests

```
☐ 1. Collect inputs
    ☐ 1a. Parse test files (extract TS/TC structure)
    ☐ 1b. Baca TS/TC/RTM docs yang ada (kalau ada)
☐ 2. Map coverage
    ☐ 2a. Kategorikan existing tests per dimensi
    ☐ 2b. Identifikasi missing dimensions per TS
☐ 3. Generate gap report
    ☐ 3a. Summary stats (total TS/TC, coverage %)
    ☐ 3b. Per-TS gap analysis
    ☐ 3c. RTM discrepancies (orphaned + undocumented)
    ☐ 3d. Prioritized recommendations
☐ 4. Offer remediation
    ☐ 4a. Generate missing TC docs?
    ☐ 4b. Generate test code stubs?
    ☐ 4c. Update RTM?

---

## 11. Key Constraints & Anti-Patterns

**MUST enforce:**
- Coverage wajib: 7 dimensi per TS (happy/empty/invalid/boundary/notfound/unauthorized/forbidden)
- Fail-first ordering: negative cases sebelum happy path
- Traceability: setiap TC link ke parent TS
- Specific error messages: bukan generic "validation failed"
- Gate compliance: user approval di phases sebelum generate artifacts

**MUST avoid:**
- Skipping coverage dimensions without justification
- Generic test names without TC-ID
- Happy path first ordering
- Generating test code without documentation artifacts
- Proceeding to next phase without user approval

---

## 12. Cross-Platform Adaptation

Agent harus adaptasi pseudo-code ke target framework:

| Framework | Suite syntax | Test syntax | Assertion |
|-----------|-------------|-------------|-----------|
| Jest/Vitest | `describe()` | `it()` | `expect()` |
| Pytest | `class Test{Feature}` | `def test_{id}_...` | `assert` |
| Go | `func Test{Feature}(t *testing.T)` | `t.Run("{TC-ID}: ...")` | `if/t.Error` |
| .NET xUnit | `public class {Feature}Tests` | `[Fact] public void {TcId}_...` | `Assert.*` |
| Ruby RSpec | `describe` | `it` | `expect().to` |

**Principle remains same:** TS = suite, TC = individual test, fail-first order.

---

## 13. Distribution & Installation

**Sumber tunggal:** seluruh isi skill berada di `skills/traceable-tests/`. Repo ini tidak menyimpan salinan per-agent; installer CLI yang membuat link atau salinan ke direktori skill milik agent yang dipilih.

**Frontmatter wajib:** `SKILL.md` memakai YAML frontmatter standar Agent Skills (`name`, `description`). `description` harus menyebut apa yang dikerjakan skill dan kapan dipakai, karena itulah yang dibaca agent untuk memutuskan apakah skill ini relevan.

**Instalasi (satu perintah):**

```bash
# Interaktif: pilih skill & agent tujuan
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests

# Non-interaktif ke semua agent yang terdeteksi
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests --all

# Global (tersedia di semua project) untuk agent tertentu
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests -g -a claude-code -a codex -a opencode

# Dari checkout lokal (mis. sebelum repo dipublikasikan)
npx skills add ./skills/traceable-tests
```

Tanpa instalasi, skill bisa dipakai sekali jalan:

```bash
npx skills use MuhammadAidil-dev/trace-test@traceable-tests
```

**Mode pemasangan:** symlink (rekomendasi — satu sumber kebenaran dan mudah di-update) atau copy (untuk lingkungan tanpa dukungan symlink).

**Kompatibilitas lintas agent:**
- Definisi markdown murni + frontmatter standar Agent Skills, tanpa kode spesifik platform
- Prinsip framework-agnostic dengan panduan adaptasi (§12); template memakai placeholder, bukan sintaks hardcoded
- Nama skill dan struktur direktori identik di semua agent; isi tidak berubah per platform

---

## 14. Success Criteria

Skill berhasil jika:

1. **Consistency:** Test cases generated across different sessions follow same structure and naming
2. **Coverage:** Every TS derives TC covering all 7 mandatory dimensions
3. **Traceability:** Clear mapping from requirements → TS → TC → test code
4. **Fail-first:** Test code consistently orders negative cases before happy path
5. **Audit mode:** Gap detection correctly identifies missing coverage dimensions
6. **Portability:** Skill runs unmodified on any agent that supports the Agent Skills standard
7. **User validation:** Gates ensure user approval before artifact generation

### Quality Metrics

- **Coverage completeness:** % of TS with all 7 dimensions covered
- **Traceability ratio:** % of implemented tests with valid TC-ID mapping to TS
- **Ordering compliance:** % of test suites following fail-first sequence
- **Documentation sync:** % of test code with corresponding TC documentation
```
