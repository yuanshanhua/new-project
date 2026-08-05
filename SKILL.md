---
name: new-project
description: >-
  Guide for creating new projects from scratch. Routes to tech-stack-specific
  sub-skills that provide scaffolding templates, toolchain setup, directory
  layout, and conventions. Use when the user asks to create, scaffold, start,
  or initialize a new project, or mentions starting a codebase.
---

# 新建项目

本 skill 根据技术栈将项目创建请求路由到对应的子 skill。
**不要凭记忆搭建** — 加载匹配的子 skill 并按步骤执行。

## 使用方式

1. 根据用户描述判断项目类型 — 参考下方路由表
2. 加载对应子 skill
3. 根据 subskill 的要求去阅读指定参考文档或模板，**禁止**直接阅读全部参考文档和模板（只读被点名的）

如果用户请求的项目类型在此 skill 中不存在，则可参考相近的子 skill，并在关键节点显示征求用户意见。

## Skill 目录结构

```text
new-project/
├── SKILL.md                  # 本文件 — 路由索引
├── reference/                # 通用参考
│   ├── python-conventions.md
│   ├── frontend-conventions.md
│   ├── go-conventions.md
│   └── rust-conventions.md
├── templates/                # 可复用模板
│   ├── ruff.toml
│   ├── Justfile.python
│   ├── Justfile.fullstack
│   ├── AGENTS.md
│   ├── env.example
│   ├── openapi-ts.config.ts
│   ├── .pre-commit-config.yaml
│   ├── ci.python.yml
│   ├── ci.fullstack.yml
│   ├── .golangci.yml
│   └── rustfmt.toml
└── subskills/
    ├── python-cli/SKILL.md
    ├── go-cli/SKILL.md
    ├── fastapi-react/SKILL.md
    └── rust-cli/SKILL.md
```

## 路由表

### CLI

| 技术栈 | 子 skill | 触发关键词 |
| -------- | ---------- | ----------- |
| Python CLI | `subskills/python-cli/` | "python cli"、"python 命令行"、"typer" |
| Go CLI | `subskills/go-cli/` | "go cli"、"golang 命令行"、"cobra" |
| Rust CLI | `subskills/rust-cli/` | "rust cli"、"rust 命令行"、"cargo"、"clap" |

### Web 全栈

| 技术栈 | 子 skill | 触发关键词 |
| -------- | ---------- | ----------- |
| FastAPI + React | `subskills/fastapi-react/` | "fastapi"、"前后端"、"python 后端 react 前端"、"全栈" |

纯前端 SPA、Python 库等暂无独立子 skill：分别近似按 `fastapi-react` 的 `web/` 部分，或 `python-cli`（去掉 Typer 入口、保留 src 布局与门禁）搭建，并征求用户确认。

## 跨类型硬约定（所有子 skill）

- **任务入口**：根目录 `just`（`just --list`）
- **提交前**：`just check`；prek hooks + GitHub Actions 调用同一套目标
- **Agent 文档**：生成 `AGENTS.md`（从 `templates/AGENTS.md` 裁剪）
- **禁止手写依赖版本号**；用各语言包管理器安装最新版
