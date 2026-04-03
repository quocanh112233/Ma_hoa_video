# Report 03 – Đảm bảo Yêu cầu Bảo mật

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | `SECURITY.md` có đủ 5 sections | ⏳ Chưa implement | |
| 2 | `security_headers.py` – middleware tạo xong | ⏳ Chưa implement | |
| 3 | Security headers được apply (X-Content-Type-Options, X-Frame-Options,...) | ⏳ Chưa implement | |
| 4 | Header "Server" bị xóa khỏi response | ⏳ Chưa implement | |
| 5 | `rate_limiter.py` – SimpleRateLimiter tạo xong | ⏳ Chưa implement | |
| 6 | Rate limit: 10 req/phút cho encrypt/decrypt | ⏳ Chưa implement | |
| 7 | Rate limit: 30 req/phút cho generate-key | ⏳ Chưa implement | |
| 8 | `file_validator.py` – validate extension + MIME + magic bytes | ⏳ Chưa implement | |
| 9 | `crypto_utils.py` – đủ hàm utility | ⏳ Chưa implement | |
| 10 | `error_handlers.py` – không expose stack trace | ⏳ Chưa implement | |
| 11 | Tất cả handler đăng ký vào `main.py` | ⏳ Chưa implement | |
| 12 | `frontend/src/utils/security.ts` – downloadBlob, clearSensitiveData | ⏳ Chưa implement | |
| 13 | `frontend/src/utils/fileValidator.ts` – validate đủ loại file | ⏳ Chưa implement | |
| 14 | `frontend/src/utils/sanitize.ts` – escapeHtml, sanitizeFilename | ⏳ Chưa implement | |
| 15 | Không có key nào bị log ra console | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Security headers trong response | ⏳ | |
| T02 | Không có "Server: uvicorn" header | ⏳ | |
| T03 | Rate limit encrypt (11 req → 429) | ⏳ | |
| T04 | Rate limit key gen (31 req → 429) | ⏳ | |
| T05 | Upload .exe đổi tên thành .mp4 → reject | ⏳ | |
| T06 | Lỗi 500 – không có stack trace Python | ⏳ | |
| T07 | Request thiếu field → 422 tiếng Việt | ⏳ | |
| T08 | CORS từ origin sai → bị chặn | ⏳ | |
| T09 | Upload file 0 bytes → reject | ⏳ | |
| T10 | DevTools: không thấy key trong console | ⏳ | |
| T11 | localStorage: không có key | ⏳ | |

---

## 🔐 Security Checklist (baseline cho prompt04-11)

> Tất cả các prompt sau phải pass checklist này:

| # | Tiêu chí | Trạng thái |
|---|----------|-----------|
| S01 | Dùng AES-256-GCM (không CBC/ECB) | ⏳ |
| S02 | Nonce mới cho mỗi chunk | ⏳ |
| S03 | RSA-OAEP-SHA256 padding | ⏳ |
| S04 | RSA-PSS cho chữ ký | ⏳ |
| S05 | os.urandom() cho key generation | ⏳ |
| S06 | Key không log ra console/file | ⏳ |
| S07 | Temp file xóa sau xử lý | ⏳ |
| S08 | File validation: extension + MIME + magic | ⏳ |
| S09 | Rate limiter được apply | ⏳ |
| S10 | Error không expose stack trace | ⏳ |
| S11 | Security headers có trong response | ⏳ |
| S12 | Frontend: key cleared sau download | ⏳ |
| S13 | Frontend: key không lưu localStorage | ⏳ |
| S14 | Frontend: validate trước khi gọi API | ⏳ |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 04 (AES Backend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
