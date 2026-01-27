# Go Documentation Tools

Tools and scripts for Go project documentation.

## Project Structure Detection

```bash
# Check if Go project
test -f go.mod && echo "Go project detected"

# Get module name
go list -m

# List all packages
go list ./...
```

## Standard Go Project Layout

```
project/
├── cmd/                  # Entry points (main packages)
│   ├── api/
│   │   └── main.go
│   └── worker/
│       └── main.go
├── internal/             # Private packages
│   ├── handlers/
│   ├── services/
│   └── repository/
├── pkg/                  # Public packages
│   └── client/
├── go.mod
├── go.sum
└── README.md
```

## Dependency Analysis

```bash
# List all dependencies
go list -m all

# Show dependency graph (text)
go mod graph

# Visual dependency graph (requires graphviz)
go mod graph | sed -Ee 's/@[^[:space:]]+//g' | sort -u | \
  awk '{print "\"" $1 "\" -> \"" $2 "\""}' | \
  (echo "digraph {"; cat; echo "}") | dot -Tsvg -o deps.svg

# Using godepgraph (install: go install github.com/kisielk/godepgraph@latest)
godepgraph -s ./... | dot -Tsvg -o deps.svg

# Using go-callvis for call graph (install: go install github.com/ofabry/go-callvis@latest)
go-callvis -focus main ./cmd/api
```

## Documentation Extraction

```bash
# Package documentation
go doc ./internal/handlers

# Specific function/type
go doc ./internal/handlers.UserHandler

# Full package with all symbols
go doc -all ./pkg/client

# Generate HTML docs (start godoc server)
godoc -http=:6060
# Then visit http://localhost:6060/pkg/your-module/

# Export docs to markdown (using gomarkdoc)
# Install: go install github.com/princjef/gomarkdoc/cmd/gomarkdoc@latest
gomarkdoc ./... > docs/api.md
```

## AST Analysis Script

```go
// scripts/analyze/main.go
package main

import (
	"fmt"
	"go/ast"
	"go/parser"
	"go/token"
	"os"
	"path/filepath"
)

func main() {
	root := "."
	if len(os.Args) > 1 {
		root = os.Args[1]
	}

	fset := token.NewFileSet()

	filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
		if err != nil || info.IsDir() || filepath.Ext(path) != ".go" {
			return nil
		}

		file, err := parser.ParseFile(fset, path, nil, parser.ParseComments)
		if err != nil {
			return nil
		}

		fmt.Printf("\n=== %s ===\n", path)
		fmt.Printf("Package: %s\n", file.Name.Name)

		// List exports
		for _, decl := range file.Decls {
			switch d := decl.(type) {
			case *ast.FuncDecl:
				if d.Name.IsExported() {
					fmt.Printf("  func %s\n", d.Name.Name)
				}
			case *ast.GenDecl:
				for _, spec := range d.Specs {
					switch s := spec.(type) {
					case *ast.TypeSpec:
						if s.Name.IsExported() {
							fmt.Printf("  type %s\n", s.Name.Name)
						}
					}
				}
			}
		}
		return nil
	})
}
```

Run with: `go run scripts/analyze/main.go ./internal`

## Codemap for Go Projects

```markdown
# Backend Codemap (Go)

**Last Updated:** YYYY-MM-DD
**Module:** github.com/org/project
**Entry Points:** cmd/api/main.go, cmd/worker/main.go

## Structure
```

cmd/
├── api/ # HTTP API server
└── worker/ # Background worker

internal/
├── handlers/ # HTTP handlers
├── services/ # Business logic
├── repository/ # Data access
└── models/ # Domain models

pkg/
└── client/ # Public SDK

```

## Key Packages

| Package | Purpose | Exports |
|---------|---------|---------|
| internal/handlers | HTTP handlers | UserHandler, AuthHandler |
| internal/services | Business logic | UserService, AuthService |
| internal/repository | Database | UserRepo, PostgresDB |
| pkg/client | Public SDK | Client, Options |

## Data Flow

Request → Handler → Service → Repository → Database

## Dependencies

From `go.mod`:
- github.com/gin-gonic/gin - HTTP framework
- github.com/jackc/pgx/v5 - PostgreSQL driver
- go.uber.org/zap - Logging
```

## Makefile Targets

```makefile
.PHONY: docs deps-graph

# Generate documentation
docs:
	gomarkdoc ./... > docs/api.md
	go mod graph | dot -Tsvg -o docs/deps.svg

# Analyze dependencies
deps-graph:
	@echo "=== Direct dependencies ==="
	go list -m -f '{{if not .Indirect}}{{.Path}}{{end}}' all
	@echo "\n=== Generating graph ==="
	godepgraph -s ./... | dot -Tsvg -o deps.svg

# List all exported symbols
exports:
	go run scripts/analyze/main.go ./...
```

## Go-Specific Considerations

1. **Internal packages** - Cannot be imported outside module
2. **Exported vs unexported** - Capital letter = public API
3. **go.mod** - Single source of truth for dependencies
4. **No circular imports** - Go enforces acyclic dependencies
5. **Package-level doc** - Use `doc.go` for package documentation
