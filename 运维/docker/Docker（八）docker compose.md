## Docker compose简介与安装

前面我们使用 Docker 的时候，定义 Dockerfile 文件，然后使用 docker build 打包成镜像、使用 docker run 运行 容器等命令。然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启动和停止，那么效率之低，维护量之大可想而知。

  Docker Compose 是一个用来定义和运行复杂应用的 Docker 开源项目，负责实现对 Docker 容器集群的快速编排，它允许用户通过一个单独的`docker-compose.yml`模板文件 (YAML 格式) 来定义一组相关联的应用容器为一个项目，在配置文件中，所有的容器通过 services 来定义，然后使用 docker-compose 脚本来启动、停止和重启应用，非常适合组合使用多个容器进行开发的场景。

Docker compose V2 已经集成到了Docker CLI platform， 安装docker后就有docker compose 的功能

```shell
root@admin:~# docker compose version
Docker Compose version v2.21.0
root@admin:~#
```

## docker-compose 模板命令讲解

docker-compose.yml文件的基本语法可以参考官方文档：

https://docs.docker.com/compose/compose-file/compose-file-v3/

docker compose文件中可以定义多个相互关联的应用容器，每一个应用容器被称为一个服务（service）。由于service就是在定义某个应用的运行时参数，因此与`docker run`参数非常相似。

对比如下：

| **docker run 参数** | **docker compose 指令** | **说明**   |
| :------------------ | :---------------------- | :--------- |
| --name              | container_name          | 容器名称   |
| -p                  | ports                   | 端口映射   |
| -e                  | environment             | 环境变量   |
| -v                  | volumes                 | 数据卷配置 |
| --network           | networks                | 网络       |

`version`：指定`docker-compose.yml`文件的版本

```yaml
格式：
    version: "版本号"
示例：
    version: "3.0"
```

`docker compose` 与 `docker`的兼容性如下 (可以向下兼容，即：`docker`版本在`19.03.0+`这个版本，可以使用`docker compose3.8版本`或者`3.8以下的版本`都是可以的，不过推荐写低两位，避免最新版本不兼容，即：`3.0~3.6`之间)

`services`：多个应用容器的集合，一个服务就代表一个应用容器

```yaml
version: "3.0"
services:
   定义服务
```

`build`：指定构建镜像时，`Dockerfile` 所在目录的路径 (可以是绝对路径，也可以是相对 `docker-compose.yml` 文件的路径)。`docker compose`指令将会利用它自动构建这个镜像，然后使用这个镜像

```yaml
/**
 * 方式一：使用的是相对路径，指的是Dockerfile文件在其
 *        docker-compose.yml当前目录下的dir目录中，
 *        文件名称必须叫Dockerfile
 * 
 * 这里的webapp指的是你的服务名，在docker-compose.yml文件中必须唯一      
 **/
version: "3.0"
services:
  webapp:
    build: ./dir



/**
 * 方式二：
 *   1、可以使用context指令指定Dockerfile所在目录的路径
 *   2、如果名称不叫Dockerfile，可以使用dockerfile指令
 *      指定其文件名             
 **/
version: "3.0"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: myfile


```

`image`：指定服务所使用的镜像，如果镜像在本地不存在，`docker-compose`指令将会尝试去拉取这个镜像。与`build`指令二选一

```shell
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
```

`container_name`：指定容器名称。如果不指定默认将会使用 `项目名称_服务名称_序号`这样的格式

```yaml
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx

```

`ports`：指定容器暴露的端口信息，使用 宿主端口: 容器端口 (`HOST:CONTAINER`) 格式，或者仅仅指定容器的端口 (宿主将会随机选择端口) 都可以

```shell
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx
    ports:
      - "80:80"

# 注：在暴露端口时，建议都采用引号包括起来的字符串格式，避免解析出错
```

`volumes`：数据卷所挂载路径设置。可以设置为 宿主机绝对路径: 容器内目录的绝对路径 或者 数据卷名称: 容器内目录的绝对路径，并且还可以设置访问模式 (默认为`rw`读写模式，`ro`表示只读)

```yaml
/**
 * 方式一：使用宿主机绝对路径进行挂载    
 **/
version: "3.0"
services:
  webapp:
    image: mysql:5.6
    container_name: myMysql
    ports:
      - "3306:3306"
    volumes:
      - /root/data:/var/lib/mysql

/**
 * 方式二：使用数据卷名称进行挂载，必须在文件中对数据卷名称进行声明
 **/
version: "3.0"
services:
  webapp:
    image: mysql:5.6
    container_name: myMysql
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql 
      - mysql_logs:/logs

volumes:   #对上面使用到的数据卷名称进行声明
  mysql_data:   #数据卷名称前面会自带项目名称，(当前docker-compose.yml所在目录的名称)，即项目名称_数据卷名称
    external: true  #如果不想加上项目名称，请设置external为true，这时就需要我们在启动服务之前，必须在外部使用命令去手动创建一个名叫mysql_data的数据卷 (创建命令: docker volume create mysql_data)
  mysql_logs:


```

`environment`：设置环境变量，可以使用数组或字典两种格式

```shell
version: "3.0"
services:
  webapp:
    image: mysql:5.6
    container_name: myMysql
    ports:
      - "3306:3306"
    environment:  #字典格式
      MYSQL_ROOT_PASSWORD: 123456
      SESSION_SECRET:  #只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。
    或者
    environment:   #数组格式
      - MYSQL_ROOT_PASSWORD=123456
      - SESSION_SECRET  

```

`env_file`：从以`.env`结尾的文件中获取环境变量，可以为单独的文件路径或列表

```shell
version: "3.0"
services:
  webapp:
    image: mysql:5.6
    container_name: myMysql
    ports:
      - "3306:3306"
    env_file: ./mysql.env  #单独单独的文件路径
    或者
    env_file:  #列表格式
      - ./mysql.env

/**
 * 1、环境变量文件中的每一行必须符合字典格式，即变量名=值
 * 2、支持 # 开头的注释行。
 * mysql.env环境变量文件内容如下
 **/
MYSQL_ROOT_PASSWORD=123456

```

`networks`：配置容器连接的网络

```yaml
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx
    networks:
      - nginx_network  

networks:     #对上面使用到的网络名称进行声明
  nginx_network:  #网络名称前面会自带项目名称，(当前docker-compose.yml所在目录的名称)，即项目名称_网络名称
    external: true  #如果不想加上项目名称，请设置external为true，这时就需要我们在启动服务之前，必须在外部使用命令去手动创建一个名叫nginx_network的网络(创建命令: docker network create -d bridge nginx_network)

```

`depends_on`：解决容器的依赖、启动先后的问题 (一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败)

下面示例就会先启动 `redis` 和 `mysql` 这两个服务，最后才启动 webapp 服务 (`提示`：`webapp` 服务不会等待 `redis` 和 `mysql` 服务`完全启动` 之后才启动，而是会先启动依赖服务，再去启动`webapp` 服务，但是依赖服务最终会在`webapp` 服务完全启动之前 完全启动)

```yaml
version: "3.0"
services:
  webapp:
    build: .
    container_name: myApp
    depends_on:    #设置webapp服务依赖于redis和mysql服务
      - redis
      - mysql  

  redis:
    image: redis:3.2
    container_name: myRedis
    
  mysql:
    image: mysql:5.6
    container_name: myMysql


```

`command`：覆盖容器启动后默认执行的命令，即：覆盖 DockerFile 中的`CMD`或第三方镜像的启动命令

```shell
version: "3.0"
services:
  webapp:
    image: redis:3.2 .
    container_name: myRedis
    ports:    
      - "6379:6379"
    command: "redis-server --appendonly yes"
    或者
    command: ["redis-server", "--appendonly", "yes"]
```

`healthcheck`：通过命令检查容器是否健康运行

```shell
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx
    ports:
      - "80:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 1m30s  #时间间隔
      timeout: 10s     #超时时间
      retries: 3       #重试次数
```

`labels`：为容器添加元数据 (metadata) 信息，例如：可以为容器添加辅助说明信息，支持数组或字典格式

```shell
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx
    ports:
      - "80:80"
    labels:
      - "com.startupteam.description=webapp for a strtup team" #数组格式
    或者
    labels:
      com.startupteam.description: "webapp for a strtup team" #字典格式
```

`restart`：指定容器退出后的重启策略，在生产环境中推荐配置为 `always` 或者 `unless-stopped`

- `no：` 在任何情况下都不会重启容器
- `always：` 容器总是会重新启动
- `on-failure：` 如果退出代码指示失败错误，则该策略会重新启动容器
- `unless-stopped：` 总是重新启动容器，除非容器停止

```yaml
version: "3.0"
services:
  webapp:
    image: nginx:1.21.1
    container_name: myNginx
    ports:
      - "80:80"
    restart: always

```

想要了解更多 docker-compose 模板命令请阅读[官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.docker.com%2Fcompose%2Fcompose-file%2Fcompose-file-v3%2F)

案例：编写一个综合的`docker-compose.yaml`，设置`mysql`服务依赖于`nginx`服务，并同处于名叫`common_network`的网络下

```yaml
version: "3.0"
services:
  mysql:
    image: mysql:5.6
    container_name: myMysql
    ports:
      - "3306:3306"
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=123456
      - SESSION_SECRET
    depends_on:
      - nginx
    networks:
      - common_network
    volumes:
      - mysql_data:/var/lib/mysql 
      - mysql_logs:/logs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 1m30s  #时间间隔
      timeout: 10s     #超时时间
      retries: 3       #重试次数

    
  nginx:
    image: nginx:1.21.1
    container_name: myNginx
    ports:
      - "80:80"
    restart: always
    networks:
      - common_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 1m30s  #时间间隔
      timeout: 10s     #超时时间
      retries: 3       #重试次数


volumes:    #对上面使用到的数据卷名称进行声明
  mysql_data: 
  mysql_logs:
networks:   #对上面使用到的网络名称进行声明  
  common_network: 


```

## docker compose 指令讲解

`提示`：

1. 在执行`docker compose`指令时，如果不指定配置文件，就需要在执行指令的目录下必须要有`docker-compose.yml`配置文件且文件名必须叫`docker-compose.yml`
2. 如果执行`docker compose`指令的目录下没有`docker-compose.yml`配置文件 或者 配置文件名不叫`docker-compose.yml`，需要在`docker compose`后面添加`-f 配置文件路径`去指定配置文件，例如：`docker compose -f /root/myfile.yml up`
3. 如果通过 `docker compose -f file` 的方式来指定`docker-compose.yml`配置文件，那么配置文件中那些使用相对路径的命令则会基于`-f file`中 file 的路径来进行对照

执行 `docker compose --help` 查看指令

```shell
oot@admin:~# docker compose --help

Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker.

Options:
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       Specify an alternate environment file.
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, quiet) (default "auto")
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Commands:
  build       Build or rebuild services
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service.
  down        Stop and remove containers, networks
  events      Receive real time events from containers.
  exec        Execute a command in a running container.
  images      List images used by the created containers
  kill        Force stop service containers.
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding.
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service.
  start       Start services
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  wait        Block until the first service container stops
```



### 运行服务命令

```shell
/**
 * 1、以交互模式启动docker-compose.yml中定义的服务
 * 2、通过ctrl+c停止命令时，将会停止所有的服务
 * 3、service可选，如果不指定启动的服务名，默认启动所有服务
 **/
docker compose up [service]

/**
 * 1、以后台方式启动docker-compose.yml中定义的服务
 * 2、一般推荐生产环境下使用该选项
 **/
docker compose up -d [service]

```

### 停止并移除服务命令

```shell
/**
 * 1、停止并移除docker-compose.yml中定义的服务
 * 2、移除docker compose up运行过程中自建的网络，自己手动使用命令创建的网络不会移除
 **/
docker compose down
```

### 删除服务

```shell
/**
 * 移除docker-compose.yml中定义的服务
 * options:
 *   -f, --force 强制直接删除，包括非停止状态的容器
 *   -v 删除容器所挂载的数据卷
 **/
docker compose rm [options] [service]

```

### 进入指定的服务内部

```shell
docker compose exec service服务名称 /bin/bash 或 /bin/sh
```

### 启动、停止和重启服务

```shell

#启动服务
docker compose start [service]

#停止服务
docker compose stop [service]

#强制停止服务
docker compose kill [service]

#重启服务
docker compose restart [service]
```

### 7、查看服务内运行的进程

```shell
docker-compose top [service]
```

### 8、查看服务的日志

```shell
docker compose logs [service]
```













