# Template: Test Cases (TC)

**File target:** `docs/testing/cases/TC-{FEATURE}-cases.md`

## Test Cases: TS-{DOMAIN}-01 ({Nama Fitur})

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-{DOMAIN}-01 | TC-{FEATURE}-01 | {Kondisi negatif/empty}<br><br>Kondisi awal: {precondition}. | 1. [Arrange] Siapkan data dengan {kondisi spesifik}.<br>2. [Act] Kirim/eksekusi {action} ke sistem. | {Status code} - {pesan validasi spesifik}. |
| TS-{DOMAIN}-01 | TC-{FEATURE}-02 | {Kondisi invalid format}<br><br>Kondisi awal: {precondition}. | 1. [Arrange] Siapkan data dengan format salah.<br>2. [Act] Kirim/eksekusi {action} ke sistem. | {Status code} - {pesan validasi spesifik}. |
| TS-{DOMAIN}-01 | TC-{FEATURE}-NN | {Kondisi happy path}<br><br>Kondisi awal: {precondition}. | 1. [Arrange] Siapkan data valid.<br>2. [Act] Kirim/eksekusi {action} ke sistem. | {Status code} - {hasil sukses spesifik}. |

**Catatan:** Urutkan TC dalam fail-first order (empty/null → invalid format → boundary → not found → unauthorized → forbidden → happy path terakhir). Setiap TC harus link ke parent TS.