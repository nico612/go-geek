### docker 安装 Mysql



```shell
# 1、拉取mysql镜像，这里以mysql:5.7为例
docker pull mysql:5.7

# 2、后台运行mysql:5.6镜像
docker run -d -p 3306:3306 --name=new_mysql -v $(pwd)/conf:/etc/mysql/conf.d -v $(pwd)/logs:/logs -v $(pwd)/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7

#注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
# 解决办法：在挂载目录后面 多加一个--privileged=true参数即可

```

`-d`：表示后台运行容器
 `-p`：指定端口映射，第一个 3306 表示对外暴露的端口 (即：应用服务端口)，第二个 3306 表示 mysql 容器端口
 `--name`：指定容器名称
 `-v $(pwd)/conf:/etc/mysql/conf.d`：将宿主机当前目录下的 webapps 目录 映射到 mysql 容器的应用配置程序目录，注意：`/conf.d`是一个目录，不是文件
 `-v $(pwd)/logs:/logs`：将宿主机当前目录下的 logs 目录 映射到 mysql 容器的日志目录
 `-v $(pwd)/data:/var/lib/mysql`：将宿主机当前目录下的 data 目录 映射到 mysql 容器的数据存储目录
 `-e MYSQL_ROOT_PASSWORD=123456`：初始化 root 用户的密码

更多可参考 Docker hub mysql文档：https://hub.docker.com/_/mysql

### docker 安装 Redis

```shell
#1、拉取redis镜像，这里以redis:7为例
docker pull redis:7

#2、后台运行redis:7镜像
docker run -d -p 6379:6379 --name=myredis -v $(pwd)/data:/data -v $(pwd)/redis.conf:/usr/local/etc/redis/redis.conf redis:3.2 redis-server /usr/local/etc/redis/redis.conf --appendonly yes

#3、在$(pwd)/redis.conf目录下，创建一个redis.conf配置文件
#redis.conf配置文件下载  https://redis.io/docs/management/config/ 找到对应的版本，并下载redis.conf然后复制到自己的redis.conf配置目录中

#redis.conf配置文件需要修改四处地方：
#1、注释掉bind 127.0.0.1，即#bind 127.0.0.1，这里的bind指的是只有指定的网段才能远程访问这个redis，注释掉后，就没有这个限制了
#2、把protected-mode属性设置成no (默认是yes， 禁止了远程访问)
#3、把daemonize属性设置成yes (表明需要在后台运行)
#4、找到# requirepass foobared，删除前面的注释符号#，并把foobared修改成自己的密码  或者  另起一行 requirepass 自己的密码

#注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
#解决办法：在挂载目录后面 多加一个--privileged=true参数即可


```



### docker 安装 [RabbitMQ](https://link.juejin.cn/?target=https%3A%2F%2Fso.csdn.net%2Fso%2Fsearch%3Fq%3DRabbitMQ%26spm%3D1001.2101.3001.7020)

```shell
/**
 *拉取RabbitMQ镜像，这里以rabbitmq:3-management为例
 *拉取RabbitMQ镜像的时候，选择带有"management"版本的，不要选择latest版本的，因为带有"management"版本的才带有管理界面。
 **/
docker pull rabbitmq:3-management

# 2、后台运行rabbitmq:3-management镜像
docker run -d --name=rabbitmq -p 5672:5672 -p 15672:15672 --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3-management

或者

# 此方式的默认账号密码为：guest：guest，默认虚拟机为：/
docker run -d --name=rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management

```

### docker 安装 [Nginx](https://link.juejin.cn/?target=https%3A%2F%2Fso.csdn.net%2Fso%2Fsearch%3Fq%3DNginx%26spm%3D1001.2101.3001.7020)

```shell
//1、拉取nginx镜像，这里以nginx:latest为例
docker pull nginx

//2、后台运行redis:3.2镜像
docker run -d -p 80:80 --name=mynginx nginx

//注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
解决办法：在挂载目录后面 多加一个--privileged=true参数即可


```

`-d`：表示后台运行容器
`-p`：指定端口映射，第一个 80 表示对外暴露的端口 (即：应用服务端口)，第二个 80 表示 mysql 容器端口
`--name`：指定容器名称

### docker 安装etcd

官方文档：https://github.com/etcd-io/etcd/releases/

```shell
rm -rf /tmp/etcd-data.tmp && mkdir -p /tmp/etcd-data.tmp && \
  docker rmi gcr.io/etcd-development/etcd:v3.5.11 || true && \
  docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
  --name etcd-gcr-v3.5.11 \
  gcr.io/etcd-development/etcd:v3.5.11 \
  /usr/local/bin/etcd \
  --name s1 \
  --data-dir /etcd-data \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://0.0.0.0:2379 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-advertise-peer-urls http://0.0.0.0:2380 \
  --initial-cluster s1=http://0.0.0.0:2380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --log-level info \
  --logger zap \
  --log-outputs stderr

docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcd --version
docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcdctl version
docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcdutl version
docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcdctl endpoint health
docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcdctl put foo bar
docker exec etcd-gcr-v3.5.11 /usr/local/bin/etcdctl get foo
```





