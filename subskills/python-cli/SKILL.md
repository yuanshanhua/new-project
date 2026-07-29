---
name: new-project-python-cli
description: >-
  Scaffold a Python CLI project. Toolchain: uv, ruff, ty, pyright, pytest.
  Layout: src-layout with click. Use for Python command-line tools,
  scripts graduating to packages, or any single-purpose Python CLI.
---

# Python CLI 项目

> 先读 `reference/python-conventions.md` — 本文仅描述 CLI 特有的增量。

## 特有依赖

```bash
uv add click
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
│   ├── cli.py            # click group + 主入口
│   └── __main__.py       # 支持 `python -m <package>`
└── tests/
    └── test_cli.py
```

其余目录及文件（`pyproject.toml`、`Justfile`、`.gitignore` 等）由 `uv init` 生成后，
按 `reference/python-conventions.md` 调整。

## cli.py

```python
"""<project-name> — <一句话描述>。"""

import click


@click.group()
@click.version_option()
def main() -> None:
    """<project-name> — <一句话描述>。"""
    pass


@main.command()
@click.argument("name", default="world")
def hello(name: str) -> None:
    """打招呼。"""
    click.echo(f"Hello, {name}!")


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
from click.testing import CliRunner
from <package>.cli import main


def test_hello():
    runner = CliRunner()
    result = runner.invoke(main, ["hello"])
    assert result.exit_code == 0
    assert "Hello" in result.output
```

## 搭建步骤

1. 询问用户：项目名、包名（默认项目名 snake_case）、CLI 命令名
2. `uv init --lib <项目名>` 生成基础项目
3. 清理 `uv init` 生成的示例代码，写入上述 CLI 特有文件
4. 按 `reference/python-conventions.md` 用 `uv add --dev` 安装 dev 依赖，添加 ruff/ty/pyright 配置
5. 从 `templates/ruff.toml` 复制到项目根目录
6. 写入 `Justfile` 和 `.gitignore`（使用 `reference/python-conventions.md` 中的模板）
7. `uv sync && just test && just fmt && just lint`
