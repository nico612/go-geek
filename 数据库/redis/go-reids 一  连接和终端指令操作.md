[TOC]



## Go语言标准库操作Redis数据库

**快速了解 Redis 数据库**

描述: Redis是一个开源的内存数据库, Redis提供了多种不同类型的数据结构，很多业务场景下的问题都可以很自然地映射到这些数据结构上。除此之外，通过复制、持久化和客户端分片等特性，我们可以很方便地将Redis扩展成一个能够包含数百GB数据、每秒处理上百万次请求的系统。

**Redis 支持的数据结构**

- 字符串（strings）
- 哈希（hashes）
- 列表（lists）
- 集合（sets）
- 带范围查询的排序集合（sorted sets）
- 位图（bitmaps）
- hyperloglogs 基数统计
- 带半径查询和流的地理空间索引等数据结构（geospatial indexes）

**Redis 应用场景**

- 高速缓存系统：减轻主数据库（MySQL）的压力 `set keyname`
- 计数场景：比如微博、抖音中的关注数和粉丝数 `incr keyname`
- 热门排行榜: 需要排序的场景特别适合使用 `ZSET`
- 实现消息队列的功能: 简单的队列操作使用list类型实现,L表示从左边(头部)开始插与弹出，R表示从右边(尾部)开始插与弹出,例如`"lpush / rpop" - (满足先进先出的队列模式)`和`"rpush / lpop" - (满足先进先出的队列模式)`。

### Redus 常用的操作命令





### Redis 环境准备

描述: 此处使用Docker快速启动一个redis环境，如有不会的朋友可以看我前面关于Docker文章或者百度。

以下是启动一个redis server，利用docker启动一个名为redis的容器,注意此处的版本为5.0.8、容器名和端口号请根据自己需要设置。

```shell
# -- Server
$ docker run --name redis -p 6379:6379 -d redis:5.0.8

# -- 查看运行的 redis 容器
$ docker ps | grep "redis"
24eb3c6f7bab  redis   "docker-entrypoint.s…"   19 months ago   Up 2 weeks  0.0.0.0:6379->6379/tcp   redis

# -- 查询redis容器资源使用状态 (扩展)
$ docker stats redis
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
24eb3c6f7bab        redis               0.10%               9.02MiB / 2.77GiB   0.32%               20.8MB / 126MB      33.5MB / 16.2MB     4

```

以下方法是启动一个 redis-cli 连接上面的 redis server

```shell
# -- Client 
docker run -it --network host --rm redis:5.0.8 redis-cli

# -- 交互式
$ docker exec -it redis bash
root@24eb3c6f7bab:/data# redis-cli
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth weiyigeek.top
OK
127.0.0.1:6379> ping
PONG

```

如果不是docker安装的执行 `redis-cli` 连接redis

```shell
~ redis-cli
127.0.0.1:6379> 
```

### Redis 客户端库安装

描述: 在web项目开发中redis数据库的使用也比较频繁，本节将介绍在Go语言中如何连接操作Redis数据库以及客户库的基本安装和使用。

Go 语言中常用的Redis Client库:

- redigo : https://github.com/gomodule/redigo
- go-redis : https://github.com/go-redis/redis

Tips: 此处我们采用go-redis来连接Redis数据库并进行一系列的操作,因为其支持连接哨兵及集群模式的Redis。

`redis`版本为7的安装v9版本

使用命令下载安装go-redis库: `go get -u github.com/go-redis/redis/v9`

### Redis 数据库连接

描述: 前面我们下载并安装了`go-redis`

第三方库, 下面我将分别进行单节点连接和集群连接演示, 并将其封装为package方便后续试验进行调用.

#### 1.Redis单节点连接

```go
package singleredis

import (
	"context"
	"fmt"
	"github.com/redis/go-redis/v9"
	"time"
)

// 简单连接 redis 示例

// Option 定义一个Option结构体
type Option struct {
	Addr         string
	WriteTimeout time.Duration
	ReadTimeout  time.Duration
	Password     string
	Database     int
}

// InitSingleRedis 结构体InitSingleRedis方法: 用于初始化redis数据库
func (o *Option) InitSingleRedis(ctx context.Context) (client *redis.Client, err error) {
	// Option

	// Redis 连接对象: NewClient将客户端返回到由选项指定的Redis服务器。
	client = redis.NewClient(&redis.Options{
		Addr:         o.Addr,     // redis服务ip:port
		Password:     o.Password, // redis的认证密码
		WriteTimeout: o.WriteTimeout,
		ReadTimeout:  o.ReadTimeout,
		DialTimeout:  2 * time.Second, // 连接超时时间
		PoolSize:     10,              // 连接池
	})
	fmt.Printf("Connecting Redis : %v\n", o.Addr)
	// 验证是否连接到redis服务端
	res, err := client.Ping(ctx).Result()
	if err != nil {
		fmt.Printf("Connect Failed! Err: %v\n", err)
		return nil, err
	}

	fmt.Printf("Connect Successful! Ping => %v\n", res)
	return client, nil

}

```

测试

```go
package main

import (
	"context"
	"fmt"
	"github.com/nico612/voyage/example/redis/single/singleredis"
	"github.com/redis/go-redis/v9"
	"time"
)

func main() {

	// 实例化
	opts := &singleredis.Option{
		Addr:         "127.0.0.1:6379",
		WriteTimeout: 1 * time.Second,
		ReadTimeout:  1 * time.Second,
		Password:     "",
		Database:     0,
	}
	ctx, cnacle := context.WithTimeout(context.Background(), 1*time.Second)
	defer cnacle()

	// 初始化连接 single redis 服务端
	client, err := opts.InitSingleRedis(ctx)
	if err != nil {
		panic(err)
	}

	// 测试
	V9Example(client)

	defer client.Close() // 关闭redis连接
}

func V9Example(client *redis.Client) {
	ctx := context.Background()

	// 设置Key, 0 表示不过期
	err := client.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		panic(err)
	}

	// 获取存在的Key
	val, err := client.Get(ctx, "key").Result()
	if err != nil {
		panic(err)
	}
	fmt.Printf("value = %s", val)

	// 获取不存在的Key
	val2, err := client.Get(ctx, "key2").Result()
	if err == redis.Nil {
		fmt.Println("key2 does not exist")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("key2", val2)
	}
}

```

执行结果

```shell
Connecting Redis : 127.0.0.1:6379
Connect Successful! Ping => PONG
value = value
key2 does not exist

```

#### 2.Redis哨兵模式连接

在 Redis 中，哨兵（Sentinel）是一个用于高可用性（High Availability）的工具，它是 Redis 的一个附属进程，专门用于监控主服务器和其对应的从服务器们。

哨兵模式的主要目标是确保 Redis 服务的高可用性，即使主服务器出现故障，也能够及时切换到备用的从服务器。这种自动故障转移的方式有助于减少服务中断时间，并且对于一些对服务稳定性要求较高的应用场景非常重要。

一般来说，哨兵模式解决了以下问题：

1. **主服务器故障切换：** 当 Redis 主服务器出现故障时，哨兵会自动检测到并进行切换，将其中一个从服务器升级为新的主服务器。
2. **监控与通知：** 哨兵负责监控 Redis 主服务器和从服务器的运行状况，一旦发现异常，会及时通知管理员或其他系统，以便进行相应的处理。
3. **配置管理：** 哨兵允许对 Redis 的配置进行动态调整，包括设置从服务器的优先级、故障转移的超时时间、以及故障切换的条件等。

哨兵模式通常用于对 Redis 服务的关键部署，如对某些数据的高可用性要求较高的生产环境，或者在进行 Redis 部署时，希望在主服务器发生故障时自动切换到备用服务器而不影响服务的连续性。

使用哨兵模式连接redis

```go
package failoverredis

import (
	"context"
	"github.com/redis/go-redis/v9"
	"time"
)

type Option struct {
	Master   string		// 主服务
	Addr     []string // redis 服务地址切片
	Password string
}

// InitSentinelClient 哨兵模式连接
func (r *Option) InitSentinelClient() (client *redis.Client, err error) {
	client = redis.NewFailoverClient(&redis.FailoverOptions{
		MasterName:    master, // 主服务器名
		SentinelAddrs: []string{"x.x.x.x:26379", "xx.xx.xx.xx:26379", "xxx.xxx.xxx.xxx:26379"},
	})

	c, cancle := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancle()

	_, err = client.Ping(c).Result()
	if err != nil {
		return nil, err
	}
	return client, nil
}

```

#### 3.Redis集群模式连接

```go
package clusterredis

import (
	"context"
	"github.com/redis/go-redis/v9"
	"time"
)


type Option struct {
	Addr []string
	Password string
}

// 结构体方法
func (o *Option) initClusterClient() (client *redis.ClusterClient, err error) {

	client = redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: o.Addr, //[]string{":7000", ":7001", ":7002", ":7003", ":7004", ":7005"},
	})

	c, cancle := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancle()

	_, err = client.Ping(c).Result()
	if err != nil {
		return nil, err
	}
	return client, nil
}

```

### Redis 数据类型指令操作实践

描述: 在使用`go-redis`
来操作redis前,我们可以通过redis-cli命令进入到交互式的命令行来执行相关命令并查看执行后相应的效果便于理解。

Step 1.首先我们连接到服务端.

```shell
# 连接到redis服务器 -a 指定密码，如果没有设置密码为空
# redis-cli -a xxxxx
redis-cli
# 验证连接状态
127.0.0.1:6379> ping  
PONG # 表示连接正常

```

Step 2. Redis 字符串数据类型的相关命令用于管理 redis 字符串值

```shell
127.0.0.1:6379> set myname "xiaowei" EX 60 # EX 过期时间默认秒, 
OK
127.0.0.1:6379> get myname
"xiaowei"
127.0.0.1:6379> get myname # 过期后返回nil
(nil)
```

Step 3. Redis hash 特别适合用于**存储对象**它是一个 string 类型的 field（字段） 和 value（值） 的映射表

设置语法：`HMSET key field value [field value ...]` 

获取语法：`HGET key field`

下面使用 key 为用户的 id 来储存对象

```shell
127.0.0.1:6379> HMSET userid name "xiaowei" age 13 hobby "Study GO!"
OK
127.0.0.1:6379> HGET userid name  # 指定键的指定字段值
"xiaowei"
127.0.0.1:6379> HGETALL userid    # 获取在哈希表中指定 key 的所有字段和值
1) "name"
2) "xiaowei"
3) "age"
4) "13"
5) "hobby"
6) "Study GO!"
127.0.0.1:6379> 
```

Step 4. Redis List 是简单的字符串列表，按照插入顺序排序。

`LPUSH key element [element...]`

```shell
127.0.0.1:6379> LPUSH mylpush one "C" tow "C#" there "Java" four "GO"  # 从左边推入
(integer) 8  # 储存后结构为 [GO, four, Java, there, C#, tow, C, one]
127.0.0.1:6379> LRANGE mylpush 0 7
1) "GO"
2) "four"
3) "Java"
4) "there"
5) "C#"
6) "tow"
7) "C"
8) "one"
127.0.0.1:6379> LINDEX mylpush 0  # 索引为0的数据
"GO"
127.0.0.1:6379> LPOP mylpush   # 从左边移除，也就是移除下标为0的数据
"GO"
127.0.0.1:6379> RPOP mylpush		# 移除最后一个元素
"one"
127.0.0.1:6379> RPUSH mylpush "末尾" # 从末尾添加数据
(integer) 7
127.0.0.1:6379> LINDEX mylpush -1  # 获取最后一个元素的
"\xe6\x9c\xab\xe5\xb0\xbe"
127.0.0.1:6379> 
```

Step 5. Set是 String 类型的无序集合且集合成员是唯一的

`SADD key member [member...]`

```shell
127.0.0.1:6379> SADD mysadds 1 redis 8 mongodb 3 mysql 4 oracle 5 db2
(integer) 10
127.0.0.1:6379> SMEMBERS mysadds # 返回集合中的所有成员
 1) "redis"
 2) "mongodb"
 3) "1"
 4) "4"
 5) "mysql"
 6) "3"
 7) "oracle"
 8) "8"
 9) "5"
10) "db2"
127.0.0.1:6379> SCARD mysadds	  # 获取集合的成员数
(integer) 10
127.0.0.1:6379> SPOP mysadds    # 随机移除成员
"mongodb"
127.0.0.1:6379> SPOP mysadds
"8"

```

Step 6. Redis 有序集合是 string 类型元素的集合且不允许重复的成员, 但是会通过分数来为集合中的成员进行从小到大的排序。

`ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]`

```shell
127.0.0.1:6379> ZADD mysets 100 "Go" 90 "Python" 80 "Ruby" 70 "C"
(integer) 4
127.0.0.1:6379> ZRANGE mysets 0 5 withscores  # 指定范围
1) "C"
2) "70"
3) "Ruby"
4) "80"
5) "Python"
6) "90"
7) "Go"
8) "100"
127.0.0.1:6379> ZRANGE mysets 0 -1   # 整个集合
1) "C"
2) "Ruby"
3) "Python"
4) "Go"
127.0.0.1:6379> ZRANGE mysets -1 3   # 获取分数最高的值-1代表倒数第一个。
1) "Go"
127.0.0.1:6379> ZRANGE mysets -2 2
1) "Python"
127.0.0.1:6379> ZRANGE mysets -3 1
1) "Ruby"
127.0.0.1:6379> ZRANGE mysets -4 0
1) "C"

```

Step 7. Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

```shell
127.0.0.1:6379> PFADD myhlkey "redis"  # 添加指定元素到 HyperLogLog 中。
(integer) 1
127.0.0.1:6379> PFADD myhlkey "memcache"
(integer) 1
127.0.0.1:6379> PFADD myhlkey "mysql"
(integer) 1
127.0.0.1:6379> PFADD myhlkey "redis"  
(integer) 0
127.0.0.1:6379> PFCOUNT myhlkey  # 返回给定 HyperLogLog 的基数估算值。
(integer) 3

```

Step 8. 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

```shell
# 第一个 redis-cli 客户端，在我们实例中我们创建了订阅频道名为 weiyigeekChat:
redis 127.0.0.1:6379> SUBSCRIBE weiyigeekChat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "weiyigeekChat"
3) (integer) 1

# 第二个 redis-cli 客户端,在同一个频道 weiyigeekChat 发布两次消息，订阅者就能接收到消息。
redis 127.0.0.1:6379> PUBLISH weiyigeekChat "Redis PUBLISH test"
(integer) 1
redis 127.0.0.1:6379> PUBLISH weiyigeekChat "Learn redis by weiyigeek.top"
(integer) 1

# 订阅者的客户端会显示如下消息
1) "subscribe"
2) "weiyigeekChat"
3) (integer) 1

1) "message"
2) "weiyigeekChat"
3) "Redis PUBLISH test"

1) "message"
2) "weiyigeekChat"
3) "Learn redis by weiyigeek.top"
```

Step 9.Redis 事务可以一次执行多个命令, 一个事务从开始到执行会经历以下三个阶段：

- 开始事务
- 命令入对
- 执行事务

```shell
redis 127.0.0.1:6379> MULTI  # 开启事务
OK

redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED
redis 127.0.0.1:6379> GET book-name
QUEUED
redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED
redis 127.0.0.1:6379> SMEMBERS tag
QUEUED

redis 127.0.0.1:6379> EXEC  # 执行所有事务块内的命令
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"

```

Step 10. Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作，该功能在 Redis 3.2 版本新增。

```shell
# 1.将一个或多个经度(longitude)|、纬度(latitude)-、位置名称(member)添加到指定的 key 中
# 重庆 经度:106.55 纬度:29.57
# 四川成都 经度:104.06	纬度:30.67
GEOADD cityAddr 106.55 29.57 ChongQing 104.06 30.67 SichuanChengDu

# 2.返回所有指定名称(member)的位置（经度和纬度），不存在的返回 nil。
GEOPOS cityAddr ChongQing SichuanChengDu NonExistKey
1) 1) "106.5499994158744812"
   2) "29.5700000136221135"
2) 1) "104.05999749898910522"
   2) "30.67000055930392222"
3) (nil)

# 3.用于返回两个给定位置之间的距离,此处计算重庆与程度的距离（m ：米，默认单位、km ：千米、mi ：英里、ft ：英尺）。
GEODIST cityAddr ChongQing SichuanChengDu km
"268.9827"

# 4.以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。
# WITHDIST: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。
# WITHCOORD: 将位置元素的经度和纬度也一并返回。
# WITHHASH: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。这个选项主要用于底层应用或者调试， 实际中的作用并不大。
127.0.0.1:6379> GEORADIUS cityAddr 105 29 200 km WITHDIST WITHCOORD
1) 1) "ChongQing"
   2) "163.1843"
   3) 1) "106.5499994158744812"
      2) "29.5700000136221135"

# 5.geohash 用于获取一个或多个位置元素的 geohash 值。
GEOHASH cityAddr ChongQing SichuanChengDu
127.0.0.1:6379> GEOHASH cityAddr ChongQing SichuanChengDu
1) "wm7b0x53dz0"
2) "wm3yrzq1tw0"

```

Step 11.Redis Stream 主要用于消息队列（MQ，Message Queue），Redis 本身是有一个 Redis 发布订阅 (pub/sub) 来实现消息队列的功能，但它有个缺点就是消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃，它是Redis 5.0 版本新增加的数据结构。

```shell
# 使用 XADD 向队列添加消息，如果指定的队列不存在，则创建一个队列
XADD mystreams * Name "WeiyiGeek" Age 25 Hobby "Computer"
"1640313258699-0"
XADD mystreams * Addr ChongQing
"1640313276946-0"

# 消息队列长度
127.0.0.1:6379> XLEN mystreams
(integer) 2

# 打印队列存储的字段与值( - 表示最小值 ,+ 表示最大值 )
127.0.0.1:6379> XRANGE mystreams - +
1) 1) "1640313258699-0"
   2) 1) "Name"
      2) "WeiyiGeek"
      3) "Age"
      4) "25"
      5) "Hobby"
      6) "Computer"
2) 1) "1640313276946-0"
   2) 1) "Addr"
      2) "ChongQing"

#  使用 XTRIM 对流进行修剪，限制长度， 语法格式：
127.0.0.1:6379> XTRIM mystreams MAXLEN 1
(integer) 1
127.0.0.1:6379> XRANGE mystreams - +
1) 1) "1640313276946-0"
   2) 1) "Addr"
      2) "ChongQing"

# 从 Stream 头部读取两条消息
127.0.0.1:6379> XADD mystreams * Name "WeiyiGeek" Age 25 Hobby "Computer"
127.0.0.1:6379> XREAD COUNT 2 STREAMS mystreams  writers 0-0 0-0
1) 1) "mystreams"
   2) 1) 1) "1640313276946-0"
         2) 1) "Addr"
            2) "ChongQing"
      2) 1) "1640313910204-0"
         2) 1) "Name"
            2) "WeiyiGeek"
            3) "Age"
            4) "25"
            5) "Hobby"
            6) "Computer"

# 从 Stream 头部读取1条消息
127.0.0.1:6379> XREAD COUNT 1 STREAMS mystreams  writers 0-0 0-0
1) 1) "mystreams"
   2) 1) 1) "1640313276946-0"
         2) 1) "Addr"
            2) "ChongQing"


# 使用 XGROUP CREATE 创建消费者组,此处从头部消费,如果想从尾部消费请将0-0改成$
127.0.0.1:6379> XGROUP CREATE mystreams consumer-group-name 0-0
OK

# 使用 XREADGROUP GROUP 读取消费组中的消息
# XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]
# group ：消费组名
# consumer ：消费者名。
# count ：读取数量。
# milliseconds ：阻塞毫秒数。
# key ：队列名。
# ID ：消息 ID。
127.0.0.1:6379> XREADGROUP GROUP consumer-group-name consumer-name COUNT 1 STREAMS mystreams >
1) 1) "mystreams"
   2) 1) 1) "1640313276946-0"
         2) 1) "Addr"
            2) "ChongQing"
127.0.0.1:6379> XREADGROUP GROUP consumer-group-name consumer-name COUNT 1 STREAMS mystreams >
1) 1) "mystreams"
   2) 1) 1) "1640313910204-0"
         2) 1) "Name"
            2) "WeiyiGeek"
            3) "Age"
            4) "25"
            5) "Hobby"
            6) "Computer"
127.0.0.1:6379> XREADGROUP GROUP consumer-group-name consumer-name COUNT 1 STREAMS mystreams >
(nil)

```









