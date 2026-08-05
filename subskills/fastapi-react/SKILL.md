---
name: new-project-fastapi-react
description: >-
  Scaffold a FastAPI + React full-stack project.
  Layout: root Python package (src/) + web/ frontend.
  Backend: FastAPI, SQLAlchemy async, Alembic, SQLite, pydantic-settings.
  Frontend: React 19, Vite, TypeScript, Tailwind, OpenAPI codegen (hey-api), OXC.
  Toolchain: uv, pnpm, just, prek. Use for Python API + React SPA apps.
---

# FastAPI + React 全栈项目

> 后端遵循 `reference/python-conventions.md`；前端遵循 `reference/frontend-conventions.md`。
> 本文仅描述全栈特有的架构、依赖和目录结构。

## 架构

根目录 Python 包 + `web/` 前端。根 **Justfile** 是唯一对外任务入口；`web/package.json` 只含前端内部 scripts，由 Just 转发。

开发时 Vite 将 `/api` 代理到后端；生产可由 FastAPI 挂载 `web/dist`（按需）。

```
<project-name>/
├── Justfile                      # 根编排（从 templates/Justfile.fullstack 复制）
├── AGENTS.md
├── .env.example
├── .pre-commit-config.yaml
├── .github/workflows/ci.yml
├── pyproject.toml
├── ruff.toml
├── scripts/
│   └── export_openapi.py
├── src/<package>/
│   ├── __init__.py
│   ├── config.py                 # pydantic-settings
│   ├── database.py               # async engine + session（SQLite）
│   ├── models/base.py
│   ├── schemas/
│   ├── services/
│   └── api/
│       ├── main.py               # FastAPI app factory
│       └── router.py
├── alembic.ini
├── alembic/
│   ├── env.py
│   └── versions/
├── tests/
└── web/                          # Vite React SPA
    ├── package.json
    ├── openapi-ts.config.ts
    ├── openapi.json              # 由 export_openapi 生成
    ├── vite.config.ts
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── client/               # hey-api 生成，禁止手改
        ├── pages/
        └── components/
```

## 后端特有内容

### 额外依赖

```bash
uv add fastapi "uvicorn[standard]" "sqlalchemy[asyncio]" aiosqlite alembic pydantic-settings
```

数据库默认 **SQLite**（`sqlite+aiosqlite`）。不要默认引入 PostgreSQL / Docker Compose。

### config.py

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    app_name: str = "<project-name>"
    database_url: str = "sqlite+aiosqlite:///./data/app.db"
    cors_origins: list[str] = ["http://localhost:5173"]


settings = Settings()
```

### database.py

```python
from collections.abc import AsyncIterator

from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine

from .config import settings

engine = create_async_engine(settings.database_url, echo=False)
async_session = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)


async def get_db() -> AsyncIterator[AsyncSession]:
    async with async_session() as session:
        yield session
```

### api/main.py

```python
from contextlib import asynccontextmanager

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from <package>.api.router import api_router
from <package>.config import settings
from <package>.database import engine


@asynccontextmanager
async def lifespan(_app: FastAPI):
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

### api/router.py

```python
from fastapi import APIRouter

api_router = APIRouter()


@api_router.get("/health")
async def health() -> dict[str, str]:
    return {"status": "ok"}
```

### models/base.py

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

### Alembic

```bash
uv run alembic init alembic
```

修改 `alembic/env.py`：对 SQLite 用同步 URL 跑 DDL（`sqlite+aiosqlite` → `sqlite`）：

```python
from <package>.config import settings
from <package>.models.base import Base

sync_url = settings.database_url.replace("+aiosqlite", "")
target_metadata = Base.metadata
```

其余按 Alembic 常规 online 迁移写法接入 `sync_url`。

### scripts/export_openapi.py

```python
#!/usr/bin/env python3
"""Export FastAPI OpenAPI schema for frontend SDK generation."""

from __future__ import annotations

import json
from pathlib import Path

from <package>.api.main import app


def main() -> int:
    root = Path(__file__).resolve().parents[1]
    out = root / "web" / "openapi.json"
    out.write_text(json.dumps(app.openapi(), indent=2) + "\n", encoding="utf-8")
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

## 前端

### 创建方式

```bash
pnpm create vite@latest web --template react-ts
cd web
pnpm add -D tailwindcss @tailwindcss/vite oxlint oxfmt npm-check-updates @hey-api/openapi-ts
```

按 `reference/frontend-conventions.md` 写入 scripts（含 `generate` / `lint` / `fix` / `typecheck`）。

从 `templates/openapi-ts.config.ts` 复制到 `web/`。

### vite.config.ts

```typescript
import { resolve } from "node:path";
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: { "@": resolve(import.meta.dirname, "./src") },
  },
  server: {
    port: 5173,
    proxy: { "/api": "http://localhost:8000" },
  },
});
```

### 骨架保持薄

默认只要：`main.tsx`、简单 `App.tsx` / 首页、以及通过生成 client 调用 `/api/health` 的示例。

**不要默认安装** React Router 或 TanStack Query。需要时再追加（见下节）。

### 按需追加

| 需求 | 可选依赖 |
|------|----------|
| 多页面路由 | `pnpm add react-router`（或 TanStack Router） |
| 服务端状态缓存 / 失效 | `pnpm add @tanstack/react-query` |
| 客户端 UI 状态 | `pnpm add zustand` |

## 根 Justfile

从 `templates/Justfile.fullstack` 复制，把 `<package>` 换成实际包名。

## 搭建步骤

1. 询问用户：项目名、Python 包名
2. 创建根目录；`uv init --lib`（或在当前目录 init），`requires-python = ">=3.12"`，src 布局
3. 按 python-conventions：`uv add --dev pytest ruff ty pyright prek`；复制 `templates/ruff.toml`；配置 pyright/ty
4. `uv add fastapi "uvicorn[standard]" "sqlalchemy[asyncio]" aiosqlite alembic pydantic-settings`
5. 写入 `config.py`、`database.py`、`models/base.py`、`api/main.py`、`api/router.py`、`scripts/export_openapi.py`
6. `uv run alembic init alembic` 并改 `env.py`；`mkdir -p data`
7. 复制 `templates/Justfile.fullstack` → `Justfile`；`templates/env.example` → `.env.example`
8. 复制 `templates/AGENTS.md`（启用全栈相关段落）；`templates/.pre-commit-config.yaml`；`templates/ci.fullstack.yml` → `.github/workflows/ci.yml`
9. 前端：`pnpm create vite@latest web --template react-ts`；安装 Tailwind / OXC / hey-api；写 `vite.config.ts`、`openapi-ts.config.ts`、scripts
10. `just sync && just generate && uv run prek install && just check`
11. 验证：`just dev`（API :8000 + Web :5173）

## 质量门禁（必须写入 AGENTS.md）

- 提交前 `just check`
- 改 API 后 `just generate`；禁止手改 `web/src/client`
- prek hook 调 `just check`；CI 调同一套 Just 目标并校验 OpenAPI 生成物已提交
