

推荐先看[官方文档](https://github.com/hibiken/asynq/wiki/Getting-Started)

[Asynq简单、可靠、高效的分布式任务队列](https://juejin.cn/post/7184406673879449658)

## asynq

### 安装

```shell
go get -u github.com/hibiken/asynq
```

### Cli 安装

```shell
go install github.com/hibiken/asynq/tools/asynq@latest
```

#### Cli 常用命令

`asynq -h`

```
USAGE
  asynq <command> <subcommand> [flags]

COMMANDS
  cron:           Manage cron  		管理 cron 任务。
  dash:           View dashboard  查看仪表板。可以看到队列中的各种信息，等待执行的，正在重试的等
  group:          Manage groups		管理任务组
  queue:          Manage queues		管理任务队列
  server:         Manage servers	管理服务器
  stats:          View current state	查看当前状态
  task:           Manage tasks		管理任务

FLAGS
  --cluster       Connect to redis cluster
  --cluster_addrs List of comma-separated redis server addresses
  --config        Config file to set flag defaut values (default is $HOME/.asynq.yaml)
  --db            Redis database number (default is 0)
  --help          Help for asynq
  --password      Password to use when connecting to redis server
  --tls_server    Server name for TLS validation
  --uri           Redis server URI
  --version       Version for asynq

EXAMPLES
  $ asynq stats
  $ asynq queue pause myqueue
  $ asynq task list --queue=myqueue --state=archived

LEARN MORE
  Use 'asynq <command> <subcommand> --help' for more information about a command.

FEEDBACK
  Open an issue at https://github.com/hibiken/asynq/issues/new/choose
```


这些是 `asynq` CLI 工具的主要命令和选项：

- `cron`: 用于管理 cron 任务。
- `dash`: 用于查看仪表板。
- `group`: 用于管理任务组。
- `queue`: 用于管理任务队列。
- `server`: 用于管理服务器。
- `stats`: 用于查看当前状态。
- `task`: 用于管理任务。

这些命令允许你执行各种操作，例如管理任务队列、查看状态、管理服务器等。在使用命令时，你可以使用一些标志（flags）来配置连接选项，比如 `--cluster` 用于连接到 Redis 集群，`--uri` 用于指定 Redis 服务器的 URI 等等。

举例来说：

- `asynq stats`：查看当前状态。
- `asynq queue pause myqueue`：暂停名为 `myqueue` 的队列。
- `asynq task list --queue=myqueue --state=archived`：列出名为 `myqueue` 的队列中处于归档状态的任务列表。
- asynq task cancel [task_id]： 取消任务
- asynq cron ls ：查看当前正在运行的 shceduler

此外，可以通过 `asynq <command> <subcommand> --help` 来获取特定命令的详细信息和用法。

如果你需要更多帮助或有其他问题，也可以在 https://github.com/hibiken/asynq/issues/new/choose 提交问题。

### WebUI

github地址：https://github.com/hibiken/asynqmon

Docker image。不支持mac m系列，m系列会报错：`no matching manifest for linux/arm64/v8 in the manifest list entries`

```shell
# Pull the latest image
docker pull hibiken/asynqmon

# Or specify the image by tag
docker pull hibiken/asynqmon[:tag]

#  Asynqmon web server listens on port 8080 and connects to a Redis server running on 127.0.0.1:6379.
docker run --rm \
    --name asynqmon \
    -p 8080:8080 \
    hibiken/asynqmon
```

可以使用 `---redis-url`  指定redis连接
