# Prompt 10 – Signature Backend: Chữ ký số & Mã hóa kết hợp (RSA-PSS + Hybrid)

---

## 📝 TL;DR

- **Input:** Video file + Sender private key + Receiver public key
- **Output:** File `.signed.enc` (ký + mã hóa) hoặc video gốc + kết quả xác minh
- **Files to create:** `routers/signature.py`, `services/signature_service.py`, `schemas/signature_schema.py`
- **Key constraint:** RSA-PSS cho chữ ký, Hybrid cho mã hóa (AES+RSA), xác minh hash SHA-256
- **Definition of Done:** 4 endpoints hoạt động + 13 test cases pass + Security Checklist S01-S11 pass

---

## 🎭 Role

Bạn là **Backend Security Engineer** chuyên về chữ ký số (Digital Signature). Nhiệm vụ là implement API **ký số + mã hóa** video kết hợp, đảm bảo 3 tính chất:
- **Bảo mật (Confidentiality):** Chỉ người nhận đọc được nội dung
- **Toàn vẹn (Integrity):** Phát hiện nếu file bị sửa đổi
- **Xác thực (Authentication):** Xác minh danh tính người gửi

---

## 📋 Context

- Phase 4 – Digital Signature (Chữ ký số kết hợp mã hóa)
- Phase 1 (AES), Phase 2 (RSA), Phase 3 (Hybrid) đã hoàn thành
- Chữ ký số dùng **RSA-PSS** (Probabilistic Signature Scheme) — KHÔNG phải PKCS1v15
- Quy trình: Người gửi **ký** video bằng private key → sau đó **mã hóa Hybrid** toàn bộ (gồm cả chữ ký) bằng public key của người nhận
- Người nhận **giải mã** → rồi **xác minh** chữ ký bằng public key của người gửi

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 04 (AES):** Import `encrypt_chunk_gcm`, `decrypt_chunk_gcm`
- **Từ Prompt 06 (RSA):** Import `rsa_encrypt_bytes`, `rsa_decrypt_bytes`
- **Từ Prompt 08 (Hybrid):** Tham khảo pattern Hybrid Encryption
- **Từ Prompt 03:** Import `validate_video_file`, `encrypt_limiter`
- **KHÔNG thay đổi:** Không sửa file AES, RSA, hoặc Hybrid — chỉ import

---

## 🎯 Mục tiêu

Implement 4 endpoint Signature:

1. `POST /api/signature/sign-encrypt` — ký video + mã hóa Hybrid
2. `POST /api/signature/decrypt-verify` — giải mã + xác minh chữ ký
3. `POST /api/signature/sign-only` — chỉ ký video (không mã hóa)
4. `POST /api/signature/verify-only` — chỉ xác minh chữ ký (không mã hóa)

---

## 📁 File cần tạo/chỉnh sửa

```
backend/app/
├── routers/signature.py           ← Implement đầy đủ
├── services/signature_service.py  ← Tạo mới
└── schemas/signature_schema.py    ← Tạo mới
```

---

## 🔐 Quy trình Chi tiết (Chain-of-Thought)

### Quy trình Ký + Mã hóa (`sign-encrypt`)

```
Bước 1: Tính hash SHA-256 của video gốc
         → video_hash = SHA256(video_bytes)

Bước 2: Ký hash bằng Sender Private Key (RSA-PSS + SHA-256)
         → signature = RSA_PSS_SIGN(sender_private_key, video_hash)

Bước 3: Tạo payload gồm: video + signature + metadata
         → payload_header (JSON): {signature_b64, hash_algorithm, video_hash_hex, ...}

Bước 4: Mã hóa Hybrid toàn bộ payload:
         → Sinh AES session key (os.urandom(32))
         → AES-GCM encrypt video chunks
         → RSA encrypt session key bằng Receiver Public Key

Bước 5: Output file .signed.enc
```

### Quy trình Giải mã + Xác minh (`decrypt-verify`)

```
Bước 1: Giải mã Hybrid:
         → RSA decrypt AES session key bằng Receiver Private Key
         → AES-GCM decrypt video chunks

Bước 2: Tính hash SHA-256 của video đã giải mã
         → actual_hash = SHA256(decrypted_video)

Bước 3: Xác minh chữ ký bằng Sender Public Key (RSA-PSS)
         → RSA_PSS_VERIFY(sender_public_key, signature, expected_hash)

Bước 4: So sánh actual_hash với expected_hash từ metadata
         → Match = "✅ Chữ ký hợp lệ"
         → Mismatch = "❌ Chữ ký KHÔNG hợp lệ - File có thể bị giả mạo"
```

---

## 🔐 Thiết kế định dạng file `.signed.enc`

```
[4 bytes: uint32 big-endian = độ dài JSON header]
[N bytes: JSON header (UTF-8)]
[256 bytes: RSA-encrypted AES key]
[Signed + Encrypted video chunks]
  → [Chunk 1: 12B nonce | ciphertext | 16B GCM tag]
  → [Chunk 2: ...]
```

**JSON Header (mở rộng từ Hybrid header):**

```json
{
  "version": "1.0",
  "algorithm": "SIGNED-HYBRID-RSA2048-AES256GCM",
  "signature": {
    "algorithm": "RSA-PSS-SHA256",
    "signature_b64": "base64_encoded_signature",
    "hash_algorithm": "SHA-256",
    "video_hash_hex": "abc123...",
    "signed_by": "sender"
  },
  "encryption": {
    "rsa_key_size_bits": 2048,
    "aes_key_size_bits": 256,
    "encrypted_aes_key_size": 256,
    "chunk_size": 10485760,
    "chunk_count": 5
  },
  "original_filename": "video.mp4",
  "original_size": 52428800,
  "created_at": "2024-01-01T00:00:00Z"
}
```

---

## ✅ Yêu cầu chức năng

### `POST /api/signature/sign-encrypt`

- **Request:** `multipart/form-data`
  - `file`: UploadFile (video file)
  - `sender_private_key_pem`: str (private key của người gửi — để ký)
  - `receiver_public_key_pem`: str (public key của người nhận — để mã hóa)
- **Response:** StreamingResponse file `.signed.enc`
  - Header: `X-Algorithm: SIGNED-HYBRID-RSA2048-AES256GCM`
  - Header: `X-Video-Hash: <sha256_hex>`
- **Logic:**
  1. Validate video file + validate cả 2 PEM keys
  2. Tính SHA-256 hash toàn bộ video
  3. Ký hash bằng RSA-PSS + SHA-256 (sender private key)
  4. Sinh AES session key
  5. Mã hóa AES key bằng RSA (receiver public key)
  6. Mã hóa video bằng AES-GCM (chunked)
  7. Build output: `[header][encrypted_aes_key][encrypted_chunks]`

### `POST /api/signature/decrypt-verify`

- **Request:** `multipart/form-data`
  - `enc_file`: UploadFile (file `.signed.enc`)
  - `receiver_private_key_pem`: str (để giải mã)
  - `sender_public_key_pem`: str (để xác minh chữ ký)
- **Response:** `StreamingResponse` file video gốc (giống pattern AES/RSA/Hybrid)
  - Header: `Content-Disposition: attachment; filename="<original_filename>"`
  - Header: `X-Signature-Valid: true` hoặc `false`
  - Header: `X-Video-SHA256-Expected: <hex>` (hash từ metadata file .signed.enc)
  - Header: `X-Video-SHA256-Actual: <hex>` (hash tính từ video sau decrypt)
  - Header: `X-Signature-Message: <message tiếng Việt URL-encoded>`
  > **Lưu ý:** Cho phép download video NGAY CẢ KHI chữ ký không hợp lệ. Frontend đọc headers để quyết định hiển thị ✅ hay ❌.
  >
  > Pattern này nhất quán với toàn bộ dự án (không cần session, không cần DB, không cần endpoint `/download/<session_id>`).
  
  **Ví dụ response headers khi hợp lệ:**
  ```
  X-Signature-Valid: true
  X-Video-SHA256-Expected: abc123def456...
  X-Video-SHA256-Actual: abc123def456...
  X-Signature-Message: %E2%9C%85%20Ch%E1%BB%AF%20k%C3%BD%20h%E1%BB%A3p%20l%E1%BB%87
  ```
  
  **Ví dụ response headers khi KHÔNG hợp lệ:**
  ```
  X-Signature-Valid: false
  X-Video-SHA256-Expected: abc123def456...
  X-Video-SHA256-Actual: 789xyz...
  X-Signature-Message: %E2%9D%8C%20Ch%E1%BB%AF%20k%C3%BD%20KH%C3%94NG%20h%E1%BB%A3p%20l%E1%BB%87
  ```

- **Xử lý lỗi RSA decrypt:**
  - Sai receiver private key → **HTTP 400** `"Private key người nhận không đúng"` (KHÔNG trả video)

- **Logic:**
  1. Parse header JSON → lấy signature info
  2. RSA decrypt AES key bằng receiver private key (fail → 400)
  3. AES-GCM decrypt video chunks → stream vào temp hoặc buffer
  4. Tính SHA-256 hash video decrypt
  5. RSA-PSS verify signature bằng sender public key
  6. So sánh actual hash vs expected hash
  7. Stream video decrypt về client kèm verify result trong headers

### `POST /api/signature/sign-only`

- **Request:** `multipart/form-data`
  - `file`: UploadFile (video)
  - `private_key_pem`: str
- **Response:**
  ```json
  {
    "signature_b64": "base64...",
    "hash_algorithm": "SHA-256",
    "video_hash_hex": "abc123...",
    "algorithm": "RSA-PSS-SHA256"
  }
  ```
- **Logic:** Tính SHA-256 → RSA-PSS sign → trả signature + hash

### `POST /api/signature/verify-only`

- **Request:**
  ```json
  {
    "signature_b64": "base64...",
    "video_hash_hex": "abc123...",
    "public_key_pem": "-----BEGIN PUBLIC KEY-----..."
  }
  ```
  HOẶC `multipart/form-data` (kèm file để tính hash tự động):
  - `file`: UploadFile (video gốc)
  - `signature_b64`: str
  - `public_key_pem`: str
- **Response:**
  ```json
  {
    "valid": true,
    "message": "✅ Chữ ký hợp lệ"
  }
  ```

---

## 🔧 Yêu cầu kỹ thuật

### `services/signature_service.py`

```python
import hashlib
from cryptography.hazmat.primitives.asymmetric import padding as asym_padding
from cryptography.hazmat.primitives import hashes

def compute_sha256(file_stream) -> str:
    """Tính SHA-256 hash cho file (streaming, không load toàn bộ vào RAM)."""
    hasher = hashlib.sha256()
    while True:
        chunk = file_stream.read(8192)
        if not chunk:
            break
        hasher.update(chunk)
    return hasher.hexdigest()

def rsa_pss_sign(video_hash_bytes: bytes, private_key_pem: str) -> bytes:
    """Ký hash bằng RSA-PSS + SHA-256."""
    # private_key.sign(
    #     video_hash_bytes,
    #     asym_padding.PSS(
    #         mgf=asym_padding.MGF1(hashes.SHA256()),
    #         salt_length=asym_padding.PSS.MAX_LENGTH
    #     ),
    #     hashes.SHA256()
    # )

def rsa_pss_verify(
    video_hash_bytes: bytes,
    signature: bytes,
    public_key_pem: str
) -> bool:
    """Xác minh chữ ký RSA-PSS. Return True/False."""
    # try:
    #     public_key.verify(
    #         signature,
    #         video_hash_bytes,
    #         asym_padding.PSS(
    #             mgf=asym_padding.MGF1(hashes.SHA256()),
    #             salt_length=asym_padding.PSS.MAX_LENGTH
    #         ),
    #         hashes.SHA256()
    #     )
    #     return True
    # except InvalidSignature:
    #     return False
```

**Tái sử dụng từ prompts trước:**
```python
from app.services.aes_service import encrypt_chunk_gcm, decrypt_chunk_gcm
from app.services.rsa_service import rsa_encrypt_bytes, rsa_decrypt_bytes
```

### `schemas/signature_schema.py`

```python
class SignEncryptRequest(BaseModel):
    sender_private_key_pem: str
    receiver_public_key_pem: str

    @field_validator("sender_private_key_pem")
    @classmethod
    def validate_sender_key(cls, v):
        if '-----BEGIN PRIVATE KEY-----' not in v and '-----BEGIN RSA PRIVATE KEY-----' not in v:
            raise ValueError("sender_private_key_pem phải là Private Key PEM hợp lệ")
        return v

    @field_validator("receiver_public_key_pem")
    @classmethod
    def validate_receiver_key(cls, v):
        if '-----BEGIN PUBLIC KEY-----' not in v:
            raise ValueError("receiver_public_key_pem phải là Public Key PEM hợp lệ")
        return v

class DecryptVerifyRequest(BaseModel):
    receiver_private_key_pem: str
    sender_public_key_pem: str

class SignOnlyRequest(BaseModel):
    private_key_pem: str

class VerifyOnlyRequest(BaseModel):
    signature_b64: str
    video_hash_hex: str
    public_key_pem: str
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Chữ ký dùng **RSA-PSS** (KHÔNG PKCS1v15)
- Hash dùng **SHA-256** (streaming, không load toàn bộ video vào RAM)
- Mã hóa dùng Hybrid (AES-GCM + RSA-OAEP) — giống Phase 3
- File `.signed.enc` phải chứa cả signature lẫn encrypted video
- Cho phép download video ngay cả khi signature invalid (kèm cảnh báo)
- **Import** code từ aes_service, rsa_service — KHÔNG copy-paste

❌ **NGHIÊM CẤM:**
- KHÔNG dùng PKCS1v15 cho signing
- KHÔNG dùng MD5, SHA-1 cho hash — chỉ SHA-256+
- KHÔNG log private key
- KHÔNG lưu video decrypt trên server vĩnh viễn — cleanup sau khi download

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| Sender key = Receiver key (cùng người) | Cho phép — dùng 2 cặp key khác nhau của cùng 1 người |
| Video bị sửa 1 byte sau khi ký | SHA-256 hash thay đổi → signature verify FAIL |
| Signature bị đổi | RSA-PSS verify sẽ raise InvalidSignature → `valid: false` |
| File `.signed.enc` bị truncated | Response 400 "File bị hỏng" |
| Sai receiver private key | RSA decrypt fail → 400 "Private key người nhận không đúng" |
| Sai sender public key | Signature verify fail nhưng video vẫn decrypt → `valid: false` |
| Video > 1GB | SHA-256 tính streaming (8KB chunks), không OOM |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG ký trên ciphertext — ký trên video GỐC (sign-then-encrypt)
2. KHÔNG dùng RSA-PKCS1v15 cho signing — PHẢI PSS
3. KHÔNG hash bằng MD5 hay SHA-1 — PHẢI SHA-256
4. KHÔNG load file vào RAM để tính hash — streaming hash (`hashlib.sha256()` + chunked read)
5. KHÔNG copy-paste code Hybrid — import functions

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Sign-Encrypt | Upload video + sender private + receiver public | Download `.signed.enc` |
| T02 | Inspect header | Xem header file `.signed.enc` (hex) | JSON có `signature.algorithm: "RSA-PSS-SHA256"` |
| T03 | Decrypt-Verify đúng | Upload `.signed.enc` + đúng key cả 2 | `signature_valid: true` + download video |
| T04 | Sai receiver key | Dùng sai receiver private key | 400 "Private key người nhận không đúng" |
| T05 | Sai sender key | Dùng sai sender public key | `signature_valid: false` + video vẫn download được (kèm cảnh báo) |
| T06 | Tamper video | Sửa 1 byte trong `.signed.enc` (phần video) | AES GCM fail hoặc signature hash mismatch |
| T07 | Sign-Only | Upload video + private key | JSON có `signature_b64`, `video_hash_hex` |
| T08 | Verify-Only file | Upload video gốc + signature_b64 + public key | `valid: true` |
| T09 | Verify-Only sai | Gửi signature sai + public key | `valid: false` |
| T10 | Video lớn | Upload 500MB+ | Hoàn thành, không OOM |
| T11 | Round-trip hash | SHA-256 gốc vs sau decrypt | Match 100% |
| T12 | Response header | Kiểm tra response headers | Có `X-Video-Hash` |
| T13 | Dùng RSA-4096 | Sinh key 4096 → sign-encrypt | Hoạt động bình thường |

---

## 📋 Checklist tự kiểm tra

- [ ] Dùng RSA-PSS (KHÔNG PKCS1v15) cho chữ ký
- [ ] Hash SHA-256 tính streaming (không load toàn bộ video)
- [ ] Mã hóa Hybrid: AES-GCM + RSA-OAEP (import từ services trước)
- [ ] File `.signed.enc` chứa cả signature lẫn encrypted video
- [ ] Header JSON có nested `signature` object
- [ ] Decrypt đúng + verify đúng → `signature_valid: true`
- [ ] Tamper detection hoạt động (hash mismatch = invalid)
- [ ] Video download được ngay cả khi signature invalid (kèm cảnh báo)
- [ ] `sign-only` và `verify-only` hoạt động độc lập
- [ ] Import code từ aes_service/rsa_service (KHÔNG copy-paste)
- [ ] Private key không log
- [ ] Tất cả endpoints thấy trong Swagger
