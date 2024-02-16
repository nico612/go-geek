

## zap的特性

- 高性能：zap 对日志输出进行了多项优化以提高它的性能
- 日志分级：有 Debug，Info，Warn，Error，DPanic，Panic，Fatal 等
- 日志记录结构化：日志内容记录是结构化的，比如 json 格式输出
- 自定义格式：用户可以自定义输出的日志格式
- 自定义公共字段：用户可以自定义公共字段，大家输出的日志内容就共同拥有了这些字段
- 调试：可以打印文件名、函数名、行号、日志时间等，便于调试程序
- 自定义调用栈级别：可以根据日志级别输出它的调用栈信息
- Namespace：日志命名空间。定义命名空间后，所有日志内容就在这个命名空间下。命名空间相当于一个文件夹
- 支持 hook 操作

## 快速开始

安装：

```go
go get -u go.uber.org/zap	
```

zap 提供了 2 种日志记录器：`SugaredLogger` 和 `Logger`。

在需要性能但不是很重要的情况下，使用 SugaredLogger 较合适。它比其它结构化日志包快 4-10 倍，包括 结构化日志和 printf 风格的 API。看下面使用 SugaredLogger 例子：

```go
logger, _ := zap.NewProduction()
defer logger.Sync() // zap底层有缓冲。在任何情况下执行 defer logger.Sync() 是一个很好的习惯
sugar := logger.Sugar()
sugar.Infow("failed to fetch URL",
 // 字段是松散类型，不是强类型
  "url", url,
  "attempt", 3,
  "backoff", time.Second,
)
sugar.Infof("Failed to fetch URL: %s", url)
```

当性能和类型安全很重要时，请使用 Logger。它比 SugaredLogger 更快，分配的资源更少，但它只支持结构化日志和强类型字段。

```go
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("failed to fetch URL",
  // 字段是强类型，不是松散类型
  zap.String("url", url),
  zap.Int("attempt", 3),
  zap.Duration("backoff", time.Second),
)
```



参考链接：

[在Go语言项目中使用Zap日志库-李文周](https://www.liwenzhou.com/posts/Go/zap/)