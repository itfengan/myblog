---
title: git
date: 2016-03-22 14:53:19
tags: 
-dev
categories: dev
password: 123456
typora-copy-images-to: ipic
---

> 前言

#### 记录一下常用的Git命令

<!--more-->

###### 初始化相关

**设置Git的user name和email**：

```
$ git config --global user.name "fengan"
$ git config --global user.email "fengan1102@gmail.com"
```

**SSH密钥**

```
1.查看是否已经有了ssh密钥：cd ~/.ssh
如果没有密钥则不会有此文件夹，有则备份删除
2.生存密钥：
$ ssh-keygen -t rsa -C “haiyan.xu.vip@gmail.com”
按3个回车，密码为空。
公钥添加到代码仓库
```

**测试链接**

```
ssh git@github.com
```

**查看配置信息**

```
git config --global  --list
```

###### 更多

**基本命令使用SourceTree即可**

**将远程最近四次提交合并为一个**

```
git rebase -i HEAD~4
然后根据提示操作
先esc
ZZ                        退出vi并保存
:q!                       退出vi，不保存
:wq                       退出vi并保存
更多Vi命令（https://www.cnblogs.com/my_life/articles/2125007.html）
然后解决冲突
然后强推 sourceTree也可以强推，但是要手动打开（https://www.phpsong.com/3262.html）
```

**将远程最后一次提交删除**

```
commit 3
commit 2
commit 1
其中最后一次提交commit 3是错误的，那么可以执行：
git reset --hard HEAD~1
你会发现，HEAD is now at commit 2
然后再使用git push --force将本次变更强行推送至服务器。这样在服务器上的最后一次错误提交也彻底消失了。

值得注意的是，这类操作比较比较危险，例如：在你的commit 3之后别人又提交了新的commit 4，那在你强制推送之后，那位仁兄的commit 4也跟着一起消失了。

```

