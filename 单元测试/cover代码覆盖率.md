[TOC]

**代码覆盖率，也就是有效代码的比例为我们提供了一种重要的衡量维度。**

代码覆盖率指的是，在测试时被执行的源代码占全部源代码的比例。测试代码覆盖率可以衡量软件的质量，我们甚至还可以用它来识别无效的代码，检验测试用例的有效性。

如果要用单元测试函数测试代码的覆盖率，Go1.2 之后，我们可以用 cover 工具来实现。

### cover的基本用法

`go test -cover` 能够用测试函数统计出源代码的覆盖率，我们在项目的 sqldb 库中测试一下代码的覆盖率，输出结果为 83.7%。它反映的是整个 package 中所有代码的覆盖率。

```shell
» go test -cover                                                                                                         jackson@bogon
PASS
coverage: 83.7% of statements
ok      github.com/dreamerjackson/crawler/sqldb 0.426s
```

另外，我们还可以收集覆盖率，并进行可视化的展示。具体做法是先将收集到覆盖率的样本文件输出到 coverage.out。

```
go test -coverprofile=coverage.out
```

接着使用 go tool cover 可视化分析代码覆盖率信息。

```
go tool cover -html=coverage.out
```

### 测试环境下的代码覆盖率

通常情况下，代码覆盖率是 go test -cover 运行了 xxx_test.go 文件的测试函数得到的。但是测试用例通常是手动添加的。 如果我们希望能够接入测试环境中的流量来测试代码覆盖率，能不能实现呢？用上一些技巧是完全可以实现的。

假设我们的服务是一个 Web 服务器，我们可以新建一个 main_test.go 文件，在 main_test.go 中执行 main() 函数，这就好像在实际执行程序一样。

```shell
func TestSystem(t *testing.T) {
    handleSignals()

    endRunning = make(chan bool, 1)

    go func() {

      main()

    }()

    <-endRunning
}
```

接着，我们可以在当前目录执行 go test -coverprofile=coverage.out ，或者用 go test -c -coverprofile=coverage.out 生成可执行的测试文件。

然后执行该测试函数，当测试结束时终止程序。退出程序后，就会自动生成 coverprofile 文件了。这时我们可以在测试环境调用 HTTP 请求，并最终统计代码覆盖率。