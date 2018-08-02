---
title: Rxjava2.0-知识碎片(二)
date: 2018-07-23 16:51:40
tags:
- Rxjava
categories: Rxjava
password:
---

Rxjava2.0中的线程调度小节

<!--more-->

# Scheduler的种类

<img src="https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/schedulers.png" width="25%" height="25%" />

- Schedulers.io( )：

  用于IO密集型的操作，例如读写SD卡文件，查询数据库，访问网络等，具有线程缓存机制，在此调度器接收到任务后，先检查线程缓存池中，是否有空闲的线程，如果有，则复用，如果没有则创建新的线程，并加入到线程池中，如果每次都没有空闲线程使用，可以无上限的创建新线程。

- Schedulers.newThread( )：


在每执行一个任务时创建一个新的线程，不具有线程缓存机制，因为创建一个新的线程比复用一个线程更耗时耗力，虽然使用Schedulers.io( )的地方，都可以使用Schedulers.newThread( )，但是，Schedulers.newThread( )的效率没有Schedulers.io( )高。

- Schedulers.computation()：


用于CPU 密集型计算任务，即不会被 I/O 等操作限制性能的耗时操作，例如xml,json文件的解析，Bitmap图片的压缩取样等，具有固定的线程池，大小为CPU的核数。不可以用于I/O操作，因为I/O操作的等待时间会浪费CPU。

- Schedulers.trampoline()：


在**当前线程**立即执行任务，如果当前线程有任务在执行，则会将其暂停，**等插入进来的任务执行完之后，再将未完成的任务接着执行。**

- Schedulers.single()：


拥有一个线程单例(**注意是子线程**)，所有的任务都在这一个线程中执行，当此线程中有任务执行时，其他任务将会按照先进先出的顺序依次执行。

- Scheduler.from(@NonNull Executor executor)：


指定一个线程调度器，由此调度器来控制任务的执行策略。

- AndroidSchedulers.mainThread()：


在Android UI线程中执行任务.需要添加RxAndroid依赖

> 注意

| 方式                                       | 子线程  | 主线程  |      备注       |
| :--------------------------------------- | :--: | :--: | :-----------: |
| Schedulers.io()                          | yes  |  --  |      线程池      |
| Schedulers.newThread()                   | yes  |  --  |     创建新的      |
| Schedulers.computation()                 | yes  |  --  | 占用CPU，不要i/o耗时 |
| Schedulers.trampoline()                  | yes  | yes  |     当前线程      |
| Schedulers.single()                      | yes  |  --  |    固定一个子线程    |
| Scheduler.from(@NonNull Executor executor) | yes  | yes  |     指定线程      |
| AndroidSchedulers.mainThread()           |  --  | yes  | 主线程，RxAndroid |

- 在RxJava2中，废弃了RxJava1中的Schedulers.immediate( )


- 在RxJava1中，Schedulers.immediate( )的作用为在当前线程立即执行任务，功能等同于RxJava2中的Schedulers.trampoline( )，不同的是，Schedulers.trampoline( )是停下当前的任务，先执行插入进来的任务，等执行完后，再将暂停的任务继续执行下去。

- Schedulers.trampoline( )在RxJava1中的作用是当其它排队的任务完成后，在当前线程排队开始执行接到的任务，有点像RxJava2中的Schedulers.single()，但也不完全相同，因为Schedulers.single()不是在当前线程而是在一个线程单例中排队执行任务.

  ​

  ​

# subscribeOn和observeOn


  ## 介绍

### **subscribeOn**

`Observable<T> subscribeOn(Scheduler scheduler)`

  subscribeOn通过接收一个Scheduler参数，来指定对数据的处理运行在特定的线程调度器Scheduler上。
  若多次设定，则只有一次起作用。

### **observeOn**

`Observable<T> observeOn(Scheduler scheduler)`

observeOn同样接收一个Scheduler参数，来指定下游操作运行在特定的线程调度器Scheduler上。
若多次设定，每次均起作用。

### 使用实例

#### 实例一：

```java
@Test
public void rxjavaThreadScheuler(){
    Integer [] nums = {1,2,3};
    Observable.fromArray(nums)
            .observeOn(Schedulers.single())
            .subscribeOn(Schedulers.io())
            .map(new Function<Integer, String>() {
                @Override
                public String apply(Integer integer) throws Exception {

                    System.out.println("mapA"+"Thread="+Thread.currentThread().getName()+"\r\n"+"integer="+integer);
                    return "num"+integer;
                }
            })
            .map(new Function<String, Integer>() {
                @Override
                public Integer apply(String s) throws Exception {
                    System.out.println("mapB"+"Thread="+Thread.currentThread().getName()+"\r\n"+"String="+s);
                    return Integer.parseInt(s.substring(3,s.length()));
                }
            })
            .subscribe(new Consumer<Integer>() {
                @Override
                public void accept(Integer integer) throws Exception {
                    System.out.println("subscribe"+"Thread="+Thread.currentThread().getName()+"\r\n"+"integer="+integer);
                    System.out.println("========");
                }
            });
}
```

```
mapAThread=RxSingleScheduler-1
integer=1
mapBThread=RxSingleScheduler-1
String=num1
subscribeThread=RxSingleScheduler-1
integer=1
========
mapAThread=RxSingleScheduler-1
integer=2
mapBThread=RxSingleScheduler-1
String=num2
subscribeThread=RxSingleScheduler-1
integer=2
========
mapAThread=RxSingleScheduler-1
integer=3
mapBThread=RxSingleScheduler-1
String=num3
subscribeThread=RxSingleScheduler-1
integer=3
========
```

结论：

打印的所有线程均在`Schedulers.single() `中执行，  `subscribeOn(Schedulers.io())`貌似没有生效，其实

`Observable.fromArray(nums)`的执行线程，是在`Schedulers.io()`线程中执行，我们通过实例二验证

#### 实例二：

```java
@Test
public void rxjavaThreadScheuler() {
    Integer[] nums = {1, 2, 3};
    Observable.create((ObservableOnSubscribe<Integer>) emitter -> {
        for (Integer integer : nums) {
            emitter.onNext(integer);
            System.out.println("emitter.onNext()" + "Thread=" + Thread.currentThread().getName() + "\r\n" + "integer=" + integer);
        }
    })
            .observeOn(Schedulers.single())
            .subscribeOn(Schedulers.io())
            .map(integer -> {
                System.out.println("mapA" + "Thread=" + Thread.currentThread().getName() + "\r\n" + "integer=" + integer);
                return "num" + integer;
            })
            .map(s -> {
                System.out.println("mapB" + "Thread=" + Thread.currentThread().getName() + "\r\n" + "String=" + s);
                return Integer.parseInt(s.substring(3, s.length()));
            })
            .subscribe(integer -> {
                System.out.println("subscribe" + "Thread=" + Thread.currentThread().getName() + "\r\n" + "integer=" + integer);
                System.out.println("========");
            });
}
```

  logcat

```java
//可见，事件的发布，是在.subscribeOn(Schedulers.io())指定的io线程
emitter.onNext()Thread=RxCachedThreadScheduler-1
integer=1
emitter.onNext()Thread=RxCachedThreadScheduler-1
integer=2
emitter.onNext()Thread=RxCachedThreadScheduler-1
integer=3
//其他均在.observeOn(Schedulers.single())指定的Single线程
mapAThread=RxSingleScheduler-1
integer=1
mapBThread=RxSingleScheduler-1
String=num1
subscribeThread=RxSingleScheduler-1
integer=1
========
mapAThread=RxSingleScheduler-1
integer=2
mapBThread=RxSingleScheduler-1
String=num2
subscribeThread=RxSingleScheduler-1
integer=2
========
mapAThread=RxSingleScheduler-1
integer=3
mapBThread=RxSingleScheduler-1
String=num3
subscribeThread=RxSingleScheduler-1
integer=3
========
```

  结论：

subscribeOn()指定的是事件的发布执行的线程，observeOn()指定的是各种操作符和subscribe的onNext，onError，onComplete执行等线程，那么调用多次subscribeOn()或者多次observeOn()，会以哪次为准呢，请看实例三

#### 实例三：多次切换场景

```java
public void rxjavaThreadScheuler() {
        Integer[] nums = {1, 2, 3};
        Observable.create((ObservableOnSubscribe<Integer>) emitter -> {
            for (Integer integer : nums) {
                emitter.onNext(integer);
                Log.e("rxjava", "emitter.onNext()--->" + Thread.currentThread().getName());
            }
            emitter.onComplete();
        })
                .subscribeOn(AndroidSchedulers.mainThread())//指定主线程发布事件
                .observeOn(Schedulers.single())//切换mapA
                .map(integer -> {
                    Log.e("rxjava", "mapA--->" + Thread.currentThread().getName());
                    return "num" + integer;
                })
                .observeOn(AndroidSchedulers.mainThread())//切换mapB
                .map(s -> {
                    Log.e("rxjava", "mapB线程--->" + Thread.currentThread().getName());
                    return Integer.parseInt(s.substring(3, s.length()));
                })
                .observeOn(Schedulers.computation())//切换Observer执行线程
                .subscribeOn(Schedulers.io())//再此指定发布线程为io
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        Log.e("rxjava", "onSubscribe--->" + 			Thread.currentThread().getName());
                    }

                    @Override
                    public void onNext(Integer integer) {
                        Log.e("rxjava", "onNext--->" + Thread.currentThread().getName());
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.e("rxjava", "onError--->" + Thread.currentThread().getName());
                    }

                    @Override
                    public void onComplete() {
                        Log.e("rxjava", "onComplete--->" + Thread.currentThread().getName());
                    }
                });
    }
```

  logcat

```java

rxjava: onSubscribe--->main//Observer的onSubscribe回调和发布事件的线程一致，也就是第一次subscribeOn()，若未指定，则在
rxjava: emitter.onNext()--->main//第一次subscribeOn()
rxjava: emitter.onNext()--->main
rxjava: mapA--->RxSingleScheduler-1//最近一次observeOn（）指定线程，若未指定，默认和subscribeOn的一致，若subscribeOn也未指定，默认是Schedulers.trampoline()
rxjava: emitter.onNext()--->main
rxjava: mapA--->RxSingleScheduler-1
rxjava: mapA--->RxSingleScheduler-1
rxjava: mapB线程--->main//最近一次observeOn（）指定线程
rxjava: mapB线程--->main
rxjava: mapB线程--->main
rxjava: onNext--->RxComputationThreadPool-1//Observer的onNext，由最近一次的observeOn（）指定
rxjava: onNext--->RxComputationThreadPool-1
rxjava: onNext--->RxComputationThreadPool-1
rxjava: onComplete--->RxComputationThreadPool-1//Observer的onComplete，由最近一次的observeOn（）
```

 结论：

- 多个subscribeOn()以第一个为主
- 多个observeOn（）指定以下操作执行线程
- 若不指定observeOn（），默认按subscribeOn()指定的线程一致
- 若不指定subscribeOn()，默认是Schedulers.trampoline()（在当前线程，停下之前任务，先执行本次任务）
- 注意：Observer的onSubscribe(Disposable d)回调，以subscribeOn()为主，而不是以observeOn（）为主

其他三个回调onNext，onError，onComplete以observeOn（）为主

#### 实例四Schedulers.trampoline()：

![](https://upload-images.jianshu.io/upload_images/6773051-6d40d0cbdb0bd590.jpg?imageMogr2/auto-orient/)

log

```java
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:0
System.out: 接收线程:RxCachedThreadScheduler-1---->接收:0
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:1
System.out: 接收线程:RxCachedThreadScheduler-1---->接收:1
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:2
System.out: 接收线程:RxCachedThreadScheduler-1---->接收:2
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:3
System.out: 接收线程:RxCachedThreadScheduler-1---->接收:3
System.out: 发射线程:RxCachedThreadScheduler-1---->发射:4
System.out: 接收线程:RxCachedThreadScheduler-1---->接收:4
//可以看到
//Schedulers.trampoline()指定Consumer回调执行线程和发布事件线程一样
//Schedulers.trampoline()的作用在当前线程立即执行任务，如果当前线程有任务在执行，则会将其暂停，等插入进来的任务执行完之后，再将未完成的任务接着执行。
```

结论：

Schedulers.trampoline()的作用在当前线程立即执行任务，如果当前线程有任务在执行，则会将其暂停，等插入进来的任务执行完之后，再将未完成的任务接着执行。【本实例的体现：当observe接受的时候，事件停止了发送】

#### 实例五Schedulers.single()

![](https://upload-images.jianshu.io/upload_images/6773051-7691bdd048bdcaad.jpg?imageMogr2/auto-orient/)

log

```java
System.out: 发射线程:RxSingleScheduler-1---->发射:0
System.out: 发射线程:RxSingleScheduler-1---->发射:1
System.out: 发射线程:RxSingleScheduler-1---->发射:2

System.out: 处理线程:RxSingleScheduler-1---->处理:0
System.out: 处理线程:RxSingleScheduler-1---->处理:1
System.out: 处理线程:RxSingleScheduler-1---->处理:2

System.out: 接收线程:RxSingleScheduler-1---->接收:0
System.out: 接收线程:RxSingleScheduler-1---->接收:1
System.out: 接收线程:RxSingleScheduler-1---->接收:2
//发布和接受都是single线程
//接收在Schedulers.single()的线程单例中排队执行，当此线程中有任务执行时，其他任务将会按照先进先出的顺序依次执行  
//所以可以达到。 完全发射---完全处理---完全接受的流程  
```

[参考连接](https://www.jianshu.com/p/12638513424f)