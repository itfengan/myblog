---
title: HandlerThread
date: 2015-12-22 13:52:02
tags: 
- Android
categories: Android
---

**HandlerThread**

HandlerThread知识点整理

<!--more-->

 **官方介绍**

> Handy class for starting a new thread that has a looper. The looper can then be used to create handler classes. Note that start() must still be called.

意思就是HandlerThread能够新建拥有Looper的线程(除了主线程,我们新建线程是需要手动调用Looper.prepare来初始化looper和messagequeue的),而这个looper能够来新建其他的Handler(新建的这个handler是属于子线程的,并且looper和messagequeue都是初始化好了的)

**如以下代码：**

```java
mHandlerThread = new HandlerThread("check-message-coming");//the name of the new thread
        mHandlerThread.start();

        mThreadHandler = new Handler(mHandlerThread.getLooper())//拥有子线程looper的handler
        {
            @Override
            public void handleMessage(Message msg)
            {
                update();//模拟数据更新

                if (isUpdateInfo)
                    mThreadHandler.sendEmptyMessage(MSG_UPDATE_INFO);
            }
        };
        
	 @Override
	    protected void onDestroy() {
	        super.onDestroy();

	        //释放资源
	        myHandlerThread.quit() ;
	    }
//其他线程可以拿mThreadHandler发消息了，来完成向该HandlerThread线程通讯
```

mThreadHandler构建的时候,传的是HandlerThread的looper对象,也就是说这个mThreadHandler是属于子线程的管理的,他的handlerMessage的回调中是可以做耗时操作的(切记,是不能做更新UI的操作的,如需要更新,需要用主线程的handler发消息来更新,或者使用runOnUiThread或者eventBus等其他方式来刷新ui)

**退出循环**

Looper是通过调用loop方法驱动着消息循环的进行: 从MessageQueue中阻塞式地取出一个消息，然后让Handler处理该消息，周而复始，loop方法是个死循环方法。

那如何终止消息循环呢？我们可以调用Looper的quit方法或quitSafely方法，二者稍有不同。

```java
/**
     * Quits the looper.
     * <p>
     * Causes the {@link #loop} method to terminate without processing any
     * more messages in the message queue.
     * </p><p>
     * Any attempt to post messages to the queue after the looper is asked to quit will fail.
     * For example, the {@link Handler#sendMessage(Message)} method will return false.
     * </p><p class="note">
     * Using this method may be unsafe because some messages may not be delivered
     * before the looper terminates.  Consider using {@link #quitSafely} instead to ensure
     * that all pending work is completed in an orderly manner.
     * </p>
     *
     * @see #quitSafely
     */
    public void quit() {
        mQueue.quit(false);
    }

    /**
     * Quits the looper safely.
     * <p>
     * Causes the {@link #loop} method to terminate as soon as all remaining messages
     * in the message queue that are already due to be delivered have been handled.
     * However pending delayed messages with due times in the future will not be
     * delivered before the loop terminates.
     * </p><p>
     * Any attempt to post messages to the queue after the looper is asked to quit will fail.
     * For example, the {@link Handler#sendMessage(Message)} method will return false.
     * </p>
     */
    public void quitSafely() {
        mQueue.quit(true);
    }
```

**quit（）和quitSafety的区别**

当我们调用Looper的quit方法时，实际上执行了MessageQueue中的removeAllMessagesLocked方法，该方法的作用是把MessageQueue消息池中所有的消息全部清空，无论是延迟消息（延迟消息是指通过sendMessageDelayed或通过postDelayed等方法发送的需要延迟执行的消息）还是非延迟消息。

当我们调用Looper的quitSafely方法时，实际上执行了MessageQueue中的removeAllFutureMessagesLocked方法，通过名字就可以看出，该方法只会清空MessageQueue消息池中所有的延迟消息，并将消息池中所有的非延迟消息派发出去让Handler去处理，quitSafely相比于quit方法安全之处在于清空消息之前会派发所有的非延迟消息。

无论是调用了quit方法还是quitSafely方法只会，Looper就不再接收新的消息。即在调用了Looper的quit或quitSafely方法之后，消息循环就终结了，这时候再通过Handler调用sendMessage或post等方法发送消息时均返回false，表示消息没有成功放入消息队列MessageQueue中，因为消息队列已经退出了。

需要注意的是Looper的quit方法从API Level 1就存在了，但是Looper的quitSafely方法从API Level 18才添加进来。

**总结**

    1. HandlerThread继承自Thread,因此调用start方法,也是执行run方法,run()方法的逻辑都是在子线程中运行的。
    2. 查看HandlerThread源码可以看到,run()中主要做了Looper.prepare()和looper.loop()创建looper和messagequeue对象并开启消息队列的循环
    3. 需要注意的是,对于网络io操作,HandlerThread并不适合,因为它只有一个线程,得排队一个一个等着。
    4. 页面消耗的时候,调用 myHandlerThread.quit() ;looper就不在接受新的消息,消息循环结束,这个时候再通过handler调用sendMessage或者post等方法发送消息时均返回false,表示没有成功的放入消息队列。