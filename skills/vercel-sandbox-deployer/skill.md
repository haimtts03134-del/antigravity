---
name: vercel-sandbox-deployer
description: "Quy trình kiểm thử cô lập (Sandbox Verification), sao lưu GitHub an toàn (Restore Point) và deploy bản Preview lên Vercel Sandbox: bắt buộc push Git tag lên GitHub trước khi build/deploy, chạy smoke test và hỗ trợ rollback tức thì."
---

# Skill: Vercel Sandbox Deployer

Skill này chuẩn hóa quy trình triển khai bản xem trước (**Vercel Preview Sandbox**) kết hợp cơ chế bảo vệ mã nguồn tuyệt đối (**GitHub Safe Backup - Restore Point Gate**), bảo đảm mọi phiên bản deploy đều có điểm khôi phục tức thì trên GitHub, vượt qua kiểm tra tĩnh và không làm gãy môi trường build.

---

## 1. Quy Trình Vận Hành 4 Bước Chuẩn Hóa

```
┌───────────────────────────────────────────────┐
│ Bước 0: GitHub Safe Backup (Restore Point)    │
│ (git remote check, tag backup & push GitHub)  │
└───────────────────────┬───────────────────────┘
                        │ (Chỉ tiếp tục khi push thành công 100%)
                        ▼
┌───────────────────────────────────────────────┐
│ Bước 1: Sandbox Verification                  │
│ (Lint, Typecheck, Test, npx vercel build)     │
└───────────────────────┬───────────────────────┘
                        │ (Nếu có lỗi -> Sửa dứt điểm)
                        ▼
┌───────────────────────────────────────────────┐
│ Bước 2: Deploy Preview Sandbox                │
│ (npx vercel link & deploy --yes)              │
└───────────────────────┬───────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────┐
│ Bước 3: Smoke Test, Rollback Info & Báo Cáo   │
│ (curl -I HTTP 200/308, Tag Name & Rollback)   │
└───────────────────────────────────────────────┘
```

---

## BƯỚC 0: GITHUB SAFE BACKUP (BẮT BUỘC TRƯỚC KHI BUILD & DEPLOY)

> [!IMPORTANT]
> **Nguyên Tắc Bất Khả Xâm Phạm**: *"No GitHub Backup = No Vercel Deploy"*. Tuyệt đối không thực hiện bất kỳ lệnh build hay deploy nào nếu source code chưa được đồng bộ và đóng dấu tag an toàn trên GitHub.

1. **Kiểm Tra Kết Nối Git Remote**:
   - Chạy lệnh: `git remote -v`
   - Nếu dự án chưa có remote GitHub:
     * **Dừng lại ngay lập tức** (Hard Stop).
     * Gọi công cụ `ask_question` / hộp thoại tương tác yêu cầu người dùng cung cấp đường link GitHub Repository (`git remote add origin <URL>`).
     * Không tự ý deploy cục bộ khi chưa có remote GitHub.

2. **Tạo Điểm Khôi Phục An Toàn (Restore Point)**:
   - Đảm bảo toàn bộ thay đổi ổn định trước đó đã được commit sạch sẽ:
     ```powershell
     git status --porcelain
     # Nếu còn uncommitted changes cần lưu:
     git add -A
     git commit -m "chore: save working state before vercel deployment"
     ```
   - Tạo Git Tag định danh theo thời gian thực (Timestamp Tag):
     * *PowerShell / Windows*:
       ```powershell
       $TAG_NAME = "backup-pre-deploy-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
       git tag -a $TAG_NAME -m "Safe backup before Vercel deployment"
       ```
     * *Bash / Linux / macOS*:
       ```bash
       TAG_NAME="backup-pre-deploy-$(date +%Y%m%d-%H%M%S)"
       git tag -a "$TAG_NAME" -m "Safe backup before Vercel deployment"
       ```

3. **Đẩy Toàn Bộ Branch & Tag Lên GitHub**:
   - Chạy lệnh đồng bộ:
     ```powershell
     git push origin --all
     git push origin --tags
     ```
   - **Cổng Kiểm Tra Nghiêm Ngặt**:
     * Kiểm tra mã phản hồi của lệnh `git push`.
     * **Chỉ khi GitHub báo đã nhận thành công 100% (exit code 0)** mới được phép chuyển sang Bước 1.
     * Nếu push lên GitHub thất bại (lỗi mạng, sai quyền, xung đột branch), **NGHIÊM CẤM** chạy lệnh build và deploy lên Vercel. Phải dừng lại và khắc phục lỗi Git trước.

---

## BƯỚC 1: SANDBOX VERIFICATION (Kiểm Thử Cô Lập Tại Local)

Sau khi mã nguồn đã được backup an toàn lên GitHub:

1. **Kiểm Tra Tĩnh & Unit Test**:
   - Linter: `npm run lint` (nếu có)
   - Typecheck: `npm run typecheck` hoặc `npx tsc --noEmit`
   - Unit Test: `npm test` hoặc `npm run test -- --passWithNoTests`

2. **Mô Phỏng Môi Trường Build Vercel**:
   - Chạy lệnh: `npx vercel build`
   - Nếu build fail: Đọc log, tự động khắc phục lỗi trong mã nguồn và chạy lại cho đến khi `Build Completed`.

---

## BƯỚC 2: DEPLOY LÊN VERCEL PREVIEW SANDBOX (Triển Khai Bản Xem Trước)

1. **Liên Kết Dự Án**: Kiểm tra `.vercel`, nếu chưa có chạy `npx vercel link --yes`.
2. **Deploy Bản Nháp**:
   - Chạy: `npx vercel deploy --yes`
   - > [!CAUTION]
     > **TUYỆT ĐỐI CẤM** dùng cờ `--prod`. Cờ `--prod` chỉ được phép dùng khi người dùng yêu cầu rõ ràng bằng từ khóa: `"deploy production"`, `"triển khai production"`.

---

## BƯỚC 3: HỖ TRỢ ROLLBACK TỨC THÌ & BÁO CÁO NGHIỆM THU

1. **Smoke Test**:
   - Lấy Preview URL từ terminal.
   - Chạy kiểm tra: `curl.exe -I "<Preview_URL>"` (HTTP 200 / 308).

2. **Báo Cáo Nghiệm Thu & Thông Tin Rollback**:
   Báo cáo trả về cho người dùng bắt buộc bao gồm:
   - **Vercel Preview URL**: Link xem trước để người dùng click trải nghiệm.
   - **GitHub Backup Tag**: Tên Tag điểm khôi phục vừa tạo trên GitHub (ví dụ: `backup-pre-deploy-20261107-160520`).
   - **Lệnh Rollback Khẩn Cấp**:
     ```powershell
     # Chiến lược an toàn: Checkout sang branch mới từ backup tag
     git checkout -b rollback-<tag-name> <tag-name>
     npx vercel deploy --yes
     ```

3. **Cơ Chế Rollback Tự Động Khi Có Yêu Cầu**:
   - Khi người dùng phản hồi bản deploy bị lỗi và yêu cầu: *"quay lại bản cũ"*, *"rollback"*, *"restore backup"*:
     1. Agent tự động tìm Git Tag backup gần nhất (`git tag -l "backup-pre-deploy-*" | Select-Object -Last 1`).
     2. Tạo branch rollback an toàn: `git checkout -b rollback-<timestamp> <backup-tag>`.
     3. Đẩy branch lên GitHub: `git push origin rollback-<timestamp>`.
     4. Chạy `npx vercel deploy --yes` để phục hồi lại bản deploy ổn định trước đó ngay lập tức.
