# 前端（React + TypeScript）通用约定

全栈项目的 `web/` 与纯前端项目遵守本文。
后端 / 根编排仍遵循 `reference/python-conventions.md`（若有 Python）。

## 工具链

| 工具 | 用途 | 备注 |
|------|------|------|
| [pnpm](https://pnpm.io/) | 包管理 | 禁止 npm / yarn 作为默认 |
| [Vite](https://vite.dev/) | 构建与开发服务器 | `pnpm create vite@latest` |
| [TypeScript](https://www.typescriptlang.org/) | 类型 | strict |
| [oxlint](https://oxc.rs/docs/guide/usage/linter) | Lint | OXC；默认，不用 ESLint |
| [oxfmt](https://oxc.rs/docs/guide/usage/formatter) | 格式化 | OXC；默认，不用 Prettier / Biome |
| [npm-check-updates](https://github.com/raineorshine/npm-check-updates) | 依赖升级交互 | `pnpm deps` |

根目录任务仍由 **Just** 编排；`web/package.json` 只定义前端内部 scripts，由 Just 转发（如 `cd web && pnpm lint`）。

## 依赖管理

**禁止手写依赖版本号。** 用 pnpm 安装最新版：

```bash
pnpm add <pkg>
pnpm add -D <pkg>
```

## package.json 脚本参考

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "typecheck": "tsc -b --pretty false",
    "lint": "oxlint",
    "format": "oxfmt --check src/",
    "fix": "oxlint --fix && oxfmt src/",
    "preview": "vite preview",
    "generate": "openapi-ts",
    "deps": "ncu -i --format group"
  }
}
```

全栈项目必须有 `generate`（`@hey-api/openapi-ts`）。纯静态前端可省略。

## 默认技术选型

| 层 | 默认 | 说明 |
|----|------|------|
| UI 运行时 | React 19 + Vite + TS | |
| 样式 | Tailwind CSS v4（`@tailwindcss/vite`） | 也可用项目选定的组件库自带方案 |
| API client | `@hey-api/openapi-ts` 生成 | 全栈强制；禁止手改 `web/src/client` |
| 路由 | 按需 | 小项目可手写 / 单页；变复杂再加 React Router 或 TanStack Router |
| 服务端状态 | 按需 | 有缓存/失效需求再加 TanStack Query；不要默认塞进骨架 |
| 客户端 UI 状态 | 按需 | 可用 Zustand；不要为了用而用 |

骨架默认保持薄：`main.tsx` + 简单页面 +（全栈）生成 client 调用示例。Query / Router 写在子 skill「按需追加」节，不强制安装。

## TypeScript 规范

- 禁止 `any`；尽量避免 `as` 断言（必要情况在注释说明原因）
- 类型导入用 `import type`
- 路径别名：`@/*` → `./src/*`

## 与根 Justfile 的关系

```text
just check     → 含 cd web && pnpm lint / format check / typecheck
just fix       → 含 cd web && pnpm fix
just generate  → 导出 OpenAPI + cd web && pnpm generate
just dev-web   → cd web && pnpm dev
```

人与 Agent 日常只敲 `just …`，不要把 `cd web && pnpm …` 当成对外文档里的主入口（调试前端时可直接用）。

## 禁止事项

- 禁止默认使用 ESLint / Prettier / Biome（新项目用 OXC；维护旧项目除外）
- 禁止手改 OpenAPI 生成的 client
- 禁止在未 `just generate` 的前提下假设前端类型与后端一致
- 禁止用 npm / yarn 作为包管理器
