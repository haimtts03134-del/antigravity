---
trigger: always_on
description: "Bắt buộc tự động kích hoạt quy trình Spec Kit kết hợp tinh hoa kỷ luật kỹ thuật Superpowers (TDD, Systematic Debugging, Acceptance Verification) cho mọi yêu cầu code."
---
# Mandatory Spec Kit & Superpowers Engineering Enforcement

Quy trình Spec Kit kết hợp kỷ luật kỹ thuật từ Superpowers là động cơ thực thi chuẩn mực, **bắt đầu vận hành ngay sau khi vượt qua Cổng phê duyệt `grill-me` (Phase 0 từ rule `confirm-before-action`)**:

---

## 1. VÒNG ĐỜI PHÁT TRIỂN TÍNH NĂNG (FEATURE LIFECYCLE)

1. **Phase 0 (Phê duyệt Tiền đề - Gate 0)**: 
   - Thống nhất yêu cầu, kiến trúc và ranh giới an ninh qua hộp thoại trắc nghiệm `ask_question` (`grill-me`).
   - Dừng lại hoàn toàn (Hard stop) chờ người dùng chọn phê duyệt trên box mới chuyển tiếp sang Phase 1.

2. **Phase 1 (Đặc tả Yêu cầu - Specification)**: 
   - Tự động chạy `@.agents/skills/speckit-specify/SKILL.md` để sinh tài liệu đặc tả chức năng (`.specify/specs/.../spec.md`).
   - Định nghĩa rõ ràng danh sách **Tiêu chí Nghiệm thu (Acceptance Criteria - AC)** có thể kiểm chứng được.

3. **Phase 2 (Kế hoạch Kiến trúc - Architectural Plan)**: 
   - Chạy `@.agents/skills/speckit-plan/SKILL.md` để lập kế hoạch kiến trúc chi tiết, xác định cấu trúc module và luồng dữ liệu.

4. **Phase 3 (Phân rã Task Siêu Nhỏ - Bite-sized Tasks)**: 
   - Chạy `@.agents/skills/speckit-tasks/SKILL.md` để chia nhỏ đầu việc thành các nhiệm vụ độc lập, thực thi trong 2 đến 5 phút.

5. **Phase 4 (Thực thi Code & Kỷ luật TDD - Test-Driven Development)**: 
   - Chạy `@.agents/skills/speckit-implement/SKILL.md` tuân thủ 100% tiêu chuẩn `hardened-backend-security` và `ui-ux-pro-architect`.
   - **Kỷ luật TDD (Superpowers Red-Green-Refactor)**: 
     * Với mọi logic nghiệp vụ mới, DAO, Service, Servlet API hay thuật toán: **BẮT BUỘC viết Unit Test / Integration Test trước**.
     * Chạy bài test xác nhận kết quả **FAIL (Red)** để chứng minh test hoạt động chính xác và không có false-positive.
     * Sau đó mới tiến hành viết implementation code tối thiểu để bài test **PASS (Green)**.
     * Tối ưu hóa, dọn dẹp mã nguồn theo chuẩn Clean Code (**Refactor**).
     * Tuyệt đối cấm viết code logic trước rồi mới bổ sung test đối phó sau.

6. **Phase 5 (Nghiệm thu Đa Chiều & Chống Thoái Lui - Verification Gate)**: 
   - Chạy `@.agents/skills/speckit-converge/SKILL.md` trước khi bàn giao hoặc kích hoạt `vercel-sandbox-deployer`.
   - Chạy lại **toàn bộ Test Suite** để đảm bảo code mới không làm hỏng tính năng cũ (Zero Regression).
   - Tự động đối soát và đánh dấu hoàn thành từng mục trong danh sách Acceptance Criteria đã lập ở Phase 1.

---

## 2. QUY TRÌNH SỬA LỖI HỆ THỐNG (SYSTEMATIC ROOT-CAUSE DEBUGGING)

Khi nhận yêu cầu sửa lỗi (bug fix), **TUYỆT ĐỐI CẤM** đoán mò (guessing) hoặc sửa hú họa phần ngọn. Bắt buộc tuân thủ chu trình:

1. **Assess & Reproduce (Đánh giá & Tái hiện)**: 
   - Chạy `@.agents/skills/speckit-bug-assess/SKILL.md`.
   - Bắt buộc tạo bài test tái hiện lỗi (Reproduction Test) hoặc kịch bản chạy thử để xác nhận lỗi xảy ra nhất quán.
2. **Root-Cause Investigation (Truy tìm Nguyên nhân Gốc rễ)**: 
   - Truy ngược stack trace, dữ liệu đầu vào và trạng thái hệ thống để tìm chính xác điểm gãy của logic, không dừng lại ở việc bắt ngoại lệ bề mặt (catch-all).
3. **Fix & Verify (Sửa triệt để & Kiểm chứng)**: 
   - Chạy `@.agents/skills/speckit-bug-fix/SKILL.md` để sửa code tại gốc rễ nguyên nhân.
   - Chạy `@.agents/skills/speckit-bug-test/SKILL.md`: Bài test tái hiện lỗi ở bước 1 phải chuyển sang trạng thái **PASS (Green)**.
   - Chạy kiểm tra hồi quy toàn bộ hệ thống để bảo đảm không sinh ra bug phụ.

---

## 3. NGUYÊN TẮC CỐT LÕI
Tuyệt đối không viết hoặc sửa code trực tiếp nếu chưa đi qua đầy đủ các cổng kiểm soát kỹ thuật ở trên.
