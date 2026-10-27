<div align="center">

# TRACEABLE-TESTS

**Dokumentasi pengujian yang traceable untuk agent AI apa pun.**

Test Scenario → Test Case → Requirements Traceability Matrix

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-8A2BE2.svg)](https://agentskills.io)
[![Node.js](https://img.shields.io/badge/Node.js-22.20%2B-339933.svg)](#persyaratan)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#kontribusi)

</div>

## Daftar Isi

[Ikhtisar](#ikhtisar) · [Fitur](#fitur) · [Quick Start](#quick-start) · [Cara Kerja](#cara-kerja) · [Contoh Keluaran](#contoh-keluaran) · [Kompatibilitas](#kompatibilitas) · [Struktur Repo](#struktur-repo) · [Batasan](#batasan) · [FAQ](#faq) · [Kontribusi](#kontribusi) · [Lisensi](#lisensi) · [Referensi](#referensi)

## Ikhtisar

Repo ini **bukan aplikasi, bukan test runner, dan bukan framework**. Isinya sebuah *skill* — instruksi markdown yang mengajari agent AI cara menyusun pengujian secara konsisten: dari skenario, turun ke test case, sampai matriks keterlacakan requirement.

Saat kamu meminta agent *"buatkan test case untuk fitur login"*, tanpa panduan agent mengarang format baru setiap sesi — hanya happy path, tanpa kasus keamanan, tanpa kaitan ke requirement. Skill ini mengunci prosesnya sehingga hasilnya sama setiap kali, di agent mana pun.

Semua isi repo berupa markdown: tidak ada kode yang dijalankan, tidak ada dependency, tidak ada build step.

## Fitur

| Fitur | Penjelasan |
|---|---|
| **Alur 3 layer** | Test Scenario (TS) → Test Case (TC) → RTM, dengan ID yang saling terhubung |
| **7 dimensi coverage wajib** | Happy path, empty/null, invalid format, boundary, not found, unauthorized, forbidden |
| **Fail-first ordering** | Negative case dinomori lebih dulu, happy path terakhir |
| **Dua gerbang persetujuan** | Kamu menyetujui daftar TS dan kelengkapan TC sebelum artefak dibuat |
| **Mode audit** | Mem-parse test yang sudah ada, melaporkan dimensi coverage yang hilang |
| **Deteksi gap RTM** | Menemukan TC *orphaned* (ada di dokumen, belum diimplementasikan) dan test tak terdokumentasi |
| **Agnostik framework** | Jest/Vitest, Pytest, Go, xUnit, RSpec — agent menyesuaikan sintaks, prinsipnya tetap |
| **Agnostik agent** | Standar [Agent Skills](https://agentskills.io); berjalan sama di 75+ agent |

## Quick Start

```bash
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests
```

Setelah terpasang, cukup bicara natural ke agent-mu:

```
buatkan test case untuk fitur login
audit test yang sudah ada, cek coverage-nya
review test completeness modul pembayaran
buatkan TS/TC/RTM untuk fitur registrasi
```

Agent juga mengenali penyebutan **TS**, **TC**, atau **RTM** secara eksplisit. Jika agent-mu mendukung pemanggilan skill langsung, gunakan `/traceable-tests`.

### Persyaratan

- **Node.js ≥ 22.20.0** untuk CLI `skills`
- Agent yang mendukung standar Agent Skills

### Opsi Instalasi

```bash
# Pilih skill & agent tujuan secara interaktif
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests

# Non-interaktif ke semua agent yang terdeteksi
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests --all

# Global (tersedia di semua project) untuk agent tertentu
npx skills add MuhammadAidil-dev/trace-test --skill traceable-tests -g -a claude-code -a codex -a opencode

# Dari checkout lokal
npx skills add ./skills/traceable-tests
```

**Tanpa instalasi**, pakai sekali jalan:

```bash
npx skills use MuhammadAidil-dev/trace-test@traceable-tests
```

**Instalasi manual**: salin `skills/traceable-tests/` ke direktori skill agent tujuan, mis. `.agents/skills/traceable-tests/` (project) atau `~/.agents/skills/traceable-tests/` (global).

CLI memasang ke direktori milik masing-masing agent — `.agents/skills/` untuk OpenCode, Codex, Cursor, dan Copilot, atau `.claude/skills/` untuk Claude Code — memakai symlink ke satu sumber di `skills/traceable-tests/` (atau copy, bila lingkunganmu tidak mendukung symlink, seperti Windows tanpa symlink privilege).

## Cara Kerja

Skill bekerja dalam tiga fase, dengan dua gerbang persetujuan supaya kamu tetap mengontrol hasilnya.

| Fase | Proses | Keluaran | Gerbang |
|:---:|---|---|:---:|
| **1** | Analisis batas fitur → abstraksi skenario | Daftar `TS-{DOMAIN}-{NN}` + deskripsi | Ya |
| **2** | Derivasi TC per TS, fail-first | Tabel TC: ID, langkah, expected result | Ya |
| **3** | Pemetaan requirement dan audit coverage | RTM + laporan gap | — |

### 7 Dimensi Coverage

Setiap Test Scenario wajib menurunkan Test Case untuk seluruh dimensi berikut:

| # | Dimensi | Contoh |
|:---:|---|---|
| 1 | Happy path | Login kredensial valid → 200 + token |
| 2 | Empty/null input | Email kosong → 400, "email wajib diisi" |
| 3 | Invalid format | `email='invalid'` → 400, "format email tidak valid" |
| 4 | Boundary | Password < 8 karakter; nama 256 karakter |
| 5 | Not found | Update produk dengan ID tidak ada → 404 |
| 6 | Unauthorized | Request tanpa token → 401 |
| 7 | Forbidden | Role viewer mencoba hapus → 403 |

### Artefak yang Dihasilkan

```
docs/testing/
├── scenarios/      TS-{DOMAIN}-scenarios.md
├── cases/          TC-{FEATURE}-cases.md
└── traceability/   RTM-{feature}.md
```

Kode test mengikuti struktur bersarang — TS sebagai suite luar, TC sebagai test di dalamnya:

```javascript
describe('TS-AUTH-01: Login - Autentikasi dengan email dan password', () => {

  it('TC-LOGIN-01: Login dengan email kosong - 400, email wajib diisi', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({ email: '', password: 'Valid123!' });

    expect(response.status).toBe(400);
    expect(response.body.message).toContain('email wajib diisi');
  });

  // ... negative cases lain ...

  it('TC-LOGIN-08: Login dengan kredensial valid - 200, return JWT token', async () => {
    const response = await request(app)
      .post('/auth/login')
      .send({ email: 'user@mail.com', password: 'Valid123!' });

    expect(response.status).toBe(200);
    expect(response.body.token).toBeDefined();
  });
});
```

### Konvensi Penamaan

| Artefak | Pola | Contoh |
|---|---|---|
| Test Scenario | `TS-{DOMAIN}-{2 digit}` | `TS-AUTH-01` |
| Test Case unit | `TC-{FEATURE}-{2 digit}` | `TC-LOGIN-01` |
| Test Case integrasi | `TC-INT-{FEATURE}-{3 digit}` | `TC-INT-LOGIN-001` |

## Contoh Keluaran

**Test Scenario**

| Scenario ID | Scenario Description |
|---|---|
| TS-AUTH-01 | Login — Autentikasi pengguna dengan email dan password |

**Test Case** (urut fail-first, dipotong)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|---|---|---|---|---|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong | 1. Siapkan data: email='', password='Valid123!'<br>2. POST /auth/login | 400 - email wajib diisi |
| TS-AUTH-01 | TC-LOGIN-03 | Login dengan format email salah | 1. Siapkan data: email='invalid', password='Valid123!'<br>2. POST /auth/login | 400 - format email tidak valid |
| TS-AUTH-01 | TC-LOGIN-06 | Login tanpa auth header | 1. Siapkan request tanpa header Authorization<br>2. POST /auth/login | 401 - unauthorized |
| TS-AUTH-01 | TC-LOGIN-08 | Login dengan kredensial valid | 1. Siapkan data: email='user@mail.com', password='Valid123!'<br>2. POST /auth/login | 200 - return JWT token |

**Requirements Traceability Matrix**

| Scenario ID | Requirements Description | Test Case IDs |
|---|---|---|
| TS-AUTH-01 (Autentikasi) | Pengguna dapat login ke sistem | TC-LOGIN-01 s.d. TC-LOGIN-08 |

<details>
<summary><b>Mode audit — contoh laporan gap</b></summary>

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

</details>

Contoh lengkap ada di [`skills/traceable-tests/examples/`](skills/traceable-tests/examples/), dan versi kosongnya di [`templates/`](skills/traceable-tests/templates/).

## Kompatibilitas

Skill mengikuti [spesifikasi Agent Skills](https://agentskills.io), jadi formatnya sama di semua agent pendukung. Beberapa contoh flag `--agent`:

| Agent | Flag | Direktori project |
|---|---|---|
| Claude Code | `claude-code` | `.claude/skills/` |
| OpenCode | `opencode` | `.agents/skills/` |
| Codex | `codex` | `.agents/skills/` |
| Cursor | `cursor` | `.agents/skills/` |
| GitHub Copilot | `github-copilot` | `.agents/skills/` |
| Gemini CLI | `gemini-cli` | `.agents/skills/` |

> **Prinsip tetap sama di semua agent:** skill memberi struktur dan prinsip, agent menyesuaikannya ke stack-mu — bukan mengikatmu ke framework atau vendor agent tertentu.

## Struktur Repo

```
.
├── LICENSE                          # MIT
├── README.md                        # dokumen ini
└── skills/
    └── traceable-tests/             # satu-satunya sumber skill
        ├── SKILL.md                 # instruksi agent (YAML frontmatter + 14 bagian)
        ├── README.md                # pusat informasi lengkap untuk manusia
        ├── examples/                # contoh dokumen TS, TC, RTM
        └── templates/               # template kosong dengan placeholder
```

Tidak ada salinan per-agent di repo ini. Installer yang membuat link ke direktori skill milik agent yang kamu pilih, sehingga tidak ada berkas yang perlu dijaga sinkron secara manual.

## Batasan

Skill ini sengaja dibatasi pada perancangan, pendokumentasian, dan pengauditan test.

| Bukan cakupan | Alasan |
|---|---|
| Menjalankan test | Tugas test runner, bukan perancangan test |
| Menulis kode produksi agar test lolos | Itu fase implementasi TDD |
| Performance/load testing | Domain berbeda |
| Mock/stub generation | Di luar cakupan |
| Detail alur otomasi E2E UI | Di luar cakupan |

## FAQ

<details>
<summary><b>Apakah skill ini menjalankan test saya?</b></summary>

Tidak. Skill menghasilkan dokumen pengujian dan struktur kode test. Eksekusi tetap tugas test runner di proyekmu.

</details>

<details>
<summary><b>Apakah saya harus memakai Jest?</b></summary>

Tidak. Skill memetakan prinsipnya ke Jest/Vitest, Pytest, Go, xUnit, dan RSpec — dan karena framework-agnostic, agent bisa menyesuaikan ke stack lain dengan prinsip yang sama.

</details>

<details>
<summary><b>Apakah ini menggantikan TDD?</b></summary>

Tidak. Ini perancangan dan dokumentasi test, bukan siklus red-green-refactor.

</details>

<details>
<summary><b>Bisakah dipakai tanpa agent AI?</b></summary>

Bisa. Template di `templates/` adalah markdown biasa yang bisa diisi manual. Nilai utama skill ini ada pada otomatisasi dan konsistensinya saat dijalankan agent.

</details>

## Kontribusi

Repo ini hanya berisi markdown — tidak ada build, dependency, atau test suite.

1. Ubah berkas di `skills/traceable-tests/`. **Jangan** membuat salinan di `.agents/` atau `.opencode/`; skill sengaja hanya punya satu sumber.
2. Jika mengubah frontmatter `SKILL.md`, pastikan skill masih terdeteksi:
   ```bash
   npx skills add . --list   # harus menemukan 1 skill: traceable-tests
   ```
3. Jaga agar `SKILL.md` dan [README skill](skills/traceable-tests/README.md) tetap sinkron: `SKILL.md` berisi instruksi yang dijalankan agent, README berisi penjelasan lengkapnya untuk manusia.
4. Buka pull request dengan penjelasan singkat alasan perubahan.

Menambah skill baru? Buat `skills/<nama-skill>/SKILL.md` dengan YAML frontmatter berisi `name` dan `description`.

## Lisensi

Dirilis dengan **MIT License** — bebas dipakai, diubah, dan didistribusikan, termasuk untuk keperluan komersial, dengan syarat menyertakan notice hak cipta. Lihat [LICENSE](LICENSE).

## Referensi

- [Agent Skills specification](https://agentskills.io) — standar format skill
- [skills.sh](https://skills.sh) — direktori skill dan CLI `npx skills`
- [SKILL.md](skills/traceable-tests/SKILL.md) — instruksi yang dijalankan agent
- [README skill](skills/traceable-tests/README.md) — pusat informasi lengkap: workflow, aturan coverage, mode audit, checklist, metrik, dan roadmap
