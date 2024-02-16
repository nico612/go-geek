

`go-nanoid` 是一个 Go 语言的库，用于生成唯一的、随机的 ID。它是基于 Twitter 的 `flake` 算法实现的，用于生成短小且高度唯一的 ID。

```shell
go get github.com/jaevor/go-nanoid
```

示例：

```go
package main

import (
	"fmt"
	"github.com/jaevor/go-nanoid"
)

func main() {
	// 使用默认的配置生成一个长度为 12 的 ID
	id, err := gonanoid.Generate()
	if err != nil {
		fmt.Println("Error generating ID:", err)
		return
	}
	fmt.Println("Generated ID:", id)
}

```

