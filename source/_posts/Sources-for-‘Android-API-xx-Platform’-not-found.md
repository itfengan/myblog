---
title: Sources for ‘Android API xx Platform’ not found
date: 2017-03-13 16:38:47
tags: 
- Android
categories: others
password: 123456
---

**前言**

在Android Studio翻源码多时候，明明下载了对应的SDK，却点进去都是.class看不到源码，本文记录一下解决办法

<!--more-->

**首先确保下载了对应的SDK**

- 找到以下路径，并打开文件

~/Library/Preferences/AndroidStudioXXX/options/jdk.table.xml

- 修改前

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/w3eyx.png)

- 修改后

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/ehg3w.png)

- 重启AS

