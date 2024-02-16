

### fx.Provider

`Provide` 函数用于注册任意数量的构造函数，告诉应用程序如何实例化各种类型。提供的构造函数可以依赖于应用程序中可用的其他类型，必须返回一个或多个对象，并且可能返回一个错误。例如：

```go
// 构造类型 *C，依赖于 *A 和 *B。
func(*A, *B) *C

// 构造类型 *C，依赖于 *A 和 *B，并通过返回错误来表示失败。
func(*A, *B) (*C, error)

// 构造类型 *B 和 *C，依赖于 *A，并且可能失败。
func(*A) (*B, *C, error)

```

提供构造函数的顺序并不重要，多次传递 `Provide` 选项将添加到应用程序的构造函数集合中。**只有在需要它们返回的类型之一时，才会调用构造函数，并且它们的结果会被缓存以供重复使用**（因此，在应用程序中，类型的实例有效上是单例）。因此，即使只使用其中一小部分，也完全可以提供大量的构造函数。

对于高级功能，包括可选参数和命名实例，请参阅 `In` 和 `Out` 类型的文档。

有关限制对构造函数的访问，请参阅 `[Private]` 的文档。

构造函数应尽可能少地进行外部交互，并且应避免启动 goroutine。诸如服务器监听循环、后台计时器循环和后台处理 goroutine 等操作应改用生命周期回调进行管理。

### fx.Supply()

`Supply` 在依赖注入中提供已实例化的值，就好像它们是使用简单返回的构造函数提供的一样。对于每个值，会使用最具体的类型（通过反射确定）。

这个功能类似于 `fx.Replace` 对 `fx.Decorate` 所做的事情。

举个例子：

```go
type (
    TypeA struct{}
    TypeB struct{}
    TypeC struct{}
)

var a, b, c = &TypeA{}, TypeB{}, &TypeC{}

// 下面的两种形式是等价的：
fx.Supply(a, b, fx.Annotated{Target: c})

fx.Provide(
    func() *TypeA { return a },
    func() TypeB { return b },
    fx.Annotated{Target: func() *TypeC { return c }},
)

```

`Supply` 函数的作用类似于提供 `fx.Provide` 函数所需的构造函数。如果值（或注释目标）是无类型的 `nil` 或错误，则 `Supply` 会引发`panic`。

#### Supply 注意事项

如上所述，`Supply` 使用所提供值的最具体类型。对于接口值，这是**指实现的类型**，而不是接口本身的类型。因此，如果你提供了一个 `http.Handler`，`fx.Supply` 将使用实现的类型。

```go
var handler http.Handler = http.HandlerFunc(f)
fx.Supply(handler)

```

上述代码等同于：

```go
fx.Provide(func() http.HandlerFunc { return f })
```

这通常不是你的本意。要将上述处理程序（handler）作为 `http.Handler` 提供，我们需要使用 `fx.Annotate` 函数和 `fx.As` 注释。

```go
fx.Supply(
    fx.Annotate(handler, fx.As(new(http.Handler))),
)
```

这样做可以确保提供的值以正确的类型被注入，从而避免了在提供值时可能出现的类型混淆问题。