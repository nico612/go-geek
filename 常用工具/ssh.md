

## 使用ssh上传和下载文件

### 从本地上传文件到服务器

重新开启一个终端窗口，通过以下指令实现本地与服务器的文件传输。

```shell
scp 本地文件路径 你的服务器登录用户名@你的服务器ip地址:服务器中的存储目录
# 如：scp /Users/hhh/Desktop/do.txt root@100.200.200.100:C:/Users/Administrator/UsersAdministrator/

```

加一个-r可以实现传输文件夹：

```shell
scp -r /Users/hhh/Desktop/FRC Administrator@100.200.200.100:C:/Users/Administrator/UsersAdministrator/

```

### 从服务器将文件传输到本地

```shell
scp 你的服务器登录用户名@你的服务器ip地址:服务器中的存储目录 本地文件目录
# 如：scp Administrator@100.200.200.100:C:/Users/Administrator/UsersAdministrator/do.txt  /Users/hhh/Desktop/
```

加一个-r可以实现传输文件夹：

```shell
scp -r Administrator@100.200.200.100:C:/Users/Administrator/UsersAdministrator/  /Users/hhh/Desktop/
```