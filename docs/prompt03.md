# Prompt 03 – Đảm bảo Yêu cầu Bảo mật (Security Baseline)

---

## 📝 TL;DR

- **Input:** Monorepo từ Prompt 01
- **Output:** SECURITY.md + middleware files + utility files + frontend security utils
- **Files to create:** `SECURITY.md`, `security_headers.py`, `rate_limiter.py`, `file_validator.py`, `crypto_utils.py`, `error_handlers.py`, `security.ts`, `fileValidator.ts`, `sanitize.ts`
- **Key constraint:** Toàn bộ security baseline PHẢI hoàn thành TRƯỚC khi implement crypto
- **Definition of Done:** 14 điểm Security Checklist pass + 11 test cases pass

---

## 🎭 Role

Bạn là **Security Engineer & Code Reviewer** chuyên về ứng dụng web xử lý dữ liệu nhạy cảm. Nhiệm vụ là xây dựng lớp bảo mật nền tảng (security baseline) cho dự án, bao gồm middleware, validators, và utilities — tất cả phải sẵn sàng TRƯỚC KHI bất kỳ logic crypto nào được implement.

---

## 📋 Context

- Prompt 01 đã hoàn thành (monorepo skeleton chạy được)
- Dự án xử lý dữ liệu mật mã → bảo mật là yêu cầu số 1
- Security baseline này sẽ được **tái sử dụng** trong tất cả prompts 04-12
- Không có authentication (không login/signup) → nhưng vẫn cần bảo vệ API

---

## 🔗 Dependencies (từ các prompt trước)

- **Từ Prompt 01:** Cấu trúc `backend/app/middleware/`, `backend/app/utils/`, `frontend/src/utils/`
- **KHÔNG thay đổi:** Không sửa `routers/`, `services/`, `schemas/` — chỉ thêm middleware và utils

---

## 🎯 Mục tiêu

1. Tạo file `SECURITY.md` ở root — tài liệu bảo mật cho dự án
2. Implement backend middleware: security headers, rate limiting
3. Implement backend utilities: file validation, crypto helpers, error handlers
4. Implement frontend utilities: secure download, data cleaning, sanitization
5. Đăng ký tất cả middleware/handlers vào `main.py`

---

## 📁 File cần tạo/chỉnh sửa

```
video-crypto/
├── SECURITY.md                              ← Tạo mới
├── backend/app/
│   ├── main.py                              ← Chỉnh sửa (đăng ký middleware)
│   ├── middleware/
│   │   ├── security_headers.py              ← Tạo mới
│   │   └── rate_limiter.py                  ← Tạo mới
│   └── utils/
│       ├── file_validator.py                ← Tạo mới
│       ├── crypto_utils.py                  ← Tạo mới
│       └── error_handlers.py                ← Tạo mới
└── frontend/src/utils/
    ├── security.ts                          ← Tạo mới
    ├── fileValidator.ts                     ← Tạo mới
    └── sanitize.ts                          ← Tạo mới
```

---

## ✅ Yêu cầu chức năng

### `SECURITY.md` (5 sections)

```markdown
# Chính sách Bảo mật – Video Crypto

## 1. Tổng quan
- Dự án mã hóa video, xử lý dữ liệu nhạy cảm
- Không lưu trữ key, không có database

## 2. Nguyên tắc Cryptographic
- AES-256-GCM (KHÔNG CBC/ECB)
- RSA-OAEP-SHA256 (KHÔNG PKCS1v15)
- RSA-PSS-SHA256 cho chữ ký (KHÔNG PKCS1v15)
- os.urandom() cho tất cả giá trị ngẫu nhiên

## 3. Chính sách Key
- Key sinh tại Runtime, KHÔNG lưu trên server
- Key truyền qua HTTPS (response header hoặc request body)
- Frontend clear key khỏi memory sau khi dùng xong

## 4. API Security
- Rate limiting: 10 req/phút cho encrypt/decrypt, 30 req/phút cho key generation
- CORS: chỉ allow origin từ .env
- Security headers trên mọi response
- File validation: extension + MIME + magic bytes

## 5. Báo cáo lỗ hổng
- Liên hệ: [email dự án]
```

### `middleware/security_headers.py`

```python
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response: Response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()"
        # Xóa header Server để không lộ thông tin backend
        if "server" in response.headers:
            del response.headers["server"]
        return response
```

### `middleware/rate_limiter.py`

```python
import time
from collections import defaultdict
from fastapi import Request, HTTPException

class SimpleRateLimiter:
    def __init__(self, max_requests: int = 10, window_seconds: int = 60):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.requests: dict[str, list[float]] = defaultdict(list)

    async def __call__(self, request: Request):
        client_ip = request.client.host if request.client else "unknown"
        now = time.time()
        # Xóa request cũ ngoài window
        self.requests[client_ip] = [
            t for t in self.requests[client_ip]
            if now - t < self.window_seconds
        ]
        if len(self.requests[client_ip]) >= self.max_requests:
            raise HTTPException(
                status_code=429,
                detail="Quá nhiều yêu cầu. Vui lòng thử lại sau."
            )
        self.requests[client_ip].append(now)

# Pre-configured instances
encrypt_limiter = SimpleRateLimiter(max_requests=10, window_seconds=60)
keygen_limiter = SimpleRateLimiter(max_requests=30, window_seconds=60)
```

### `utils/file_validator.py`

```python
import mimetypes
from fastapi import UploadFile, HTTPException

# Magic bytes cho định dạng video phổ biến
VIDEO_MAGIC_BYTES = {
    b'\x00\x00\x00': ['mp4', 'mov'],     # ftyp box (MP4/MOV)
    b'\x1a\x45\xdf\xa3': ['mkv', 'webm'], # EBML header (MKV/WebM)
    b'\x52\x49\x46\x46': ['avi'],          # RIFF header (AVI)
}

ALLOWED_EXTENSIONS = {'mp4', 'mkv', 'avi', 'mov', 'webm'}
ALLOWED_MIME_TYPES = {'video/mp4', 'video/x-matroska', 'video/avi', 'video/quicktime', 'video/webm'}

async def validate_video_file(file: UploadFile, max_size_bytes: int = None) -> None:
    """Validate video file: extension + MIME type + magic bytes."""
    # 1. Extension check
    filename = file.filename or ""
    ext = filename.rsplit('.', 1)[-1].lower() if '.' in filename else ""
    if ext not in ALLOWED_EXTENSIONS:
        raise HTTPException(400, f"Định dạng file không hỗ trợ: .{ext}. Chỉ chấp nhận: {', '.join(ALLOWED_EXTENSIONS)}")

    # 2. MIME type check
    mime = file.content_type or mimetypes.guess_type(filename)[0] or ""
    if mime not in ALLOWED_MIME_TYPES:
        raise HTTPException(400, f"MIME type không hợp lệ: {mime}")

    # 3. Magic bytes check
    header = await file.read(16)
    await file.seek(0)  # Reset file pointer

    if len(header) == 0:
        raise HTTPException(400, "File rỗng")

    magic_valid = False
    for magic, exts in VIDEO_MAGIC_BYTES.items():
        if header.startswith(magic) and ext in exts:
            magic_valid = True
            break
    # Một số file có thể có magic bytes khác, chỉ cảnh báo
    # if not magic_valid:
    #     raise HTTPException(400, "Magic bytes không khớp với định dạng file")

    # 4. Size check (nếu có giới hạn)
    if max_size_bytes:
        content = await file.read()
        await file.seek(0)
        if len(content) > max_size_bytes:
            size_mb = max_size_bytes / (1024 * 1024)
            raise HTTPException(413, f"File vượt quá giới hạn {size_mb:.0f}MB")

async def validate_enc_file(file: UploadFile, expected_ext: str) -> None:
    """Validate encrypted file: chỉ kiểm tra extension."""
    filename = file.filename or ""
    if not filename.endswith(expected_ext):
        raise HTTPException(400, f"Yêu cầu file {expected_ext}")
```

### `utils/crypto_utils.py`

```python
import os
import base64

def generate_random_bytes(length: int) -> bytes:
    """Sinh random bytes an toàn bằng os.urandom()."""
    return os.urandom(length)

def generate_aes_key() -> tuple[bytes, str]:
    """Sinh AES-256 key (32 bytes) và trả về cả dạng bytes lẫn base64."""
    key = os.urandom(32)
    key_b64 = base64.b64encode(key).decode('utf-8')
    return key, key_b64

def decode_key_b64(key_b64: str, expected_length: int = 32) -> bytes:
    """Decode base64 key và validate độ dài."""
    try:
        key_b64 = key_b64.strip()
        key = base64.b64decode(key_b64)
    except Exception:
        raise ValueError("Key không phải base64 hợp lệ")
    if len(key) != expected_length:
        raise ValueError(f"Key phải có đúng {expected_length} bytes, nhận được {len(key)} bytes")
    return key
```

### `utils/error_handlers.py`

```python
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

async def http_exception_handler(request: Request, exc: HTTPException):
    """Trả lỗi HTTP không chứa stack trace."""
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail}
    )

async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """Chuẩn hóa lỗi validation — KHÔNG lộ internal structure."""
    errors = []
    for error in exc.errors():
        field = " → ".join(str(loc) for loc in error["loc"])
        errors.append(f"{field}: {error['msg']}")
    return JSONResponse(
        status_code=422,
        content={"detail": "Dữ liệu không hợp lệ", "errors": errors}
    )

async def general_exception_handler(request: Request, exc: Exception):
    """Catch-all: KHÔNG expose stack trace Python."""
    # TODO: Log error chi tiết vào file log (không phải console)
    return JSONResponse(
        status_code=500,
        content={"detail": "Lỗi hệ thống. Vui lòng thử lại sau."}
    )
```

### Đăng ký vào `main.py`

```python
# Thêm vào main.py (sau CORS middleware):
from app.middleware.security_headers import SecurityHeadersMiddleware
from app.utils.error_handlers import (
    http_exception_handler,
    validation_exception_handler,
    general_exception_handler
)
from fastapi.exceptions import RequestValidationError

app.add_middleware(SecurityHeadersMiddleware)

app.add_exception_handler(HTTPException, http_exception_handler)
app.add_exception_handler(RequestValidationError, validation_exception_handler)
app.add_exception_handler(Exception, general_exception_handler)
```

### Frontend: `utils/security.ts`

```typescript
/**
 * Download blob an toàn — tự dọn dẹp URL object sau khi download.
 */
export function downloadBlob(blob: Blob, filename: string): void {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url); // Dọn memory
}

/**
 * Xóa chuỗi nhạy cảm (key) khỏi memory.
 * Lưu ý: JavaScript không cho phép xóa hoàn toàn khỏi memory,
 * nhưng overwrite giúp giảm rủi ro.
 */
export function clearSensitiveData(setter: (val: string) => void): void {
  setter('');  // Overwrite giá trị
}
```

### Frontend: `utils/fileValidator.ts`

```typescript
const ALLOWED_VIDEO_EXTENSIONS = ['mp4', 'mkv', 'avi', 'mov', 'webm'];
const ALLOWED_ENC_EXTENSIONS = ['.enc', '.rsa.enc', '.hybrid.enc', '.signed.enc'];

export function isVideoFile(file: File): boolean {
  const ext = file.name.split('.').pop()?.toLowerCase() || '';
  return ALLOWED_VIDEO_EXTENSIONS.includes(ext);
}

export function isEncryptedFile(file: File, expectedExt?: string): boolean {
  if (expectedExt) {
    return file.name.endsWith(expectedExt);
  }
  return ALLOWED_ENC_EXTENSIONS.some(ext => file.name.endsWith(ext));
}

export function formatFileSize(bytes: number): string {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  if (bytes < 1024 * 1024 * 1024) return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
  return `${(bytes / (1024 * 1024 * 1024)).toFixed(2)} GB`;
}
```

### Frontend: `utils/sanitize.ts`

```typescript
export function escapeHtml(str: string): string {
  const map: Record<string, string> = {
    '&': '&amp;', '<': '&lt;', '>': '&gt;',
    '"': '&quot;', "'": '&#039;'
  };
  return str.replace(/[&<>"']/g, c => map[c]);
}

export function sanitizeFilename(filename: string): string {
  return filename.replace(/[^a-zA-Z0-9._-]/g, '_').substring(0, 255);
}
```

---

## 🔐 Security Checklist (Baseline cho Prompt 04-12)

> **Tất cả prompt từ 04 trở đi PHẢI pass checklist này.**
> Mỗi report sẽ đánh dấu từng mục là ✅ Pass / ❌ Fail / — N/A.

| # | Tiêu chí | Mô tả |
|---|----------|-------|
| S01 | AES-256-GCM | Dùng AES-256-GCM (KHÔNG CBC/ECB) |
| S02 | Nonce unique | Nonce mới cho mỗi chunk (không tái sử dụng) |
| S03 | RSA-OAEP-SHA256 | RSA dùng OAEP-SHA256 padding (KHÔNG PKCS1v15) |
| S04 | RSA-PSS | Chữ ký dùng RSA-PSS (KHÔNG PKCS1v15) |
| S05 | os.urandom() | Tất cả random values dùng `os.urandom()` |
| S06 | Không log key | Key KHÔNG xuất hiện trong log, console, file |
| S07 | Cleanup temp | Temp file phải xóa sau khi xử lý xong |
| S08 | File validation | Validate extension + MIME type + magic bytes |
| S09 | Rate limiting | Rate limiter apply trên tất cả endpoint nhạy cảm |
| S10 | No stack trace | Error response KHÔNG chứa Python stack trace |
| S11 | Security headers | X-Content-Type-Options, X-Frame-Options, etc. |
| S12 | Key cleared (FE) | Frontend xóa key khỏi state sau khi download |
| S13 | No localStorage (FE) | Key KHÔNG lưu vào localStorage/sessionStorage |
| S14 | Validate trước API (FE) | Frontend validate input (Zod) TRƯỚC khi gọi API |

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Mọi thuật toán crypto: AES-256-GCM, RSA-OAEP-SHA256, RSA-PSS
- Mọi random values: `os.urandom()` (KHÔNG dùng `random.randbytes()`)
- Mọi response có security headers
- Error response trả message tiếng Việt, KHÔNG trả stack trace

❌ **NGHIÊM CẤM:**
- AES-CBC, AES-ECB, DES, 3DES, RC4 — KHÔNG dùng bất kỳ thuật toán yếu nào
- PKCS1v15 padding — KHÔNG dùng cho encryption hay signature
- `random.random()`, `random.randbytes()` — KHÔNG dùng cho crypto
- `print(key)`, `logging.info(key)` — KHÔNG log key ra bất kỳ đâu
- `localStorage.setItem("key", ...)` — KHÔNG lưu key phía client

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| Rate limiter bị reset khi restart server | Chấp nhận — in-memory rate limiter đủ cho dự án học thuật |
| File upload 0 bytes | `file_validator.py` phải reject |
| File .exe đổi tên thành .mp4 | Magic bytes check nên phát hiện (nhưng không bắt buộc 100%) |
| Concurrent requests flood | Rate limiter xử lý theo IP |
| Exception chưa handle | `general_exception_handler` catch-all, trả 500 generic |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG tạo middleware phức tạp với database connection — dùng in-memory đơn giản
2. KHÔNG implement JWT/session authentication — dự án không yêu cầu
3. KHÔNG dùng bcrypt hay argon2 — không có password trong dự án
4. KHÔNG tạo custom cipher implementations — dùng `cryptography` library
5. KHÔNG catch exception rồi `pass` silently — luôn trả response có ý nghĩa

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Security headers | `GET /health`, kiểm tra response headers | Có X-Content-Type-Options, X-Frame-Options |
| T02 | No Server header | Kiểm tra response headers | KHÔNG có "Server: uvicorn" |
| T03 | Rate limit encrypt | Gửi 11 requests POST `/api/aes/` trong 1 phút | Request 11 → HTTP 429 |
| T04 | Rate limit keygen | Gửi 31 requests GET `/api/aes/` trong 1 phút | Request 31 → HTTP 429 |
| T05 | Fake file | Upload file `.exe` đổi tên thành `.mp4` | Bị reject (MIME hoặc magic bytes) |
| T06 | Error 500 | Gây lỗi server (ví dụ: endpoint lỗi) | Response KHÔNG chứa stack trace, chỉ message tiếng Việt |
| T07 | Validation error | Gửi request thiếu required field | HTTP 422 với message tiếng Việt |
| T08 | CORS sai origin | Gọi API từ `http://evil.com` | CORS bị chặn |
| T09 | Empty file | Upload file 0 bytes | HTTP 400 "File rỗng" |
| T10 | DevTools Console | Thực hiện bất kỳ thao tác → kiểm tra console | KHÔNG thấy key nào trong log |
| T11 | localStorage | Thực hiện thao tác → DevTools → Application → Local Storage | Không có key |

---

## 📋 Checklist tự kiểm tra

- [ ] `SECURITY.md` có đủ 5 sections
- [ ] `security_headers.py` tạo xong, headers được apply
- [ ] Header "Server" bị xóa khỏi response
- [ ] `rate_limiter.py` tạo xong với 2 instances (encrypt + keygen)
- [ ] `file_validator.py` validate extension + MIME + magic bytes + size
- [ ] `crypto_utils.py` có `generate_random_bytes`, `generate_aes_key`, `decode_key_b64`
- [ ] `error_handlers.py` có 3 handlers, không expose stack trace
- [ ] Tất cả handlers đăng ký vào `main.py`
- [ ] `security.ts` có `downloadBlob` và `clearSensitiveData`
- [ ] `fileValidator.ts` validate đủ loại file
- [ ] `sanitize.ts` có `escapeHtml` và `sanitizeFilename`
- [ ] Không có key nào bị log ra console ở bất kỳ file nào

---

## 🔄 Self-Verification

Sau khi implement xong, bạn PHẢI:

1. **Grep toàn bộ codebase:** `grep -rn "print.*key" app/` → Phải trả về 0 kết quả
2. **Kiểm tra headers:** `curl -I http://localhost:8000/health` → Thấy security headers
3. **Test rate limit:** Chạy loop `for i in $(seq 15); do curl -s -o /dev/null -w "%{http_code}" http://localhost:8000/api/aes/; done` → Cuối cùng phải thấy 429
4. **Đọc lại Security Checklist** → Đánh dấu tất cả mục pass

Chỉ báo **"HOÀN THÀNH"** khi 14 điểm Security Checklist đều sẵn sàng sử dụng.
