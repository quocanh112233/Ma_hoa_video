# Prompt 06 – RSA Backend: FastAPI Encrypt/Decrypt RSA-2048

---

## 📝 TL;DR

- **Input:** Video file ≤ 5MB + RSA public key PEM
- **Output:** File `.rsa.enc` (mã hóa) hoặc video gốc (giải mã) + RSA keypair
- **Files to create:** `routers/rsa.py`, `services/rsa_service.py`, `schemas/rsa_schema.py`
- **Key constraint:** RSA thuần chỉ cho mục đích học thuật, giới hạn 5MB, OAEP-SHA256
- **Definition of Done:** 4 endpoints hoạt động + 12 test cases pass + Security Checklist S03,S05-S11 pass

---

## 🎭 Role

Bạn là **Backend Security Engineer** hiểu rõ giới hạn của RSA asymmetric encryption. Nhiệm vụ là implement API mã hóa/giải mã video bằng **RSA-2048** thuần túy — CHỈ cho mục đích học thuật và so sánh hiệu suất, vì RSA encrypt trực tiếp data lớn cực kỳ chậm.

---

## 📋 Context

- Phase 2 – RSA Asymmetric Encryption (Educational)
- Phase 1 (AES) đã hoàn thành
- RSA-2048 chỉ encrypt được ~190 bytes/lần (OAEP overhead) → video phải chia chunks rất nhỏ → cực chậm
- Giới hạn 5MB cho mục đích học thuật (so sánh tốc độ vs AES)
- Dùng **OAEP-SHA256 padding** (KHÔNG phải PKCS1v15)

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** Config `MAX_CHUNK_SIZE` từ `.env`
- **Từ Prompt 03:** Import `validate_video_file`, `validate_enc_file`, `encrypt_limiter`, `keygen_limiter`
- **Từ Prompt 04:** Tham khảo pattern file format `.enc` (header + chunks)
- **KHÔNG thay đổi:** Không sửa file AES nào (routers/aes.py, services/aes_service.py)

---

## 🎯 Mục tiêu

Implement 4 endpoint RSA:

1. `POST /api/rsa/generate-keypair` — sinh cặp RSA key mới
2. `POST /api/rsa/encrypt` — mã hóa video ≤ 5MB bằng public key
3. `POST /api/rsa/decrypt` — giải mã file `.rsa.enc` bằng private key
4. `POST /api/rsa/validate-keypair` — kiểm tra cặp key có khớp nhau

---

## 📁 File cần tạo/chỉnh sửa

```
backend/app/
├── routers/rsa.py              ← Implement đầy đủ
├── services/rsa_service.py     ← Tạo mới
└── schemas/rsa_schema.py       ← Tạo mới
```

---

## 🔐 Thiết kế định dạng file `.rsa.enc`

```
[4 bytes: uint32 big-endian = độ dài JSON header]
[N bytes: JSON header (UTF-8)]
[RSA encrypted chunks...]
  → Mỗi chunk = [4 bytes chunk_len] + [RSA ciphertext]
```

**JSON Header:**

```json
{
  "version": "1.0",
  "algorithm": "RSA-2048-OAEP-SHA256",
  "key_size_bits": 2048,
  "chunk_size": 190,
  "chunk_count": 5300,
  "original_filename": "video.mp4",
  "original_size": 1048576,
  "warning": "RSA thuần rất chậm. Chỉ dùng cho mục đích học thuật.",
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

## ✅ Yêu cầu chức năng

### `POST /api/rsa/generate-keypair`

- **Request body:**
  ```json
  { "key_size": 2048 }
  ```
  - `key_size`: 2048 (mặc định) hoặc 4096. Bất kỳ giá trị khác → 422.
- **Response:**
  ```json
  {
    "public_key_pem": "-----BEGIN PUBLIC KEY-----\n...",
    "private_key_pem": "-----BEGIN PRIVATE KEY-----\n...",
    "key_size_bits": 2048
  }
  ```

### `POST /api/rsa/encrypt`

- **Request:** `multipart/form-data`
  - `file`: UploadFile (video ≤ 5MB)
  - `public_key_pem`: str
- **Response:** StreamingResponse file `.rsa.enc`
  - Header: `X-Algorithm: RSA-2048-OAEP-SHA256`
  - Header: `X-Warning: RSA thuần — chỉ dùng cho mục đích học thuật`
- **Logic:**
  1. Validate file ≤ 5MB, nếu vượt → **HTTP 413** "File vượt quá 5MB. RSA thuần chỉ hỗ trợ file nhỏ."
  2. Validate PEM public key syntax + parse thực sự bằng cryptography
  3. Chia video thành chunks 190 bytes (max plaintext size cho RSA-2048 OAEP-SHA256)
  4. Encrypt từng chunk bằng RSA public key (OAEP-SHA256)
  5. Build output file `.rsa.enc`

### `POST /api/rsa/decrypt`

- **Request:** `multipart/form-data`
  - `enc_file`: UploadFile (file `.rsa.enc`)
  - `private_key_pem`: str
- **Response:** StreamingResponse video gốc
- **Logic:**
  1. Parse JSON header
  2. Đọc từng encrypted chunk → decrypt bằng RSA private key
  3. Nếu decrypt thất bại → **HTTP 400** "Private key không đúng hoặc file bị hỏng"

### `POST /api/rsa/validate-keypair`

- **Request body:**
  ```json
  {
    "public_key_pem": "...",
    "private_key_pem": "..."
  }
  ```
- **Response:**
  ```json
  {
    "valid": true,
    "message": "Cặp key hợp lệ"
  }
  ```
- **Logic:** Encrypt 1 test message bằng public key → decrypt bằng private key → so sánh → `valid: true/false`

---

## 🔧 Yêu cầu kỹ thuật

### `services/rsa_service.py`

```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes, serialization

def generate_rsa_keypair(key_size: int = 2048) -> tuple[str, str]:
    """Sinh cặp RSA key, trả về (public_pem, private_pem)."""
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=key_size,
    )
    # Serialize to PEM...

def rsa_encrypt_bytes(plaintext: bytes, public_key_pem: str) -> bytes:
    """Encrypt bytes bằng RSA-OAEP-SHA256."""
    # Load public key from PEM
    # public_key.encrypt(plaintext, OAEP(mgf=MGF1(SHA256()), algorithm=SHA256(), label=None))

def rsa_decrypt_bytes(ciphertext: bytes, private_key_pem: str) -> bytes:
    """Decrypt bytes bằng RSA private key."""
    # Load private key from PEM
    # private_key.decrypt(ciphertext, OAEP(...))
```

### `schemas/rsa_schema.py`

```python
from pydantic import BaseModel, field_validator

class RSAKeyGenRequest(BaseModel):
    key_size: int = 2048

    @field_validator("key_size")
    @classmethod
    def validate_key_size(cls, v):
        if v not in (2048, 4096):
            raise ValueError("key_size phải là 2048 hoặc 4096")
        return v

class RSAPEMRequest(BaseModel):
    public_key_pem: str
    private_key_pem: str

    @field_validator("public_key_pem")
    @classmethod
    def validate_public_key(cls, v):
        if '-----BEGIN PUBLIC KEY-----' not in v:
            raise ValueError("Phải là RSA Public Key PEM hợp lệ")
        return v

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
- RSA-OAEP với SHA-256 (KHÔNG PKCS1v15)
- Giới hạn file 5MB (hardcode) — vì RSA thuần quá chậm cho file lớn
- Response header X-Warning để nhắc người dùng
- PEM keys phải parse được thực sự bằng `cryptography` library (không chỉ check string)

❌ **NGHIÊM CẤM:**
- KHÔNG dùng PKCS1v15 padding
- KHÔNG cho phép file > 5MB (trả 413)
- KHÔNG nâng giới hạn lên — giữ 5MB theo thiết kế
- KHÔNG log private key

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| key_size = 1024 | HTTP 422 "key_size phải là 2048 hoặc 4096" |
| PEM string có extra whitespace | `.strip()` trước khi parse |
| RSA-4096 generate chậm | Chấp nhận — có thể mất vài giây |
| File 5.1MB | HTTP 413 "File vượt quá 5MB" |
| Decrypt với wrong keypair | HTTP 400 "Private key không đúng" |
| PEM invalid (text bừa) | HTTP 422 "Phải là RSA Public Key PEM hợp lệ" |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG dùng `rsa` Python package thuần — dùng `cryptography` library
2. KHÔNG implement RSA từ đầu (modular exponentiation) — dùng library
3. KHÔNG stream output (file ≤ 5MB nên load vào RAM đủ nhỏ, nhưng vẫn nên StreamingResponse)
4. KHÔNG reuse code AES bằng copy-paste — import nếu cần utilities

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Generate RSA-2048 | POST `/api/rsa/generate-keypair` body `{key_size:2048}` | JSON có PEM keys |
| T02 | Generate RSA-4096 | body `{key_size:4096}` | Thành công (chậm hơn) |
| T03 | key_size = 1024 | body `{key_size:1024}` | HTTP 422 validation error |
| T04 | Validate keypair đúng | POST cặp key vừa sinh | `{valid: true}` |
| T05 | Validate keypair sai | Mix public key 1 + private key 2 | `{valid: false}` |
| T06 | Encrypt ~1MB | Upload video 1MB + public key | Download `.rsa.enc` |
| T07 | Decrypt đúng | Upload `.rsa.enc` + private key đúng | Video gốc, xem được |
| T08 | Decrypt sai key | Upload + sai private key | HTTP 400 |
| T09 | File 10MB | Upload video 10MB | HTTP 413 "vượt quá 5MB" |
| T10 | PEM sai | Gửi string bừa làm public_key_pem | HTTP 422 |
| T11 | X-Warning header | Kiểm tra response headers khi encrypt | Có X-Warning |
| T12 | Đo thời gian | Encrypt file 5MB | Ghi lại thời gian (___ giây) |

---

## 📋 Checklist tự kiểm tra

- [ ] Dùng RSA-OAEP-SHA256 padding (KHÔNG PKCS1v15)
- [ ] key_size chỉ chấp nhận 2048/4096
- [ ] File > 5MB bị reject (HTTP 413)
- [ ] PEM validation bằng `cryptography` library (thực sự parse, không chỉ check string)
- [ ] Validate keypair thực sự encrypt/decrypt thử, không chỉ so sánh
- [ ] Header JSON có field `warning`
- [ ] Response headers có `X-Warning`
- [ ] Decrypt sai key → 400, message rõ ràng
- [ ] Private key không log
- [ ] Tất cả endpoints thấy trong Swagger
