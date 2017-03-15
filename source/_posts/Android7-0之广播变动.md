---
title: Android7.0之广播变动
date: 2017-01-23 17:45:15
tags: Android
---
#### Android7.0的后台优化 ####
Android中有一些系统的隐式广播,我们可以利用这些广播注册BroadCastReceiver来监听,比如手机网络变动(Wifi的时候自动下载更新包,发送错误日志),当这些广播到来的时候(网络从Wifi到移动数据来回切换的时候),后台会频繁的启动已经监听这些的应用,并且现在很多应用都会注册这些广播(如网络变化),那么就会带来大量的电量消耗,所以Android7.0中删除了三项隐式广播,又花了内存和电量的消耗

<!--more-->
#### 具体体现 ####
- Android7.0以上应用不会接受和发送以下三种广播
1. CONNECTIVITY_ACTION广播:网络状态改变
2. ACTION_NEW_PICTURE广播:一个新的相机,拍照和图片的添加
3. ACTION_NEW_VIDEO广播:一个新的视频摄像记录下来

#### 解决办法 ####
可以使用JobScheduler API(Android5.0提供),任务调度,可以使你再未来的某个时间点满足某个特定条件执行一个任务(当设备连接到Wifi,连通电源适配器的时候),具体用法以后再次整理.

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=28285910&auto=1&height=66"></iframe>