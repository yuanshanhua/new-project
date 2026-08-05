# AGENTS.md

## 项目概览

<一句话描述>.

- **后端 / 包**: Python >=3.12, `uv` 管理依赖, src 布局
- **任务入口**: 根目录 `just`（`just --list`）；不要绕开 Justfile 自造正式入口
- **质量门禁**: 提交前必须 `just check` 通过

<!-- 全栈时取消注释并填写
- **前端**: React + Vite + TypeScript, 目录 `web/`, `pnpm` 管理
- **API 契约**: OpenAPI → `web/src/client`（hey-api）；改路由/schema 后必须 `just generate`
-->

## 目录结构

```text
src/<package>/     # 主包
tests/             # pytest
<!-- 全栈
web/               # React SPA；src/client/ 由 OpenAPI 生成，勿手改
scripts/           # 含 export_openapi.py
-->
```

## 工具与门禁

- 常用任务走 `just`（`just --list`）。
- **提交前必须跑 `just check`**（lint + format + typecheck + test）。可用 `just fix` 先自动修。
- Git hooks（prek）：提交时执行 `just check`。
- CI（GitHub Actions）：push / PR 上执行同一套 `just check`。

<!-- 全栈
- 改 FastAPI 路由 / schema / operation_id 后，必须 `just generate` 再改前端；**禁止手改** `web/src/client`。
-->

## Always

- 用 `just` 跑任务；提交前 `just check`
- 类型安全：Python 完备标注；TS 禁止 `any`
- 密钥只走环境变量 / 本地 `.env`（勿提交）

## Never

- 绕过 Justfile 把临时脚本当成正式检查/启动入口
- 提交 `.env`、secrets、`data/` 产物
<!-- 全栈
- 在未 `generate` 时假设前端类型与后端一致
-->

## 文档约定

- `AGENTS.md` 保持短；细节放 `docs/`（若有）。
- 文档只写边界、顺序、契约、取舍、踩坑；**不**抄字段/签名，**不**写演进史（「从 A 改成 B」）。
- 改架构或工作流时同步更新文档。
