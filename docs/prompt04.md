# Prompt 04 – AES Backend: FastAPI Encrypt/Decrypt AES-256-GCM

---

## 📝 TL;DR

- **Input:** Video file + AES key (optional — có thể tự sinh)
- **Output:** File `.enc` (mã hóa) hoặc video gốc (giải mã) + File `.key`
- **Files to create:** `routers/aes.py`, `services/aes_service.py`, `schemas/aes_schema.py`
- **Key constraint:** Chunked processing 10MB, nonce-per-chunk, StreamingResponse
- **Definition of Done:** 3 endpoints hoạt động + 12 test cases pass + Security Checklist S01,S02,S05-S11 pass

---

## 🎭 Role

Bạn là **Backend Security Engineer** chuyên về cryptographic implementations. Nhiệm vụ là implement API mã hóa/giải mã video bằng **AES-256-GCM** (Galois/Counter Mode), xử lý file video lớn (từ vài MB đến vài GB) bằng kỹ thuật chunked streaming — đảm bảo không load toàn bộ video vào RAM.

---

## 📋 Context

- Phase 1 – AES Symmetric Encryption
- Prompt 01 (Setup) và Prompt 03 (Security) đã hoàn thành
- AES-256-GCM là thuật toán mã hóa đối xứng, nhanh, authenticated
- File video có thể từ vài MB đến vài GB → PHẢI chunked processing
- Mỗi chunk mã hóa bằng nonce riêng (12 bytes, sinh bằng `os.urandom()`)

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** Config `MAX_CHUNK_SIZE` đọc từ `.env` (default 10MB)
- **Từ Prompt 03:** Import và sử dụng:
  - `from app.utils.file_validator import validate_video_file, validate_enc_file`
  - `from app.utils.crypto_utils import generate_aes_key, decode_key_b64`
  - `from app.middleware.rate_limiter import encrypt_limiter, keygen_limiter`
- **KHÔNG thay đổi:** Không sửa bất kỳ file nào trong `middleware/` hoặc `utils/`

---

## 🎯 Mục tiêu

Implement 3 endpoint AES:

1. `GET /api/aes/generate-key` — sinh AES-256 key mới
2. `POST /api/aes/encrypt` — mã hóa video → file `.enc`
3. `POST /api/aes/decrypt` — giải mã file `.enc` → video gốc

---

## 📁 File cần tạo/chỉnh sửa

```
backend/app/
├── routers/aes.py              ← Implement đầy đủ (thay thế placeholder)
├── services/aes_service.py     ← Tạo mới
└── schemas/aes_schema.py       ← Tạo mới
```

---

## 🔐 Thiết kế định dạng file `.enc`

```
[4 bytes: uint32 big-endian = độ dài JSON header]
[N bytes: JSON header (UTF-8)]
[Chunk 1: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
[Chunk 2: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
[...]
```

> **Mỗi chunk** = nonce (12B) + encrypted_data + GCM tag (16B).
> **Nonce PHẢI khác nhau** cho mỗi chunk (sinh bằng `os.urandom(12)`).

**JSON Header:**

```json
{
  "version": "1.0",
  "algorithm": "AES-256-GCM",
  "chunk_size": 10485760,
  "chunk_count": 5,
  "original_filename": "video.mp4",
  "original_size": 52428800,
  "created_at": "2024-01-01T00:00:00Z"
}
```

> ⚠️ **QUAN TRỌNG:** Key KHÔNG được nhúng vào file `.enc`. Key phải trả riêng qua response header hoặc file `.key` riêng.

---

## ✅ Yêu cầu chức năng

### `GET /api/aes/generate-key`

- **Response:**
  ```json
  {
    "key_b64": "base64_encoded_32_bytes",
    "key_size_bits": 256,
    "algorithm": "AES-256-GCM"
  }
  ```
- **Logic:** `os.urandom(32)` → base64 encode
- **Rate limit:** `keygen_limiter` (30 req/phút)

### `POST /api/aes/encrypt`

- **Request:** `multipart/form-data`
  - `file`: UploadFile (video)
  - `key_b64`: str (optional — nếu không gửi, server tự sinh)
- **Response:** `StreamingResponse` trả file `.enc`
  - Header: `Content-Disposition: attachment; filename="video.mp4.enc"`
  - Header: `X-Algorithm: AES-256-GCM`
  - Header: `X-Key-B64: <key_base64>` (CHỈ khi server tự sinh key)
- **Logic:**
  1. Validate video file (extension + MIME + magic bytes)
  2. Nếu có `key_b64` → decode + validate 32 bytes. Nếu không → sinh key mới
  3. Build JSON header (version, algorithm, chunk_size, chunk_count, original_filename, ...)
  4. Write: `[4 bytes header_len][header JSON]`
  5. Đọc file theo chunks (`MAX_CHUNK_SIZE`), mỗi chunk:
     - Sinh nonce 12 bytes (`os.urandom(12)`)
     - Encrypt bằng `AESGCM(key)`: `nonce + ciphertext + tag`
  6. Stream output
- **Rate limit:** `encrypt_limiter` (10 req/phút)

### `POST /api/aes/decrypt`

- **Request:** `multipart/form-data`
  - `enc_file`: UploadFile (file `.enc`)
  - `key_b64`: str (bắt buộc)
- **Response:** `StreamingResponse` trả video gốc
  - Header: `Content-Disposition: attachment; filename="<original_filename>"`
- **Logic:**
  1. Validate file có đuôi `.enc`
  2. Decode `key_b64` → validate 32 bytes
  3. Đọc 4 bytes → header_len → đọc header JSON
  4. Validate header (version, algorithm)
  5. Đọc từng chunk: 12 bytes nonce → ciphertext+tag → decrypt bằng AESGCM
  6. Nếu GCM tag fail → **HTTP 400** `"Giải mã thất bại: key sai hoặc file bị hỏng"`
  7. Stream output
- **Rate limit:** `encrypt_limiter` (10 req/phút)

---

## 🔧 Yêu cầu kỹ thuật

### `services/aes_service.py`

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os, struct, json
from datetime import datetime, timezone
from typing import Generator

def encrypt_chunk_gcm(key: bytes, chunk: bytes) -> bytes:
    """Encrypt 1 chunk bằng AES-256-GCM. Return: nonce(12) + ciphertext + tag(16)."""
    nonce = os.urandom(12)
    aesgcm = AESGCM(key)
    ciphertext = aesgcm.encrypt(nonce, chunk, None)
    return nonce + ciphertext  # ciphertext đã bao gồm tag (16 bytes cuối)

def decrypt_chunk_gcm(key: bytes, data: bytes) -> bytes:
    """Decrypt 1 chunk. data = nonce(12) + ciphertext + tag(16)."""
    nonce = data[:12]
    ciphertext = data[12:]
    aesgcm = AESGCM(key)
    return aesgcm.decrypt(nonce, ciphertext, None)

def encrypt_video_stream(
    file_stream,
    key: bytes,
    original_filename: str,
    chunk_size: int
) -> Generator[bytes, None, None]:
    """
    Generator: yield binary output theo định dạng .enc
    [4 bytes header_len][JSON header][chunk1][chunk2]...
    """
    # Bước 1: Đọc toàn bộ file để biết original_size và chunk_count
    # (Hoặc đọc chunks vào temp, nhưng cần biết trước chunk_count cho header)
    # ...implement...

def decrypt_video_stream(
    enc_stream,
    key: bytes
) -> tuple[Generator[bytes, None, None], str]:
    """
    Parse .enc file → yield decrypted chunks.
    Return: (generator, original_filename)
    """
    # Bước 1: Đọc 4 bytes header_len
    # Bước 2: Đọc header JSON
    # Bước 3: Yield decrypted chunks
    # ...implement...
```

### `schemas/aes_schema.py`

```python
from pydantic import BaseModel, field_validator
import base64

class AESKeyRequest(BaseModel):
    key_b64: str

    @field_validator("key_b64")
    @classmethod
    def validate_key(cls, v: str) -> str:
        v = v.strip()
        try:
            key = base64.b64decode(v)
        except Exception:
            raise ValueError("key_b64 không phải base64 hợp lệ")
        if len(key) != 32:
            raise ValueError(f"Key phải có đúng 32 bytes (256 bit), nhận được {len(key)} bytes")
        return v
```

### File `.key` (output khi server tự sinh key)

```json
{
  "key_b64": "base64_encoded_32_bytes",
  "algorithm": "AES-256-GCM",
  "key_size_bits": 256,
  "note": "Lưu file này cẩn thận. Mất key = mất video."
}
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Dùng `AESGCM` từ `cryptography.hazmat.primitives.ciphers.aead`
- Stream response bằng `StreamingResponse` (KHÔNG load toàn bộ video vào RAM)
- Mỗi chunk phải có nonce riêng (12 bytes, `os.urandom(12)`)
- Chunk size đọc từ `.env` (`MAX_CHUNK_SIZE`)
- Validate key_b64 bằng Pydantic (đúng 32 bytes sau decode)

❌ **NGHIÊM CẤM:**
- KHÔNG dùng AES-CBC, AES-ECB — chỉ dùng AES-GCM
- KHÔNG nhúng key vào file `.enc`
- KHÔNG tái sử dụng nonce giữa các chunks
- KHÔNG dùng `random.randbytes()` — chỉ dùng `os.urandom()`
- KHÔNG load toàn bộ video vào memory bằng `await file.read()`

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| `key_b64` có whitespace/newlines | `.strip()` trước khi decode |
| `key_b64` sai format (không phải base64) | HTTP 422 "key_b64 không phải base64 hợp lệ" |
| `key_b64` đúng base64 nhưng sai độ dài | HTTP 422 "Key phải có đúng 32 bytes" |
| File `.enc` bị truncated (thiếu chunks) | `aesgcm.decrypt()` sẽ raise → HTTP 400 |
| File upload bị ngắt giữa chừng | FastAPI tự handle, cleanup temp |
| Decrypt sai key | GCM tag verification fail → HTTP 400 "Giải mã thất bại" |
| Sửa 1 byte trong file `.enc` | GCM auth fail → HTTP 400 "File bị hỏng" |
| Video ~2GB (rất lớn) | Chunked processing, streaming response, không OOM |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG dùng `base64.encodebytes()` — dùng `base64.b64encode()` (không có newline chars)
2. KHÔNG return `JSONResponse` chứa binary data — dùng `StreamingResponse`
3. KHÔNG `await file.read()` đọc toàn bộ file lớn — dùng chunked reading
4. KHÔNG tự implement AES — dùng `cryptography` library
5. KHÔNG lưu file `.enc` tạm trên server — stream thẳng sang response

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Generate key | `GET /api/aes/generate-key` | JSON có `key_b64` (44 chars base64 = 32 bytes) |
| T02 | Decode key | Decode `key_b64` bằng base64 | Đúng 32 bytes |
| T03 | Encrypt (auto key) | POST video ~10MB, không gửi `key_b64` | Download `.enc`, response header có `X-Key-B64` |
| T04 | Encrypt (manual key) | POST video ~10MB + `key_b64` hợp lệ | Download `.enc`, không có `X-Key-B64` header |
| T05 | Encrypt lớn | POST video >100MB | Streaming response hoàn thành, không timeout |
| T06 | Decrypt đúng key | POST file `.enc` + `key_b64` đúng | Download video, xem được |
| T07 | Decrypt sai key | POST file `.enc` + `key_b64` sai | HTTP 400 "Giải mã thất bại: key sai hoặc file bị hỏng" |
| T08 | Decrypt sai file | POST file `.mp4` thường (không phải .enc) | HTTP 400 "Yêu cầu file .enc" |
| T09 | Key sai format | `key_b64` = "hello world" | HTTP 422 "key_b64 không phải base64 hợp lệ" |
| T10 | Key sai độ dài | `key_b64` = base64 của 16 bytes | HTTP 422 "Key phải có đúng 32 bytes" |
| T11 | Tamper detection | Sửa 1 byte trong file `.enc` → decrypt | HTTP 400 "File bị hỏng" (GCM tag fail) |
| T12 | Round-trip lớn | Encrypt+Decrypt video ~1 tiếng | SHA-256 gốc = SHA-256 sau decrypt |

---

## 📋 Checklist tự kiểm tra

- [ ] Dùng `AESGCM` từ `cryptography` (KHÔNG phải AES-CBC)
- [ ] `os.urandom(12)` sinh nonce MỚI cho MỖI chunk
- [ ] `os.urandom(32)` sinh key (KHÔNG dùng `random`)
- [ ] Key KHÔNG nhúng vào file `.enc`
- [ ] File `.enc` có cấu trúc đúng: `[4 bytes][header][chunks]`
- [ ] JSON header có đủ fields: version, algorithm, chunk_size, chunk_count, original_filename, original_size, created_at
- [ ] `StreamingResponse` — không load toàn bộ video vào RAM
- [ ] Decrypt sai key → HTTP 400 (không 500)
- [ ] Pydantic validator: key_b64 phải decode được và = 32 bytes
- [ ] Validate file upload: extension + MIME (dùng `file_validator`)
- [ ] Rate limiter apply trên cả 3 endpoints
- [ ] Tất cả endpoints hiện trong Swagger `/docs`
