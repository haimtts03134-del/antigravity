---
trigger: always_on
description: "Quy tắc bắt buộc sao lưu GitHub (Restore Point Gate), rà soát Prelaunch Checklist 3 chốt, kiểm thử Sandbox và kết nối đề xuất deploy Vercel Preview qua grill-me."
---

# Vercel Sandbox & GitHub Safe Backup Deployment Rule

## 1. Nguyên Tắc Bất Khả Xâm Phạm: "No GitHub Backup = No Vercel Deploy"

Bất kỳ khi nào Agent thực hiện lệnh build và deploy lên Vercel:

1. **Bắt Buộc Tạo Điểm Khôi Phục (Restore Point) Trên GitHub Trước Khi Deploy**:
   - Phải kiểm tra kết nối `git remote -v`. Nếu chưa có remote GitHub, phải dừng lại và yêu cầu người dùng cấu hình repo trước.
   - Bắt buộc tạo Git Tag định danh theo thời gian thực: `backup-pre-deploy-YYYYMMDD-HHMMSS`.
   - Bắt buộc đẩy toàn bộ branch và tag lên GitHub: `git push origin --all && git push origin --tags`.
   - **Chỉ khi GitHub báo thành công 100% (exit code 0)** mới được phép tiến hành kiểm thử cục bộ và deploy Vercel. Nếu push GitHub lỗi, **TUYỆT ĐỐI CẤM** chạy deploy Vercel.

2. **Hỗ Trợ Rollback Tức Thì Khi Có Sự Cố**:
   - Khi người dùng phản hồi bản deploy mới bị lỗi hoặc yêu cầu *"quay lại bản cũ"*, *"rollback"*, *"restore"* :
     * Tự động xác định Git Tag backup gần nhất (`backup-pre-deploy-*`).
     * Checkout tag sang branch mới an toàn: `git checkout -b rollback-<timestamp> <backup-tag>`.
     * Đẩy branch lên GitHub: `git push origin rollback-<timestamp>`.
     * Thực hiện redeploy ngay lập tức lên Vercel: `npx vercel deploy --yes`.

---

## 2. Kết Nối Quy Trình Với Prelaunch Checklist & `grill-me`

Sau khi Agent hoàn tất viết/sửa code theo kế hoạch đã được phê duyệt:

1. **Thực Thi Rà Soát 3 Chốt (Prelaunch Quality Gate)**:
   - Bắt buộc vượt qua 100% các tiêu chí trong rule `vercel-prelaunch-checklist` (Chốt 1: Code Hygiene $\rightarrow$ Chốt 2: Auth & Access $\rightarrow$ Chốt 3: Input, Upload, Rate Limit, Headers).
   - Chạy kiểm tra tĩnh (`npm run lint`, `tsc --noEmit`, `npm test`) và mô phỏng build bằng `npx vercel build`.
   - Tự động sửa dứt điểm lỗi build nếu có.

2. **Kích Hoạt Hộp Thoại `grill-me` Đề Xuất Deploy Preview**:
   - Khi Code, Prelaunch Checklist và Local Build đã pass 100%, Agent **bắt buộc** gọi công cụ `ask_question` để hỏi người dùng:
     * *Câu hỏi*: *"Code, Prelaunch Checklist và Local Build đã pass 100%, bạn có muốn backup GitHub và deploy lên Vercel Preview Sandbox để lấy link test trực tiếp không?"*
     * *Options*:
       - `(Recommended) Có, backup GitHub và deploy lên Vercel Preview Sandbox`
       - `Không, chỉ lưu code tại local và hoàn tất tác vụ`
   - **Ngoại lệ kích hoạt tự động**: Khi người dùng yêu cầu trực tiếp bằng các từ khóa như: *"test trên sandbox"*, *"deploy thử"*, *"preview lên vercel"*, Agent tự động thực thi tuần tự quy trình (Prelaunch Checklist -> Backup GitHub -> Sandbox Verify -> Deploy Preview -> Smoke test) mà không cần hỏi lại.

---

## 3. Quy Tắc An Toàn Môi Trường (Environment Safety Gate)

- **Cấm Tuyệt Đối Tự Ý Deploy Lên Production (`--prod`)**:
  * Mọi lệnh deploy tự động từ quy trình kiểm thử hoặc đề xuất xem trước **CHỈ ĐƯỢC PHÉP** chạy `npx vercel deploy --yes` (Preview Sandbox).
  * Nghiêm cấm tuyệt đối việc thêm cờ `--prod` vào lệnh deploy trừ khi người dùng chỉ định rõ ràng bằng văn bản với từ khóa: `"deploy production"` hoặc `"triển khai production"`.
- **Bắt Buộc Smoke Test & Báo Cáo Thông Tin Rollback**:
  * Luôn chạy kiểm tra HTTP Status bằng `curl` đối với Preview URL trước khi báo cáo kết quả cho người dùng.
  * Báo cáo bắt buộc kèm theo **Tên Tag GitHub Backup** và **câu lệnh Rollback khẩn cấp**.
