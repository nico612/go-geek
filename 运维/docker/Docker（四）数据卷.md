[TOC]



## 容器数据卷

### 什么是容器数据卷

  docker 的理念将运行的环境打包形成容器运行，运行可以伴随容器，但是我们对数据的要求是希望持久化，容器之间可以共享数据，Docker 容器产生的数据，如果不通过 docker commit 生成新的镜像，使得数据作为容器的一部分保存下来，那么当容器被删除之后，数据也就没了，为了能够保存数据，在 docker 容器中使用卷。卷就是文件或者目录，存在于一个或者多个容器中，但是不属于联合文件系统，因此能够绕过 Union File System 提供一些用于持久化数据或共享数据的特点。

### 容器数据卷的作用与特性

  容器数据卷是一个特殊的文件或者目录，**它将主机文件或者目录直接映射进容器中**，可供一个或多个容器使用。容器数据卷设计的目的就是为了 数据的持久化，它完全独立与容器的生命周期。因此，**容器删除时，不会删除其挂载的数据卷**，也不会存在类似的垃圾机制对容器存在的数据卷进行处理。

**容器数据卷的特性：**

- 数据卷可以在容器之间进行数据共享和重用
- 对数据卷里的内容做更改不会影响镜像的更新
- 对数据卷里的内容做修改，能直接生效，无论是在容器内操作还是本地操作
- 数据卷的生命周期一直持续到没有容器使用它为止，即使挂载数据卷的容器已经被删除。
- 数据卷在容器启动时初始化，如果容器使用的镜像在挂载点包含了数据，这些数据会拷贝到新初始化的数据卷中

### 数据卷命令

数据卷的相关命令有：

| **命令**              | **说明**             | **文档地址**                                                 |
| :-------------------- | :------------------- | :----------------------------------------------------------- |
| docker volume create  | 创建数据卷           | [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/) |
| docker volume ls      | 查看所有数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_ls/) |
| docker volume rm      | 删除指定数据卷       | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_prune/) |
| docker volume inspect | 查看某个数据卷的详情 | [docs.docker.com](https://docs.docker.com/engine/reference/commandline/volume_inspect/) |
| docker volume prune   | 清除数据卷           | [docker volume prune](https://docs.docker.com/engine/reference/commandline/volume_prune/) |

注意：容器与数据卷的挂载要在创建容器时配置，对于创建好的容器，是不能设置数据卷的。而且**创建容器的过程中，数据卷会自动创建**。

### 容器数据卷的添加

#### 方式一：通过命令添加数据卷

##### 匿名目录挂载（不推荐）

匿名目录挂载只需要写容器内目录或者文件即可，而宿主机对应的目录会在`/var/lib/docker/volumes`路径下生成

```shell
#以交互模式运行容器，并使用-v 匿名挂载容器数据卷
docker run -it -v 容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

#以后台方式运行容器，并使用-v 匿名挂载容器数据卷 (推荐)
docker run -d -v 容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

#注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
# 解决办法：在挂载目录后面 多加一个--privileged=true参数即可

```

案例：使用 centos 容器内的 根目录 (/) 下的 centosVolume 目录 匿名挂载到宿主机中

1、创建并运行 centos 容器，同时匿名挂载数据卷

```shell
# 这里没有使用--name=容器名称 去指定容器名称，则docker随机一个容器名称
docker run -it -v /centosVolume centos
```

2、查看数据卷是否挂载成功

```shell
docker inspect 81b687fb8e88 (容器ID)
```

输出的挂载信息

```shell
 "Mounts": [
            {
                "Type": "volume",
                "Name": "f475af08b5669bfac425a9d0db468c4aa06a33a6716d5082bb8c5f145194030c",
                "Source": "/var/lib/docker/volumes/f475af08b5669bfac425a9d0db468c4aa06a33a6716d5082bb8c5f145194030c/_data",
                "Destination": "/centosValume",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
```

可以看到，centos 容器内的`/centosVolume`数据卷挂载到宿主机`/var/lib/docker/volumes`路径下的`/f475af08b5669bfac425a9d0db468c4aa06a33a6716d5082bb8c5f145194030c/_data`目录了

##### 具名目录挂载

具名目录挂载相对于匿名目录挂载，就是在宿主机生成对应的目录时可以指定该目录的名称，同样目录也会在`/var/lib/docker/volumes`路径下生成。例如：匿名目录挂载生成的对应宿主机目录为`0a4ea838c43ccc9af377af6d2e641b2c9fd4977c4a56408d76d31c8719b9dd8f`一串随机的数字，而具名目录挂载就是可以将这一串随机的数字改成指定的目录名称

```shell
# 以交互模式运行容器，并使用-v 具名挂载容器数据卷
docker run -it -v 宿主机目录名称:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

# 以后台方式运行容器，并使用-v 具名挂载容器数据卷 (推荐)
docker run -d -v 宿主机目录名称:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

# 注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
# 解决办法：在挂载目录后面 多加一个--privileged=true参数即可

```

案例：使用 centos 容器内的 根目录 (/) 下的 centosVolume 目录 匿名挂载到宿主机中

1、创建并运行 centos 容器，同时匿名挂载数据卷

```shell
# 这里没有使用--name=容器名称 去指定容器名称，则docker随机一个容器名称
docker run -d -v hostVolume:/myVolume centos
```

2、查看数据卷是否挂载成功

```shell
docker ps -a
docker inspect 8961c0a39ce8 (容器ID)
```

挂载信息

```json
"Mounts": [
            {
                "Type": "volume",
                "Name": "hostVolume",
                "Source": "/var/lib/docker/volumes/hostVolume/_data",
                "Destination": "/myVolume",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```

可以看到，centos 容器内的`/myVolume`数据卷挂载到 宿主机的`/var/lib/docker/volumes/hostVolume/_data`目录了，这时的宿主机目录名称不在是一串随机的数字了，而是我们指定的目录名称`hostVolume`。

##### 指定目录挂载

指定目录挂载就是我们可以将容器内部的数据卷指定挂载到宿主机的某一文件或者目录下

```shell
#以交互模式运行容器，并使用-v 挂载容器数据卷
docker run -it -v 宿主机目录/文件的绝对路径:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

#以后台方式运行容器，并使用-v 挂载容器数据卷 (推荐)
docker run -d -v 宿主机目录/文件的绝对路径:容器内目录/文件的绝对路径[:rw/ro] -p 主机端口:容器端口 --name=容器名称 镜像ID/镜像名称[:版本号]

#注意：如果出现Docker挂载宿主机目录显示cannot open directory .:Permission denied
#解决办法：在挂载目录后面 多加一个--privileged=true参数即可

```

案例：在 Host 宿主机的 /root 目录下使用 hostVolume 目录 挂载数据卷到 centos 容器内的 根目录 (/) 下的 containerVolume 目录

1、创建并运行 centos 容器，同时挂载数据卷

```shell
# 这里没有使用--name=容器名称 去指定容器名称，则docker随机一个容器名称
root@admin:/home/ad# docker run -it -v /root/hostVolume:/containerVolume centos
[root@0ed42d7e67a9 /]# ls
bin  containerVolume  dev  etc	home  lib  lib64  lost+found  media  mnt  opt  proc  root  run	sbin  srv  sys	tmp  usr  var
[root@0ed42d7e67a9 /]#
```

可以看到，此时由 root@localhost 转变成为 root@13243e227243 (容器 ID)，表示我们已经由宿主机进入到 centos 容器中。然后执行 ls查看目录可以看到`centos容器`的`根目录`下多了一个 containerVolume 目录，`宿主机`的 `/root目录`下多了一个 hostVolume 目录

同时还可以在`宿主机`中使用指令`docker inspect 容器ID/容器名称`来查看是否挂载成功

```json
"Mounts": [
            {
                "Type": "bind",
                "Source": "/root/hostVolume",
                "Destination": "/containerVolume",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
```

2、容器与宿主机之间进行通信

如果在宿主机中添加文件等操作，会同步到容器中去，就算关闭容器后，更改宿主机 hostVolume 目录中的 文件，容器再次启动后数据仍然同步

#### 方式二：通过 DockerFile 添加数据卷

案例：在 Host 宿主机的 /root 目录下创建一个 DockerFile 文件 (`名称随意`)，并通过 docker build 指令生成镜像来添加数据卷

1、在 Host 宿主机的 /root 目录下创建一个 DockerFile 文件，并添加如下内容到文件中

```shell
#基于centos镜像进行构建
FROM centos

#数据卷只能指定容器数据卷，不能指定宿主机数据卷，因为并不能够保证在所有的宿主机上都存在这样的特定目录。
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]

#以 /bin/bash方式启动
CMD /bin/bash

```

2、使用如下指令，把编写的 DockerFile 文件执行生成镜像，`注意：命令最后面是空格 + .`

```
docker build -f 宿主机中DockerFile文件的绝对路径 -t 新镜像名称[:版本号] .
```

命令如下

```shell
root@admin:~# docker build -f /root/Dockerfile -t new-centos .
[+] Building 0.0s (5/5) FINISHED                                                                           docker:default
 => [internal] load build definition from Dockerfile                                                                 0.0s
 => => transferring dockerfile: 339B                                                                                 0.0s
 => [internal] load .dockerignore                                                                                    0.0s
 => => transferring context: 2B                                                                                      0.0s
 => [internal] load metadata for docker.io/library/centos:latest                                                     0.0s
 => [1/1] FROM docker.io/library/centos                                                                              0.0s
 => exporting to image                                                                                               0.0s
 => => exporting layers                                                                                              0.0s
 => => writing image sha256:0b8c6230027543a5eaf1bd6ae8ea0d689bf6b9bacd581dcde5e02c7f294e3f2c                         0.0s
 => => naming to docker.io/library/new-centos                                                                        0.0s
root@admin:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
mysql        latest    9c61872d4987   7 weeks ago     628MB
nginx        latest    eeb9db34b331   23 months ago   134MB
centos       latest    e6a0117ec169   2 years ago     272MB
new-centos   latest    0b8c62300275   2 years ago     272MB
```

3、运行我们生成的 new-centos 镜像，就能够查看到在容器内中生成的数据卷

```shell
root@admin:~# docker run -it new-centos
[root@8e823028bb3b /]# ls
bin		      dataVolumeContainer2  etc   lib	 lost+found  mnt  proc	run   srv  tmp	var
dataVolumeContainer1  dev		    home  lib64  media	     opt  root	sbin  sys  usr
```

4、那么容器内的数据卷文件 / 目录地址已经知道，对应的宿主机文件 / 目录的地址怎么查看？通过如下指令

```shell
[root@8e823028bb3b /]# exit
exit
root@admin:~# docker inspect 8e823028bb3b
# ... 省略部分信息
"Mounts": [
            {
                "Type": "volume",
                "Name": "a97db0f944ee8cc0fd094c5f4a0fbdae6164adbf45fbc3eaa66bf383afe2fea5",
                "Source": "/var/lib/docker/volumes/a97db0f944ee8cc0fd094c5f4a0fbdae6164adbf45fbc3eaa66bf383afe2fea5/_data",
                "Destination": "/dataVolumeContainer1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "e1d792ba4efc567740abe433ce93da792130959a2d470744d01ec12fb08fe799",
                "Source": "/var/lib/docker/volumes/e1d792ba4efc567740abe433ce93da792130959a2d470744d01ec12fb08fe799/_data",
                "Destination": "/dataVolumeContainer2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
  # ... 省略部分信息
```

## 数据卷容器

### 什么是数据卷容器

  命名的容器`已挂载数据卷`，其他的容器通过挂载这个容器 (父容器) 实现数据共享，挂载数据卷的容器，称为数据卷容器。通过数据卷容器可以实现容器间的数据共享。

### 添加数据卷容器

```shell
docker run -it/-d  -p 主机端口:容器端口 --name=容器名称 --volumes-from 数据卷容器ID/数据卷容器名称 生成数据卷容器的镜像ID/镜像名称[:版本号]

```

案例：创建父容器，并在父容器的数据卷中添加数据，以挂载父容器生成子容器，实现数据共享

1、启动父容器，并在父容器的 dataVolumeContainer1 目录下新增内容

```shell
root@admin:~# docker run -it --name=father-centos new-centos
[root@5744f96ba362 /]# ls
bin  dataVolumeContainer1  dataVolumeContainer2  dev  etc  home  lib  lib64  lost+found  media	mnt  opt  proc	root  run  sbin  srv  sys  tmp	usr  var
[root@5744f96ba362 /]# cd dataVolumeContainer1
[root@5744f96ba362 dataVolumeContainer1]# echo "add a new line" > data.txt
[root@5744f96ba362 dataVolumeContainer1]# ls
data.txt
[root@5744f96ba362 dataVolumeContainer1]# cat data.txt
add a new line
[root@5744f96ba362 dataVolumeContainer1]#
```

2、基于父容器生成子容器 son-centos，注意是在`宿主机中`使用如下指令

```shell
[root@5744f96ba362 dataVolumeContainer1]# exit
exit
root@admin:~# docker run -it --name=son-centos --volumes-from father-centos new-centos
[root@781a6d62f07c /]# cd dataVolumeContainer1/
[root@781a6d62f07c dataVolumeContainer1]# ls
data.txt
[root@781a6d62f07c dataVolumeContainer1]# cat data.txt
add a new line
[root@781a6d62f07c dataVolumeContainer1]#
```

3、子容器添加数据，父容器查看数据

```shell
[root@781a6d62f07c dataVolumeContainer1]# echo "add a other line" >> data.txt
[root@781a6d62f07c dataVolumeContainer1]# cat data.txt
add a new line
add a other line
[root@781a6d62f07c dataVolumeContainer1]# exit
exit
root@admin:~# docker ps -a
CONTAINER ID   IMAGE        COMMAND                  CREATED              STATUS                          PORTS                                                  NAMES
781a6d62f07c   new-centos   "/bin/sh -c /bin/bash"   About a minute ago   Exited (0) 13 seconds ago                                                              son-centos
5744f96ba362   new-centos   "/bin/sh -c /bin/bash"   3 minutes ago        Exited (0) About a minute ago                                                          father-centos

# root@admin:~# docker attach father-centos 这个命令必须是容器正在运行的状态，而我这里容器已经停止了，所以使用 docker start -i 781a6d62f07c 重新运行容器，并进入交互式
root@admin:~# docker start -i 781a6d62f07c
[root@781a6d62f07c dataVolumeContainer1]# cat data.txt
add a new line
add a other line
```

4、删除父容器，子容器数据依然保留

```shell
root@admin:~# docker rm -f father-centos
father-centos
root@admin:~# docker start -i son-centos
[root@781a6d62f07c /]# cd dataVolumeContainer1
[root@781a6d62f07c dataVolumeContainer1]# ls
data.txt
[root@781a6d62f07c dataVolumeContainer1]# cat data.txt
add a new line
add a other line
```

