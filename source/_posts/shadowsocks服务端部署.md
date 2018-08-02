---
title: shadowsocks服务端部署
date: 2018-07-18 13:05:35
tags:
- Dev
categories: Dev
password:
---

基于Ubuntu 16.04的Shadowsocks服务端安装

<!--more-->

### 安装步骤

##### 1，首先检查下Python 版本，要有 2.6 or 2.7.

```
python --version
Python 2.7.4
```

##### 2，安装python包管理器

```
apt-get install python-pip
```

##### 3，安装shadowsocks

```
pip install shadowsocks
```

##### 4，修改配置文件

```
vim /etc/shadowsocks/config.json
```

```
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"yourpsw",
    "timeout":300,
    "method":"bf-cfb",
    "fast_open": true,
    "workers": 10
}
注意：
server 0，0，0，0（其他可能出问题）
method bf-cfb（其他可能出问题）
```

##### 5，开启

```
sudo ssserver -c /etc/shadowsocks/config.json
```

##### 6，设置为开机启动

- ###### 编辑

```
vim /etc/rc.local
```



- ###### 然后在exit 0之前加入 

  ```
   sudo ssserver -c /etc/shadowsocks/config.json
  ```

- ###### 重启

```
  ● reboot
```

- ###### 重启后查看进程

```
ps aux | grep shadowsocks
```

### 参考：

[参考1](https://blog.csdn.net/sjtu_mc/article/details/79207427)

[参考2](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)

[参考3](https://hceasy.com/2013/12/shadowsocks-%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%83%A8%E7%BD%B2/)

