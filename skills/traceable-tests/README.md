# Traceable Tests Skill

Skill metodologi pengujian 3-layer (**TS → TC → RTM**) untuk konsistensi struktur dan coverage tracking. Skill ini language-agnostic dan portable ke agent apa pun yang mendukung standar Agent Skills.

## Struktur Repo

```
skills/traceable-tests/   ← satu-satunya sumber skill
├── SKILL.md              ← instruksi agent (wajib, ber-YAML frontmatter)
├── README.md             ← dokumentasi untuk manusia
├── examples/             ← contoh dokumen TS, TC, RTM
└── templates/            ← template kosong dengan placeholder
```

Tidak ada salinan per-agent di repo ini. Installer yang membuat link ke direktori skill milik agent yang dipilih.

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

Skill ini mengikuti standar Agent Skills, sehingga bisa dipasang ke agent mana pun (Claude Code, OpenCode, Codex, Cursor, Gemini CLI, GitHub Copilot, dll.) dengan satu perintah:

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

Salin folder `skills/traceable-tests/` ke direktori skill agent tujuan — mis. `.agents/skills/traceable-tests/` (project) atau `~/.agents/skills/traceable-tests/` (global). Isi skill adalah markdown murni, sehingga berjalan sama di semua agent.

## Trigger

Invoke saat user meminta: "write test", "buat test case", "audit test", "cek coverage", menyebutkan TS/TC/RTM, atau memanggil `/traceable-tests`. Pemicu ini juga tertulis di field `description` pada frontmatter `SKILL.md`, karena agent membaca field itu untuk memutuskan kapan skill dipakai.
