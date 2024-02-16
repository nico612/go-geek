[TOC]



## Docker 常用命令

| **命令**       | **说明**                       | **文档地址**                                                 |
| :------------- | :----------------------------- | :----------------------------------------------------------- |
| docker pull    | 拉取镜像                       | [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) |
| docker push    | 推送镜像到DockerRegistry       | [docker push](https://docs.docker.com/engine/reference/commandline/push/) |
| docker images  | 查看本地镜像                   | [docker images](https://docs.docker.com/engine/reference/commandline/images/) |
| docker rmi     | 删除本地镜像                   | [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/) |
| docker run     | 创建并运行容器（不能重复创建） | [docker run](https://docs.docker.com/engine/reference/commandline/run/) |
| docker stop    | 停止指定容器                   | [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) |
| docker start   | 启动指定容器                   | [docker start](https://docs.docker.com/engine/reference/commandline/start/) |
| docker restart | 重新启动容器                   | [docker restart](https://docs.docker.com/engine/reference/commandline/restart/) |
| docker rm      | 删除指定容器                   | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/rm/) |
| docker ps      | 查看容器                       | [docker ps](https://docs.docker.com/engine/reference/commandline/ps/) |
| docker logs    | 查看容器运行日志               | [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) |
| docker exec    | 进入容器                       | [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) |
| docker save    | 保存镜像到本地压缩文件         | [docker save](https://docs.docker.com/engine/reference/commandline/save/) |
| docker load    | 加载本地压缩文件到镜像         | [docker load](https://docs.docker.com/engine/reference/commandline/load/) |
| docker inspect | 查看容器详细信息               | [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) |

![image-20231213151458677](../img/image-20231213151458677.png)

### docker 相关命令

#### docker 启动与停止

```shell
//启动docker
systemctl start docker

//停止docker
systemctl stop docker

//重启docker
systemctl restart docker

//设置docker开机自启
systemctl enable docker

```

#### 查看docker状态

```shell
systemctl status docker
```

#### 查看 docker 版本信息

```
docker version
```

#### 查看 docker 概要信息

```shell
docker info
```

#### 查看 docker 帮助文档

```shell
docker --help
```

### 镜像相关命令

#### 查看本地主机上的镜像

```shell
➜  ~ docker images
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
hello-world              latest    b038788ddb22   7 months ago   9.14kB
```

`REPOSITORY`：镜像的仓库源，即镜像名称

`TAG`：镜像版本标签，即版本号

- 同一镜像仓库源可以有多个 TAG，代表这个镜像仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。
- 如果不指定一个镜像的版本标签，docker 将默认使用最新版，即 latest 版本，例如你使用 docker pull tomcat，docker 将默认拉取 tomcat 最新版镜像，即 docker pull tomcat:latest。

`IMAGE ID`：镜像 ID

`CREATED`：镜像的创建日期（不是获取该镜像的日期）

`SIZE`：镜像大小

#### 查看 docker images 镜像命令的帮助文档

```
docker images --help
```

**docker images 相关命令的主要用法为：** `docker images [OPTIONS] [REPOSITORY[:TAG]]`，即 docker images [docker 镜像命令选项] [镜像名称 [: 版本号]]，`中括号[]`表示该命令选项 `可选可不选`

常用的 docker images 镜像命令选项有：

`-a` ：列出本地所有的镜像 (包含中间镜像层)

`-q`：只显示镜像 ID

`--digests`：显示镜像的摘要信息

`--no-trunc`：显示完整的镜像信息

**docker images 常用命令：**

```shell
//查看镜像
docker images

//查看所有镜像(包含中间镜像层)
docker images -a

//查看镜像ID
docker images -q

//查看所有镜像ID
docker images -aq

```

#### 查找 docker 镜像

```
docker search 镜像名称
```

`NAME`：仓库名称

`DESCRIPTION`：镜像描述

`STARS`：点赞数，反应一个镜像的受欢迎程度

`OFFICIAL`：是否为官方镜像

`AUTOMATED`：自动构建，表示该镜像由 Docker Hub 自动构建流程创建的

#### 查看 docker search 镜像命令的帮助文档

```shell
docker search --help
```

**docker search 相关命令的主要用法为：** `docker search [OPTIONS] TERM`，即 docker search [docker 镜像命令选项] 镜像，`中括号[]`表示该命令选项 `可选可不选`

常用的 docker search 镜像命令选项有：

`-f stars=数字` ：列出点赞数不小于指定值的镜像

`--limit 数字`：列出指定数目的镜像，默认数目为 25

**docker search 常用命令：**

```shell
//查询镜像
docker search 镜像名称

例如：docker search tomcat  //查询tomcat镜像

//查询点赞数不小于指定值的镜像
docker search -f stars=n 镜像名称

例如：docker search -f stars=5 tomcat  //查询点赞数大于等于5的tomcat镜像

//查询点赞数排名前n的镜像
docker search --limit n 镜像名称

例如：docker search --limit 5 tomcat  //查询点赞数排名前5的tomcat镜像


```

#### 拉取 docker 镜像

```shell
docker pull 镜像名称[:版本号]
```

例如：`docker pull mysql:5.7`

#### 删除 docker 镜像

```shell
//删除单个镜像
docker rmi 镜像ID/镜像名称[:版本号]

//-f:表示强制删除，强制删除单个镜像
docker rmi -f 镜像ID/镜像名称[:版本号]

//删除多个镜像
docker rmi [-f] 镜像1ID/镜像1名称[:版本号] 镜像2ID/镜像2名称[:版本号]

//删除全部镜像
docker rmi [-f] $(docker images -aq)

```

例如：`docker rmi -f mysql:5.7`

#### 获取 docker 镜像元信息

```
docker inspect 镜像ID/镜像名称[:版本号]
```

### 容器相关命令

#### 查看 docker run 容器命令的帮助文档

```
docker run --help
```

**docker run 相关命令的主要用法为：** `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`，即 docker run [docker 容器命令选项] 镜像 [命令] [参数]，`中括号[]`表示该命令选项 `可选可不选`

常用的 docker run 容器命令选项有：

`-i` ：表示以交互模式运行容器，通常与`-t`结合使用

`-t`：为容器重新分配一个伪输入终端，通常与`-i`结合使用

`-d`：后台运行容器，并返回容器 ID，即启动守护式容器 (这样创建的容器不会分配一个伪输入终端，如果是以`-it`两个参数启动，启动后则会分配一个伪输入终端)

`-p`：指定端口映射，格式为：`-p 主机(宿主机)端口:容器映射端口`，可以使用多个`-p`做多个端口映射

`-v`：指定挂载主机目录 / 文件 到容器目录 / 文件 上，即挂载容器数据卷，格式为：`-v 主机(宿主机)目录/文件的绝对路径:容器内目录/文件的绝对路径[:读取权限]`，可以使用多个`-v`做多个目录或文件映射，默认为`rw读写模式`，`ro表示只读`。

  `rw读写模式`：表示宿主机能对数据卷进行读取和更改，容器也能对其进行读取和更改。
 `ro表示只读`：表示宿主机能对数据卷进行读取和更改，容器只能对其进行读取不能更改。

```
--name`：为创建的容器指定一个名称，格式为：`--name=容器名称
```

#### 创建并运行容器

```shell
//以交互模式运行容器
docker run -it -v 宿主机目录/文件的绝对路径:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

//以后台方式运行容器 (推荐)
docker run -d -v 宿主机目录/文件的绝对路径:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

```

**注意：这里启动容器时，没有挂载容器数据卷，一般建议挂载容器数据卷，实现数据持久化操作**

**在浏览器中，输入你的 Linux 系统的 ip 地址: 主机 (宿主机) 端口，即可访问**

#### 查看 docker ps 容器命令的帮助文档

```
docker ps --help
```

**docker ps 相关命令的主要用法为：** `docker ps [OPTIONS]`，即 docker ps [docker 容器命令选项]，`中括号[]`表示该命令选项 `可选可不选`

常用的 docker ps 容器命令选项有：

`-a` ：列出当前所有`正在运行`的容器 和 `之前运行过但已停止`的容器

`-l`：显示最近创建的容器

`-q`：只显示容器编号

`-n 数字`：显示最近创建的 n 个容器

`-f status=exited`：查看已停止的容器

**docker ps 常用命令：**

```shell
//查看当前运行的容器
docker ps

//查看所有容器 (运行和停止的)
docker ps -a

//查看最近创建的容器
docker ps -l

//查看最近创建指定个数的容器
docker ps -n 数量

//查看停止的容器
docker ps -f status=exited

//查看所有容器的ID
docker ps -aq


```

#### 列出当前所有正在运行的容器

```
docker ps
```

`CONTAINER ID`：容器 ID

`IMAGE`：镜像

`COMMAND`：命令

`CREATED`：创建时间

`STATUS`：启动时长

`PORTS`：端口映射

`NAMES`：容器名称

#### 容器启动与停止

```shell
//启动容器
docker start 容器ID/容器名称

//重启容器
docker restart 容器ID/容器名称

//停止容器
docker stop 容器ID/容器名称

//强制停止容器
docker kill 容器ID/容器名称


```

#### 删除容器

```shell
//删除已停止的容器
docker rm 容器ID/容器名称

//-f:表示强制删除，删除正在运行的容器
docker rm -f 容器ID/容器名称

//删除全部的容器
docker rm -f $(docker ps -qa)

```

#### 查看容器内部运行的进程

```shell
docker top 容器ID/容器名称
```

#### 查看容器内部元信息

```shell
docker inspect 容器ID/容器名称
```

#### 进入正在运行的容器内并以命令行交互

```bash

docker exec -it 容器ID/容器名称 /bin/bash 或 /bin/sh

//以attach方式进入到容器
docker attach 容器ID/容器名称

//如果不想进入容器，直接获取相关指令的运行结果，可在后面填写相关操作指令
docker exec -it 容器ID/容器名称 相关命令
```

exec 与 attach 的区别：

- `exec`：是在容器中打开新的终端，并且可以启动新的进程 **(推荐)**
- `attach`：是直接进入容器启动命令的终端，不会启动新的进程

#### 退出容器

**上面说过，可以使用命令进入到正在运行的容器内，那么该如何退出容器呢？请使用以下命令：**

```less
//退出并停止容器
exit

//退出但容器不停止
ctrl + p + q
```

#### 文件拷贝

```shell
//从容器内拷贝文件到宿主机
docker cp 容器ID/容器名称:容器内目录/文件的绝对路径 宿主机目录/文件的绝对路径

//从宿主机中拷贝文件到容器内
docker cp 宿主机目录/文件的绝对路径 容器ID/容器名称:容器内目录/文件的绝对路径
```

#### 查看 docker logs 容器命令的帮助文档

```shell
docker logs --help
```

**docker logs 相关命令的主要用法为：** `docker logs [OPTIONS]`CONTAINER，即 docker ps [docker 容器命令选项] 容器，`中括号[]`表示该命令选项 `可选可不选`

常用的 docker logs 容器命令选项有：

`-f` ：显示最新的打印日志

`-t`：显示时间戳

`--tail 数字`：显示最后多少条日志

**docker logs 常用命令：**

```shell
//查看容器日志并显示时间戳
docker logs -t 容器ID/容器名称

//持续输出容器日志
docker logs -f 容器ID/容器名称

//查看最后n条容器日志
docker logs --tail n 容器ID/容器名称
```

#### 查看容器日志

```shell
docker logs -f -t 容器ID/容器名称
```





原文链接：https://juejin.cn/post/7154440096219922462







