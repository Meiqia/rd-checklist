# Go 编码规范

## 目录

- [Go 编码规范](#go-编码规范)
	- [目录](#目录)
	- [Prefer to struct](#prefer-to-struct)
	- [Package naming](#package-naming)
	- [Empty string check](#empty-string-check)
	- [Interfaces](#interfaces)
	- [Error strings](#error-strings)
- [Handle errors](#handle-errors)
	- [Imports](#imports)
	- [Goroutine lifetimes](#goroutine-lifetimes)
	- [In-band errors](#in-band-errors)
	- [Use consistent spelling of certain words](#use-consistent-spelling-of-certain-words)
	- [Group similar declarations](#group-similar-declarations)
	- [Reduce scope of variables](#reduce-scope-of-variables)
	- [Unnecessary else](#unnecessary-else)
	- [Variable Names](#variable-names)
		- [example](#example)
			- [ID](#id)


## Prefer to struct

golang是一门静态语言，避免使用通用型`map[string]interface{}`，能用结构体的地方都用结构体

## Package naming

Do this: `strconv`
Don't do this: `str_conv` `strConv`

- 全部小写，没有大写或下划线
- 大多数使用命名导入的情况下，不需要重命名
- 简短而简洁
- 不用复数

参考:
- https://blog.golang.org/package-names
- https://rakyll.org/style-packages/


## Empty string check

Do this:

```go
if s == "" {
	...
}
```

Don't do this:

```go
if len(s) == 0 {
	...
}
```

参考: https://dmitri.shuralyov.com/idiomatic-go#empty-string-check.


## Interfaces

Do this:

```go
package consumer

type Thinger interface { Thing() bool }

func Foo(t Thinger) string { … }
```

```go
package producer

type DefaultThinger struct{ … }
func (t *DefaultThinger) Thing() bool { … }

func NewDefaultThinger() *DefaultThinger { return &DefaultThinger{ … } }

// Interface guards
var _ consumer.Thinger = (*DefaultThinger)(nil)
```

Don't do this:

```go
package producer

type Thinger interface { Thing() bool }

type defaultThinger struct{ … }
func (t *defaultThinger) Thing() bool { … }

func NewThinger() Thinger { return &defaultThinger{ … } }
```

说明:

- Interface 定义应该出现在使用的地方, 而不是实现的地方
- Interface 实现应该返回具体类型（通常是指针或结构体）
- 使用 Interface guards 可以在编译期间保证 Interface 接口实现的完整性

参考:

- https://github.com/golang/go/wiki/CodeReviewComments#interfaces
- https://github.com/uber-go/guide/blob/master/style.md#verify-interface-compliance


## Error strings

Do this:

```go
fmt.Errorf("something bad")
```

Don't do this:

```go
fmt.Errorf("Something bad.")
```

说明: 错误字符串不要以大写字母开头,也不要包括标点符号。

参考: https://github.com/golang/go/wiki/CodeReviewComments#error-strings.


# Handle errors

Do this:

```go
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}

// ...
```

Don't do this:

```go
resp, _ := http.Get("http://example.com/")

// ...
```

说明:

- 不要用 `_` 来忽略错误
- 总是处理错误: return, 打日志, 或者极端情况下直接 panic

参考: https://github.com/golang/go/wiki/CodeReviewComments#handle-errors.


## Imports

Do this:

```go
import (
	"fmt"
	"os"

	"github/Meiqia/pkg"
)
```

Don't do this:

```go
import (
	"fmt"
	"os"
	"github/Meiqia/pkg"
)
```

说明:

- import 语句至少分为两组: 标准库, 其他库
- 也可以分为三组: 标准库, 三方库, 本地库

参考:

- https://github.com/golang/go/wiki/CodeReviewComments#imports
- https://github.com/uber-go/guide/blob/master/style.md#import-group-ordering


## Goroutine lifetimes

启动一个 goroutine 之前, 必须要知道它应该如何终止。

等待 goroutine 终止的常用方式:

- 自定义 channel
- [sync.WaitGroup](https://pkg.go.dev/sync#WaitGroup)

参考:

- https://github.com/golang/go/wiki/CodeReviewComments#goroutine-lifetimes
- [Never start a goroutine without knowning when it will stop](https://dave.cheney.net/practical-go/presentations/qcon-china.html#_never_start_a_goroutine_without_knowning_when_it_will_stop)


## In-band errors

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
// Case 1: 不会返回错误
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

参考: https://github.com/golang/go/wiki/CodeReviewComments#in-band-errors.

golangci-lint linter: [nilnil](https://golangci-lint.run/usage/linters/#nilnil).


## Use consistent spelling of certain words

Do this:

```
// marshaling
// unmarshaling
// canceling
// canceled
// cancellation
```

Don't do this:

```
// marshalling
// unmarshalling
// cancelling
// cancelled
// cancelation
```

参考: https://dmitri.shuralyov.com/idiomatic-go#use-consistent-spelling-of-certain-words.


## Group similar declarations

Do this:
```go
const (
	a = 1
	b = 2
)

var (
	a = 1
	b = 2
)
```

Don't do this:
```go
const a = 1
const b = 2

var a = 1
var b = 2
```

参考: https://github.com/uber-go/guide/blob/master/style.md#group-similar-declarations


## Reduce scope of variables

Do this:
```go
data, err := ioutil.ReadFile(name)
if err != nil {
	return err
}

if err := cfg.Decode(data); err != nil {
	return err
}

fmt.Println(cfg)
return nil
```

Don't do this:
```go
if data, err := ioutil.ReadFile(name); err == nil {
	err = cfg.Decode(data)
  	if err != nil {
		return err
  	}

	fmt.Println(cfg)
	return nil
} else {
	return err
}
```

如果返回只有一个err，用一行；如果有多个，换行处理err

参考: https://github.com/uber-go/guide/blob/master/style.md#reduce-scope-of-variables


## Unnecessary else

Do this:
```go
a := 10
if b {
	a = 100
}
```

Don't do this:
```go
var a int
if b {
	a = 100
} else {
	a = 10
}
```

参考：https://github.com/uber-go/guide/blob/master/style.md#unnecessary-else


## Variable Names
Variable names in Go should be short rather than long. This is especially true for local variables with limited scope. Prefer c to lineCount. Prefer i to sliceIndex.

The basic rule: the further from its declaration that a name is used, the more descriptive the name must be. For a method receiver, one or two letters is sufficient. Common variables such as loop indices and readers can be a single letter (i, r). More unusual things and global variables need more descriptive names.

### example

#### ID
Do this:
```go
var UUID string
var ID string
```
Don't do this:
```go
var uuid string
var Uuid string
var Id string
```

参考：https://github.com/golang/go/wiki/CodeReviewComments#variable-names