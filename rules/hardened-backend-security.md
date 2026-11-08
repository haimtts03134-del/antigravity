---
trigger: always_on
description: "Bắt buộc tuân thủ 100% tiêu chuẩn bảo mật Hardened Backend Security khi viết hoặc sửa code Backend, API, Database."
---

# Hardened Backend Security Enforcement Rule

## Quy Tắc Bắt Buộc Khi Viết/Sửa Code Backend, API & Database

Bất kỳ khi nào Agent thực hiện viết mới, tái cấu trúc hoặc chỉnh sửa code liên quan đến Backend, API Endpoint, Business Logic, Database Migration / Query hoặc xử lý dữ liệu, **BẮT BUỘC** phải tuân thủ 100% tiêu chuẩn trong skill `hardened-backend-security`:

1. **Ưu Tiên Bảo Mật & Toàn Vẹn Dữ Liệu Tuyệt Đối**:
   - Luôn ưu tiên tính an toàn, toàn vẹn dữ liệu và phòng vệ chiều sâu hơn là độ ngắn gọn của code.
   - **Tuyệt đối cấm** viết tắt hoặc dùng comment giữ chỗ như `// TODO: validate`, `// TODO: add auth`, `/* check IDOR later */`. Mọi đoạn code sinh ra phải hoàn chỉnh bảo mật ngay từ đầu.

2. **Kiểm Định Dữ Liệu Vào (Strict Input Validation) & Chống Mass Assignment**:
   - Mọi endpoint phải validate chặt chẽ `body`, `query`, `params` bằng Schema (Zod/Pydantic/Joi/DTO).
   - Chống Mass Assignment bằng cách chỉ trích xuất đúng các trường trong whitelist được phép cập nhật. Cấm truyền trực tiếp toàn bộ payload (`req.body`, `**kwargs`) vào ORM / Database.

3. **Triệt Tiêu BOLA / IDOR Bằng Query-Level Scoping**:
   - Mọi thao tác đọc, cập nhật hoặc xóa tài nguyên theo ID bắt buộc phải ràng buộc quyền sở hữu (`user_id`, `tenant_id`) ngay tại tầng câu lệnh truy vấn dữ liệu. Không chỉ dựa vào role check ở middleware.

4. **Bọc Database Transaction, Xử Lý Concurrency & Idempotency**:
   - Nghiệp vụ tài chính, trừ kho hàng, biến động số dư, hoặc cập nhật từ 2 bảng trở lên bắt buộc phải bọc trong Transaction.
   - Phòng chống Race Condition bằng câu lệnh nguyên tử (Atomic updates) hoặc Row-level locking (`SELECT FOR UPDATE`).
   - Hỗ trợ Idempotency (kiểm tra `Idempotency-Key`) cho các write API nhạy cảm.

5. **Lọc Dữ Liệu Ra Bằng DTO & Bắt Lỗi An Toàn**:
   - Sử dụng Response DTO / Serializer để loại bỏ hoàn toàn các trường nhạy cảm (`password_hash`, secret, token nội bộ).
   - Xử lý lỗi tập trung; log chi tiết an toàn tại server nhưng **tuyệt đối không bao giờ trả về client** stack trace, câu lỗi SQL raw, tên bảng/cột DB.

6. **Phòng Chống Injection & DoS Qua Phân Trang Giới Hạn Limit**:
   - Chỉ dùng Parameterized Queries / ORM chuẩn (không cộng chuỗi SQL/NoSQL).
   - Mọi endpoint lấy danh sách phải có phân trang với `limit` tối đa được giới hạn cứng (chống cạn kiệt tài nguyên / DoS).

