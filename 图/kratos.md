微服务框架

kratos

官方开发文档：https://go-kratos.dev/docs/

简单的商城微服务实战：https://learnku.com/articles/64942

[kratos框架系列教程](https://www.bilibili.com/video/BV1de4y1y7eu/)

常用命令

| 命令                                                         | 描述                                              |
| ------------------------------------------------------------ | ------------------------------------------------- |
| kratos new <project>                                         | 新建项目                                          |
| kratos new <project> -r https://gitee.com/go-kratos/kratos-layout.git | 新建项目指定源                                    |
| kratos new app/user --nomod                                  | 使用 `--nomod` 添加服务，共用 `go.mod` ，大仓模式 |
| kratos proto add api/helloworld/v1/demo.proto                | 添加proto文件                                     |
| make api 或者 kratos proto client api/helloworld/v1/demo.proto | 生成 Proto 代码                                   |
| kratos proto server api/helloworld/v1/demo.proto -t internal/service | 生成 Service 代码，使用 `-t` 指定生成目录         |
| kratos -h<br/>kratos new -h                                  | 查看帮助                                          |





