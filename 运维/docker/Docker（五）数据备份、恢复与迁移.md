[TOC]



## docker 数据的备份、恢复与迁移

    有时在 docker 中产生的数据，我们需要进行相应的备份和迁移到另外一台服务器上，并在另一台服务器上进行数据的恢复，那么改如何操作呢？如下将介绍三种方式进行数据的备份、恢复与迁移：

### 方式一：docker 容器的备份、恢复与迁移

#### docker 容器的备份 (导出)

```shell
docker export -o 容器导出文件(格式为tar压缩文件) 容器ID或容器名称
或
docker export 容器ID或容器名称 > 容器导出文件(格式为tar压缩文件) 

示例：
docker export -o $(pwd)/newtomcat.tar mytomcat
或
docker export mytomcat > $(pwd)/newtomcat.tar 

注释：
$(pwd)是docker支持的获取当前目录路径的方法，与linux的pwd类似
$(pwd)/newtomcat.tar 表示在当前目录下生成一个newtomcat.tar压缩文件

备注：
容器可以不启动进行备份操作


```

```shell
root@admin:~# docker run -d -p 8080:8080 --name mytomcat tomcat
root@admin:~# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                                  NAMES
b1cfc6fd9e0b   tomcat    "catalina.sh run"        About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp              mytomcat
013b39bd0366   mysql     "docker-entrypoint.s…"   20 hours ago         Up 20 hours         0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql
root@admin:~# docker export -o $(pwd)/newtomact.tar mytomcat
root@admin:~# ls
Dockerfile  hostVolume  newtomact.tar  snap
root@admin:~#
```

#### 2、docker 容器的迁移与恢复 (导入)

```shell
docker import 容器导出文件(格式为tar压缩文件) 新镜像名称[:版本号]
或
docker import /URL 新镜像名称[:版本号]

示例：
docker import $(pwd)/newtomcat.tar newtomcat:v1.0
或
docker import http://example.com/exampleimage.tgz example/imagerepo


```

```shell
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~# ls
Dockerfile  hostVolume  newtomact.tar  snap
root@admin:~# docker import $(pwd)/newtomact.tar newtomcat:v1.0
sha256:c8dcb1b186bd164fd9c313db359ebcbe9e6a74abe77e261881b01bba52b4d0f5
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newtomcat    v1.0      c8dcb1b186bd   6 seconds ago   662MB
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~#
```

### 方式二：docker 镜像的备份、恢复与迁移

#### 1、docker 镜像的备份 (导出)

```shell
docker save -o 镜像导出文件(格式为tar压缩文件) 镜像ID或镜像名称[:版本号]
或
docker save 镜像ID或镜像名称[:版本号] > 镜像导出文件(格式为tar压缩文件)

示例：
docker save -o $(pwd)/mytomcat.tar newtomcat:v1.0
或
docker save newtomcat:v1.0 > $(pwd)/mytomcat.tar 


```

将newtomcat:v1.0 镜像备份为mytocat.tar

```shell
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newtomcat    v1.0      c8dcb1b186bd   6 seconds ago   662MB
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~# ls
Dockerfile  hostVolume  newtomact.tar  snap
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newtomcat    v1.0      c8dcb1b186bd   2 minutes ago   662MB
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~# docker save -o $(pwd)/mytomcat.tar newtomcat:v1.0
root@admin:~# ls
Dockerfile  hostVolume  mytomcat.tar  newtomact.tar  snap
root@admin:~#
```

#### 2、docker 镜像的迁移与恢复 (导入)

```shell
docker load -i 镜像导出文件(格式为tar压缩文件)
或
docker load < 镜像导出文件(格式为tar压缩文件)

示例：
docker load -i $(pwd)/mytomcat.tar
或
docker load < $(pwd)/mytomcat.tar

```

```shell
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newtomcat    v1.0      c8dcb1b186bd   6 minutes ago   662MB
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~# docker rmi -f newtomcat:v1.0
Untagged: newtomcat:v1.0
Deleted: sha256:c8dcb1b186bd164fd9c313db359ebcbe9e6a74abe77e261881b01bba52b4d0f5
Deleted: sha256:742745511b86733da0d72624e18eea36c25e5cd0ea6a4ac0603c37d8967cdca0
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~# ls
Dockerfile  hostVolume  mytomcat.tar  newtomact.tar  snap
root@admin:~# docker load -i $(pwd)/mytomcat.tar
742745511b86: Loading layer [==================================================>]  673.8MB/673.8MB
Loaded image: newtomcat:v1.0
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newtomcat    v1.0      c8dcb1b186bd   7 minutes ago   662MB
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
tomcat       latest    b64abfdee99c   24 months ago   668MB
centos       latest    e6a0117ec169   2 years ago     272MB
root@admin:~#
```

注意：用户既可以使用 `docker load` 来导入`镜像存储文件`到本地镜像库，也可以使用 `docker import` 来导入一个`容器快照`到本地镜像库。这两者的区别在于`容器快照文件`将丢弃所有的`历史记录和元数据信息`（即仅保存容器当时的快照状态），而镜像存储文件将`保存完整记录，体积也要大`。此外，`容器快照文件`方式导入可以`重新指定镜像标签等元数据信息`。

### 方式三：docker 数据卷的备份、恢复与迁移

#### 1、docker 数据卷的备份 (导出)

```shell
#单个数据卷的备份
docker run --volumes-from 数据卷容器ID/数据卷容器名称 -v 宿主机备份目录:容器备份目录 镜像ID/镜像名称[:版本号] tar cvf 容器目录/数据卷压缩文件(格式为tar压缩文件) 容器数据卷文件/目录

#多个数据卷的备份
docker run --volumes-from 数据卷容器ID/数据卷容器名称 -v 宿主机备份目录:容器备份目录 镜像ID/镜像名称[:版本号] tar cvf 容器目录/数据卷压缩文件(格式为tar压缩文件) 容器数据卷文件1/目录1 容器数据卷文件2/目录2

# 示例：
#单个数据卷的备份
docker run --volumes-from mycentos -v $(pwd):/backup centos tar cvf /backup/newcentos.tar /containerVolume

# 多个数据卷的备份
docker run --volumes-from mycentos -v $(pwd):/backup centos tar cvf /backup/newcentos.tar /containerVolume1 /containerVolume2

# 如果想要在执行完备份指令后，删除临时容器，请在run 后面加上--rm属性，表示在执行完后立即删除该容器
docker run --rm --volumes-from mycentos -v $(pwd):/backup centos tar cvf /backup/newcentos.tar /containerVolume

```

注意：

1. 数据卷容器可以`不启动(即容器退出停止)`进行备份操作
2. 这里的`数据卷压缩文件的路径`要选择导出到`容器备份目录下`，即 如果`容器备份目录`为 / backup，则`数据卷压缩文件的路径`为`/backup/newcentos.tar`，因为`宿主机备份目录`与`容器备份目录`做了`数据卷挂载`，只有将`数据卷压缩文件的导出路径`选择在`容器备份目录`下，才能同步到`宿主机备份目录`下。

```shell
root@admin:~# docker run -it  --name=mycentos -v /root/hostVolume/:/containerVolume centos
[root@3a2021394ce5 /]# ls
bin  containerVolume  dev  etc	home  lib  lib64  lost+found  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
[root@62c8b59ea52d containerVolume]# ls
[root@62c8b59ea52d containerVolume]# echo "create a new file" > data.txt
[root@62c8b59ea52d containerVolume]# ls
data.txt
[root@62c8b59ea52d containerVolume]# cat data.txt
create a new file
[root@62c8b59ea52d containerVolume]# exit
exit
root@admin:~# docker run --volumes-from mycentos -v $(pwd):/backup centos tar cvf /backup/newcentos.tar /containerVolume
/containerVolume/
/containerVolume/data.txt
tar: Removing leading `/' from member names
root@admin:~# ls
Dockerfile  hostVolume  mytomcat.tar  newcentos.tar  snap
```

#### docker 数据卷的迁移与恢复 (导入)

```shell
# 单个数据卷 与 多个数据卷的 数据卷恢复指令相同
docker run --volumes-from 需要恢复数据的数据卷容器ID/名称 -v 宿主机备份目录:容器备份目录 镜像ID/镜像名称[:版本号] tar xvf 容器备份目录/数据卷压缩文件(格式为tar压缩文件)

# 示例：
docker run --volumes-from mycentos -v $(pwd):/backup centos tar xvf /backup/newcentos.tar

#如果想要在执行完数据恢复指令后，删除临时容器，请在run 后面加上--rm属性，表示在执行完后立即删除该容器
docker run --rm --volumes-from mycentos -v $(pwd):/backup centos tar xvf /backup/newcentos.tar

```

注意：

1. 需要恢复数据的数据卷容器可以`不启动(即容器退出停止)`进行数据恢复操作
2. 如果要把数据卷恢复到`新的容器`中，那么`新的容器`中的`数据卷`要与之前`备份的容器数据卷的路径和名称要一致`，(即：之前要`备份的容器数据卷`为`/data` 或者 `/var/containerVolume`，那么`新容器`的`数据卷`的路径和名称也要为`/data` 或者 `/var/containerVolume`)，例子：之前需要备份的容器与宿主机的挂载情况为`-v ~/hostVolume:/containerVolume`，那么需要备份的新容器与宿主机的挂载情况为：`-v ~/host:/containerVolume`，`即新容器数据卷的路径和名称 要与 备份的容器数据卷路径和名称必须一致，为/containerVolume`，宿主机的数据卷路径和名称可以不一致
3. 这里的`数据卷压缩文件`要选择`容器备份目录下`的，(即 如果`容器备份目录`为`/backup`，则`数据卷导出文件的路径`为`/backup/newcentos.tar`)

1、删除数据卷容器 mycentos 中的数据卷里的 data.txt 文件，模拟数据的丢失

```shell
root@admin:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
4ceb74873fbf   centos    "/bin/bash"   57 seconds ago   Exited (0) 26 seconds ago             mycentos
root@admin:~# docker start mycentos
mycentos
root@admin:~# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS         PORTS     NAMES
4ceb74873fbf   centos    "/bin/bash"   About a minute ago   Up 4 seconds             mycentos
root@admin:~# docker exec -it mycentos /bin/bash
[root@4ceb74873fbf /]# ls
bin  containerVolume  dev  etc	home  lib  lib64  lost+found  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
[root@4ceb74873fbf /]# cd ./containerVolume/
[root@4ceb74873fbf containerVolume]# ls
data.txt
[root@4ceb74873fbf containerVolume]# rm -rf data.txt
[root@4ceb74873fbf containerVolume]# ls
[root@4ceb74873fbf containerVolume]# exit
exit
```

2、对丢失数据的数据卷容器 mycentos 进行数据的恢复

```shell
root@admin:~# docker run --volumes-from mycentos -v $(pwd):/backup centos tar xvf /backup/newcentos.tar
containerVolume/
containerVolume/data.txt
root@admin:~# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
4ceb74873fbf   centos    "/bin/bash"   3 minutes ago   Up 2 minutes             mycentos
root@admin:~# docker exec -it mycentos /bin/bash
[root@4ceb74873fbf /]# ls
bin  containerVolume  dev  etc	home  lib  lib64  lost+found  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
[root@4ceb74873fbf /]# cd containerVolume/
[root@4ceb74873fbf containerVolume]# ls
data.txt
[root@4ceb74873fbf containerVolume]# cat data.txt
create a new file
[root@4ceb74873fbf containerVolume]#
```



















