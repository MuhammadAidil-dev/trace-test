# Contoh: Test Cases (TC)

**File:** `docs/testing/cases/TC-LOGIN-cases.md`

## Test Cases: TS-AUTH-01 (Login - Autentikasi)

| Scenario ID | Test Case ID | Scenario Description | Test Steps | Expected Result |
|-------------|--------------|---------------------|-----------|----------------|
| TS-AUTH-01 | TC-LOGIN-01 | Login dengan email kosong.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='', password='Valid123!'<br>2. POST /auth/login | 400 - email wajib diisi |
| TS-AUTH-01 | TC-LOGIN-02 | Login dengan password kosong.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='user@mail.com', password=''<br>2. POST /auth/login | 400 - password wajib diisi |
| TS-AUTH-01 | TC-LOGIN-03 | Login dengan format email salah.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='invalid', password='Valid123!'<br>2. POST /auth/login | 400 - format email tidak valid |
| TS-AUTH-01 | TC-LOGIN-04 | Login dengan password kurang dari 8 karakter.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='user@mail.com', password='123'<br>2. POST /auth/login | 400 - password minimal 8 karakter |
| TS-AUTH-01 | TC-LOGIN-05 | Login dengan email tidak terdaftar.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='notfound@mail.com', password='Valid123!'<br>2. POST /auth/login | 404 - akun tidak ditemukan |
| TS-AUTH-01 | TC-LOGIN-06 | Login tanpa auth header.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan request tanpa header Authorization<br>2. POST /auth/login | 401 - unauthorized |
| TS-AUTH-01 | TC-LOGIN-07 | Login dengan akun suspended.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email milik akun yang suspended<br>2. POST /auth/login | 403 - akun dinonaktifkan |
| TS-AUTH-01 | TC-LOGIN-08 | Login dengan kredensial valid.<br><br>Kondisi awal: Belum ada sesi masuk sebelumnya. | 1. Siapkan data: email='user@mail.com', password='Valid123!'<br>2. POST /auth/login | 200 - return JWT token |

**Urutan fail-first:** empty/null → invalid format → boundary → not found → unauthorized → forbidden → happy path (terakhir).