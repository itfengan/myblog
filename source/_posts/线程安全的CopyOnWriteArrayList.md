---
title: 线程安全的CopyOnWriteArrayList
date: 2017-02-16 00:28:47
tags: 
- Android
categories: notes
password: 123456
---

**前言**

最近翻阅EventBus源码，在subscribe方法中，看到了CopyOnWriteArrayList这个类，一时竟不知其原理是什么，本文简要记录一下CopyOnWriteArrayList的相关介绍

<!--more-->

### 先看一段EventBus的subscribe（）的代码

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/vj1ty.png)

这段代码是EventBus中subscribe方法，里面涉及到了subscriptionsByEventType的遍历过程中按照优先级排序

关于EventBus的具体源码我将在后面会整理，这里我们只关注CopyOnWriteArrayList即可

可以看到，在遍历的过程中，调用了`subscriptions.add(i,newSubscription)`来按照优先级进行排序，我相信任何一个懂java的都知道如果使用普通Arraylist，都会报错（并发修改异常），那么我们下面来证实一下

### 不使用CopyOnWriteArrayList

先写一段代码证明`CopyOnWriteArrayList`确实是线程安全的。

> ReadThread.java

```java
import java.util.List;

public class ReadThread implements Runnable {
    private List<Integer> list;

    public ReadThread(List<Integer> list) {
        this.list = list;
    }

    @Override
    public void run() {
        for (Integer ele : list) {
            System.out.println("ReadThread:"+ele);
        }
    }
}
```

> WriteThread.java

```java
import java.util.List;

public class WriteThread implements Runnable {
    private List<Integer> list;

    public WriteThread(List<Integer> list) {
        this.list = list;
    }

    @Override
    public void run() {
        this.list.add(9);
    }
}
```

> TestCopyOnWriteArrayList.java

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestCopyOnWriteArrayList {

    private void test() {
        //1、初始化CopyOnWriteArrayList
        List<Integer> tempList = Arrays.asList(new Integer [] {1,2});
        CopyOnWriteArrayList<Integer> copyList = new CopyOnWriteArrayList<>(tempList);


        //2、模拟多线程对list进行读和写
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        executorService.execute(new ReadThread(copyList));
        executorService.execute(new WriteThread(copyList));
        executorService.execute(new WriteThread(copyList));
        executorService.execute(new WriteThread(copyList));
        executorService.execute(new ReadThread(copyList));
        executorService.execute(new WriteThread(copyList));
        executorService.execute(new ReadThread(copyList));
        executorService.execute(new WriteThread(copyList));

        System.out.println("copyList size:"+copyList.size());
    }


    public static void main(String[] args) {
        new TestCopyOnWriteArrayList().test();
    }
}
```

运行上面的代码，没有报出

java.util.ConcurrentModificationException

说明了CopyOnWriteArrayList并发多线程的环境下，仍然能很好的工作。

### CopyOnWriteArrayList如何做到线程安全

`CopyOnWriteArrayList`使用了一种叫**写时复制**的方法，当有新元素添加到`CopyOnWriteArrayList`时，先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。

当有新元素加入的时候，如下图，创建新数组，并往新数组中加入一个新元素,这个时候，array这个引用仍然是指向原数组的。

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/7cbd4.png)

当元素在新数组添加成功后，将array这个引用指向新数组。

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/bmcdz.png)





### CopyOnWriteArrayList的源码

```java
public boolean add(E e) {
    //1、先加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        //2、拷贝数组
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //3、将元素加入到新数组中
        newElements[len] = e;
        //4、将array引用指向到新数组
        setArray(newElements);
        return true;
    } finally {
       //5、解锁
        lock.unlock();
    }
}
```

由于所有的写操作都是在新数组进行的，这个时候如果有线程并发的写，则通过锁来控制，如果有线程并发的读，则分几种情况： 
1、如果写操作未完成，那么直接读取原数组的数据； 
2、如果写操作完成，但是引用还未指向新数组，那么也是读取原数组数据； 
3、如果写操作完成，并且引用已经指向了新的数组，那么直接从新数组中读取数据。

可见，`CopyOnWriteArrayList`的**读操作**是可以不用**加锁**的。

### CopyOnWriteArrayList使用场景

通过上面的分析，`CopyOnWriteArrayList` 有几个缺点： 
1、由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致`young gc`或者`full gc`

2、不能用于**实时读**的场景，像拷贝数组、新增元素都需要时间，所以调用一个`set`操作后，读取到数据可能还是旧的,虽然`CopyOnWriteArrayList` 能做到**最终一致性**,但是还是没法满足实时性要求；

`CopyOnWriteArrayList` 合适**读多写少**的场景，不过这类慎用 
因为谁也没法保证`CopyOnWriteArrayList` 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

### 思想

如上面的分析CopyOnWriteArrayList表达的一些思想： 
1、读写分离，读和写分开 
2、最终一致性 
3、使用另外开辟空间的思路，来解决并发冲突

















参考文章：

[1](http://blog.csdn.net/linsongbin1/article/details/54581787)