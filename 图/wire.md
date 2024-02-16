[依赖注入工具-wire](https://www.liwenzhou.com/posts/Go/wire/)

[Go工程化 - 依赖注入](https://go-kratos.dev/blog/go-project-wire)

使用方法

1. 定义wire.go文件

   ```go
   //go:build wireinject
   // +build wireinject
   
   // The build tag makes sure the stub is not built in the final build.
   
   package main
   
   
   // wireApp init kratos application.
   func wireApp(*conf.Server, *conf.Data, log.Logger) (*kratos.App, func(), error) {
   	panic(wire.Build(server.ProviderSet, data.ProviderSet, biz.ProviderSet, service.ProviderSet, newApp))
   }
   
   ```

2. 生成代码，在Makefile中执行

   ```makefile
   .PHONY: generate
   # generate
   generate:
   	go mod tidy
   	go get github.com/google/wire/cmd/wire@latest
   	go generate ./...
   
   ```

常见用法

- `wire.Build`：注入器

- `wire.NewSet`：`provider`集合

  ```go
  wire.NewSet(NewGRPCServer, NewHTTPServer)
  ```

- `wire.Bind`：绑定接口

  ```go
  
  // Fooer 为接口，MyFooer为实现了Fooer接口的类型
  var ProviderSet = wire.NewSet {
  	NewMyFooer,
    wire.Bind(new(Fooer), new(*MyFooer)),	
  }
  
  ```

  