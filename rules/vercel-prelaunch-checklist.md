---
trigger: always_on
description: "Bắt buộc thực thi bộ rà soát 3 chốt bảo mật & chất lượng tinh gọn (Code Hygiene, Auth & Access, Input/Upload/RateLimit/Headers) theo cơ chế Step-by-step Hard Stop trước khi Backup GitHub và Deploy Vercel."
---

# Vercel Prelaunch Checklist & Quality Gate Rule

## Quy Tắc Bắt Buộc Trước Khi Backup GitHub & Deploy Lên Vercel

Bất kỳ khi nào người dùng yêu cầu:
- *"kiểm tra trước khi deploy"*, *"audit bảo mật"*, *"rà soát trước khi deploy"*, *"prelaunch checklist"*
- Hoặc hệ thống chuẩn bị kích hoạt quy trình deploy của `vercel-sandbox-deployer`:

Agent **BẮT BUỘC** phải tuân thủ 100% quy trình rà soát 3 Chốt được định nghĩa trong skill `vercel-prelaunch-checklist`:

---

### 1. QUY TẮC THỰC THI TỪNG CHỐT MỘT (STEP-BY-STEP HARD STOP)
- **Nghiêm cấm quét gộp 1 lèo**: Không được gộp cả 3 chốt lại rồi tự ý bỏ qua các cảnh báo.
- **Tuần tự 3 Chốt**:
  1. **CHỐT 1: DỌN DẸP SECRET, LOG & THƯ VIỆN (CODE HYGIENE)**
     - Quét API keys, biến môi trường, rò rỉ `.env` trong `.gitignore`.
     - Xóa toàn bộ debug log in PII, token, password.
     - Ẩn lỗi hệ thống, không để lộ raw stack trace hay câu lỗi SQL.
     - Kiểm tra lỗ hổng thư viện (`npm audit`).
  2. **CHỐT 2: TÀI KHOẢN, PHÂN QUYỀN & DATABASE (AUTH & ACCESS)**
     - Kiểm tra mã hóa mật khẩu (`bcrypt`/`argon2`).
     - Session & Cookie flags (`HttpOnly`, `Secure`, `SameSite`).
     - Chống IDOR/BOLA trên từng API và bảo vệ route Admin ở tầng Server.
  3. **CHỐT 3: INPUT, UPLOAD, RATE LIMIT & VERCEL HEADERS**
     - Validate Server, Parameterized queries / ORM chống SQLi.
     - Whitelist file upload và giới hạn dung lượng $\le$ 5MB.
     - Rate limit cho login và các API gọi mô hình AI.
     - Cấu hình Security Headers trong `vercel.json`/`next.config.js` và kiểm tra a11y UI cơ bản.

---

### 2. BẮT BUỘC HIỂN THỊ KẾT QUẢ QUA HỘP THOẠI `grill-me`
- Tại mỗi Chốt:
  * Xuất bảng báo cáo trực quan các tiêu chí với `[x]` (Đạt) và `[ ]` (Chưa đạt kèm vị trí file:dòng cụ thể).
  * **Kích hoạt công cụ `ask_question`**: Dừng lại hoàn toàn (Hard Stop) hỏi người dùng xác nhận kế hoạch sửa chữa dứt điểm các mục chưa đạt.
  * Chỉ khi toàn bộ các mục trong Chốt đó được xác nhận khắc phục thành công mới được chuyển sang Chốt kế tiếp.

---

### 3. CỔNG CHẶN TRIỂN KHAI (DEPLOYMENT BLOCKER)
- **Tuyệt đối cấm**:
  * Tạo Git Tag `backup-pre-deploy-*`
  * Chạy lệnh `git push`
  * Chạy lệnh `npx vercel deploy`
- Khi và chỉ khi **toàn bộ 3 Chốt** đều đạt trạng thái `[x]` 100% thì Cổng mới được mở khóa để chuyển giao sang quy trình backup và deploy an toàn.

