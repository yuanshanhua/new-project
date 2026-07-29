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

1. 根据用户描述判断项目类型 — 参考上表关键词
2. 加载对应子 skill
3. 根据 subskill 的要求去阅读指定参考文档或模板, **禁止**直接阅读参考文档和模板

如果用户请求的项目类型在此 skill 中不存在，则可参考相近的子 skill, 并在关键节点显示征求用户意见.

## Skill 目录结构

```text
new-project/
├── SKILL.md                  # 本文件 — 路由索引
├── reference/                # 通用参考
│   ├── python-conventions.md
│   ├── go-conventions.md
│   └── rust-conventions.md
├── templates/                # 可复用模板
│   ├── ruff.toml
│   ├── .golangci.yml
│   └── rustfmt.toml
└── subskills/                # 子 skill
    ├── python-cli/SKILL.md
    ├── go-cli/SKILL.md
    ├── fastapi-react/SKILL.md
    └── rust-cli/SKILL.md
```

## 路由表

### CLI

| 技术栈 | 子 skill | 触发关键词 |
| -------- | ---------- | ----------- |
| Python CLI | `subskills/python-cli/` | "python cli"、"python 命令行"、"click"、"typer" |
| Go CLI | `subskills/go-cli/` | "go cli"、"golang 命令行"、"cobra" |
| Rust CLI | `subskills/rust-cli/` | "rust cli"、"rust 命令行"、"cargo"、"clap" |

### Web 全栈

| 技术栈 | 子 skill | 触发关键词 |
| -------- | ---------- | ----------- |
| FastAPI + React | `subskills/fastapi-react/` | "fastapi"、"前后端"、"python 后端 react 前端" |

### 库 / SDK

| 技术栈 | 子 skill | 触发关键词 |
| -------- | ---------- | ----------- |
| Python 库 | `subskills/python-lib/` | "python 库"、"python sdk"、"pip 包" |
