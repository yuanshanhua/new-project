# Go 项目通用约定

所有 Go 项目（CLI、服务等）必须遵守以下约定。
各子 skill 直接引用本文，不再重复声明。

## 工具链

| 工具 | 用途 | 备注 |
|------|------|------|
| [Go modules](https://go.dev/doc/modules/managing-dependencies) | 依赖管理 | 现代 Go 唯一选项 |
| [golangci-lint](https://golangci-lint.run/) | Lint 聚合 | 替代单用 govet/staticcheck |
| [just](https://github.com/casey/just) | 任务运行器 | 替代 Makefile |

## 项目元数据

- **Go 版本**：`go.mod` 中声明 `go 1.22` 或更新
- **模块路径**：`github.com/<owner>/<repo>`
- **包可见性**：非公开包放 `internal/`，禁止使用 `/pkg/`
- **依赖安装**：始终用 `go get <pkg>@latest` 安装最新版本，禁止手写版本号

## golangci-lint 配置

从 `templates/.golangci.yml` 复制到项目根目录。

## Justfile 标准目标

```make
default:
    @just --list

build:
    go build ./cmd/<binary>/

fmt:
    gofmt -w .
    goimports -w .

lint:
    golangci-lint run ./...

test:
    go test -v -race ./...

clean:
    go clean
    rm -f <binary-name>

install:
    go install ./cmd/<binary>/
```

## 通用 .gitignore

```
# 二进制文件
*.exe

# Go
*.test
*.out
vendor/

# IDE
.idea/
.vscode/
```

## 入口模块约定

- `cmd/<binary>/main.go`：唯一入口，调用 `internal/` 中的逻辑
- `internal/` 下按功能分包：`internal/cli/`、`internal/config/`、`internal/domain/`
- 版本号通过 `ldflags` 注入，不由源码硬编码

## 禁止事项

- 禁止使用 `/pkg/` 目录（Go 生态已不推荐）
- 禁止使用 `dep`、`glide`、`govendor` 等旧包管理工具
- 禁止手写 `flag` 解析（CLI 项目统一用 cobra）
- 禁止在 `main.go` 中写业务逻辑
