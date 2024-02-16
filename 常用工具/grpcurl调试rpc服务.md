## grpc测试工具

Protobuf 本身具有反射功能，可以在运行时获取对象的 Proto 文件。gRPC 同样也提供了一个名为 reflection 的反射包，用于为 gRPC 服务提供查询。gRPC 官方提供了一个 C++ 实现的 grpc_cli 工具，可以用于查询 gRPC 列表或调用 gRPC 方法。但是 C++ 版本的 grpc_cli 安装比较复杂，我们推荐用纯 Go 语言实现的 grpcurl 工具。本节将简要介绍 grpcurl 工具的用法。

更多测试工具可查看该文章：https://blog.wu-boy.com/2022/08/three-grpc-testing-tool/

grpcurl使用方法：https://chai2010.cn/advanced-go-programming-book/ch4-rpc/ch4-08-grpcurl.html

安装：

```go
 go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

### 启动反射服务

reflection 包中只有一个 Register 函数，用于将 grpc.Server 注册到反射服务中。reflection 包文档给出了简单的使用方法：

```shell
import (
    "google.golang.org/grpc/reflection"
)

func main() {
    s := grpc.NewServer()
    pb.RegisterYourOwnServer(s, &server{})

    // Register reflection service on gRPC server.
    reflection.Register(s)

    s.Serve(lis)
}

```

### 查看服务列表

列出`gRPC` 服务器上支持的服务，

- 本地端没有启用`tls`协议认证，需要加上`-plaintext`忽略 `tls` 证书的验证过程，详细参数可通过`-h`查看
- 如果启用了`tls`协议服务需要通过 `-cert`和`key` 参数设置公钥和私钥文件

```go
$ grpcurl -plaintext localhost:8080 list
grpc.channelz.v1.Channelz
grpc.health.v1.Health
grpc.reflection.v1.ServerReflection
grpc.reflection.v1alpha.ServerReflection

```

如果 grpc 服务正常，但是服务没有启动 reflection 反射服务，将会遇到以下错误：

```go
$ grpcurl -plaintext localhost:1234 list
Failed to list services: server does not support the reflection API
```

- `grpc.channelz.v1.Channelz`: 这是 gRPC 提供的 Channelz 服务。Channelz 是一个诊断工具，允许你检查和监视 gRPC 通道的状态和性能指标。它提供了有关 gRPC 通道（包括连接和流）的详细信息。
- `grpc.health.v1.Health`: 这是 gRPC 健康检查服务。健康检查服务允许客户端查询服务器的健康状态，用于确定服务器是否可用和健康。通常，它用于负载均衡器等组件检查服务的健康情况。
- `grpc.reflection.v1.ServerReflection`: 这是 gRPC 提供的服务器反射服务的 v1 版本。服务器反射服务允许客户端查询服务器支持的服务、方法和消息类型的元数据信息。
- `grpc.reflection.v1alpha.ServerReflection`: 这是 gRPC 提供的服务器反射服务的 v1alpha 版本。v1alpha 版本通常包含一些实验性或较新的功能，可能与 v1 版本有所不同或新增一些特性。

这些服务提供了对 gRPC 服务器状态、健康、元数据信息的访问，以及对 gRPC 通道的诊断和监控。这些信息对于调试、监控和管理 gRPC 服务器非常有用。

#### 检查健康状态

```go
$ grpcurl -plaintext localhost:4560 grpc.health.v1.Health/Check

{
  "status": "SERVING"
}

```

### 服务的方法列表

protobuf 文件如下：

```go
syntax = "proto3";

package HelloService;

message String {
	string value = 1;
}

service HelloService {
	rpc Hello (String) returns (String);
	rpc Channel (stream String) returns (stream String);
}

```

继续使用 list 子命令还可以查看 HelloService 服务的方法列表：

```go
$ grpcurl -plaintext localhost:1234 list HelloService.HelloService
Channel
Hello
```

从输出可以看到 HelloService 服务提供了 Channel 和 Hello 两个方法，和 Protobuf 文件的定义是一致的。

如果还想了解方法的细节，可以使用 grpcurl 提供的 describe 子命令查看更详细的描述信息：

```go
$ grpcurl -plaintext localhost:1234 describe HelloService.HelloService
```

### 获取类型信息

在获取到方法的参数和返回值类型之后，还可以继续查看类型的信息。下面是用 describe 命令查看参数 HelloService.String 类型的信息：

```go
$ grpcurl -plaintext localhost:1234 describe HelloService.String
```

### 调用方法

在获取 gRPC 服务的详细信息之后就可以 json 调用 gRPC 方法了。

下面命令通过 `-d` 参数传入一个 json 字符串作为输入参数，调用的是 HelloService 服务的 Hello 方法：

```go
$ grpcurl -plaintext -d '{"value":"gopher"}' \
    localhost:1234 HelloService.HelloService/Hello
{
  "value": "hello:gopher"
}
```

如果 `-d` 参数是 `@` 则表示从标准输入读取 json 输入参数，这一般用于比较输入复杂的 json 数据，也可以用于测试流方法。



