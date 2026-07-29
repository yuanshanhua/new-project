# Python 项目通用约定

所有 Python 项目（CLI、库、后端服务等）必须遵守以下约定。
各子 skill 直接引用本文，不再重复声明。

## 工具链

| 工具 | 用途 | 备注 |
|------|------|------|
| [uv](https://docs.astral.sh/uv/) | 包管理、虚拟环境、构建 | 替代 pip/poetry/pipenv |
| [ruff](https://docs.astral.sh/ruff/) | Lint + 格式化 | 替代 flake8/isort/black |
| [mypy](https://mypy-lang.org/) | 静态类型检查 | `strict = true` |
| [pyright](https://github.com/microsoft/pyright) | 静态类型检查（第二道） | 与 mypy 互补 |
| [pytest](https://docs.pytest.org/) | 测试 | 替代 unittest |
| [just](https://github.com/casey/just) | 任务运行器 | 替代 Makefile/shell 脚本 |

## 项目元数据

- **Python 版本**：`>=3.10`
- **构建系统**：`hatchling`（通过 `[build-system]` 声明）
- **包布局**：`src/` 布局（`src/<package>/`），禁止平铺
- **配置位置**：全部收敛到 `pyproject.toml`，仅 mypy 可用 `mypy.ini`

## 依赖管理原则

**禁止手动编写依赖版本号。** 始终使用包管理器安装最新版本：

```bash
uv add --dev pytest ruff mypy pyright
```

`uv add` 自动写入 `pyproject.toml` 并锁定版本到 `uv.lock`。
不要手动编辑 `dependencies` 或 `dependency-groups.dev` 中的版本约束。

## ruff 配置

从 `templates/ruff.toml` 复制到项目根目录，在 `pyproject.toml` 中引用：

```toml
[tool.ruff]
extend = "ruff.toml"
```

## mypy 配置

```toml
[tool.mypy]
strict = true
python_version = "3.10"
```

## pyright 配置

```toml
[tool.pyright]
typeCheckingMode = "strict"
pythonVersion = "3.10"
```

## Justfile 标准目标

```make
default:
    @just --list

build:
    uv build

fmt:
    uv run ruff format src/ tests/
    uv run ruff check --fix src/ tests/

lint:
    uv run ruff check src/ tests/
    uv run mypy src/
    uv run pyright src/

test:
    uv run pytest {{args}}

clean:
    rm -rf dist/ build/ *.egg-info .mypy_cache .pytest_cache .ruff_cache
    find . -type d -name __pycache__ -exec rm -rf {} +

install:
    uv sync
```

## 通用 .gitignore

```
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.mypy_cache/
.pytest_cache/
.ruff_cache/
.venv/
```

## 入口模块约定

- `src/<package>/__init__.py`：公开 API 的 re-export
- `src/<package>/__main__.py`：支持 `python -m <package>`
- CLI 入口函数统一命名为 `main()`，由 `[project.scripts]` 指向

## 禁止事项

- 禁止手动编写依赖版本号（用 `uv add` 代替手写 `pyproject.toml`）
- 禁止使用 `requirements.txt`、`setup.py`、`setup.cfg`、`Pipfile`
- 禁止使用 `flake8`、`isort`、`black`、`autopep8`
- 禁止使用 `poetry`、`pipenv`
- 禁止使用 `unittest`（用 pytest）
- 禁止平铺包（必须 `src/` 布局）
