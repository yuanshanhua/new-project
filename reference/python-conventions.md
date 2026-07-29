# Python 项目通用约定

所有 Python 项目（CLI、库、后端服务等）必须遵守以下约定。
各子 skill 直接引用本文，不再重复声明。

## 工具链

| 工具 | 用途 | 备注 |
|------|------|------|
| [uv](https://docs.astral.sh/uv/) | 包管理、虚拟环境、构建 | 替代 pip/poetry/pipenv |
| [ruff](https://docs.astral.sh/ruff/) | Lint + 格式化 | 替代 flake8/isort/black |
| [ty](https://docs.astral.sh/ty/) | 静态类型检查（Rust 实现，极快） | Astral 出品，与 ruff/uv 同源 |
| [pyright](https://github.com/microsoft/pyright) | 静态类型检查（第二道） | 与 ty 互补，双重检查 |
| [pytest](https://docs.pytest.org/) | 测试 | 替代 unittest |
| [just](https://github.com/casey/just) | 任务运行器 | 替代 Makefile/shell 脚本 |

## 项目元数据

- **Python 版本**：`>=3.10`
- **构建系统**：`hatchling`（通过 `[build-system]` 声明）
- **包布局**：`src/` 布局（`src/<package>/`），禁止平铺
- **配置位置**：全部收敛到 `pyproject.toml`

## 依赖管理原则

**禁止手动编写依赖版本号。** 始终使用包管理器安装最新版本：

```bash
uv add --dev pytest ruff ty pyright
```

`uv add` 自动写入 `pyproject.toml` 并锁定版本到 `uv.lock`。
不要手动编辑 `dependencies` 或 `dependency-groups.dev` 中的版本约束。

## ruff 配置

从 `templates/ruff.toml` 复制到项目根目录，在 `pyproject.toml` 中引用：

```toml
[tool.ruff]
extend = "ruff.toml"
```

## ty 配置

ty 自动从 `pyproject.toml` 的 `requires-python` 读取目标 Python 版本，无需额外配置。
默认所有规则为 `error` 级别，以下为推荐配置：

```toml
[tool.ty]
# Python 版本自动从 requires-python 读取
```

## pyright 配置

pyright 同样从 `pyproject.toml` 读取 Python 版本：

```toml
[tool.pyright]
typeCheckingMode = "strict"
```

## Justfile 参考模板

以下为通用模板，可根据项目实际情况调整

```make
default:
    @just --list

# 检查的目标目录
CHECK_DIRS := "src tests"

# Ruff 代码检查
lint:
    uv run ruff check {{CHECK_DIRS}}

# Ruff 格式化检查（只检查不修改）
format:
    uv run ruff format --check {{CHECK_DIRS}}

# 自动修复 lint/format 问题
fix:
    uv run ruff check --fix {{CHECK_DIRS}}
    uv run ruff format {{CHECK_DIRS}}

# 类型检查（Pyright + ty，两个检查器必须同时通过）
typecheck:
    uv run pyright {{CHECK_DIRS}}
    uv run ty check {{CHECK_DIRS}}

# 运行测试（允许无测试文件的情况）
test:
    -uv run pytest tests/ -v

# 全部检查
check: lint format typecheck test

# 检查依赖更新
deps:
    uv tree -d 1 --outdated
```

## 通用 .gitignore

```
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
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
