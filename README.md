# new-project skill

引导 AI Agent 创建新项目的 skill。按技术栈分层，主 skill 为路由索引，
子 skill 仅描述该类型项目的增量内容，通用约定和可复用模板集中管理。

## 目录结构

```
new-project/
├── README.md                      # 本文件 — 开发约定与扩展指南
├── SKILL.md                       # 路由索引（AI 入口）
├── reference/                     # 语言级通用约定
│   ├── python-conventions.md
│   ├── go-conventions.md
│   └── rust-conventions.md
├── templates/                     # 可复用工具配置文件
│   ├── ruff.toml
│   ├── .golangci.yml
│   └── rustfmt.toml
└── subskills/                     # 按技术栈分类的子 skill
    ├── python-cli/SKILL.md
    ├── go-cli/SKILL.md
    ├── fastapi-react/SKILL.md
    └── rust-cli/SKILL.md
```

## 设计原则

### 1. 不硬编码版本号

所有 skill 中**禁止出现依赖的版本号**。始终指示 AI 使用包管理器的安装命令自动获取最新版本：

- Python：`uv add <pkg>` / `uv add --dev <pkg>`
- Rust：`cargo add <pkg>` / `cargo add --dev <pkg>`
- Go：`go get <pkg>@latest`
- Node：`pnpm add <pkg>` / `pnpm add -D <pkg>`

唯一例外：`requires-python`、`go` 指令、`python_version` 等**工具链元数据**，这些是项目级别约束而非依赖版本。

### 2. 子 skill 只写增量

子 skill 不得重复语言级约定。每个子 skill 仅包含：

- 该类型特有的依赖列表（以及安装命令）
- 目录结构差异（相对于语言标准布局）
- 入口代码模板（仅该类型独有的文件）
- 搭建步骤（引用内置脚手架 + 约定文件）

通用内容（工具链说明、Justfile 模板、.gitignore、禁止事项等）全部在 `reference/<lang>-conventions.md` 中定义。

### 3. 优先使用内置脚手架

不手动创建 `pyproject.toml`、`Cargo.toml`、`go.mod`。始终从官方脚手架起步：

| 语言 | 命令 |
|------|------|
| Python | `uv init --lib` |
| Rust | `cargo init --lib` |
| Go | `go mod init <path>` |
| Node | `pnpm create vite@latest` |

然后在此基础上增量修改。

### 4. 约定文件即权威来源

每种语言有且只有一份 `reference/<lang>-conventions.md`。
该文件定义的内容，子 skill 只需引用，不得重述或覆盖。
如果一个约定需要更改，只需修改这一份文件。

### 5. 正文中文，元数据英文

SKILL.md 文件：

- YAML frontmatter（`name`、`description`）保持英文 — 这是 skill 系统的索引字段
- Markdown 正文使用中文 — 面向中文 AI 交互

## 如何扩展

### 添加新语言

1. 创建 `reference/<lang>-conventions.md`
   - 定义工具链、通用依赖、Justfile 模板、.gitignore、禁止事项
2. 如需可复用配置文件，放入 `templates/`
3. 创建 `subskills/<type>/SKILL.md`，引用上述约定

### 添加新项目类型（已有语言）

1. 创建 `subskills/<type>/SKILL.md`
   - 开头引用 `reference/<lang>-conventions.md`
   - 描述该类型特有依赖和安装命令
   - 描述目录结构差异
   - 给出入口代码模板和搭建步骤
2. 在 `SKILL.md` 路由表中添加一行

### 子 skill 文件模板

```markdown
---
name: new-project-<type>
description: >-
  ...
---

# <标题>

> 先读 `reference/<lang>-conventions.md` — 本文仅描述 <类型> 特有的增量。

## 特有依赖

```bash
<包管理器安装命令>
```

## 目录结构

（仅展示与语言标准布局的差异部分）

## <入口代码>

（该类型独有的源文件模板）

## 搭建步骤

（引用 scaffolding 命令 + 约定文件 + 模板复制 + 验证）
```

## 文件约定

- 所有 markdown 正文使用中文
- 目录名使用 kebab-case（`python-cli`、`fastapi-react`）
- 模板文件保留其原始扩展名和格式（`.toml`、`.yml`）
- 不要创建空的 `__init__` 文件或占位目录 — 每个文件必须有内容
