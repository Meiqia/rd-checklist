# Go 编码规范

## 目录

- [In-Band Errors](#in-band-errors)


## In-Band Errors

Do this:

```go
// Case 1: 不会返回错误
func Get(key string) (value string, ok bool) {
	...
}

func main() {
	v, ok := Get("foo")
	if !ok {
		return // 处理无效的 v
	}

	// 处理有效的 v
}

// Case 2: 可能会返回错误
func Load(key string) (value string, err error) {
	...
}

func main() {
	v, err := Load("foo")
	if err == ErrNotFound {
		return // 处理无效的 v
	}
	if err != nil {
		return // 处理其他错误
	}

	// 处理有效的 v
}
```

Don't do this:

```go
func Get(key string) string {
	...
}

func main() {
	v := Get("foo")
	if v == "" {
		return // 处理无效的 v
	}

	// 处理有效的 v
}

// Case 2: 可能会返回错误
func Load(key string) (value string, err error) {
	...
}

func main() {
	v, err := Load("foo")
	if v == "" {
		return // 处理无效的 v
	}
	if err != nil {
		return // 处理其他错误
	}

	// 处理有效的 v
}
```

golangci-lint linter: [nilnil](https://golangci-lint.run/usage/linters/#nilnil).