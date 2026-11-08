---
trigger: always_on
description: "Bắt buộc kiểm soát chống rò rỉ Secret & Dữ liệu Khách hàng (PII) trước khi commit, push GitHub hoặc deploy Vercel."
---

# Secret & User Data Leak Prevention Rule

## Quy Tắc An Ninh Bắt Buộc Khi Thao Tác Git & Triển Khai (Deploy)

Bất kỳ khi nào Agent thực hiện tạo commit (`git commit`), đẩy mã nguồn (`git push`), sao lưu an toàn (GitHub Restore Point) hoặc triển khai lên Vercel (`npx vercel deploy`), **BẮT BUỘC** phải tuân thủ 100% tiêu chuẩn trong skill `secret-leak-prevention`:

1. **Cấu Hình Chặn File Dữ Liệu & Logs Trong `.gitignore` & `.vercelignore`**:
   - Phải đảm bảo các định dạng chứa dữ liệu khách hàng hoặc nội bộ không bao giờ lọt vào Git/Vercel:
     * File bảng biểu & DB: `*.csv`, `*.xlsx`, `*.xls`, `*.sql`, `*.dump`, `*.bak`, `*.sqlite`, `*.db`, `*.jsonl`.
     * Nhật ký & tạm: `*.log`, `logs/`, `temp/`, `tmp/`, `backups/`.
     * Thư mục upload & export: `uploads/`, `public/uploads/`, `storage/`, `media/`, `user_data/`, `exports/`.

2. **Quét PII (Personally Identifiable Information) Bắt Buộc Trước Khi Push/Deploy**:
   - Quét nội dung các file thay đổi để phát hiện:
     * Số điện thoại thật (đầu số Việt Nam `09x`, `03x`, `07x`, `08x`, `05x`, `+84...`).
     * Số CCCD / CMND (chuỗi 9 hoặc 12 số liên quan đến định danh cá nhân).
     * Email cá nhân thật (chỉ cho phép domain test như `@example.com`, `@test.local`, `@mock.dev`).
     * Thông tin tài khoản ngân hàng, số thẻ tín dụng, bảng lương, địa chỉ riêng tư.

3. **Ẩn Danh Hóa Dữ Liệu Test (100% Mock / Dummy Data)**:
   - Dữ liệu test/seed phải được tạo qua thư viện faker hoặc dữ liệu ẩn danh giả lập.
   - Tuyệt đối cấm sao chép dữ liệu thật từ cơ sở dữ liệu khách hàng vào file test trong codebase.

4. **Cơ Chế Hard Stop & Kích Hoạt `grill-me`**:
   - Nếu phát hiện file dữ liệu nhạy cảm hoặc thông tin PII:
     * **Lập tức HỦY** lệnh `git push` và `vercel deploy`.
     * Bắt buộc hiển thị hộp thoại `grill-me` (`ask_question`) thông báo rõ file/dòng vi phạm và đề xuất phương án ẩn danh hoặc ignore.
