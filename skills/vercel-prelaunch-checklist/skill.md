---
name: vercel-prelaunch-checklist
description: "Bộ rà soát bảo mật & chất lượng 3 giai đoạn (Code Hygiene, Auth & Access, Input/Upload/RateLimit/Headers) theo cơ chế Step-by-step Hard Stop cho dự án chạy trên Vercel trước khi Backup GitHub và Deploy."
---

# Student Vercel Prelaunch Checklist (3-Phase Security & Quality Gate)

Quy trình rà soát bảo mật và chất lượng mã nguồn tinh gọn dành riêng cho các dự án triển khai trên Vercel (không phụ thuộc vào Cloudflare WAF hoặc Sentry ngoại vi).

---

## NGUYÊN TẮC VẬN HÀNH: TỪNG CHỐT MỘT (STEP-BY-STEP HARD STOP)

- **TUYỆT ĐỐI KHÔNG quét 1 lèo rồi bỏ qua**: Agent bắt buộc phải thực thi quét độc lập và hoàn tất từng Chốt theo thứ tự $1 \rightarrow 2 \rightarrow 3$.
- Tại mỗi Chốt, hiển thị bảng đánh dấu trực quan:
  * `[x]`: Hạng mục đã đạt tiêu chuẩn.
  * `[ ]`: Hạng mục vi phạm hoặc chưa đạt (ghi rõ tên file, số dòng và giải pháp khắc phục cụ thể).
- **Hard Stop**: Bắt buộc kích hoạt hộp thoại tương tác `grill-me` (`ask_question`) dừng lại chờ người dùng xác nhận sửa dứt điểm các lỗi của Chốt hiện tại trước khi chuyển sang Chốt kế tiếp.

---

## CHI TIẾT 3 CHỐT KIỂM SOÁT

### CHỐT 1: DỌN DẸP SECRET, LOG & THƯ VIỆN (CODE HYGIENE)
1. **API Key & Biến môi trường**:
   - Mọi API Key (đặc biệt là Gemini API, OpenAI, Database URL/Connection Strings) phải được lưu trong `.env` và chỉ đọc từ phía Server (`process.env.KEY` hoặc Java Environment).
   - Tuyệt đối không để lộ key nhạy cảm ra Frontend (tiền tố `NEXT_PUBLIC_` hoặc `VITE_` chỉ dùng cho các khóa công khai như Client ID/Public Map Key).
   - Kiểm tra file `.env`, `.env.local` bắt buộc phải được khai báo trong `.gitignore`.
2. **Xóa Debug Log**:
   - Quét và loại bỏ tất cả các dòng log debug (`console.log`, `System.out.println`, `print`) đang in ra: tokens, JWT, passwords, request bodies chứa thông tin nhạy cảm hoặc PII người dùng.
3. **Ẩn lỗi hệ thống (Safe Error Handling)**:
   - Client không bao giờ được nhận trực tiếp raw stack trace, câu lệnh SQL lỗi (ví dụ: syntax error SQL Server/Postgres), hoặc đường dẫn thư mục vật lý server (`C:\Users\...` hoặc `/var/www/...`).
   - Mọi lỗi phải được bọc trong Response DTO thân thiện: `{ "success": false, "message": "Đã xảy ra lỗi, vui lòng thử lại sau" }`.
4. **Kiểm tra lỗ hổng thư viện**:
   - Chạy lệnh kiểm tra dependencies: `npm audit --audit-level=high` (hoặc kiểm tra Maven/Gradle dependencies). Phát hiện và cảnh báo các thư viện có CVE nghiêm trọng.

---

### CHỐT 2: TÀI KHOẢN, PHÂN QUYỀN & DATABASE (AUTH & ACCESS)
1. **Mã hóa mật khẩu**:
   - Mật khẩu người dùng trong Database bắt buộc phải được băm (hash) bằng thuật toán an toàn: `bcrypt` (work factor >= 10) hoặc `argon2`. Tuyệt đối cấm lưu Plaintext hoặc MD5/SHA-1 không có salt.
2. **Session & Cookie Security**:
   - Phiên đăng nhập (Session/JWT) phải có thời hạn hết hạn (`maxAge`/`expiresIn`) và phải bị thu hồi/xóa khi người dùng bấm Đăng xuất.
   - Cookie lưu phiên bắt buộc phải bật đủ các cờ:
     * `HttpOnly`: Chống rò rỉ qua XSS (`document.cookie`).
     * `Secure`: Chỉ truyền qua HTTPS (mặc định trên Vercel).
     * `SameSite`: Đặt là `Lax` hoặc `Strict` để chống CSRF.
3. **Chống vượt quyền (BOLA/IDOR & Admin Route Protection)**:
   - Mọi API lấy, sửa hoặc xóa dữ liệu theo ID (ví dụ: `/api/orders/:id` hay `/user/:id`) phải ràng buộc quyền sở hữu ở tầng truy vấn DB (`WHERE id = ? AND user_id = ?`).
   - Bảo vệ toàn bộ endpoint Admin ở tầng Server / Middleware. Tuyệt đối không chỉ ẩn nút bấm trên giao diện mà bỏ ngỏ URL API.
   - Nếu dự án dùng Supabase: kiểm tra đã bật Row Level Security (RLS) cho tất cả các bảng dữ liệu.

---

### CHỐT 3: INPUT, UPLOAD, RATE LIMIT & VERCEL HEADERS
1. **Validate Server & Chống SQLi**:
   - Validate nghiêm ngặt mọi trường dữ liệu đầu vào (`body`, `query`, `params`) ở tầng Server bằng Schema (Zod/Pydantic/DTO/Java Validator).
   - 100% câu lệnh truy vấn CSDL phải dùng Parameterized Queries (PreparedStatement) hoặc ORM chuẩn. Cấm cộng chuỗi SQL.
2. **Giới hạn Upload File an toàn**:
   - Whitelist định dạng file cho phép: chỉ chấp nhận các đuôi file cần thiết (`.jpg`, `.jpeg`, `.png`, `.webp`, `.pdf`). Chặn đứng hoàn toàn `.exe`, `.sh`, `.bat`, `.html`, `.js`, `.jsp`.
   - Giới hạn cứng dung lượng file upload (mặc định tối đa $\le$ 5MB).
3. **Rate Limit (Chống spam & cạn kiệt tài nguyên)**:
   - Áp dụng cơ chế giới hạn tần suất gọi (Rate Limiting) cho các endpoint nhạy cảm: `/api/login`, `/api/register`, `/api/forgot-password` và các API gọi mô hình AI để chống tấn công brute-force và tránh cạn kiệt ngân sách API.
4. **Cấu hình Vercel Security Headers & UI cơ bản**:
   - Cấu hình file `vercel.json` (hoặc `next.config.js`) bổ sung các Security Headers cơ bản:
     * `X-Content-Type-Options: nosniff`
     * `X-Frame-Options: DENY` hoặc `SAMEORIGIN`
     * `Referrer-Policy: strict-origin-when-cross-origin`
     * `Permissions-Policy: camera=(), microphone=(), geolocation=()`
   - Rà soát nhanh Accessibility: Thẻ `<img>` có `alt`, độ tương phản màu văn bản đạt chuẩn WCAG, các nút bấm và form có thể điều hướng mượt mà bằng phím `Tab` và kích hoạt bằng `Enter`.

---

## BÀN GIAO TIẾP THEO
Chỉ sau khi cả **3 Chốt** đều đạt `[x]` 100% và được người dùng xác nhận trên Box, hệ thống mới được phép chuyển giao sang rule `vercel-sandbox-deployer`:
1. Tạo Git Tag Timestamp sao lưu: `backup-pre-deploy-YYYYMMDD-HHMMSS`.
2. Đẩy toàn bộ branch và tag lên GitHub: `git push origin --all && git push origin --tags`.
3. Chạy lệnh deploy Vercel Sandbox: `npx vercel deploy --yes`.

