---
title: AsyncTask的弊端
date: 2016-12-20 10:42:03
tags: 
- Android
categories: Android
---

Android开发中，AsyncTask可以替代Handler和Thread来处理后台操作和通知Ui刷新，适用于处理异步数据，并将更新Ui的场景，AsyncTask适用于后台操作只有几秒的短时操作。但是AsyncTask本身存在很多糟糕的问题，如果使用中不注意，将会影响程序的健壮性。

<!--more-->

## AsyncTask和Handler对比

**注意：按照Android官方文档支出,AsyncTask被推荐为处理短时间(10秒以内)的操作,即本地的轻量IO操作.不适合使用网络这样时间不定的操作.**

### 共同点

都是为了解决耗时操作的问题，主要区别在于一个流程完善，拿来就用（AsyncTask），一个偏向自主定制，扩展性高（Handler+Thread）。

- **AsyncTask比Handler轻量级，对吗？**

1. 通过看源码，发现AsyncTask实际上就是一个线程池，而网上的说法是AsyncTask比handler要轻量级，显然上不准确的，只能这样说，AsyncTask在代码上比handler要轻量级别，而实际上要比handler更耗资源，因为AsyncTask底层是一个线程池！而Handler仅仅就是发送了一个消息队列，连线程都没有开。
2. 但是，如果异步任务的数据特别庞大，AsyncTask这种线程池结构的优势就体现出来了。

- **AsyncTask实现的原理,和适用的优缺点**

AsyncTask,是android提供的轻量级的异步类,可以直接继承AsyncTask,在类中实现异步操作,并提供接口反馈当前异步执行的程度(可以通过接口实现UI进度更新),最后反馈执行的结果给UI主线程.

使用的优点:

l  简单,快捷

l  过程可控

使用的缺点:

l 在使用多个异步操作的同时，共同进行Ui变更时,就变得复杂起来.

l 最大并发数不超过5

-  **Handler异步实现的原理和适用的优缺点**

在Handler 异步实现时,涉及到 Handler, Looper, Message,Thread四个对象，实现异步的流程是主线程启动Thread（子线程）àthread(子线程)运行并生成Message-àLooper获取Message并传递给HandleràHandler逐个获取Looper中的Message，并进行UI变更。

**使用的优点：**

l  结构清晰，功能定义明确

l  对于多个后台任务时，简单，清晰

**使用的缺点：**

l  在单个后台异步处理时，显得代码过多，结构过于复杂（相对性）

## AsyncTask的使用：

- **必选方法：**

1，  doinbackground(params…) 后台执行，比较耗时的操作都可以放在这里。

注意这里不能直接操作UI。此方法在后台线程执行，完成任务的主要工作

，通常需要较长的时间。在执行过程中可以调用

Public progress(progress…)来更新任务的进度。

 

2，  onpostexecute(result)相当于handler处理UI的方式，在这里可以使用在

doinbackground得到的结果处理操作UI。此方法在主线程执行，任务执行的结果作为此方法的参数返回。

 

- **可选方法：**

1，  onprogressupdate(progress…) 可以使用进度条增加用户体验度。此方法在主线程执行，用户显示任务执行的进度。

2，  onpreExecute()  这里是最新用户调用excute时的接口，当任务执行之前开始调用此方法，可以在这里显示进度对话框。

3，  onCancelled()  用户调用取消时，要做的操作

- **AsyncTask三个状态：**

pending,running,finished

- **使用AsyncTask类，遵守的准则：**

1，  Task的实例必须在UI thread中创建；

2，  Execute方法必须在UI thread中调用

3，  不要手动的调用onPfreexecute()，onPostExecute(result)

Doinbackground(params…),onProgressupdate(progress…)这几个方法；

4，  该task只能被执行一次，否则多次调用时将会出现异常;

## AsyncTask缺陷总结：

- **生命周期**

  很多开发者会认为在一个Activity中创建的AsyncTask会随着Activity销毁而销毁，事实并非如此，AsyncTask会随着doInBackground（）方法执行完毕才销毁，然后，cancel（）被调用，那么onCancel会执行；否则执行postExecute方法会执行。如果在AsyncTask没有执行完毕，就销毁了Activity，AsyncTask可能会崩溃，因为它想要处理的view已经不存在了。所以，我们总是必须确保在销毁活动之前取消任务。总之，我们使用AsyncTask需要确保AsyncTask正确地取消。

  另外，即使我们正确地调用了cancle() 也未必能真正地取消任务。因为如果在doInBackgroud里有一个不可中断的操作，比如BitmapFactory.decodeStream()，那么这个操作会继续下去。

- **内存泄漏**

  如果AsyncTask未声明成静态，则会持有外部类Activity的引用，当Activity销毁之后，AsyncTask还在执行，它将在内存中依旧保持这个引用，会造成内存泄漏

- **结果丢失**

  当屏幕旋转Activity销毁重新创建（未配置android:configChanges="orientation|screenSize"的情况）之前运行的AsyncTask会持有之前Activity的引用，这时调用onPostExecute()再去更新界面将不再生效。

- **并行还是串行**

  当想要串行执行时，直接执行execute()方法，如果需要并行执行，则要执行executeOnExecutor(Executor)。

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=29761059&auto=1&height=66"></iframe>