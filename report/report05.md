# Report 05 – AES Frontend: React UI Mã hóa/Giải mã AES

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | AESPage.tsx có 2 tab: "🔒 Mã hóa" và "🔓 Giải mã" | ⏳ Chưa implement | |
| 2 | FileDropZone hỗ trợ drag & drop + click chọn file | ⏳ Chưa implement | |
| 3 | Chỉ nhận file video (validate Zod VideoFileSchema) | ⏳ Chưa implement | |
| 4 | Hiển thị thông tin file đã chọn (tên, kích thước) | ⏳ Chưa implement | |
| 5 | Radio chọn key mode: auto / manual | ⏳ Chưa implement | |
| 6 | Nút "Tạo key mới" gọi GET /api/aes/generate-key | ⏳ Chưa implement | |
| 7 | Progress bar cập nhật realtime (onUploadProgress) | ⏳ Chưa implement | |
| 8 | Trạng thái upload: "Đang tải lên...", "Đang mã hóa...", "Hoàn thành!" | ⏳ Chưa implement | |
| 9 | Download 2 file sau mã hóa: .enc và .key | ⏳ Chưa implement | |
| 10 | Cảnh báo màu vàng về lưu key sau mã hóa | ⏳ Chưa implement | |
| 11 | Nút "Mã hóa video khác" reset form | ⏳ Chưa implement | |
| 12 | Tab Giải mã: upload .enc + upload/nhập .key | ⏳ Chưa implement | |
| 13 | Giải mã sai key → alert đỏ rõ ràng | ⏳ Chưa implement | |
| 14 | Zustand store (useAESStore) đủ state theo spec | ⏳ Chưa implement | |
| 15 | Zod validation chạy trước khi gọi API | ⏳ Chưa implement | |
| 16 | Nút disabled khi đang xử lý (status !== 'idle') | ⏳ Chưa implement | |
| 17 | Toàn bộ text tiếng Việt | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Mở trang /aes – hiện 2 tab | ⏳ | |
| T02 | Drag & drop file .mp4 vào FileDropZone | ⏳ | |
| T03 | Upload file .txt → hiện lỗi validation ngay | ⏳ | |
| T04 | Nhấn "Tạo key mới" → key base64 hiện trong input | ⏳ | |
| T05 | Mã hóa với auto key → progress bar chạy | ⏳ | |
| T06 | Download file .enc thành công | ⏳ | |
| T07 | Download file .key thành công (JSON có key_b64) | ⏳ | |
| T08 | Cảnh báo màu vàng hiện sau mã hóa | ⏳ | |
| T09 | Tab Giải mã: upload .enc + .key đúng → download video | ⏳ | |
| T10 | Giải mã sai key → alert đỏ "Giải mã thất bại" | ⏳ | |
| T11 | Nhập key thủ công đúng → mã hóa thành công | ⏳ | |
| T12 | Nhập key sai format → lỗi Zod, không gọi API | ⏳ | |
| T13 | Video >500MB → progress bar cập nhật, không crash | ⏳ | |
| T14 | Click "Mã hóa video khác" → form reset về ban đầu | ⏳ | |
| T15 | Switch tab Mã hóa ↔ Giải mã → mượt mà | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM (không CBC/ECB) | — N/A | — | Crypto ở backend (Prompt 04) |
| S02 | Nonce mới cho mỗi chunk | — N/A | — | Crypto ở backend (Prompt 04) |
| S03 | RSA-OAEP-SHA256 padding | — N/A | — | Không dùng RSA ở phase AES |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase AES |
| S05 | os.urandom() cho key generation | — N/A | — | Ở backend (Prompt 04) |
| S06 | Key không log ra console/file | ✅ | ⏳ | Không console.log(key) trong service |
| S07 | Temp file xóa sau xử lý | — N/A | — | Không có temp file ở frontend |
| S08 | File validation: extension + MIME | ✅ | ⏳ | Zod VideoFileSchema trước khi upload |
| S09 | Rate limiter được apply | — N/A | — | Ở backend (Prompt 03) |
| S10 | Error không expose stack trace | ✅ | ⏳ | Hiển thị message tiếng Việt, không raw error |
| S11 | Security headers có trong response | — N/A | — | Ở backend (Prompt 03) |
| S12 | Frontend: key cleared sau download | ✅ | ⏳ | Dùng clearSensitiveData() từ security.ts |
| S13 | Frontend: key không lưu localStorage | ✅ | ⏳ | Kiểm tra DevTools → Application |
| S14 | Frontend: validate trước khi gọi API | ✅ | ⏳ | Zod schema chạy trước axios call |

---

## 🔍 Phân tích

### Items ngoài yêu cầu

-

### Items thiếu sót

-

### Items vi phạm

-

---

## 📝 Ghi chú kỹ thuật

---

## ✅ Kết luận

- [ ] **ĐẠT** – Chuyển sang Prompt 06 (RSA Backend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
