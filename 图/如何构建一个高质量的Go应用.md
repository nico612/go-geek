开发一个 Go 项目，最核心的目的是开发一个能够满足产品需求的应用，那么如何构建应用呢？不同的开发者有不同的构建方法。

有些开发者将所有的逻辑代码放在一个 main 函数中，有些开发者将不同的逻辑代码放在不同的文件中，并构造一种结构化的 main 函数。虽然，最终都能实现产品需求，但造成的结果却完全不同：

- 有些开发者开发的代码结构混乱，难以阅读，并且后期很难维护，改动一个功能，可能会影响另外一个功能；
- 有些开发者开发的代码结构清晰、易于阅读和维护，功能之间隔离性很强，代码变更互不影响。

作为一名初学者，你一开始就要养成构建健壮应用的习惯，并掌握其构建技巧。在我看来一个合格的应用构建方式具有以下几个特点：

- **应用代码结构清晰：** 清晰的结构不仅利于阅读，还能减少后期改动带来的 Bug；
- **应用代码按功能隔离：** 好的隔离性可以确保后期代码变更不会相互影响。例如功能类的代码变更，不会影响应用框架；
- **应用代码具有扩展性：** 能够清晰地知道在哪里添加新的功能代码，并且添加之后仍然能够保持整个应用框架逻辑一致性。

这节课，我们就来一起展示如何构建一个健壮的应用。

## 应用程序组成部分及构建方法

想要构建一个应用，首先要知道应用程序的组成部分。一个 Go 应用一般由以下 3 部分组成：

- 应用配置；
- 应用业务逻辑；
- 应用启动框架。

### 应用配置

应用配置用来配置应用程序，例如可以通过配置，指定应用需要连接的 MySQL 地址、监听指定的 HTTP 端口、指定支持的机型列表等等。这些配置项，为什么要通过配置来指定？主要有以下 3 大原因：

- **不改变代码而连接不同的环境：** 例如测试环境、开发环境、线上环境使用了不同的 MySQL 地址，通过配置，可以连接不同的 MySQL，而不用改变程序代码；
- **安全：** 一些敏感信息，不适合直接硬编码在代码中，需要通过配置的方式存放在配置文件中，这些配置文件会保存在比较安全的环境下；
- **提高发布效率和应用程序稳定性：** 通过指定配置，而非修改应用代码来改变应用的行为，一方面可以快速实现期望的应用行为，另一方面也会提高应用程序的稳定性（因为配置变更比代码变更更安全）。

那么应用程序从哪里读取配置呢？一般可以从以下 4 个地方来读取配置：

- 命令行选项（option）：选项参数，通常以 `-` 或者 `--` 开头，包含 0 个或 1 个值，例如：`--out-format=json` 或者 `--help`；
- 命令行参数（argument）：非选项参数，命令行中除了 option，其余的部分都是 argument，如 `pip install click`；
- 配置文件：通常为 YAML、JSON、TOML、INI、XML 等格式的文本文件；
- 分布式配置存储服务：例如 Apollo、Etcd、Consul、Ncos 等。

> 提示：执行 shell 命令的一般方式为 `command [options] [arguments]`，`[]` 中的内容是可选的，传递给命令的内容都统一称为参数（argument），参数又分为：选项（option）和参数（argument）。

那么具体如何来读取配置呢？毫无疑问，你可以自己从 0 开始，实现一种读取方法。但更建议的方式是用社区成熟的 Go 包来读取不同位置的配置。

- 命令行选项、命令行参数：可以通过一些 flag 包来读取，例如：标准库 `os` 包、标准库 `flag` 包、[pflag](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspf13%2Fpflag)。
  - `os` 包：只能读取命令行参数；
  - `flag` 包：Go 标准库中的包，内置在 Go 源码中，不需要额外安装，可以用在小项目中，并且项目本身仅需要比较简单的参数解析功能；
  - `pflag`：`pflag` 兼容 `flag` 包，支持丰富的参数类型，具有很多高级的功能，建议命令行参数解析场景统一使用 `pflag` 包。很多大型的开源项目都是使用 `pflag` 来进行参数解析的，例如：Kubernetes、Istio、Helm、Docker、Etcd 等。
- 配置文件：根据不同的配置文件类型，可以选择不同的 Go 包来解析，例如：
  - YAML 格式：[ghodss/yaml](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fghodss%2Fyaml)、[go-yaml/yaml](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgo-yaml%2Fyaml) 等；
  - JSON格 式：encoding/json、[jsoniter](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjson-iterator%2Fgo)、[jsonparser](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fbuger%2Fjsonparser)、[easyjson](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmailru%2Feasyjson) 等；
  - TOML 格式：[toml](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FBurntSushi%2Ftoml)、[go-toml](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fpelletier%2Fgo-toml)；
  - INI 格式：[ini](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgo-ini%2Fini) 等；
  -  还有一些包能够同时解析多种格式的配置文件，例如：[viper](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspf13%2Fviper)、[configor](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjinzhu%2Fconfigor)、[koanf](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fknadh%2Fkoanf)、[config](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgookit%2Fconfig) 等。
- 环境变量：`os.Getenv`、[envconfig](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fkelseyhightower%2Fenvconfig) 等。
- 分布式配置存储服务：这类服务使用官方提供的 client 包即可，例如：[go.etcd.io/etcd/client…](https://link.juejin.cn/?target=http%3A%2F%2Fgo.etcd.io%2Fetcd%2Fclient%2Fv3)（etcd）、[github.com/hashicorp/c…](https://link.juejin.cn/?target=http%3A%2F%2Fgithub.com%2Fhashicorp%2Fconsul%2Fapi)（consul）、[github.com/nacos-group…](https://link.juejin.cn/?target=http%3A%2F%2Fgithub.com%2Fnacos-group%2Fnacos-sdk-go)（ncos）等。

那么我们应该如何选择应用配置的方式呢？建议通过以下思路进行选择：

- **命令行选项、命令行参数：** 选择 `pflag`；
- **配置文件：** 建议选择支持多种配置文件格式的包，也即从：`viper`、`configor`、`koanf`、`config` 中选择其一，毫无疑问 `viper` 胜出：
  - `viper` 有 21000+ 的 Star 数，比其它包更受欢迎；
  - `viper` 功能强大，并且经过很多大型项目验证过；
  - `viper` 同时也可以实现分布式配置中心的功能。
- **环境变量：** 如果环境变量不多，可以使用 `os.Getenv`，如果环境变量很多，可以使用 `envconfig` 直接将环境变量读取到 Go 结构体变量中。
- **配置中心：** 可根据需要选择 `viper`、`apollo`、`etcd`、`consul` 等。其实一般的项目不需要引入配置中心，因为使用配置中心，会带来一些部署、维护的复杂度。

对于一个 Go 应用程序，通常需要解析以下类别的配置：命令行选项、命令行参数、配置文件。根据上面的分析，这里我们选择使用 `pflag`、`viper` 来实现这些能力。值得高兴的是 `pflag`、`viper` 这2 个包不是完全独立的，中间还有联系：`viper` 支持绑定` pflag` 的参数，通过这种绑定，可以统一 `pflag`、`viper` 中的配置项（本课程后面会有案例介绍）。

环境变量因为感知力很差，所以一般不会采用这种方式，作为应用程序的配置，但有些情况下，环境变量却很适用：

- **内容敏感：** 对于一些敏感的配置内容，我们不希望通过命令行选项/参数、配置文件等直接明文暴露，这时候，我们可以通过设置环境变量的方式，让应用程序加载。
- **一些临时的配置：** 例如为了测试，我们需要针对某个用户采用不同的处理逻辑，这时候可以将该用户名通过环境变量传递给程序,而不需要为这种临时的配置场景，去改动代码新增一个命令行选项。

### 应用业务逻辑处理

应用的业务逻辑根据业务的不同差别很大。一般而言，一个 Go 应用中会执行以下类别的业务逻辑处理（可能会用到其中一个或多个）：

- 初始化缓存；
- 初始化并创建各类数据库客户端，例如：Redis、MySQL、Kafka、MongoDB、Etcd 等；
- 初始化并创建其他服务的客户端等；
- 初始化并启动Web服务，例如：HTTP、HTTPS、GRPC；
- 启动异步任务，这些异步任务可以执行任何业务需要的操作，例如：watch kube-apiserver、定期从第三方服务拉取数据，并缓存、注册 `/metrics` 并监听指定的端口、启动 kafka 消费队列等等；
- 执行特定的业务处理，并退出程序；
- 还有很多其他业务逻辑。

### 应用启动框架

首先，这里解释下何为应用启动框架？启动框架你可以理解为一个 main 函数，只不过这里的 main 函数是有代码结构的，并可能分散在多个 Go 源码文件中，在这个大函数中，你可以读取配置文件、初始化业务逻辑、启动 Web 服务等，例如：

```go
go
复制代码package main

import (
    "fmt"
    "net/http"

    "github.com/spf13/pflag"
)

const helpText = `Usage: main [flags] arg [arg...]

This is a very simple app framework (does nothing).

Flags:`

var (
    addr = pflag.String("addr", ":8777", "The address to listen to.")
    help = pflag.BoolP("help", "h", false, "Show this help message.")

    usage = func() {
        fmt.Println(helpText)
        pflag.PrintDefaults()
    }
)

func main() {
    // 1. 命令行参数处理：解析，并读取命令行参数
    pflag.Usage = usage
    pflag.Parse()
    if *help {
        pflag.Usage()
        return
    }

    // 2. 业务处理：初始化路由
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprint(w, "Hello world")
    })
    server := http.Server{Addr: *addr}
    fmt.Printf("Starting http server at %s\n", *addr)

    // 3. 业务处理：启动 HTTP Web 服务
    if err := server.ListenAndServe(); err != nil {
        panic(err)
    }
}
```

上述的 main 函数，可以理解为是最简单的一种平铺式的应用框架，这个框架其实什么也没做，里面内嵌我们启动服务需要的任何操作：打印 Help 信息、读取配置、初始化路由、启动 Web 服务等。

当我们的业务逻辑简单时，使用上述这种平铺式的空白应用框架，仍然能够保持代码可读、易维护。但是当应用逻辑变得复杂，上面的代码就会变得异常臃肿，难以阅读和维护。这时候，就需要一个优秀的应用框架来构建我们的应用。

业界有很多这类应用框架，其中比较受欢迎的有（按 Star 数排序）：[cobra](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fspf13%2Fcobra)、[urfave/cli](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Furfave%2Fcli)、[kingpin](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Falecthomas%2Fkingpin)。

这里建议选择 cobra，理由如下：

- cobra 功能强大、易用、代码质量高；
- 业界有很多优秀的开源项目已经使用cobra来构建应用，例如： Kubernetes、Docker、etcd、Rkt、Hugo 等；
- cobra 可以和 pflag 组合，实现更强大、更易用的命令行参数处理能力。

## 最佳构建方法

所以，最终结论就是：我们可以使用 `pflag`、`viper`、`cobra` 来构建一个强大的应用程序（这也是当前大部分团队选择的构建方法）。因为 `pflag`、`viper`、`cobra` 功能强大，每一个包都有不少内容可以学习，并且网上也有很多教程，本节课就不再详细介绍，这里推荐我整理过的学习文档供你参考：

- pflag：[如何使用Pflag给应用添加命令行标识](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmarmotedu%2Fgeekbang-go%2Fblob%2Fmaster%2F%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8Pflag%E7%BB%99%E5%BA%94%E7%94%A8%E6%B7%BB%E5%8A%A0%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%A0%87%E8%AF%86.md)；
- viper：[配置解析神器-Viper全解](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmarmotedu%2Fgeekbang-go%2Fblob%2Fmaster%2F%E9%85%8D%E7%BD%AE%E8%A7%A3%E6%9E%90%E7%A5%9E%E5%99%A8-Viper%E5%85%A8%E8%A7%A3.md)；
- cobra：[现代化的命令行框架-Cobra全解](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmarmotedu%2Fgeekbang-go%2Fblob%2Fmaster%2F%E7%8E%B0%E4%BB%A3%E5%8C%96%E7%9A%84%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%A1%86%E6%9E%B6-Cobra%E5%85%A8%E8%A7%A3.md)。

之前我们介绍过，提升 Go 开发能力一个非常重要的方法是阅读优秀开源项目的源码。`pflag`、`viper`、`cobra` 3 个包功能强大、代码质量很高。在你学习本课程的时候，也可以借此机会阅读这 3 个项目的源码，以提升自己的开发能力。

## miniblog 应用构建

miniblog 也使用了 `pflag`、`viper`、`cobra` 来构建。具体步骤如下：

1. 修改 `main.go` 文件，确保 `main.go` 的入口属性。

使用 cobra 框架创建应用，我们需要创建 `*cobra.Command` 对象，创建该对象，会引入一些代码量，考虑到未来还有更多的业务逻辑代码需要增加，为了保证 `cmd/miniblog/main.go` 文件代码清晰易读，我们需要将这些复杂的业务存放到 `internal/miniblog/miniblog.go` 文件中。修改后的 `cmd/miniblog/main.go` 文件内容如下：

```go
go
复制代码package main

import (
    "os"

    _ "go.uber.org/automaxprocs"

    "github.com/marmotedu/miniblog/internal/miniblog"
)

// Go 程序的默认入口函数(主函数).
func main() {
    command := miniblog.NewMiniBlogCommand()
    if err := command.Execute(); err != nil {
        os.Exit(1)
    }
}
```

上面的 main 文件只有简单几行代码，具体的代码实现都放在 `internal/miniblog/miniblog.go` 文件中。将 main 入口放在 `cmd/miniblog` 目录下，将具体的代码实现放在 `internal/miniblog` 目录下的好处如下：

- 查阅代码时，方便定位到 miniblog 服务的入口函数。
- 方便通过 `go install` 安装 miniblog 服务，例如：` go install ``github.com/marmotedu/miniblog/cmd/miniblog@latest`。

另外，上述代码，通过导入匿名包 `go.uber.org/automaxprocs` 来使程序自动设置 `GOMAXPROCS` 以匹配 Linux 容器 CPU 配额。通过正确设置容器的 CPU 配额，可以解决 `GOMAXPROCS` 可能设置过大，导致生成线程过多，从而导致严重的上下文切换，浪费 CPU，降低程序性能的潜在问题。

> 提示：容器中 `GOMAXPROCS` 设置的问题，详细可参考 [GOMAXPROCS 的“坑”](https://link.juejin.cn/?target=https%3A%2F%2Fpandaychen.github.io%2F2020%2F02%2F28%2FGOMAXPROCS-POT%2F) 和 [Uber-Automaxprocs 分析](https://link.juejin.cn/?target=https%3A%2F%2Fpandaychen.github.io%2F2020%2F02%2F29%2FAUTOMAXPROCS-ANALYSIS%2F) 2 篇文章。

上述代码通过 `miniblog.NewMiniBlogCommand()` 函数调用，将业务的具体实现存放在了 `miniblog` 包中。

1. 创建 `*cobra.Command` 对象。

新建 `internal/miniblog/miniblog.go` 文件，内容如下：

```go
go
复制代码package miniblog

import (
    "fmt"

    "github.com/spf13/cobra"
)

// NewMiniBlogCommand 创建一个 *cobra.Command 对象. 之后，可以使用 Command 对象的 Execute 方法来启动应用程序.
func NewMiniBlogCommand() *cobra.Command {
    cmd := &cobra.Command{
        // 指定命令的名字，该名字会出现在帮助信息中
        Use: "miniblog",
        // 命令的简短描述
        Short: "A good Go practical project",
        // 命令的详细描述
        Long: `A good Go practical project, used to create user with basic information.

Find more miniblog information at:
        https://github.com/marmotedu/miniblog#readme`,

        // 命令出错时，不打印帮助信息。不需要打印帮助信息，设置为 true 可以保持命令出错时一眼就能看到错误信息
        SilenceUsage: true,
        // 指定调用 cmd.Execute() 时，执行的 Run 函数，函数执行失败会返回错误信息
        RunE: func(cmd *cobra.Command, args []string) error {
            return run()
        },
        // 这里设置命令运行时，不需要指定命令行参数
        Args: func(cmd *cobra.Command, args []string) error {
            for _, arg := range args {
                if len(arg) > 0 {
                    return fmt.Errorf("%q does not take any arguments, got %q", cmd.CommandPath(), args)
                }
            }

            return nil
        },
    }

    return cmd
}

// run 函数是实际的业务代码入口函数.
func run() error {
    fmt.Println("Hello MiniBlog!")
    return nil
}
```

通过 `NewMiniBlogCommand` 函数初始化并创建了一个 `*cobra.Command` 对象，通过该对象，我们实现了以下应用功能：

- 指定命令的名字，该名字会出现在帮助信息中；
- 指定命令的简短描述；
- 指定命令的详细描述；
- 设置命令出错时，不打印帮助信息；
- 指定调用 `cmd.Execute()` 时，执行的 `Run` 函数;
- 设置命令运行时，不需要指定命令行参数。

## 编译并运行 miniblog

在 miniblog 项目根目录下执行以下命令：

```bash
bash
复制代码$ make # 编译 miniblog
$ _output/miniblog -h # 打印 miniblog 使用帮助信息
A good Go practical project, used to create user with basic information.

Find more miniblog information at:
        https://github.com/marmotedu/miniblog#readme

Usage:
  miniblog [flags]

Flags:
  -h, --help   help for miniblog
$ _output/miniblog test # 指定命令行参数
Error: "miniblog" does not take any arguments, got ["test"]
$ _output/miniblog # 运行 miniblog
Hello MiniBlog!
```

可以看到，cobra 框架会根据设置自动校验是否传入了命令行参数，并给出可读的报错信息。cobra 还帮我们整理并输出了可读的帮助信息。

