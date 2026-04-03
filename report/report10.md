# Report 10 – Chữ ký số Backend: Digital Signature + Hybrid

> **Trạng thái:** ⏳ Chưa thực hiện  
> **Ngày hoàn thành:** ___________  
> **Người thực hiện:** ___________

---

## 📊 Bảng kiểm tra yêu cầu

| STT | Yêu cầu | Trạng thái | Ghi chú |
|-----|---------|-----------|---------|
| 1 | POST /api/signature/sign-encrypt hoạt động | ⏳ Chưa implement | |
| 2 | Dùng RSA-PSS với SHA-256 (không phải PKCS1v15) | ⏳ Chưa implement | |
| 3 | SHA-256 hash video được tính TRƯỚC khi mã hóa | ⏳ Chưa implement | |
| 4 | video_sha256 được nhúng vào JSON header file | ⏳ Chưa implement | |
| 5 | Cấu trúc file .signed.enc đúng định dạng | ⏳ Chưa implement | |
| 6 | POST /api/signature/decrypt-verify hoạt động | ⏳ Chưa implement | |
| 7 | Chữ ký sai → vẫn trả video + header X-Signature-Valid: false | ⏳ Chưa implement | |
| 8 | RSA decrypt fail → 400, không trả video | ⏳ Chưa implement | |
| 9 | Response headers: X-Signature-Valid, X-Video-SHA256-Expected, X-Video-SHA256-Actual | ⏳ Chưa implement | |
| 10 | POST /api/signature/sign-only hoạt động | ⏳ Chưa implement | |
| 11 | POST /api/signature/verify-only hoạt động | ⏳ Chưa implement | |
| 12 | Dùng temp file cho file lớn (tránh OOM) | ⏳ Chưa implement | |
| 13 | Pydantic validator cho sender_private_key_pem + receiver_public_key_pem | ⏳ Chưa implement | |

---

## 🧪 Kết quả Test thủ công

| # | Test | Kết quả | Ghi chú |
|---|------|---------|---------|
| T01 | Sign & Encrypt video 10MB → download .signed.enc | ⏳ | |
| T02 | Hex editor: header có "SIGNED-HYBRID-RSA2048-AES256GCM" | ⏳ | |
| T03 | Decrypt & Verify đúng → header X-Signature-Valid: true | ⏳ | |
| T04 | Sai receiver key → HTTP 400, không trả video | ⏳ | |
| T05 | Sai sender public key → video trả về + X-Signature-Valid: false | ⏳ | |
| T06 | Sửa 1 byte phần video → GCM tag fail → 400 | ⏳ | |
| T07 | POST sign-only → JSON có signature_b64 + video_sha256 | ⏳ | |
| T08 | POST verify-only đúng → `{"valid": true}` | ⏳ | |
| T09 | Verify video đã sửa 1 byte → `{"valid": false}` | ⏳ | |
| T10 | Sign & Encrypt video >500MB → dùng temp file, không OOM | ⏳ | |
| T11 | X-Video-SHA256-Expected = X-Video-SHA256-Actual (khi hợp lệ) | ⏳ | |

---

## 🔐 Security Checklist (từ Prompt 03)

> `✅ Áp dụng` = phải pass | `— N/A` = không áp dụng cho scope này

| # | Tiêu chí | Áp dụng | Trạng thái | Ghi chú |
|---|----------|---------|-----------|---------|
| S01 | Dùng AES-256-GCM cho video | ✅ | ⏳ | Tầng AES của Hybrid trong Signature |
| S02 | Nonce mới cho mỗi chunk AES | ✅ | ⏳ | Tái dùng từ aes_service |
| S03 | RSA-OAEP-SHA256 để encrypt AES key | ✅ | ⏳ | Tầng RSA-encrypt AES key |
| S04 | RSA-PSS cho chữ ký số | ✅ | ⏳ | **Core của phase này** |
| S05 | os.urandom() cho AES session key | ✅ | ⏳ | Session key mới mỗi lần |
| S06 | Sender private key không log ra console/file | ✅ | ⏳ | **Đặc biệt quan trọng** |
| S07 | Temp file xóa sau xử lý | ✅ | ⏳ | **Bắt buộc** cho file >500MB |
| S08 | File validation: extension + MIME + magic bytes | ✅ | ⏳ | |
| S09 | Rate limiter trên endpoint sign-and-encrypt | ✅ | ⏳ | |
| S10 | Error phân biệt rõ: RSA fail vs AES fail | ✅ | ⏳ | Không expose stack trace |
| S11 | Security headers trong response | ✅ | ⏳ | Middleware từ Prompt 03 |
| S12 | Frontend: key cleared sau download | — N/A | — | Thuộc Prompt 11 |
| S13 | Frontend: key không lưu localStorage | — N/A | — | Thuộc Prompt 11 |
| S14 | Frontend: validate trước khi gọi API | — N/A | — | Thuộc Prompt 11 |

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

- [ ] **ĐẠT** – Chuyển sang Prompt 11 (Chữ ký số Frontend)
- [ ] **CHƯA ĐẠT** – Cần sửa: ___________

---
*Cập nhật lần cuối: ___________*
