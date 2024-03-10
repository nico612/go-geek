## MySQL的使用步骤

登录，连接MySQL服务器

```solidity
mysql -uroot -p密码
```

远程连接MySQL

```
mysql -h your_mysql_server_ip -u your_username -p
```

带端口

```
mysql -h your_mysql_server_ip -P your_port_number -u your_username -p
```

## SQL语句

SQ语句主要分为4大类

- DDL（Data Definition Language）：数据定义语言，创建（CREATE）、修改（UPDATE）、删除（DELETE） 数据库\表
- DQL（Data Query Language）：数据查询语言，查询记录（SELECT）
- DML（Data Manipulation Language）：数据操纵语言，增加（INSERT）、删除（DELETE）、修改（UPDATE） 记录
- DCL（Data Control Language ）：数据控制语言、控制访问权限（GRANT、REVOKE）

### 数据库

#### 创建

- `CREATE DATABASES 数据库名` 创建数据库（使用默认字符编码，MySQL 5.5.3及更高版本的默认字符集为`utf8mb4`，排序规则为`utf8mb4_general_ci`。）
- `CREATE DATABASE 数据库名 CHARACTER SET 字符编码`
- `CREATE DATABASE IF NOT EXISTS 数据库名`

#### 查询

### 表



select * from table_name where condition

update table_name set column1=value1, column2=value2 where condition

insert into table_name (column1, coumn2,,,) values (value1, value2)

delete from table_name where condition