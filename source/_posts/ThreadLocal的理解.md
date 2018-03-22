---
title: ThreadLocal的理解
date: 2018-03-02 14:02:18
tags: 
- Android
categories: notes
password: 123456
---

### 什么是ThreadLocal

JDK1.2提供

- 根据JDK文档中的解释：

ThreadLocal的作用是提供线程内的局部变量，这种变量在多线程环境下访问时能够保证各个线程里变量的独立性。

当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立改变该变量的副本，而不会影响其他线程所对应的副本。

<!--more-->

### ThreadLocal的使用：

![](https://ws4.sinaimg.cn/large/006tNc79gy1foyhets3pvj31kw15p4ac.jpg)

可以看到我们用ThreadLocal存放了一个String字符串，在不同的线程set数值后，只在当前线程管用，所以说，如同上述所说的

- 一个ThreadLocal可以被多个线程共享
- 每个线程对同一个ThreadLocal的set get操作只针对当前线程管用

### ThreadLocal的原理以及源码介绍

大概了解了ThreadLocal如何使用，那么请问，ThreadLocal如何保证不同线程的独立性的呢？

#### ThreadLocal几个内部方法

##### **protected T initialValue()**（如果不想初始值返回null，需要重写initialValue方法）

```java
protected T initialValue() {
    return null;
}
```

![如果不想初始值返回null，需要重写initialValue方法](https://ws2.sinaimg.cn/large/006tNc79gy1foyhxjqs0qj31kw0zyn9s.jpg)

##### **public T get()**（该方法返回当前线程变量副本。如果这是线程第一次调用该方法，则创建并初始化此副本。）

/**

```java
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```
##### **public void set(T value)**

/**

```java
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
1. 由get和set源码可以看出，数据的存取都是先获取ThreadLocalMap对象，从**ThreadLocalMap**存取
2. ThreadLocalMap是一个map，它的key，就是threadLocal本身，值就是存放的变量副本
3. **每个线程对应一个本地变量的map，每个可以存放多个线程本地变量（即不同的ThreadLocal）**

##### **public void remove()**（jdk1.5后出现）

/**

```java
 * Removes the current thread's value for this thread-local
 * variable.  If this thread-local variable is subsequently
 * {@linkplain #get read} by the current thread, its value will be
 * reinitialized by invoking its {@link #initialValue} method,
 * unless its value is {@linkplain #set set} by the current thread
 * in the interim.  This may result in multiple invocations of the
 * <tt>initialValue</tt> method in the current thread.
 *
 * @since 1.5
 */
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null)
         m.remove(this);
 }
```
通过remove源码可以看到，

1. 先通过ThreadLocal的getMap（Thread.currentThread()）方法拿到当前线程的ThreadLocalMap
2. 然后再在当前线程的ThreadLocalMap中get，set，remove

关于remove需要知道的几点：

- 为什么移除某个ThreadLocal的值：

目的是减少内存缓存，remove之后如果再次访问此线程局部变量的值，将返回initiValue初始值

线程结束后，该线程对应的所有局部变量将自动被垃圾回收，但是显示调用remove清楚线程局部变量不是必须操作，但是可以加快内存回收的速度

### ThreadLocal和同步机制synchonzied的区别

- ThreadLocal：以空间换时间
- synchonzied：以时间换空间

synchonzied同步机制：

为多线程对相同资源的并发访问控制，保证了多线程之间的数据共享，**同步会带来巨大的性能开销，所以同步操作应该是细粒度的（对象中的不同元素使用不同的锁，而不是整个对象一个锁）**，以时间换空间的意思是：**使用同步真正的风险是复杂性和可能破坏资源安全,而不是性能。**

ThreadLocal线程局部变量机制：

空间换取时间，不同线程访问同一ThreadLocal，数据的存取是当前线程的数据副本，也就是说不同线程在某一时间访问到的并不是同一对象，所以效率比较高，但是占用内存比较大，当线程结束之后，remove会加快内存的回收速度。

Synchronized着重于线程间的数据共享，而ThreadLocal则着重于线程间的数据隔离。 

### ThreadLocal的弊端（内存泄露）

#### 内存泄露原因

ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部的强引用，那么在系统GC的时候，这个ThreadLocal就会被回收掉

ThreadLocal被回收掉之后，那么当前Thread的ThreadLocalMap中间就会出现key为null的Entry

key为null的话就意味着，没有办法访问这些key对应的值，就会存在以下的这样一个强引用链

value —Entry—TreadLocalMap--Thread

#### 内存泄露解决

`ThreadLocalMap`的设计中已经考虑到这种情况，也加上了一些防护措施：在`ThreadLocal`的`get()`,`set()`,`remove()`的时候都会清除线程`ThreadLocalMap`里所有`key`为`null`的`value`。

#### 以下操作会导致内存泄露

1. 使用static的ThreadLocal，延长了`ThreadLocal`的生命周期，导致某个线程Thread结束后，但是Thread内部的ThreadLocalMap中存在这个静态的ThreadLocal，导致ThreadLocalMap没法被回收，导致该Thread没法被回收

2. 分配使用了`ThreadLocal`又不再调用`get()`,`set()`,`remove()`方法，那么就会导致内存泄漏。因为如上所说

   `get()`,`set()`,`remove()`会清理线程ThreadLocalMap里所有key为null的value

### Android中ThreadLocal的体现

#### Handler消息机制

熟悉Handler机制的都知道

在ActivityThread的main方法中Looper.prepareMainLooper();或者在自己创建的线程中Looper.pepare()的时候

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

创建了一个Looper对象并使用sThreadLocal的set方法进行保存

并且这个ThreadLocal在Looper类中是静态的，如下

![](https://ws4.sinaimg.cn/large/006tNc79gy1foylpjjmbuj31jg0w6k0i.jpg)

那就是说，这个静态的ThreadLocal，可以供任何线程访问，但是任意线程中取出来的looper，都只是线程局部变量，都是在副本

所以说，每个线程对应一个looper，

![](https://ws4.sinaimg.cn/large/006tNc79gy1foylttuf5dj31i40ectj5.jpg)

**对于ThreadLocal的总结，暂时整理到这里，后续补充 ♨**