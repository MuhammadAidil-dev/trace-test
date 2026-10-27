# Traceable Tests Skill

Skill metodologi pengujian 3-layer (**TS → TC → RTM**) untuk konsistensi struktur dan coverage tracking. Skill ini bersifat language-agnostic dan portable di OpenCode, Claude Code, Codex, dll.

## Fitur Utama

1. **Generate Test Scenario** — dari analisis fitur/domain dengan ID `TS-{DOMAIN}-{NUM}`
2. **Derive Test Cases** — fail-first approach (negative cases duluan, happy path terakhir)
3. **Audit Test Existing** — parse, map ke struktur, dan laporan gap coverage
4. **Generate RTM** — requirement traceability matrix dari requirements → TS → TC

## Cakupan Wajib (7 Dimensi)

Setiap Test Scenario harus derive Test Cases yang mencakup:
1. Happy path
2. Empty/null input
3. Invalid format
4. Boundary cases
5. Not found
6. Unauthorized
7. Forbidden

## Struktur Artefak Dokumentasi

```
docs/testing/
├── scenarios/
│   └── TS-{DOMAIN}-scenarios.md
├── cases/
│   └── TC-{FEATURE}-cases.md
└── traceability/
    └── RTM-{feature}.md
```

## Konvensi Penamaan

- **TS ID:** `TS-{DOMAIN}-{2-digit-num}` (contoh: TS-AUTH-01, TS-PRODUCT-01)
- **TC ID (unit):** `TC-{FEATURE}-{2-digit-num}` (contoh: TC-LOGIN-01)
- **TC ID (integration):** `TC-INT-{FEATURE}-{3-digit-num}` (contoh: TC-INT-LOGIN-001)

## Struktur Test Code (Pseudocode)

```
TEST_SUITE([TS-ID]: [TS-DESCRIPTION]) {
  TEST_CASE([TC-ID]: [CONDITION] - [EXPECTED_RESULT]) {
    // Arrange
    // Act
    // Assert
  }
}
```

TS = outer block (suite), TC = inner test. Urutan: fail-first.

## Contoh & Template

- `examples/` — contoh dokumen TS, TC, RTM (berdasarkan Appendix A spec)
- `templates/` — template kosong dengan placeholders untuk TS, TC, RTM

## Instalasi

Salin folder `traceable-tests/` ke salah satu lokasi berikut sesuai platform yang digunakan:

| Platform | Lokasi global | Lokasi repo-specific |
|----------|--------------|---------------------|
| OpenCode | `~/.config/opencode/skills/traceable-tests/` | `.opencode/skills/traceable-tests/` |
| Claude Code | `~/.claude/skills/traceable-tests/` | `.claude/skills/traceable-tests/` |
| Codex | `~/.codex/skills/traceable-tests/` | `.codex/skills/traceable-tests/` |
| Generic agents | `~/.agents/skills/traceable-tests/` | `.agents/skills/traceable-tests/` |

Skill ini berupa markdown murni tanpa kode spesifik platform, sehingga berjalan sama di semua platform setelah terpasang.

## Trigger

Invoke saat user meminta: "write test", "buat test case", "audit test", "cek coverage", menyebutkan TS/TC/RTM, atau memanggil `/traceable-tests`.