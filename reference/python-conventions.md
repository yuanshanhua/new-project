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
| [just](https://github.com/casey/just) | 任务运行器 | **唯一对外任务入口** |
| [prek](https://github.com/j178/prek) | Git hooks（兼容 pre-commit 配置） | 提交前跑 `just check` |

## 项目元数据

- **Python 版本**：`>=3.12`
- **构建系统**：`hatchling`（通过 `[build-system]` 声明）
- **包布局**：`src/` 布局（`src/<package>/`），禁止平铺
- **配置位置**：全部收敛到 `pyproject.toml`
- **CLI 框架**：Typer（禁止 Click，除非维护既有 Click 代码）

## 依赖管理原则

**禁止手动编写依赖版本号。** 始终使用包管理器安装最新版本：

```bash
uv add --dev pytest ruff ty pyright prek
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

## pyright 配置

```toml
[tool.pyright]
typeCheckingMode = "strict"
```

## Justfile 参考模板

根目录 Justfile 是**唯一对外任务入口**（人 / Agent / CI / git hooks 都调它）。
从 `templates/Justfile.python` 复制后按项目调整；标准目标如下：

```make
default:
    @just --list

CHECK_DIRS := "src tests"

lint:
    uv run ruff check {{CHECK_DIRS}}

format:
    uv run ruff format --check {{CHECK_DIRS}}

fix:
    uv run ruff check --fix {{CHECK_DIRS}}
    uv run ruff format {{CHECK_DIRS}}

typecheck:
    uv run pyright {{CHECK_DIRS}}
    uv run ty check {{CHECK_DIRS}}

test:
    -uv run pytest tests/ -v

check: lint format typecheck test

deps:
    uv tree -d 1 --outdated
```

全栈项目在此基础上追加 `generate` / `dev` / `build` 等，见 `subskills/fastapi-react/`。

## 质量门禁

### 本地

- **提交前必须 `just check` 通过**（写入 `AGENTS.md`）。
- 用 `prek` 安装 hooks：配置见 `templates/.pre-commit-config.yaml`，hook 调用 `just check`（或先 `just fix` 再 check，按项目偏好）。
- 安装：`uv run prek install`（可放在 `just sync` / setup 目标里）。

### CI

- 提供 GitHub Actions 工作流：push / PR 上执行与本地相同的 `just check`。
- 模板见 `templates/ci.python.yml`；全栈见 `templates/ci.fullstack.yml`。
- CI 不得发明第二套命令名；只调用 Justfile 已有目标。

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
.env
data/
```

## 入口模块约定

- `src/<package>/__init__.py`：公开 API 的 re-export
- `src/<package>/__main__.py`：支持 `python -m <package>`
- CLI 入口函数统一命名为 `main()`，由 `[project.scripts]` 指向

## Agent 文档

脚手架必须生成根目录 `AGENTS.md`（可从 `templates/AGENTS.md` 复制并替换占位符）：

- 项目定位 + 技术栈一句话
- 任务入口：`just --list`
- Always / Never（含提交前 `just check`）
- 目录地图（短）
- 文档约定（若有 `docs/`）：只写边界/契约/取舍，不抄源码、不写演进史；细节放 `docs/`，`AGENTS.md` 保持短

## 禁止事项

- 禁止手动编写依赖版本号（用 `uv add` 代替手写 `pyproject.toml`）
- 禁止使用 `requirements.txt`、`setup.py`、`setup.cfg`、`Pipfile`
- 禁止使用 `flake8`、`isort`、`black`、`autopep8`
- 禁止使用 `poetry`、`pipenv`
- 禁止使用 `unittest`（用 pytest）
- 禁止使用 Click 作为新项目 CLI 框架（用 Typer）
- 禁止平铺包（必须 `src/` 布局）
- 禁止绕过 Justfile 自造「正式」检查/启动入口
