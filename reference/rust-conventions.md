# Rust 项目通用约定

所有 Rust 项目（CLI、库、服务等）必须遵守以下约定。
各子 skill 直接引用本文，不再重复声明。

## 工具链

| 工具 | 用途 | 备注 |
|------|------|------|
| [Cargo](https://doc.rust-lang.org/cargo/) | 构建系统、依赖管理 | Rust 唯一构建系统 |
| [clippy](https://github.com/rust-lang/rust-clippy) | Lint | `-D warnings` 拒绝所有警告 |
| [rustfmt](https://github.com/rust-lang/rustfmt) | 格式化 | — |
| [just](https://github.com/casey/just) | 任务运行器 | 替代 Makefile |

## 项目元数据

- **Edition**：`2024`
- **crate 类型**：二进制项目使用库 crate + 薄二进制入口模式（`main.rs` 仅调用 `lib.rs` 的函数）
- **错误处理**：统一使用 `anyhow`（应用层）或 `thiserror`（库层）

## 依赖管理原则

**禁止手动编写依赖版本号。** 始终使用 Cargo 安装最新版本：

```bash
cargo add anyhow tracing
cargo add tracing-subscriber --features env-filter
cargo add --dev assert_cmd predicates
```

`cargo add` 自动写入 `Cargo.toml` 并锁定版本到 `Cargo.lock`。
不要手动编辑 `[dependencies]` 中的版本约束。

## rustfmt 配置

从 `templates/rustfmt.toml` 复制到项目根目录。

## Justfile 标准目标

```make
default:
    @just --list

build:
    cargo build

release:
    cargo build --release

fmt:
    cargo fmt

lint:
    cargo clippy -- -D warnings

test:
    cargo test

clean:
    cargo clean

install:
    cargo install --path .
```

## 通用 .gitignore

```
/target/
**/*.rs.bk
```

## 入口模块约定

- `src/main.rs`：仅调用库入口，不包含业务逻辑
- `src/lib.rs`：库根，公开模块
- 二进制与库共享同一 crate

## 禁止事项

- 禁止手动编写依赖版本号（用 `cargo add` 代替手写 `Cargo.toml`）
- 禁止在 `main.rs` 中写业务逻辑
- 禁止忽略 clippy 警告（必须 `-D warnings`）
- 禁止使用已废弃的 `structopt`（用 clap derive）
