---
name: new-project-rust-cli
description: >-
  Scaffold a Rust CLI project. Toolchain: Cargo, clap, rustfmt, clippy.
  Layout: standard Cargo project with a library crate + thin binary.
  Use for Rust command-line tools, system utilities, or performance-critical
  CLI applications.
---

# Rust CLI 项目

> 先读 `reference/rust-conventions.md` — 本文仅描述 CLI 特有的增量。

## 特有依赖

```bash
cargo add clap --features derive
```

通用依赖（`anyhow`、`tracing` 等）由 `reference/rust-conventions.md` 定义，一并 `cargo add`。

## 目录结构

```
<project-name>/
├── src/
│   ├── main.rs             # 薄入口：调用 Cli::run()
│   ├── lib.rs              # 库根
│   └── cli.rs              # clap derive 结构体
└── tests/
    └── cli_tests.rs
```

其余文件（`Cargo.toml`、`Justfile`、`.gitignore` 等）由 `cargo init` 生成后，
按 `reference/rust-conventions.md` 调整。

## src/lib.rs

```rust
pub mod cli;
```

## src/cli.rs

```rust
use clap::{Parser, Subcommand};

/// <一句话描述>。
#[derive(Parser)]
#[command(version, about)]
pub struct Cli {
    #[command(subcommand)]
    pub command: Commands,
}

#[derive(Subcommand)]
pub enum Commands {
    /// 打招呼。
    Hello {
        /// 要问候的名字。
        #[arg(default_value = "world")]
        name: String,
    },
}

impl Cli {
    pub fn run() -> anyhow::Result<()> {
        let cli = Self::parse();
        match cli.command {
            Commands::Hello { name } => println!("Hello, {name}!"),
        }
        Ok(())
    }
}
```

## src/main.rs

```rust
use <project_name>::cli::Cli;

fn main() -> anyhow::Result<()> {
    tracing_subscriber::fmt()
        .with_env_filter(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "info".into()),
        )
        .init();

    Cli::run()
}
```

## 搭建步骤

1. 询问用户：项目名（snake_case，即 crate 名）、简要描述
2. `cargo init --lib <项目名>` 生成库 crate
3. 按 `reference/rust-conventions.md` 用 `cargo add` 安装通用依赖，再 `cargo add clap --features derive`
4. 手动添加 `src/main.rs`（`cargo init --lib` 不生成此文件，Cargo 自动检测）
5. 写入 `src/cli.rs`、`src/lib.rs`
6. 从 `templates/rustfmt.toml` 复制到项目根目录
7. 写入 `Justfile` 和 `.gitignore`（使用 `reference/rust-conventions.md` 中的模板）
8. `cargo build && cargo test && cargo fmt && cargo clippy -- -D warnings`
