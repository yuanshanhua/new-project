---
name: new-project-python-cli
description: >-
  Scaffold a Python CLI project. Toolchain: uv, ruff, ty, pyright, pytest, just, prek.
  Layout: src-layout with Typer. Use for Python command-line tools,
  scripts graduating to packages, or any single-purpose Python CLI.
---

# Python CLI 项目

> 先读 `reference/python-conventions.md` — 本文仅描述 CLI 特有的增量。

## 特有依赖

```bash
uv add typer
```

在 `pyproject.toml` 中手动添加入口脚本：

```toml
[project.scripts]
<cli-name> = "<package>.cli:main"
```

## 目录结构

```
<project-name>/
├── src/<package>/
│   ├── __init__.py
│   ├── cli.py            # Typer app + 主入口
│   └── __main__.py       # 支持 `python -m <package>`
├── tests/
│   └── test_cli.py
├── Justfile
├── AGENTS.md
├── .pre-commit-config.yaml
└── .github/workflows/ci.yml
```

其余文件（`pyproject.toml`、`.gitignore` 等）由 `uv init` 生成后，
按 `reference/python-conventions.md` 调整。

## cli.py

```python
"""<project-name> — <一句话描述>。"""

import typer

app = typer.Typer(no_args_is_help=True, add_completion=False)


@app.command()
def hello(name: str = "world") -> None:
    """打招呼。"""
    typer.echo(f"Hello, {name}!")


def main() -> None:
    app()


if __name__ == "__main__":
    main()
```

## __main__.py

```python
from <package>.cli import main

main()
```

## tests/test_cli.py

```python
from typer.testing import CliRunner

from <package>.cli import app

runner = CliRunner()


def test_hello() -> None:
    result = runner.invoke(app, ["hello"])
    assert result.exit_code == 0
    assert "Hello" in result.output
```

## 搭建步骤

1. 询问用户：项目名、包名（默认项目名 snake_case）、CLI 命令名
2. `uv init --lib <项目名>` 生成基础项目；将 `requires-python` 设为 `>=3.12`
3. 清理 `uv init` 生成的示例代码，写入上述 CLI 特有文件
4. 按 `reference/python-conventions.md` 用 `uv add --dev` 安装 dev 依赖（含 `prek`），添加 ruff/ty/pyright 配置
5. `uv add typer`；写入 `[project.scripts]`
6. 从 `templates/ruff.toml` 复制到项目根目录
7. 从 `templates/Justfile.python` 复制为 `Justfile`
8. 从 `templates/AGENTS.md` 复制并替换占位符（去掉全栈注释块）
9. 从 `templates/.pre-commit-config.yaml` 复制；从 `templates/ci.python.yml` 复制为 `.github/workflows/ci.yml`
10. 写入 `.gitignore`（见 python-conventions）
11. `uv sync && uv run prek install && just check`
