# Prompt 08 – Hybrid Backend: Mã hóa Lai (RSA + AES)

---

## 📝 TL;DR

- **Input:** Video file (không giới hạn kích thước) + RSA public key PEM
- **Output:** File `.hybrid.enc` (mã hóa) hoặc video gốc (giải mã)
- **Files to create:** `routers/hybrid.py`, `services/hybrid_service.py`, `schemas/hybrid_schema.py`
- **Key constraint:** AES encrypt video + RSA encrypt AES key, tái sử dụng code từ Phase 1 & 2
- **Definition of Done:** 3 endpoints hoạt động + 11 test cases pass + Security Checklist S01-S03,S05-S11 pass

---

## 🎭 Role

Bạn là **Backend Security Engineer** hiểu rõ kiến trúc mã hóa lai (Hybrid Encryption) được dùng trong TLS/HTTPS. Nhiệm vụ là implement API mã hóa lai: dùng **AES-256-GCM** để mã hóa nội dung video (nhanh, không giới hạn kích thước), và dùng **RSA-OAEP-SHA256** để mã hóa AES session key (bảo vệ key an toàn).

---

## 📋 Context

- Phase 3 – Hybrid Encryption
- Phase 1 (AES) và Phase 2 (RSA) đã hoàn thành
- Hybrid = AES encrypt video + RSA encrypt AES key
- Không giới hạn kích thước file (video 1 phút đến 1+ tiếng)
- Đây là cách hoạt động thực tế của HTTPS và email mã hóa

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 04 (AES):** Import và tái sử dụng:
  ```python
  from app.services.aes_service import encrypt_chunk_gcm, decrypt_chunk_gcm
  ```
- **Từ Prompt 06 (RSA):** Import và tái sử dụng:
  ```python
  from app.services.rsa_service import rsa_encrypt_bytes, rsa_decrypt_bytes
  ```
- **Từ Prompt 03:** Import `validate_video_file`, `encrypt_limiter`
- **KHÔNG thay đổi:** Không sửa `aes_service.py` hoặc `rsa_service.py` — chỉ import

---

## 🎯 Mục tiêu

Implement 3 endpoint Hybrid:

1. `POST /api/hybrid/encrypt` – mã hóa video bằng AES, mã hóa AES key bằng RSA public key
2. `POST /api/hybrid/decrypt` – dùng RSA private key giải mã AES key, rồi giải mã video
3. `POST /api/hybrid/generate-session-key` – sinh AES session key (hữu ích cho kiểm tra)

---

## 📁 File cần tạo/chỉnh sửa

```
backend/app/
├── routers/hybrid.py           ← Implement đầy đủ
├── services/hybrid_service.py  ← Implement đầy đủ
└── schemas/hybrid_schema.py    ← Implement đầy đủ
```

---

## 🔐 Thiết kế định dạng file `.hybrid.enc`

```
[4 bytes: uint32 big-endian = độ dài JSON header]
[N bytes: JSON header (UTF-8)]
[256 bytes: RSA-encrypted AES key (fixed size cho RSA-2048)]
[Video chunks: giống định dạng AES từ Phase 1]
  → [Chunk 1: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
  → [Chunk 2: ...]
  → ...
```

**JSON Header:**

```json
{
  "version": "1.0",
  "algorithm": "HYBRID-RSA2048-AES256GCM",
  "rsa_key_size_bits": 2048,
  "aes_key_size_bits": 256,
  "encrypted_aes_key_size": 256,
  "chunk_size": 10485760,
  "chunk_count": 5,
  "original_filename": "video.mp4",
  "original_size": 52428800,
  "created_at": "2024-01-01T00:00:00Z"
}
```

> ✅ Tất cả thông tin cần thiết để decrypt đều nằm trong file `.hybrid.enc` — người nhận chỉ cần RSA private key.

---

## ✅ Yêu cầu chức năng

### `POST /api/hybrid/encrypt`

- **Request:** `multipart/form-data`
  - `file`: UploadFile (video, không giới hạn kích thước)
  - `public_key_pem`: str (RSA public key PEM, bắt buộc)
- **Response:** StreamingResponse file `.hybrid.enc`
  - Header: `X-Algorithm: HYBRID-RSA2048-AES256GCM`
  - Header: `X-Original-Filename: <tên file>`
- **Logic (Chain-of-Thought):**
  1. Validate public key PEM (parse thực sự, không chỉ check string)
  2. Sinh AES session key 32 bytes ngẫu nhiên (`os.urandom(32)`)
  3. Mã hóa AES key bằng RSA public key (OAEP SHA-256) → 256 bytes encrypted key
  4. Build JSON header (ghi đủ metadata)
  5. Yield: `[4 bytes header_len]` → `[header JSON]` → `[encrypted AES key]`
  6. Đọc video file theo chunks (10MB), mỗi chunk: sinh nonce → encrypt AES-GCM → yield
  7. Stream hoàn tất

### `POST /api/hybrid/decrypt`

- **Request:** `multipart/form-data`
  - `enc_file`: UploadFile (file `.hybrid.enc`)
  - `private_key_pem`: str (RSA private key PEM)
- **Response:** StreamingResponse file video gốc
- **Logic (Chain-of-Thought):**
  1. Đọc 4 bytes → header_len → đọc header JSON
  2. Đọc `encrypted_aes_key_size` bytes → giải mã bằng RSA private key → lấy AES key
  3. Nếu RSA decrypt thất bại → **400** "Private key không đúng"
  4. Giải mã video chunks bằng AES-256-GCM
  5. Nếu AES GCM tag fail → **400** "File bị hỏng hoặc key không khớp"
  6. Stream video gốc

### `POST /api/hybrid/generate-session-key`

- **Response:**
  ```json
  {
    "session_key_b64": "base64...",
    "key_size_bits": 256,
    "note": "Session key này sẽ được mã hóa bằng RSA public key khi encrypt"
  }
  ```

---

## 🔧 Yêu cầu kỹ thuật

### `services/hybrid_service.py`

**Tái sử dụng code (KHÔNG copy-paste):**

```python
from app.services.aes_service import encrypt_chunk_gcm, decrypt_chunk_gcm
from app.services.rsa_service import rsa_encrypt_bytes, rsa_decrypt_bytes

def encrypt_hybrid_stream(
    file_stream,
    public_key_pem: str,
    original_filename: str,
    chunk_size: int
) -> Generator:
    """
    1. Sinh AES key (os.urandom(32))
    2. RSA encrypt AES key
    3. Build header
    4. Yield: [header_len][header][encrypted_aes_key][aes_chunks...]
    """

def decrypt_hybrid_stream(
    enc_stream,
    private_key_pem: str
) -> tuple[Generator, str]:
    """
    1. Parse header
    2. Đọc encrypted_aes_key_size bytes
    3. RSA decrypt → AES key
    4. AES decrypt chunks
    5. Yield plain chunks, return original_filename
    """
```

### `schemas/hybrid_schema.py`

```python
class HybridEncryptRequest(BaseModel):
    public_key_pem: str

    @field_validator("public_key_pem")
    @classmethod
    def validate_public_key(cls, v):
        if '-----BEGIN PUBLIC KEY-----' not in v:
            raise ValueError("Phải là RSA Public Key PEM hợp lệ")
        return v

class HybridDecryptRequest(BaseModel):
    private_key_pem: str

    @field_validator("private_key_pem")
    @classmethod
    def validate_private_key(cls, v):
        if '-----BEGIN PRIVATE KEY-----' not in v and '-----BEGIN RSA PRIVATE KEY-----' not in v:
            raise ValueError("Phải là RSA Private Key PEM hợp lệ")
        return v
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- AES session key PHẢI sinh mới mỗi lần encrypt (`os.urandom(32)`) — KHÔNG reuse
- AES key được RSA-encrypt và nhúng vào file output — KHÔNG tách file
- Người decrypt chỉ cần 1 file `.hybrid.enc` + RSA private key
- **Import** `encrypt_chunk_gcm` / `rsa_encrypt_bytes` — KHÔNG copy-paste code
- Stream response — KHÔNG load toàn bộ video vào RAM

❌ **NGHIÊM CẤM:**
- KHÔNG copy-paste code từ `aes_service.py` hoặc `rsa_service.py`
- KHÔNG giới hạn kích thước file (video lớn phải hoạt động)
- KHÔNG lưu AES session key vào file riêng — nhúng vào `.hybrid.enc`
- KHÔNG log AES session key

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| RSA-4096 key thay vì 2048 | `encrypted_aes_key_size` sẽ là 512 bytes thay vì 256 — header cần ghi đúng |
| File video > 1GB | Chunked streaming, không OOM |
| RSA decrypt fail (sai key) | HTTP 400 "Private key không đúng" — phân biệt với lỗi AES |
| AES GCM tag fail | HTTP 400 "File bị hỏng hoặc key không khớp" — phân biệt với lỗi RSA |
| PEM string invalid | HTTP 422 validation error |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG copy-paste 100 dòng code từ `aes_service.py` — import functions
2. KHÔNG tạo file key riêng cho AES session key — nhúng vào `.hybrid.enc`
3. KHÔNG dùng `await file.read()` cho video lớn — chunked reading
4. KHÔNG trả response chung chung "Lỗi giải mã" — phân biệt RSA fail vs AES fail

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Generate session key | POST `/api/hybrid/generate-session-key` | JSON có `session_key_b64` 32 bytes |
| T02 | Encrypt nhỏ | Upload video 10MB + public key RSA-2048 | Download `.hybrid.enc` |
| T03 | Inspect header | Mở `.hybrid.enc` bằng hex editor | JSON header có `HYBRID-RSA2048-AES256GCM` |
| T04 | Decrypt đúng | Upload `.hybrid.enc` + private key đúng | Download video gốc, xem được |
| T05 | Decrypt sai key | Upload `.hybrid.enc` + sai private key | HTTP 400 "Private key không đúng" |
| T06 | Encrypt lớn | Upload video >500MB | Streaming hoàn thành, không timeout |
| T07 | Encrypt 1 tiếng | Upload video ~1-2GB | Hoàn thành, không OOM |
| T08 | Decrypt video lớn | Decrypt kết quả T06/T07 | Video gốc, xem được |
| T09 | RSA-4096 key | Dùng RSA-4096 public key | Hoạt động, header ghi `rsa_key_size_bits: 4096` |
| T10 | PEM sai | Gửi chuỗi bừa làm public_key_pem | HTTP 422 |
| T11 | Round-trip hash | SHA-256 video gốc vs sau decrypt | Hash match 100% |

---

## 📋 Checklist tự kiểm tra

- [ ] AES session key sinh mới mỗi lần encrypt (KHÔNG reuse)
- [ ] AES key được RSA-encrypt và nhúng vào file output
- [ ] Người decrypt chỉ cần private key, không cần file key riêng
- [ ] Import code từ aes_service/rsa_service (KHÔNG copy-paste)
- [ ] Stream response, không load toàn bộ video vào RAM
- [ ] Decrypt sai key → 400 với message phân biệt RSA vs AES
- [ ] Header JSON có đủ fields theo spec
- [ ] Tất cả endpoints thấy trong Swagger `/docs`
