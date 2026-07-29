---
name: new-project-fastapi-react
description: >-
  Scaffold a FastAPI backend + React frontend full-stack project.
  Backend: FastAPI, SQLAlchemy (async), Alembic, Pydantic v2.
  Frontend: React 19, Vite, TypeScript, React Router, Tailwind CSS.
  Toolchain: uv (Python), pnpm (Node), Docker Compose.
  Use for full-stack web apps with a Python API and React SPA.
---

# FastAPI + React 全栈项目

> 后端遵循 `reference/python-conventions.md`。本文仅描述全栈项目特有的架构、依赖和目录结构。

## 架构

Monorepo，两个顶层目录。前端开发服务器通过 Vite proxy 将 `/api` 转发到后端。

```
<project-name>/
├── docker-compose.yml            # PostgreSQL
├── Justfile                      # 顶层编排
├── backend/
│   ├── pyproject.toml            # Python 约定 + 后端特有依赖
│   ├── Justfile                  # Python 标准目标 + db-* 目标
│   ├── .env.example
│   ├── alembic.ini
│   ├── alembic/
│   │   ├── env.py
│   │   └── versions/
│   └── src/<package>/
│       ├── __init__.py
│       ├── main.py               # FastAPI app factory
│       ├── config.py             # pydantic-settings
│       ├── database.py           # 异步 engine + session 依赖
│       ├── models/base.py        # SQLAlchemy DeclarativeBase
│       ├── api/router.py         # APIRouter
│       ├── schemas/
│       └── services/
└── frontend/
    ├── src/
    │   ├── main.tsx
    │   ├── App.tsx               # React Router 路由
    │   ├── index.css             # @import "tailwindcss"
    │   ├── api/client.ts         # fetch 封装
    │   ├── pages/
    │   └── components/
    ├── vite.config.ts            # 含 /api 代理
    └── package.json
```

## 后端特有内容

### 额外依赖

在 `uv init` 生成的项目上，除 `reference/python-conventions.md` 定义的 dev 依赖外：

```bash
uv add fastapi[standard] "sqlalchemy[asyncio]" asyncpg alembic pydantic-settings
```

### backend/Justfile

基于 `reference/python-conventions.md` 的模板，追加后端特有目标。**按项目实际需求调整**。

```make
default:
    @just --list

CHECK_DIRS := "src tests"

lint:
    uv run ruff check {{CHECK_DIRS}}

format:
    uv run ruff format --check {{CHECK_DIRS}}

fix:
    uv run ruff check --fix {{CHECK_DIRS}}
    uv run ruff format {{CHECK_DIRS}}

typecheck:
    uv run pyright {{CHECK_DIRS}}
    uv run ty check {{CHECK_DIRS}}

test:
    -uv run pytest tests/ -v

check: lint format typecheck test

deps:
    uv tree -d 1 --outdated

# ↓ 后端特有目标 ↓

dev:
    uv run fastapi dev src/<package>/main.py

db-up:
    docker compose -f ../docker-compose.yml up -d db

db-migrate:
    uv run alembic upgrade head

db-revision msg:
    uv run alembic revision --autogenerate -m "{{msg}}"
```

### config.py

```python
from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    app_name: str = "<project-name>"
    database_url: str = "postgresql+asyncpg://postgres:postgres@localhost:5432/<db-name>"
    cors_origins: list[str] = ["http://localhost:5173"]

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}


settings = Settings()
```

### database.py

```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from .config import settings

engine = create_async_engine(settings.database_url, echo=False)
async_session = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)


async def get_db() -> AsyncSession:
    async with async_session() as session:
        yield session
```

### main.py

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from .config import settings
from .database import engine
from .api.router import api_router


@asynccontextmanager
async def lifespan(app: FastAPI):
    yield
    await engine.dispose()


def create_app() -> FastAPI:
    app = FastAPI(title=settings.app_name, version="0.1.0", lifespan=lifespan)
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.cors_origins,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    app.include_router(api_router, prefix="/api")
    return app


app = create_app()
```

### models/base.py

```python
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

### api/router.py

```python
from fastapi import APIRouter

api_router = APIRouter()

@api_router.get("/health")
async def health():
    return {"status": "ok"}
```

### Alembic 配置

初始化后修改 `alembic/env.py`，使用同步 URL 执行 DDL：

```python
from <package>.config import settings
from <package>.models.base import Base

sync_url = settings.database_url.replace("+asyncpg", "")
target_metadata = Base.metadata

def run_migrations_online() -> None:
    from sqlalchemy import create_engine
    connectable = create_engine(sync_url)
    with connectable.connect() as connection:
        context.configure(connection=connection, target_metadata=target_metadata)
        with context.begin_transaction():
            context.run_migrations()
```

## 前端

### 创建方式

```bash
pnpm create vite@latest frontend --template react-ts
cd frontend
pnpm add react-router-dom
pnpm add -D tailwindcss @tailwindcss/vite
```

### package.json 脚本参考

以下为 Vite + React 项目的推荐脚本，**按项目需求调整**（如替换格式化工具、添加项目特有命令等）：

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "tsc --noEmit",
    "format": "prettier --check src/",
    "fix": "prettier --write src/",
    "dep": "ncu -i --format group"
  }
}
```

> `ncu` 由 `npm-check-updates` 提供：`pnpm add -D npm-check-updates`。
> 如使用 biome 替代 prettier，相应替换 `format`/`fix` 命令。

### vite.config.ts

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  server: {
    port: 5173,
    proxy: { "/api": "http://localhost:8000" },
  },
});
```

### src/App.tsx

```tsx
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { Layout } from "./components/Layout";
import { Home } from "./pages/Home";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<Layout />}>
          <Route path="/" element={<Home />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### src/api/client.ts

```typescript
const BASE = "/api";

async function request<T>(path: string, init?: RequestInit): Promise<T> {
  const res = await fetch(`${BASE}${path}`, {
    headers: { "Content-Type": "application/json", ...init?.headers },
    ...init,
  });
  if (!res.ok) throw new Error(`${res.status} ${res.statusText}`);
  return res.json();
}

export const api = { get: <T>(p: string) => request<T>(p) };
```

`Layout.tsx`、`Home.tsx`、`pages/` 等页面组件按需创建，内容从简。

## docker-compose.yml

```yaml
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: <db-name>
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

## 顶层 Justfile

以下为 monorepo 顶层编排模板，**按项目需求调整**：

```make
default:
    @just --list

dev:
    just dev-backend & just dev-frontend & wait

dev-backend:
    cd backend && just dev

dev-frontend:
    cd frontend && pnpm dev

check:
    cd backend && just check
    cd frontend && pnpm lint

fix:
    cd backend && just fix
    cd frontend && pnpm fix

db-up:
    docker compose up -d db

db-migrate:
    cd backend && just db-migrate
```

## 搭建步骤

1. 询问用户：项目名、Python 包名、数据库名
2. 创建项目根目录，写入 `docker-compose.yml`、顶层 `Justfile`、`.gitignore`
3. **后端**：
   - `cd backend && uv init --lib`
   - 按 `reference/python-conventions.md` 用 `uv add --dev` 安装 dev 依赖，添加 ruff/ty/pyright 配置
   - `uv add fastapi[standard] "sqlalchemy[asyncio]" asyncpg alembic pydantic-settings`
   - 从 `templates/ruff.toml` 复制到 `backend/`
   - 写入 `Justfile`（标准目标 + db-* 目标）
   - 写入 `main.py`、`config.py`、`database.py`、`models/base.py`、`api/router.py`
   - `uv sync`
   - `uv run alembic init alembic` → 修改 `alembic/env.py`
4. **前端**：
   - `pnpm create vite@latest frontend --template react-ts`
   - 安装 react-router-dom、tailwind
   - 写入 `vite.config.ts`、`App.tsx`、`api/client.ts`
5. `docker compose up -d db && cd backend && just db-migrate`
6. 验证：`just dev-backend`（端口 8000）+ `just dev-frontend`（端口 5173）
