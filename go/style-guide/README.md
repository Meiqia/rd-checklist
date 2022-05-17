# Go 编码规范

## 目录

- [Empty string check](#empty-string-check)
- [Interfaces](#interfaces)
- [Error strings](#error-strings)
- [Handle errors](#handle-errors)
- [Imports](#imports)
- [Goroutine lifetimes](#goroutine-lifetimes)
- [In-band errors](#in-band-errors)
- [Use consistent spelling of certain words](#use-consistent-spelling-of-certain-words)


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


## scale function


每行代码不超过 100 个字符。
函数不超过 100 行。

说明：函数/文件太长一般说明函数定义不明确/程序结构划分不合理，不利于维护


## clear field meaning

入参和返回值的字段有明确含义,例如：为空则代表值为空，不为空就是有明确意义。

