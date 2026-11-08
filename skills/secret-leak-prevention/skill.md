---
name: secret-leak-prevention
description: "Bộ quy chuẩn và kỹ năng phòng chống rò rỉ Secret & Dữ liệu Khách hàng (PII): cấm lộ API Keys/Tokens, kiểm soát file .env, cấu hình chặn file dữ liệu (*.csv, *.sql, *.db), quét PII (SĐT, CCCD, Email thật, Ngân hàng) trước khi git push hoặc vercel deploy, và kích hoạt grill-me khi vi phạm."
---

# Secret & User Data Leak Prevention Standard

Bộ quy chuẩn và kỹ năng an ninh bắt buộc nhằm ngăn chặn triệt để nguy cơ rò rỉ bí mật công nghệ (API Keys, Tokens, Private Keys) và dữ liệu định danh cá nhân (PII & Customer Data) của người dùng/khách hàng lên các nền tảng công khai (GitHub) hoặc môi trường triển khai (Vercel).

---

## 1. Tổng Quan Cổng Chặn Bảo Vệ Đa Lớp (Defense-in-Depth)

Mọi thao tác liên quan đến tạo commit (`git commit`), đẩy mã nguồn (`git push`), sao lưu an toàn (GitHub Restore Point) hoặc triển khai ứng dụng (`vercel deploy`) đều phải vượt qua 4 lớp kiểm soát nghiêm ngặt:

```
┌────────────────────────────────────────────────────────┐
│ Lớp 1: Chặn Secret & Credentials (API Keys, Tokens)    │
├────────────────────────────────────────────────────────┤
│ Lớp 2: Kiểm Soát Biến Môi Trường (.env, .env.local)    │
├────────────────────────────────────────────────────────┤
│ Lớp 3: Kiểm Soát Định Dạng File Nhạy Cảm (Gitignore)   │
├────────────────────────────────────────────────────────┤
│ Lớp 4: Quét PII & Dữ Liệu Khách Hàng (Zero User Leak)  │
└───────────────────────────┬────────────────────────────┘
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
         [✅ PASS 100%]             [❌ VI PHẠM]
         Cho phép Commit/Deploy    Hủy lệnh ngay lập tức
                                   Bật hộp thoại grill-me
```

---

## 2. Quy Tắc 1: Phòng Chống Rò Rỉ API Keys, Token & Private Keys

- **Tuyệt đối cấm hardcode bí mật**: Không bao giờ ghi trực tiếp API Key, Secret Token, JWT Secret, Database Password, Khóa mã hóa vào mã nguồn.
- **Các mẫu chuỗi nhạy cảm bị cấm**:
  * OpenAI / Anthropic / Gemini API Keys (`sk-...`, `AIzaSy...`).
  * AWS Access Key & Secret (`AKIA...`, `40 ký tự ngẫu nhiên`).
  * GitHub Personal Access Tokens (`ghp_...`, `github_pat_...`).
  * Stripe Secret Keys (`sk_live_...`, `rk_live_...`).
  * Private Keys: `-----BEGIN RSA PRIVATE KEY-----`, `-----BEGIN PRIVATE KEY-----`, `*.pem`, `*.key`.

---

## 3. Quy Tắc 2: Quản Lý Biến Môi Trường Chuẩn Hóa

- **Bảo vệ tuyệt đối các file cấu hình**:
  * Các file chứa giá trị thật: `.env`, `.env.local`, `.env.production`, `.env.development.local` **BẮT BUỘC** phải nằm trong `.gitignore` và `.vercelignore`.
- **Mẫu tham chiếu `.env.example`**:
  * Chỉ duy nhất file `.env.example` được phép đưa vào Git repo.
  * Toàn bộ giá trị trong `.env.example` phải là chuỗi giữ chỗ giả lập (ví dụ: `DATABASE_URL=postgresql://user:password@localhost:5432/mydb`, `API_KEY=your_api_key_here`), tuyệt đối không để lọt key thật vào file mẫu.

---

## 4. Quy Tắc 3: Kiểm Soát Git & Pre-commit Scanning

- Trước khi thực hiện `git add` và `git commit`:
  * Phải kiểm tra danh sách file staged: `git diff --cached --name-only`.
  * Không bao giờ dùng `git add .` hoặc `git add -A` một cách cẩu thả mà không rà soát lại các file sắp commit.
  * Nếu một file nhạy cảm lỡ bị commit vào Git History, không chỉ dùng `git rm` mà phải dùng `git filter-repo` hoặc BFG Repo-Cleaner để xóa sạch lịch sử commit trước khi push.

---

## 5. QUY TẮC 4: CHỐNG RÒ RỈ DỮ LIỆU NGƯỜI DÙNG & KHÁCH HÀNG (ZERO USER DATA LEAK)

> [!CAUTION]
> **Dữ Liệu Người Dùng Là Ranh Giới Đỏ**: Tuyệt đối không bao giờ để rò rỉ bất kỳ tệp dữ liệu khách hàng, cơ sở dữ liệu thật hoặc thông tin định danh cá nhân (PII) nào lên GitHub hay Vercel.

### 5.1. Cấm Đẩy File Chứa Dữ Liệu Khách Hàng / Người Dùng
Tự động cấu hình `.gitignore` và `.vercelignore` chặn toàn bộ các định dạng và thư mục có nguy cơ chứa dữ liệu thật:

- **File bảng biểu & cơ sở dữ liệu**:
  ```gitignore
  # Customer & Database exports
  *.csv
  *.xlsx
  *.xls
  *.sql
  *.dump
  *.bak
  *.sqlite
  *.sqlite3
  *.db
  *.jsonl
  ```
- **File nhật ký & bộ nhớ tạm**:
  ```gitignore
  # Logs & Temporary files
  *.log
  logs/
  temp/
  tmp/
  backups/
  ```
- **Thư mục tệp người dùng tải lên & xuất dữ liệu**:
  ```gitignore
  # User Uploads & Data Directories
  uploads/
  public/uploads/
  storage/
  media/
  user_data/
  exports/
  ```

### 5.2. Quét Dữ Liệu Định Danh Cá Nhân (PII Scanner) Trước Khi Push/Deploy
Trước khi chạy `git commit`, `git push` hoặc `npx vercel deploy`, Agent bắt buộc phải quét nội dung các file thay đổi (bao gồm cả file `.json`, `.js`, `.ts`, `.py`, file seed/mock):

1. **Số điện thoại thật**:
   - Định dạng đầu số Việt Nam: `09[0-9]{8}`, `03[2-9]{8}`, `07[06-9]{8}`, `08[1-9]{8}`, `05[689]{8}`, hoặc `(\+84|84)[0-9]{9}`.
2. **Số CCCD / CMND**:
   - Chuỗi 9 hoặc 12 chữ số gắn với các trường định danh như `cccd`, `cmnd`, `id_card`, `citizen_id`, `identity_card`, `so_cccd`.
3. **Email cá nhân thật của người dùng/nhân viên**:
   - Các email sử dụng domain thật (Gmail, Yahoo, Hotmail, domain công ty nội bộ).
   - **Chỉ cho phép** các domain giả lập chuẩn trong code và test: `@example.com`, `@example.org`, `@test.local`, `@mock.dev`.
4. **Dữ liệu tài chính & cá nhân nhạy cảm**:
   - Số tài khoản ngân hàng (Bank Account Numbers), số thẻ tín dụng/ghi nợ (Visa, Mastercard, CVV/CVC).
   - Bảng thông tin tiền lương, thu nhập cá nhân.
   - Địa chỉ nhà riêng chi tiết (số nhà, tên đường, khu phố gắn liền với tên cá nhân).

### 5.3. Quy Tắc Dữ Liệu Test (Mock / Seed Data Anonymization)
- Khi viết unit test hoặc tạo dữ liệu khởi tạo (database seed):
  * **BẮT BUỘC** sử dụng thư viện sinh dữ liệu giả (như `@faker-js/faker`, `faker` Python) hoặc dữ liệu ẩn danh hoàn toàn:
    - Tên: `"Người Dùng A"`, `"Nguyen Van Test"`, `"Test User 01"`.
    - SĐT giả định: `"0900000001"`, `"0900000002"`.
    - Email: `"user1@example.com"`, `"admin@test.local"`.
  * **Tuyệt đối cấm**: Export một phần hoặc toàn bộ dữ liệu từ Database production của khách hàng vào file test/seed trong codebase.

### 5.4. Hard Stop & Cảnh Báo Qua `grill-me`
Nếu phát hiện bất kỳ file danh sách khách hàng (`.csv`, `.xlsx`, `.sql`...) hoặc thông tin cá nhân (SĐT, CCCD, Email thật, Thẻ ngân hàng) nằm trong danh sách chuẩn bị đưa lên GitHub hoặc Vercel:

1. **HỦY NGAY LẬP TỨC** lệnh `git push` và `vercel deploy`.
2. **Bật Hộp Thoại `grill-me` Cảnh Báo**:
   - Chỉ rõ chính xác: Tên file, số dòng, và loại dữ liệu nhạy cảm phát hiện được (ví dụ: *"Phát hiện 12 số điện thoại thật và 3 số CCCD tại file `data/seed.ts:L45-L60`"*).
   - Đưa ra các giải pháp khắc phục qua câu hỏi trắc nghiệm:
     * `(Recommended) Tự động ẩn danh hóa (masking) dữ liệu bằng Faker.js / dummy data`
     * `Đưa file vi phạm vào .gitignore và loại bỏ khỏi Git stage`
     * `Xóa bỏ hoàn toàn file chứa dữ liệu thật khỏi repository`

---

## 6. Checklist Kiểm Định Trước Khi Commit / Push / Deploy

1. [ ] Đã kiểm tra `.gitignore` và `.vercelignore` đã chứa các định dạng `.csv`, `.xlsx`, `.sql`, `.db`, `uploads/`, `logs/` chưa?
2. [ ] Đã quét nội dung các file thay đổi để đảm bảo không chứa API Key, Private Key, Token bí mật chưa?
3. [ ] Đã quét PII để đảm bảo không có Số điện thoại, CCCD/CMND, Email cá nhân thật của khách hàng chưa?
4. [ ] Dữ liệu mock/seed có sử dụng 100% domain giả lập (`@example.com`) và tên giả lập không?
5. [ ] Nếu phát hiện vi phạm, đã dừng lại và kích hoạt hộp thoại `grill-me` báo cáo cho người dùng chưa?
