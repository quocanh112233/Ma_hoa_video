# Report 04 – AES Backend: FastAPI Encrypt/Decrypt

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | Endpoint POST /api/aes/generate-key hoạt động | ⏳ Chưa implement | |
| 2 | generate-key dùng os.urandom(32) | ⏳ Chưa implement | |
| 3 | Endpoint POST /api/aes/encrypt hoạt động | ⏳ Chưa implement | |
| 4 | Encrypt xử lý chunked 10MB | ⏳ Chưa implement | |
| 5 | Mỗi chunk có nonce độc lập 12 bytes | ⏳ Chưa implement | |
| 6 | Key KHÔNG nhúng vào file .enc | ⏳ Chưa implement | |
| 7 | Cấu trúc file .enc đúng [4 bytes header_len][header][chunks] | ⏳ Chưa implement | |
| 8 | Header JSON có đủ fields (version, algorithm, chunk_count,...) | ⏳ Chưa implement | |
| 9 | Response header X-Key-B64 khi tự sinh key | ⏳ Chưa implement | |
| 10 | Endpoint POST /api/aes/decrypt hoạt động | ⏳ Chưa implement | |
| 11 | Decrypt sai key → 400 với message rõ | ⏳ Chưa implement | |
| 12 | Stream response (không load toàn bộ vào RAM) | ⏳ Chưa implement | |
| 13 | Pydantic validator key_b64 (đúng 32 bytes) | ⏳ Chưa implement | |
| 14 | Validate MIME type / extension video | ⏳ Chưa implement | |
| 15 | Chunk size đọc từ .env | ⏳ Chưa implement | |
| 16 | Dùng AESGCM từ cryptography library | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | POST /api/aes/generate-key → JSON với key_b64 | ⏳ | |
| T02 | Decode key_b64 → đúng 32 bytes | ⏳ | |
| T03 | Encrypt video ~10MB, không truyền key | ⏳ | |
| T04 | Encrypt video >100MB | ⏳ | |
| T05 | Encrypt với key có sẵn | ⏳ | |
| T06 | Decrypt đúng key → video xem được | ⏳ | |
| T07 | Decrypt sai key → HTTP 400 | ⏳ | |
| T08 | Decrypt file không phải .enc → 400 | ⏳ | |
| T09 | key_b64 sai format → 422 | ⏳ | |
| T10 | key_b64 sai độ dài → 422 "32 bytes" | ⏳ | |
| T11 | Sửa 1 byte trong .enc → decrypt 400 (GCM tag fail) | ⏳ | |
| T12 | Video ~1 tiếng → encrypt + decrypt thành công | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM (không CBC/ECB) | ✅ | ⏳ | Core của phase này |
| S02 | Nonce mới cho mỗi chunk (không tái sử dụng) | ✅ | ⏳ | Kiểm tra bằng script parse nonce |
| S03 | RSA-OAEP-SHA256 padding | — N/A | — | Không dùng RSA ở phase AES |
| S04 | RSA-PSS cho chữ ký | — N/A | — | Không có chữ ký ở phase AES |
| S05 | os.urandom() cho key generation | ✅ | ⏳ | Review code |
| S06 | Key không log ra console/file | ✅ | ⏳ | Grep log statements |
| S07 | Temp file xóa sau xử lý | ✅ | ⏳ | Nếu có dùng temp file |
| S08 | File validation: extension + MIME + magic bytes | ✅ | ⏳ | Dùng file_validator từ Prompt 03 |
| S09 | Rate limiter được apply trên endpoint | ✅ | ⏳ | Dùng rate_limiter từ Prompt 03 |
| S10 | Error không expose stack trace | ✅ | ⏳ | Test T07, T08 |
| S11 | Security headers có trong response | ✅ | ⏳ | Middleware từ Prompt 03 |
| S12 | Frontend: key cleared sau download | — N/A | — | Thuộc Prompt 05 |
| S13 | Frontend: key không lưu localStorage | — N/A | — | Thuộc Prompt 05 |
| S14 | Frontend: validate trước khi gọi API | — N/A | — | Thuộc Prompt 05 |

---

## 🔍 Phân tích

### Items ngoài yêu cầu

_Liệt kê những gì đã implement nhưng không có trong prompt:_
-

### Items thiếu sót

_Liệt kê những gì implement chưa đúng/đủ:_
-

### Items vi phạm

_Liệt kê những gì implement sai/ngược yêu cầu:_
-

---

## 📝 Ghi chú kỹ thuật

_Các vấn đề gặp phải và cách giải quyết:_

---

## ✅ Kết luận

- [ ] **ĐẠT** – Chuyển sang Prompt 05 (AES Frontend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
_Cập nhật lần cuối: ____________
