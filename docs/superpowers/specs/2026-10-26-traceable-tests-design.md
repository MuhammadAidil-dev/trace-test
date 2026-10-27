# Traceable Tests Skill Design

**Date:** 2026-10-26  
**Skill Name:** `traceable-tests`  
**Author:** AI Agent + User Collaboration  
**Status:** Design Phase

---

## 1. Overview & Scope

### Purpose

Skill ini encode workflow 3-layer testing (TS → TC → RTM) dengan focus konsistensi struktur dan coverage tracking. Agent dapat:
1. Generate Test Scenario dari analisis fitur/domain
2. Derive Test Cases dari TS dengan fail-first approach
3. Audit test existing: parse, map ke struktur, report gap
4. Generate RTM untuk requirement traceability

### Skill Characteristics

- **Type:** Methodology/Process skill (language-agnostic)
- **Platform:** Cross-platform portable (OpenCode, Claude Code, Codex, dll)
- **Install location:** `~/.config/opencode/skills/traceable-tests/` atau `~/.agents/skills/traceable-tests/`
- **Artifact format:** Markdown documentation + test code structure guidance

### Trigger Conditions

**When to invoke this skill:**
- User minta "write test", "buat test case", "testing untuk fitur X"
- User minta "audit test", "cek coverage", "review test completeness"
- User mention TS/TC/RTM explicitly
- User provide requirement doc + ask untuk testing strategy
- User invoke skill explicitly: `/traceable-tests` atau "gunakan traceable-tests"

**IMPORTANT: This skill is NOT for:**
- Executing/running test cases (test execution)
- Writing production code to pass tests (TDD implementation phase)
- This skill ONLY creates traceable test documentation, test case structure, audit, and consistency

### Out of Scope

- Framework-specific syntax generation (skill provides structure/principles, agent adapts to stack)
- Performance/load testing
- E2E UI automation flow details
- Mock/stub generation specifics

### Coverage Dimensions (Mandatory)

Every Test Scenario must derive Test Cases covering:
1. **Happy path** - normal flow expected success
2. **Empty/null input** - per required field
3. **Invalid format** - per formatted field (email, regex, type)
4. **Boundary cases** - min/max length, min/max value, edge values
5. **Not found** - for read/update/delete operations (resource doesn't exist)
6. **Unauthorized** - without auth token/session
7. **Forbidden** - valid auth but insufficient role/permission

---

## 2. Workflow Architecture

### Three-Phase Workflow with Gates

#### Phase 1: Test Scenario Generation

**Input:** Feature/domain description, requirement doc, or existing code

**Process:**
1. Analyze feature boundary & responsibilities
2. Abstract into high-level test scenarios
3. Generate TS ID: `TS-{DOMAIN}-{NUM}` (e.g., TS-PRODUCT-01, TS-MHS-01)
4. Write scenario description (broad coverage statement)

**Output:** Test Scenario list dengan ID + description

**Gate:** User reviews TS list before deriving TC

**Example:**
```
TS-PRODUCT-01: Kelola produk - Mengambil, menambah, edit, hapus data produk
TS-MHS-01: Autentikasi - Proses pendaftaran akun baru, masuk ke sistem, dan keluar dari sistem
```

#### Phase 2: Test Case Derivation (Fail-First)

**Input:** Approved Test Scenarios

**Process per TS:**
1. Identify test dimensions (happy/error/security)
2. Generate TC dengan **fail-first order**: negative cases first, happy path last
3. Structure per TC:
   - Scenario ID
   - Test Case ID: `TC-{FEATURE}-{NUM}` (e.g., TC-LOGIN-01)
   - Scenario Description
   - Test Steps (include: kondisi awal + numbered steps)
   - Expected Result (explicit status code + message)

**Output:** Test Case table/document

**Gate:** User reviews TC completeness before implementation

**Fail-First Ordering Sequence:**
1. Empty/null cases (TC-XXX-01 to 0N)
2. Invalid format cases
3. Boundary cases
4. Not found cases
5. Unauthorized cases
6. Forbidden cases
7. Happy path cases (last)

#### Phase 3: RTM & Coverage Audit

**Input:** TS list + TC list (+ optional: existing test code)

**Process:**
1. Map requirements → TS → TC IDs
2. Generate RTM table
3. Audit mode: parse existing tests, identify missing TC, report gap

**Output:** RTM document + gap report

**No gate:** Final artifact

### Cross-Cutting Rules

- Coverage wajib per TC group: 7 dimensions (happy/empty/invalid/boundary/notfound/unauthorized/forbidden)
- Fail-first sequence enforce: negative/edge cases numbered first, happy path last
- Traceability enforce: every TC must link to parent TS
- Specific error messages: avoid generic "error occurred", use specific validation messages
- Concrete test data: use explicit examples ("Valid123!", "mahasiswa@mail.com")

---

## 3. Output Architecture

### A. Documentation Artifacts

**File Organization: Separated (Modular)**

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
| TS-MHS-01   | Autentikasi: Proses pendaftaran akun baru, masuk ke sistem, dan keluar dari sistem |
| TS-MHS-02   | Kelola Profil: Mengubah data diri seperti nama, NIM, dan foto profil |
```

**Test Case Format:**

```markdown
## Test Cases: TS-MHS-01 (Autentikasi)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-MHS-01 | TC-LOGIN-01 | Login dengan email kosong.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data login dengan mengosongkan kolom email dan mengisi kolom kata sandi dengan "Valid123!".<br>2. Kirim data login ke sistem. | Permintaan ditolak (400 - data tidak valid). Sistem menampilkan pesan bahwa email wajib diisi. |
```

**RTM Format:**

```markdown
## Requirements Traceability Matrix

| Scenario ID | Requirements Description | Test Case IDs |
|-------------|-------------------------|---------------|
| TS-MHS-01 (Autentikasi) | Pengguna dapat login sebagai mahasiswa | TC-LOGIN-01 s.d. TC-LOGIN-10, TC-INT-LOGIN-001 s.d. TC-INT-LOGIN-024 |
```

### B. Test Code Structure

**Principle:** Nested structure dengan TS sebagai outer block, TC sebagai inner test

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

**Example - Jest/Vitest:**

```javascript
describe('TS-MHS-01: Autentikasi - Proses pendaftaran, login, dan logout', () => {
  
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
- Keep concise: max ~80 chars for readability

**Fail-First Ordering in Code:**
Tests within suite must be ordered: negative cases first, happy path last

### C. Naming Conventions

- **TS ID:** `TS-{DOMAIN}-{2-digit-num}` (e.g., TS-MHS-01, TS-PRODUCT-01)
- **TC ID (unit):** `TC-{FEATURE}-{2-digit-num}` (e.g., TC-LOGIN-01, TC-AUTH-15)
- **TC ID (integration):** `TC-INT-{FEATURE}-{3-digit-num}` (e.g., TC-INT-LOGIN-001)
- **Feature abbreviations:** Consistent per domain (LOGIN, AUTH, PROFILE, REG, etc.)

---

## 4. Coverage Rules & Test Generation Logic

### Coverage Matrix per Test Scenario

Every TS must derive TC covering these dimensions:

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

### Test Case Generation Rules

**Rule 1: Fail-First Ordering**
Sequence TC dalam 1 TS:
1. Empty/null cases (TC-XXX-01 s.d. 0N)
2. Invalid format cases
3. Boundary cases
4. Not found cases
5. Unauthorized cases
6. Forbidden cases
7. Happy path cases (last)

**Rule 2: Test Steps Structure**
Every TC must contain:
```
Test Steps:
- Kondisi awal: [precondition state]
- Step 1: [Arrange] Siapkan data dengan [specific condition]
- Step 2: [Act] Kirim/eksekusi [action] ke sistem

Expected Result:
[HTTP status/error code] - [specific error category].
[User-facing message/validation message].
```

**Rule 3: Specific vs Generic**
- Status code: explicit (400, 401, 403, 404, 500)
- Error message: specific validation message, not generic "error occurred"
- Data values: concrete examples in test steps ("Valid123!", "mahasiswa@mail.com")

**Rule 4: Naming Consistency**
TC ID pattern:
- Unit test: `TC-{FEATURE}-{NUM}`
- Integration test: `TC-INT-{FEATURE}-{NUM}`

Feature name abbreviation consistent per domain.

### Example Coverage Derivation

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

## 5. Audit Mode & Gap Detection

### Audit Triggers

Skill enters audit mode when:
- User asks "audit test coverage", "cek gap testing", "review test completeness"
- User provides existing test files for analysis
- User provides TS/TC documentation but no test code yet

### Audit Process

**Input Artifacts:**
1. Existing test files (code)
2. Test documentation (TS/TC tables) — optional
3. Source code/requirements — optional for inferring expected coverage

**Audit Steps:**

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

## 6. Agent Instructions & Execution Checklist

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

### Agent Execution Checklist

**Mode 1: Generate New Tests**

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

**Mode 2: Audit Existing Tests**

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

### Key Constraints & Anti-Patterns

**MUST enforce:**
- Coverage wajib: 7 dimensions per TS (happy/empty/invalid/boundary/notfound/unauthorized/forbidden)
- Fail-first ordering: negative cases before happy path
- Traceability: every TC links to parent TS
- Specific error messages: not generic "validation failed"
- Gate compliance: user approval at phases 2d and 3e before generating artifacts

**MUST avoid:**
- Skipping coverage dimensions without justification
- Generic test names without TC-ID
- Happy path first ordering
- Generating test code without documentation artifacts
- Proceeding to next phase without user approval

### Cross-Platform Adaptation

**Framework-Specific Syntax Mapping:**

Agent must adapt pseudo-code to target framework:

| Framework | Suite syntax | Test syntax | Assertion |
|-----------|-------------|-------------|-----------|
| Jest/Vitest | `describe()` | `it()` | `expect()` |
| Pytest | `class Test{Feature}` | `def test_{id}_...` | `assert` |
| Go | `func Test{Feature}(t *testing.T)` | `t.Run("{TC-ID}: ...")` | `if/t.Error` |
| .NET xUnit | `public class {Feature}Tests` | `[Fact] public void {TcId}_...` | `Assert.*` |
| Ruby RSpec | `describe` | `it` | `expect().to` |

**Principle remains same:** TS = suite, TC = individual test, fail-first order.

---

## 7. Implementation Considerations

### Skill File Structure

```
traceable-tests/
├── SKILL.md                    # Main skill instructions for agent
├── README.md                   # Human-readable documentation
├── examples/
│   ├── example-ts-scenarios.md
│   ├── example-tc-cases.md
│   └── example-rtm.md
└── templates/
    ├── ts-template.md
    ├── tc-template.md
    └── rtm-template.md
```

### SKILL.md Structure

The main skill file should contain:
1. Skill name and description
2. Trigger conditions (when to use)
3. Scope boundaries (what NOT to do)
4. Workflow overview with gates
5. Coverage rules (7 dimensions)
6. Output formats and examples
7. Agent checklist (step-by-step)
8. Anti-patterns to avoid

### Installation & Portability

**Installation paths:**
- Global: `~/.config/opencode/skills/traceable-tests/` or `~/.agents/skills/traceable-tests/`
- Repo-specific: `.opencode/skills/traceable-tests/` (if project-specific customization needed)

**Cross-agent compatibility:**
- Pure markdown skill definition (no platform-specific code)
- Framework-agnostic principles with adaptation guidance
- Template files use placeholders, not hardcoded syntax

---

## 8. Success Criteria

### Skill is successful if:

1. **Consistency:** Test cases generated across different sessions follow same structure and naming
2. **Coverage:** Every TS derives TC covering all 7 mandatory dimensions
3. **Traceability:** Clear mapping from requirements → TS → TC → test code
4. **Fail-first:** Test code consistently orders negative cases before happy path
5. **Audit accuracy:** Gap detection correctly identifies missing coverage dimensions
6. **Portability:** Skill works across OpenCode, Claude Code, Codex without modification
7. **User validation:** Gates ensure user approval before artifact generation

### Quality Metrics

- **Coverage completeness:** % of TS with all 7 dimensions covered
- **Traceability ratio:** % of implemented tests with valid TC-ID mapping to TS
- **Ordering compliance:** % of test suites following fail-first sequence
- **Documentation sync:** % of test code with corresponding TC documentation

---

## 9. Future Enhancements (Out of Scope for v1)

- Visual coverage dashboard generation
- Integration with test runners for automated gap detection
- AI-powered test case prioritization based on risk
- Support for property-based testing guidance
- Mutation testing integration
- Custom coverage dimension templates per domain

---

## Appendix A: Real-World Example

### Feature: User Authentication

**Phase 1: Test Scenarios**

| Scenario ID | Scenario Description |
|-------------|---------------------|
| TS-AUTH-01  | Login - Autentikasi pengguna dengan email dan password |
| TS-AUTH-02  | Registrasi - Pendaftaran akun pengguna baru |
| TS-AUTH-03  | Logout - Keluar dari sesi aktif |

**Phase 2: Test Cases (TS-AUTH-01 only, abbreviated)**

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong | 1. Siapkan data: email='', password='Valid123!'<br>2. POST /auth/login | 400 - email wajib diisi |
| TS-AUTH-01 | TC-LOGIN-02 | Login dengan password kosong | 1. Siapkan data: email='user@mail.com', password=''<br>2. POST /auth/login | 400 - password wajib diisi |
| TS-AUTH-01 | TC-LOGIN-03 | Login dengan format email salah | 1. Siapkan data: email='invalid', password='Valid123!'<br>2. POST /auth/login | 400 - format email tidak valid |
| TS-AUTH-01 | TC-LOGIN-04 | Login dengan password <8 karakter | 1. Siapkan data: email='user@mail.com', password='123'<br>2. POST /auth/login | 400 - password minimal 8 karakter |
| TS-AUTH-01 | TC-LOGIN-05 | Login dengan email tidak terdaftar | 1. Siapkan data: email='notfound@mail.com', password='Valid123!'<br>2. POST /auth/login | 404 - akun tidak ditemukan |
| TS-AUTH-01 | TC-LOGIN-06 | Login tanpa auth header | 1. Siapkan request tanpa header Authorization<br>2. POST /auth/login | 401 - unauthorized |
| TS-AUTH-01 | TC-LOGIN-07 | Login dengan akun suspended | 1. Siapkan data: email suspended user<br>2. POST /auth/login | 403 - akun dinonaktifkan |
| TS-AUTH-01 | TC-LOGIN-08 | Login dengan kredensial valid | 1. Siapkan data: email='user@mail.com', password='Valid123!'<br>2. POST /auth/login | 200 - return JWT token |

**Phase 3: RTM (abbreviated)**

| Scenario ID | Requirements Description | Test Case IDs |
|-------------|-------------------------|---------------|
| TS-AUTH-01 | Pengguna dapat login dengan email dan password | TC-LOGIN-01 s.d. TC-LOGIN-08 |

**Test Code (Jest):**

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
  
  // ... more negative cases ...
  
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

---

## End of Design Document
