---
name: new-project-go-cli
description: >-
  Scaffold a Go CLI project. Toolchain: Go modules, cobra, golangci-lint.
  Layout: standard Go project layout with cmd/. Use for Go command-line
  tools, system utilities, or any single-purpose Go binary.
---

# Go CLI 项目

> 先读 `reference/go-conventions.md` — 本文仅描述 CLI 特有的增量。

## 特有依赖

```bash
go get github.com/spf13/cobra@latest
```

## 目录结构

```
<project-name>/
├── cmd/<binary>/
│   └── main.go            # 入口：调用 Execute()
├── internal/cli/
│   ├── root.go            # cobra 根命令（版本、全局标志）
│   └── commands.go        # 子命令
```

其余文件（`go.mod`、`Justfile`、`.gitignore` 等）由 `go mod init` 生成后，
按 `reference/go-conventions.md` 调整。

## internal/cli/root.go

```go
package cli

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
)

// Version 在构建时通过 ldflags 注入。
var Version = "dev"

func NewRoot() *cobra.Command {
	cmd := &cobra.Command{
		Use:   "<binary-name>",
		Short: "<一句话描述>",
		RunE: func(cmd *cobra.Command, args []string) error {
			return cmd.Help()
		},
	}
	cmd.AddCommand(newHello())
	cmd.Version = Version
	return cmd
}

func Execute() {
	if err := NewRoot().Execute(); err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}
}
```

## internal/cli/commands.go

```go
package cli

import (
	"fmt"

	"github.com/spf13/cobra"
)

func newHello() *cobra.Command {
	var name string
	cmd := &cobra.Command{
		Use:   "hello",
		Short: "打招呼",
		RunE: func(cmd *cobra.Command, args []string) error {
			if name == "" {
				name = "world"
			}
			fmt.Printf("Hello, %s!\n", name)
			return nil
		},
	}
	cmd.Flags().StringVarP(&name, "name", "n", "", "要问候的名字")
	return cmd
}
```

## cmd/<binary>/main.go

```go
package main

import "github.com/<owner>/<project>/internal/cli"

func main() {
	cli.Execute()
}
```

## 搭建步骤

1. 询问用户：模块路径、二进制名称
2. `go mod init <模块路径>` 生成 `go.mod`
3. `go get github.com/spf13/cobra@latest && go mod tidy`
4. 创建 `cmd/<binary>/` 和 `internal/cli/` 目录，写入上述 Go 文件
5. 从 `templates/.golangci.yml` 复制到项目根目录
6. 写入 `Justfile` 和 `.gitignore`（使用 `reference/go-conventions.md` 中的模板），替换 `<binary>` 占位符
7. `go build ./cmd/<binary>/ && go vet ./...`
