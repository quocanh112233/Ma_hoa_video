# Prompt 01 – Khởi tạo Project Monorepo (FastAPI + React)

---

## 📝 TL;DR

- **Input:** Không có (khởi tạo từ đầu)
- **Output:** Monorepo hoàn chỉnh với backend (FastAPI) và frontend (React/Vite)
- **Files to create:** ~15 files cấu trúc thư mục, config, skeleton code
- **Key constraint:** Chỉ setup skeleton, KHÔNG implement logic crypto
- **Definition of Done:** Backend chạy port 8000 + Frontend chạy port 5173 + 11 test cases pass

---

## 🎭 Role

Bạn là **Project Architect & DevOps Engineer** chuyên về Python/FastAPI và React. Nhiệm vụ là khởi tạo monorepo cho dự án mã hóa video, đảm bảo cấu trúc sạch sẽ, đúng chuẩn, tất cả config chính xác, và cả hai phần backend/frontend chạy được ngay sau khi setup.

---

## 📋 Context

- Dự án xây dựng website mã hóa/giải mã video sử dụng nhiều thuật toán
- Kiến trúc Monorepo: 1 repository chứa cả backend và frontend
- Không có database, không có authentication
- Đây là prompt ĐẦU TIÊN — chưa có code nào tồn tại

---

## 🎯 Mục tiêu

1. Tạo cấu trúc thư mục monorepo hoàn chỉnh
2. Cấu hình backend FastAPI (Python 3.11+) với 4 routers placeholder
3. Cấu hình frontend React + Vite + TypeScript + Tailwind CSS v3
4. Đảm bảo cả hai phần có thể chạy độc lập không lỗi

---

## 📁 Cấu trúc thư mục yêu cầu

```
video-crypto/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              ← FastAPI app entry point
│   │   ├── config.py            ← Load .env
│   │   ├── routers/
│   │   │   ├── __init__.py
│   │   │   ├── aes.py           ← Placeholder router
│   │   │   ├── rsa.py           ← Placeholder router
│   │   │   ├── hybrid.py        ← Placeholder router
│   │   │   └── signature.py     ← Placeholder router
│   │   ├── services/            ← Tạo folder rỗng + __init__.py
│   │   ├── schemas/             ← Tạo folder rỗng + __init__.py
│   │   ├── middleware/          ← Tạo folder rỗng + __init__.py
│   │   └── utils/               ← Tạo folder rỗng + __init__.py
│   ├── requirements.txt
│   ├── .env
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   ├── index.css
│   │   ├── pages/
│   │   │   ├── HomePage.tsx
│   │   │   ├── AESPage.tsx      ← Placeholder
│   │   │   ├── RSAPage.tsx      ← Placeholder
│   │   │   ├── HybridPage.tsx   ← Placeholder
│   │   │   └── SignaturePage.tsx ← Placeholder
│   │   ├── components/          ← Tạo folder rỗng
│   │   ├── stores/              ← Tạo folder rỗng
│   │   ├── services/            ← Tạo folder rỗng
│   │   ├── schemas/             ← Tạo folder rỗng
│   │   └── utils/               ← Tạo folder rỗng
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   └── tailwind.config.js
├── README.md
└── .gitignore
```

---

## ✅ Yêu cầu chức năng

### Backend (FastAPI)

#### `app/main.py`

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.config import settings
from app.routers import aes, rsa, hybrid, signature

app = FastAPI(
    title="Video Crypto API",
    description="API mã hóa và giải mã video",
    version="1.0.0"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=[settings.CORS_ORIGIN],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routers
app.include_router(aes.router, prefix="/api/aes", tags=["AES"])
app.include_router(rsa.router, prefix="/api/rsa", tags=["RSA"])
app.include_router(hybrid.router, prefix="/api/hybrid", tags=["Hybrid"])
app.include_router(signature.router, prefix="/api/signature", tags=["Signature"])

@app.get("/")
async def root():
    return {
        "project": "Video Crypto",
        "version": "1.0.0",
        "description": "Hệ thống mã hóa và giải mã video"
    }

@app.get("/health")
async def health_check():
    return {"status": "ok"}
```

#### `app/config.py`

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    CORS_ORIGIN: str = "http://localhost:5173"
    MAX_CHUNK_SIZE: int = 10 * 1024 * 1024  # 10MB

    class Config:
        env_file = ".env"

settings = Settings()
```

#### Mỗi router placeholder (`routers/aes.py`, `rsa.py`, `hybrid.py`, `signature.py`)

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/")
async def index():
    return {"module": "<tên module>", "status": "not_implemented"}
```

#### `requirements.txt` (pin exact versions)

```
fastapi==0.115.6
uvicorn[standard]==0.34.0
python-multipart==0.0.20
pydantic-settings==2.7.1
cryptography==44.0.0
python-dotenv==1.0.1
```

#### `.env`

```
CORS_ORIGIN=http://localhost:5173
MAX_CHUNK_SIZE=10485760
```

### Frontend (React + Vite)

#### Dependencies (`package.json` — devDependencies riêng)

```json
{
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^7.1.1",
    "axios": "^1.7.9",
    "zustand": "^5.0.3",
    "zod": "^3.24.1"
  },
  "devDependencies": {
    "@types/react": "^18.3.1",
    "@types/react-dom": "^18.3.1",
    "@vitejs/plugin-react": "^4.3.4",
    "tailwindcss": "^3.4.17",
    "postcss": "^8.4.49",
    "autoprefixer": "^10.4.20",
    "typescript": "^5.7.3",
    "vite": "^6.0.7"
  }
}
```

#### `App.tsx` — React Router

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Navbar from './components/Navbar';
import HomePage from './pages/HomePage';
import AESPage from './pages/AESPage';
import RSAPage from './pages/RSAPage';
import HybridPage from './pages/HybridPage';
import SignaturePage from './pages/SignaturePage';

function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/aes" element={<AESPage />} />
        <Route path="/rsa" element={<RSAPage />} />
        <Route path="/hybrid" element={<HybridPage />} />
        <Route path="/signature" element={<SignaturePage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

#### `components/Navbar.tsx`

- 4 links điều hướng: Trang chủ, Mã hóa AES, Mã hóa RSA, Mã hóa Lai, Chữ ký số
- Toàn bộ text tiếng Việt
- Active state highlight

#### Mỗi page placeholder

```tsx
export default function AESPage() {
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold">Mã hóa AES</h1>
      <p className="text-gray-500 mt-2">Đang phát triển...</p>
    </div>
  );
}
```

#### Axios config (`services/api.ts`)

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:8000',
  timeout: 300000, // 5 phút cho video lớn
});

export default api;
```

---

## 🛡️ Constraints

✅ **BẮT BUỘC:**
- Pin exact versions trong `requirements.txt` (không dùng `>=` không giới hạn)
- CORS origin phải đọc từ `.env`, KHÔNG hardcode
- Tất cả text hiển thị trên UI phải bằng tiếng Việt
- Mỗi router phải có ít nhất 1 endpoint placeholder
- Frontend dùng TypeScript strict mode

❌ **NGHIÊM CẤM:**
- KHÔNG implement bất kỳ logic mã hóa/giải mã nào ở bước này
- KHÔNG tạo database hoặc ORM model
- KHÔNG tạo authentication/authorization
- KHÔNG cài thêm dependencies ngoài danh sách đã liệt kê
- KHÔNG dùng JavaScript thuần — phải dùng TypeScript

---

## ⚠️ Edge Cases & Error Recovery

| Tình huống | Cách xử lý |
|-----------|------------|
| `pip install` bị lỗi version conflict | Tạo virtual environment mới: `python -m venv venv` |
| `npm install` bị lỗi peer dependency | Chạy `npm install --legacy-peer-deps` |
| Port 8000 đã bị chiếm | Thêm `--port 8001` vào lệnh uvicorn |
| Port 5173 đã bị chiếm | Vite sẽ tự chuyển sang port khác |
| Tailwind CSS không apply | Kiểm tra `tailwind.config.js` có đúng `content` paths |
| Import error khi chạy FastAPI | Chạy từ thư mục `backend/`: `uvicorn app.main:app --reload` |

---

## 🚫 Anti-patterns (KHÔNG làm)

1. KHÔNG tạo file `database.py` hay bất kỳ ORM nào
2. KHÔNG dùng `from fastapi import Depends` để inject dependencies phức tạp ở bước này
3. KHÔNG tạo `Dockerfile` hay `docker-compose.yml` ở bước này
4. KHÔNG viết unit tests ở bước này — chỉ manual test
5. KHÔNG dùng `npm create vite` interactive mode — phải chạy non-interactive

---

## 🧪 Test thủ công (Manual Test Cases)

| # | Test | Hành động | Kết quả mong đợi |
|---|------|-----------|-----------------|
| T01 | Backend khởi động | `cd backend && uvicorn app.main:app --reload` | Server chạy trên port 8000 không lỗi |
| T02 | Health check | `GET http://localhost:8000/health` | `{"status":"ok"}` |
| T03 | Swagger UI | Mở browser `http://localhost:8000/docs` | Thấy Swagger UI |
| T04 | Root endpoint | `GET http://localhost:8000/` | JSON có `project: "Video Crypto"` |
| T05 | Frontend khởi động | `cd frontend && npm run dev` | Vite dev server chạy port 5173 |
| T06 | Trang chủ | Mở `http://localhost:5173/` | Thấy trang chủ tiếng Việt |
| T07 | Navigate AES | Click link "Mã hóa AES" trên navbar | Chuyển sang `/aes`, thấy "Đang phát triển..." |
| T08 | Navigate RSA | Click link "Mã hóa RSA" | Chuyển sang `/rsa` |
| T09 | Navigate Hybrid | Click link "Mã hóa Lai" | Chuyển sang `/hybrid` |
| T10 | Navigate Signature | Click link "Chữ ký số" | Chuyển sang `/signature` |
| T11 | Tailwind CSS | Kiểm tra styling trên trang chủ | CSS classes được apply (font, color, spacing) |

---

## 📋 Checklist tự kiểm tra

- [ ] Cấu trúc thư mục khớp 100% với spec ở trên
- [ ] Backend: `uvicorn app.main:app --reload` chạy không lỗi
- [ ] Backend: `GET /health` trả `{"status":"ok"}`
- [ ] Backend: `GET /` trả JSON đầy đủ
- [ ] Backend: 4 routers đăng ký thành công, thấy trong Swagger
- [ ] Backend: `.env` có `CORS_ORIGIN` và `MAX_CHUNK_SIZE`
- [ ] Backend: `requirements.txt` có exact versions
- [ ] Frontend: `npm run dev` chạy không lỗi
- [ ] Frontend: Tailwind CSS hoạt động
- [ ] Frontend: React Router có đủ 5 routes
- [ ] Frontend: Navbar có 5 links tiếng Việt (Trang chủ + 4 modules)
- [ ] Frontend: Mỗi page placeholder hiển thị "Đang phát triển..."
- [ ] Frontend: Axios baseURL trỏ đúng `http://localhost:8000`
- [ ] Frontend: `npm run build` không có lỗi TypeScript
- [ ] `.gitignore` bao gồm `node_modules/`, `venv/`, `.env`, `__pycache__/`

---

## 🔄 Self-Verification

Sau khi implement xong, bạn PHẢI tự kiểm tra:

1. **Chạy backend** → Mở Swagger → Thấy 4 tag modules
2. **Chạy frontend** → Navigate tất cả 5 trang → Không có trang nào trắng
3. **Kiểm tra CORS** → Frontend gọi `GET /health` từ browser → Không có CORS error
4. **Đếm files** → Đảm bảo tất cả folder có `__init__.py` (backend) hoặc file placeholder

Chỉ báo **"HOÀN THÀNH"** khi tất cả mục trong checklist đều ✅.
