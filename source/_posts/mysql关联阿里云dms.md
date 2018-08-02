---
title: mysql关联阿里云dms
date: 2018-06-14 12:09:47
tags: Dev
categories: Dev
password: 123456
---

将远程服务器中的mysql绑定到阿里云DMS中

<!--more-->

#### 阿里云DMS绑定mysql

![查看截图](https://img-blog.csdn.net/20180608174133433?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Zlbmdhbml0/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### **数据管理DMS登录云服务器的IP段（经典网络）：**

[DMS的ip段](https://help.aliyun.com/knowledge_detail/51251.html)

120.55.177.0/24

121.43.18.0/24

101.37.74.0/24

10.153.176.106/24

10.137.42.136/24

11.193.54.0/24

数据管理DMS登录云服务器的IP段（VPC）：

100.104.175.0/24

100.104.72.0/24

100.104.5.0/24

100.104.205.0/24

**添加白名单**
1，登录mysql：

```
mysql -h host -u username -p password
```

2，切换至mysql库：

```
use mysql;
```

3 查看当前允许登录IP及用户

```
select Host,User from user;
```

4 删除不必要而表中存在的IP和用户

```
DELETE FROM user WHERE User='username' and Host='host';
```

(host值为“%”或空表示所有IP都可登录，一般来说此类行需要删掉)

5 增加需要而表中没有的IP和用户

```
GRANT ALL PRIVILEGES ON *.* TO 'username'@'host' IDENTIFIED BY 'password' WITH GRANT OPTION;
 
```

6 使更新的配置生效

```
FLUSH PRIVILEGES;
```

#### 登录mysql失败问题

**错误 ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YE**S)

- vim /etc/my.cnf(注：windows下修改的是my.ini)

```
[mysqld]



sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

# 一般配置选项
basedir = /usr/local/mysql
datadir = /usr/local/mysql/data
port = 3306
socket = /tmp/mysql.sock
character-set-server=utf8
#在此添加【主意是在[mysqld]内部】
skip-grant-tables

back_log = 300
max_connections = 1000
max_connect_errors = 50
table_open_cache = 4096
max_allowed_packet = 32M
#binlog_cache_size = 4M
```

:wq 保存

- 接下来我们需要重启MySQL：

[各个平台重启mysql的方式](https://www.cnblogs.com/adolfmc/p/5497974.html)

```
service mysqld restart
```



- 重启之后输入#mysql即可进入mysql。


- 接下来就是用sql来修改root的密码

```
mysql> use mysql;
mysql> update user set password=password("你的新密码") where user="root";
mysql> flush privileges;
mysql> quit

到这里root账户就已经重置成新的密码了。
```

- 5.编辑my.cnf,去掉刚才添加的内容，然后重启MySQL。

###### 查看当前mysql用户表

host为% 代表任意域名下都可以访问

```
mysql> select Host,User from user;
+-----------------+-----------+
| Host            | User      |
+-----------------+-----------+
| %               | dms       |
| 120.55.177.0/24 | dms       |
| localhost       | mysql.sys |
| localhost       | root      |
+-----------------+-----------+
4 rows in set (0.00 sec)
```

