# 🔐 Video Crypto Project – Tổng quan Prompt

## Thông tin dự án

- **Tên dự án:** Website mã hóa & giải mã video
- **Backend:** Python + FastAPI
- **Frontend:** React + Vite + Tailwind CSS v3 + Zustand + Zod
- **Không dùng database**
- **Ngôn ngữ UI:** Tiếng Việt
- **Cấu trúc:** Monorepo

---

## Cấu trúc thư mục dự án (sau khi hoàn thành)

```
video-crypto/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── middleware/
│   │   │   ├── security_headers.py
│   │   │   ├── rate_limiter.py
│   │   │   └── file_validator.py
│   │   ├── routers/
│   │   │   ├── aes.py
│   │   │   ├── rsa.py
│   │   │   ├── hybrid.py
│   │   │   └── signature.py
│   │   ├── services/
│   │   │   ├── aes_service.py
│   │   │   ├── rsa_service.py
│   │   │   ├── hybrid_service.py
│   │   │   └── signature_service.py
│   │   ├── schemas/
│   │   │   ├── aes_schema.py
│   │   │   ├── rsa_schema.py
│   │   │   ├── hybrid_schema.py
│   │   │   └── signature_schema.py
│   │   └── utils/
│   │       ├── crypto_utils.py
│   │       └── error_handlers.py
│   ├── requirements.txt
│   └── .env
├── frontend/
│   ├── wireframes/
│   │   ├── index.html
│   │   ├── aes.html
│   │   ├── rsa.html
│   │   ├── hybrid.html
│   │   ├── signature.html
│   │   ├── components.html
│   │   └── wireframe-guide.md
│   ├── src/
│   │   ├── components/
│   │   │   ├── common/
│   │   │   └── crypto/
│   │   ├── pages/
│   │   │   ├── HomePage.tsx
│   │   │   ├── AESPage.tsx
│   │   │   ├── RSAPage.tsx
│   │   │   ├── HybridPage.tsx
│   │   │   └── SignaturePage.tsx
│   │   ├── stores/
│   │   ├── schemas/
│   │   ├── services/
│   │   └── utils/
│   │       ├── security.ts
│   │       ├── fileValidator.ts
│   │       └── sanitize.ts
│   └── package.json
├── SECURITY.md
├── docs/                                    ← 12 prompt files
├── report/                                  ← 12 report files
└── README.md
```

---

## 📋 Danh sách Prompt (12 prompts)

| Prompt | Giai đoạn | Nội dung | Report |
|--------|-----------|----------|--------|
| [prompt01.md](./docs/prompt01.md) | 🏗️ Setup | Khởi tạo monorepo, cấu hình project | [report01.md](./report/report01.md) |
| [prompt02.md](./docs/prompt02.md) | 🎨 Design | Wireframe toàn bộ UI (HTML tĩnh) | [report02.md](./report/report02.md) |
| [prompt03.md](./docs/prompt03.md) | 🔐 Security | Yêu cầu bảo mật, middleware, utils | [report03.md](./report/report03.md) |
| [prompt04.md](./docs/prompt04.md) | Phase 1 – AES | Backend: AES-256-GCM encrypt/decrypt | [report04.md](./report/report04.md) |
| [prompt05.md](./docs/prompt05.md) | Phase 1 – AES | Frontend: React UI AES | [report05.md](./report/report05.md) |
| [prompt06.md](./docs/prompt06.md) | Phase 2 – RSA | Backend: RSA-2048 encrypt/decrypt | [report06.md](./report/report06.md) |
| [prompt07.md](./docs/prompt07.md) | Phase 2 – RSA | Frontend: React UI RSA | [report07.md](./report/report07.md) |
| [prompt08.md](./docs/prompt08.md) | Phase 3 – Hybrid | Backend: Hybrid Encryption | [report08.md](./report/report08.md) |
| [prompt09.md](./docs/prompt09.md) | Phase 3 – Hybrid | Frontend: React UI Hybrid | [report09.md](./report/report09.md) |
| [prompt10.md](./docs/prompt10.md) | Phase 4 – Chữ ký số | Backend: Digital Signature + Hybrid | [report10.md](./report/report10.md) |
| [prompt11.md](./docs/prompt11.md) | Phase 4 – Chữ ký số | Frontend: React UI Digital Signature | [report11.md](./report/report11.md) |
| [prompt12.md](./docs/prompt12.md) | 🔍 Final Audit | Kiểm tra toàn bộ dự án (Security + Crypto + Integration + Code + UX) | [report12.md](./report/report12.md) |

---

## 🔄 Quy trình thực hiện

```
[Đọc promptXX.md]
      ↓
[Implement code theo đúng spec]
      ↓
[Tự test thủ công theo Test Cases trong prompt]
      ↓
[Fail?] ──Yes──▶ [Sửa code] ──▶ [Test lại]
  │
  No
  ↓
[Điền reportXX.md – đánh dấu từng yêu cầu]
      ↓
[Kiểm tra Security Checklist (report04-11)]
      ↓
[ĐẠT] ──▶ [Chuyển sang prompt tiếp theo]
      ↓
[Sau prompt11 – Thực hiện Prompt 12: Final Audit]
      ↓
[Tất cả tiêu chí đạt] ──▶ ✅ DỰ ÁN HOÀN THÀNH
```

---

## 📏 Quy chuẩn đánh giá

| Trạng thái | Ký hiệu | Ý nghĩa |
|-----------|---------|---------|
| Tuân thủ | ✅ | Code khớp hoàn toàn với yêu cầu |
| Thiếu sót | ⚠️ | Implement nhưng thiếu/sai một phần |
| Vi phạm | ❌ | Implement sai/ngược yêu cầu |
| Chưa implement | ⏳ | Có trong yêu cầu nhưng không có code |
| Ngoài yêu cầu | 🔵 | Có trong code nhưng không trong yêu cầu |

---

## 🔑 Thiết kế định dạng file

### AES – File `.enc`

```
[4 bytes: uint32 big-endian = độ dài header]
[N bytes: JSON header]
[Chunk×N: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
```

### RSA – File `.rsa.enc`

```
[4 bytes: header_len][N bytes: JSON header]
[Chunk×N: 2 bytes chunk_len | RSA ciphertext (256 bytes)]
```

### Hybrid – File `.hybrid.enc`

```
[4 bytes: header_len][N bytes: JSON header]
[256 bytes: RSA-encrypted AES session key]
[Chunk×N: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
```

### Digital Signature – File `.signed.enc`

```
[4 bytes: header_len][N bytes: JSON header]
[256 bytes: RSA-PSS Digital Signature]
[256 bytes: RSA-encrypted AES session key]
[Chunk×N: 12 bytes nonce | ciphertext | 16 bytes GCM tag]
```

---

## 🔐 Security Baseline (Prompt 03)

Từ Prompt 04 trở đi, mỗi report đều có **Security Checklist 14 điểm** cần pass trước khi chuyển prompt:

- AES-256-GCM | RSA-OAEP-SHA256 | RSA-PSS
- os.urandom() cho tất cả random
- Nonce không tái sử dụng
- Key không log, không lưu server
- File validation: extension + MIME + magic bytes
- Rate limiting trên endpoint
- Error không expose stack trace
- Security headers trong response
- Frontend: key cleared sau download, không lưu localStorage
