---
title: Rxjava2.0-知识碎片（三）
date: 2018-07-24 09:47:45
tags:
- Rxjava
categories: Rxjava
password:
---

基于Rxjava2.0的操作符小结

<!--more-->

# 一：创建相关

### creat操作符

```
Observable.create(new Observable.OnSubscribe<String>() {

        @Override
        public void call(Subscriber<? super String> subscriber) {

            subscriber.onNext("item1");
            subscriber.onNext("item2");
            subscriber.onCompleted();
        }
    });

```

### just

```
Observable observable = Observable.just("Hello", "Hi", "Aloha");
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
```

### from

```java
String[] words = {"Hello", "Hi", "Aloha"};
Observable observable = Observable.from(words);
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
```
### interval操作符

不多bb，看以下代码

```java
public void rxJava() {
        Observable.interval(1, TimeUnit.SECONDS)
                .subscribe(new Consumer<Long>() {
                    public Disposable mDisposable;

                    @Override
                    public void accept(Long aLong) {
                        if (aLong == 10) {
                            mDisposable.dispose();
                        }
                        System.out.println("计时器" + aLong);
                    }
                }, throwable -> {

                }, () -> {
                    System.out.println("action");
                }, disposable -> mDisposable = disposable);
    }
//log
07-24 11:24:52.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器0
07-24 11:24:53.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器1
07-24 11:24:54.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器2
07-24 11:24:55.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器3
07-24 11:24:56.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器4
07-24 11:24:57.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器5
07-24 11:24:58.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器6
07-24 11:24:59.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器7
07-24 11:25:00.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器8
07-24 11:25:01.718 12306-12332/com.fengandev.rxjavademo I/System.out: 计时器9
```

可以简易的封装，获取一个倒计时Observable

```java
/**
     * 产生一个倒计时的 Observable
     * @param time
     * @return
     */
    public Observable<Long> countdown(final long time) {
        return Observable.interval(1, TimeUnit.SECONDS)
                .map(new Function<Long, Long>() {
                    @Override
                    public Long apply(@NonNull Long aLong) throws Exception {
                        return time - aLong;
                    }
                }).take( time + 1 );
    }

//使用
    public void rxJava() {
        countdown(10).subscribe(new Consumer<Long>() {
            @Override
            public void accept(Long aLong) throws Exception {
                System.out.println("倒计时"+aLong);
            }
        });
    }
```

### range操作符

range 发射特定整数序列的 Observable

- range( int start , int end ) //start :开始的值 ， end ：结束的值

```java
 Observable.range(0,5)
                .subscribe(integer -> System.out.println(integer));
//log
0
1
2
3
4
```

老规矩，包前不包后
### empty和error和never

**empty**

`Observable observable1=Observable.empty();`//直接调用onCompleted。

```java
//Observable observable1=Observable.empty();//直接调用onCompleted。
@Test
    public void testRxJava() {
        Observable<String> empty = Observable.empty();
        empty.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe");
            }

            @Override
            public void onNext(String s) {
                System.out.println("onNext");
            }

            @Override
            public void onError(Throwable e) {
                System.out.println("onError");
            }

            @Override
            public void onComplete() {
                System.out.println("onComplete");
            }
        });
    }
//log:
//onSubscribe
//onComplete
```

**never**

`Observable observable3=Observable.never();`//啥都不做

```java
@Test
public void testRxJava() {
    Observable<String> empty = Observable.never();
    empty.subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            System.out.println("onSubscribe");
        }

        @Override
        public void onNext(String s) {
            System.out.println("onNext");
        }

        @Override
        public void onError(Throwable e) {
            System.out.println("onError");
        }

        @Override
        public void onComplete() {
            System.out.println("onComplete");
        }
    });
}
//log
//onSubscribe
```

**error**

`Observable observable2=Observable.error(new RuntimeException());`//直接调用onError。这里可以自定义异常

## 功能操作

### map操作符

map操作符算是Rxjava中最简单的一个操作符了，在2.x中的用法和1.x差不多；它的作用就是对发射的每一个事件应用一个函数，每一个事件都按照map操作符指定的函数去变化转换，下面我们看一个栗子🌰

```java
@Test
    public void rxJava() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onComplete();
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                Log.e("fengan", "apply---->" + Thread.currentThread().getName());
                return "this is value = " + integer;
            }
        }).subscribeOn(Schedulers.single())
//                .observeOn(Schedulers.single())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Exception {
                        Log.e("fengan", "accept---->" + Thread.currentThread().getName());
                        Log.e("fengan", "accept---->" + s);
                        Log.e("fengan","========");

                    }
                });
    }
```

log

```java
    apply---->RxSingleScheduler-1
    accept---->RxSingleScheduler-1
    accept---->this is value = 1
    ========
    apply---->RxSingleScheduler-1
    accept---->RxSingleScheduler-1
    accept---->this is value = 2
    ========
    apply---->RxSingleScheduler-1
    accept---->RxSingleScheduler-1
    accept---->this is value = 3
    ========

```

从这个例子，我们能看出

- 我们发射的integer事件，被转换为String，继续进行
- 我们也顺便验证了两个问题（在碎片化2整理的线程调度）
  -  只指定`subscribeOn`未指定`observeOn`的情况下，观察者的事件的接受（下游）按照`subscribeOn`指定的线程进行
  -  `Schedulers.single()`是在指定线程中按照队列的形式进行，先进先出（体现在：一个事件发布和接受完毕后，才发送第二个事件）


### debounce操作符

`debounce`：防抖；

only emit an item from an Observable if a particular time-span has passed without it emitting another item,

当一个事件发送出来之后，在约定时间内没有再次发送这个事件，则发射这个事件，如果再次触发了，则重新计算时间。

[参考链接](https://www.jianshu.com/p/ee1f0d21a856)

```java
 Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                // send events with simulated time wait
                emitter.onNext(1); // skip
                Thread.sleep(400);
                emitter.onNext(2); // deliver
                Thread.sleep(505);
                emitter.onNext(3); // skip
                Thread.sleep(100);
                emitter.onNext(4); // deliver
                Thread.sleep(605);
                emitter.onNext(5); // deliver
                Thread.sleep(510);
                emitter.onComplete();
            }
        }).debounce(500,TimeUnit.MILLISECONDS)
            .subscribe(new Consumer<Integer>() {
                @Override
                public void accept(Integer integer) throws Exception {
                    System.out.println(integer);
                }
            });

//2
//4
//5
```

### onTerminateDetach操作符

[Rxjava导致内存泄漏的问题](https://blog.csdn.net/johnny901114/article/details/67640594)

[一张图搞定-RxJava2的线程切换原理和内存泄露问题](https://blog.csdn.net/johnny901114/article/details/67640594)

看完这两个参考链接，基本也就知道了onTerminateDetach的使用方法和作用

### defer操作符

只有当订阅者订阅才创建Observable，为每个订阅创建一个新的Observable。

```java
@Test
public void rxjava() {
    Observable.defer(new Callable<ObservableSource<Integer>>() {
        @Override
        public ObservableSource<Integer> call() throws Exception {
            return Observable.just(1,2,3);
        }
    }).subscribe(new Consumer<Integer>() {
        @Override
        public void accept(Integer integer) throws Exception {
            System.out.println(integer);
        }
    });
}
1
2
3
```
### flatMap操作符

把一个发射器`Observable` 通过某种方法转换为多个`Observables`，然后再把这些分散的`Observables`装进一个单一的发射器`Observable`。但有个需要注意的是，`flatMap`并不能保证事件的顺序，如果需要保证，需要用到我们下面要讲的`ConcatMap`。

```java
@Test
public void rxJava() {
    Observable.create(new ObservableOnSubscribe<Integer>() {
        @Override
        public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
            emitter.onNext(1);
            emitter.onNext(2);
            emitter.onNext(3);
            emitter.onComplete();
        }
    }).flatMap(new Function<Integer, ObservableSource<String>>() {
        @Override
        public ObservableSource<String> apply(Integer integer) throws Exception {
            List<String> res = new ArrayList<>();
            res.add("i am value =" + integer);
            Log.e("fengan","flatMap size"+res.size());
            return Observable.fromIterable(res);
        }
    }).subscribe(new Consumer<String>() {
        @Override
        public void accept(String s) throws Exception {
            Log.e("fengan","accept="+s);
        }
    });
}
```

log

```
	flatMap size1
	accept=i am value =1
    flatMap size1
    accept=i am value =2
    flatMap size1
    accept=i am value =3
```

**结论**

- 验证了我之前的一个误区（size大小一直为1）：误以为所有事件发送完毕之后，在flatmap统一处理，其实不是，事件源每发射一个事件，`flatmap`就处理转换一个事件
- `flatMap`的作用是，接受一个事件，作出处理完之后，重新发射一个新的事件

刚才说到`flatMap`并不能保证事件的顺序，请看下面的例子就明白了

### concatMap操作符

flatMap操作符可以将一个`Observable`转换为另一个`Observable`发射出去,并且可以将多个事件转化为1个，但是最后输出的事件序列顺序是不确定的，如果想要最后输出的事件顺序和源数据的顺序一致只要换成`concatMap`就可以了。 
flatMap和Map操作符的不同是map一次只能转换一个事件。

先看一个`flatmap`的使用例子，如下

```java
 Observable.create((ObservableOnSubscribe<Integer>) emitter -> {
            emitter.onNext(1);
            emitter.onNext(2);
            emitter.onNext(3);
            emitter.onComplete();
        }).subscribeOn(Schedulers.io()).flatMap((Function<Integer, ObservableSource<String>>) integer -> {
            List<String> res = new ArrayList<>();
            for (int i = 0; i < 3; i++) {
                res.add("I am value " + integer);
            }
            int delayTime = (int) (1 + Math.random() * 10);
            return Observable.fromIterable(res).delay(delayTime, TimeUnit.MILLISECONDS);
        }).subscribe(s -> Log.e("fengan", "accept=" + s));
```

log

```java
    accept=I am value 3
    accept=I am value 3
    accept=I am value 2
    accept=I am value 2
    accept=I am value 2
    accept=I am value 3
    accept=I am value 1
    accept=I am value 1
    accept=I am value 1
```

可见不是按照源数据到顺序一致，若想一致使用`concatMap`就可以了


### repeat操作符

repeat 重复地发射数据

- repeat( ) //无限重复
- repeat( int time ) //设定重复的次数

```java
Observable.just(1,2,3)
                .repeat(3)
                .subscribe(integer -> System.out.println(integer));
1
2
3
1
2
3
1
2
3
```

### fromArray和fromIterble操作符

一个是发射一个数组一个是发射一个集合

### toList

将数组转换为集合

### delay操作符

```java
public void rxJava() {
        Observable.just(1, 2, 3)
                .delay(3, TimeUnit.SECONDS)
                .observeOn(Schedulers.io())
                .subscribe(integer -> {
                    int a = integer;
                    System.out.println(a);
                });
    }
//注意：延迟3秒钟，然后在发射数据
//是延迟三秒，发送1，2，3数据
//而不是--延迟3秒--发送1--延迟3秒---发送2
```

- 是延迟三秒，发送1，2，3数据
- 而不是--延迟3秒--发送1--延迟3秒---发送2

若想达到这种效果，可以类似这么做

```java
public void rxJava() {
        Observable.just(1, 2, 3)
                .concatMap(new Function<Integer, ObservableSource<Integer>>() {
                    @Override
                    public ObservableSource<Integer> apply(Integer integer) throws Exception {
                        return Observable.just(integer) .delay(3, TimeUnit.SECONDS);
                        
                    }
                })
                .subscribe(integer -> System.out.println(integer));
    }
//--延迟三秒---发送1---延迟三秒---发送2---延迟3秒---发送3
```
### doOnNext操作符

```java
@Test
public void testRxJava() {
    Observable.create(new ObservableOnSubscribe<Integer>() {
        @Override
        public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
            emitter.onNext(1);
            emitter.onNext(2);
        }
    }).doOnNext(new Consumer<Integer>() {
        @Override
        public void accept(Integer integer) throws Exception {
            System.out.println("前置操作" + integer);
            integer+=1;
        }
    })
            .subscribe(System.out::println);
}
//log
前置操作1
1
前置操作2
2
```

在每次 OnNext() 方法被调用前执行

使用场景：从网络请求数据，在数据被展示前，缓存到本地
## 合并操作

### concat操作符

按顺序连接多个Observables。需要注意的是Observable.concat(a,b)等价于a.concatWith(b)。

```java
Observable<Integer> observable1=Observable.just(1,2,3,4);
    Observable<Integer>  observable2=Observable.just(4,5,6);

    Observable.concat(observable1,observable2)
            .subscribe(item->Log.d("JG",item.toString()));//1,2,3,4,4,5,6

```

### startWith操作符

在数据序列的开头增加一项数据。startWith的内部也是调用了concat

```java
 Observable.just(1,2,3,4,5)
            .startWith(6,7,8)
    .subscribe(item->Log.d("JG",item.toString()));//6,7,8,1,2,3,4,5
```

### merge操作符

合并被观察者

```java
final String[] aStrings = {"A1", "A2", "A3", "A4"};
        final String[] bStrings = {"B1", "B2", "B3"};

        final Observable<String> aObservable = Observable.fromArray(aStrings);
        final Observable<String> bObservable = Observable.fromArray(bStrings);

        Observable.merge(aObservable, bObservable)//使用merge操作符将两个被观察者合并
                .subscribe(getObserver());//这里的观察者依然不重要
//"A1", "B1", "A2", "A3", "A4", "B2", "B3" 
```

操作符merge将两个被观察者合并,这里要注意merge之后的Observable是不能保证和原来的Observable发射顺序相同.

### Zip操作符

zip我们想到”压缩“，那么在这个操作符是什么意思呢，请看下面图片

![Zip](https://upload-images.jianshu.io/upload_images/1931185-14134e499db9d0c4.png?imageMogr2/auto-orient/)

```java
@Test
    public void rxJava() {
        Observable.zip(Observable.just("A", "B", "C"), Observable.just(1,2,3,4,5), new BiFunction<String, Integer, String>() {

            @Override
            public String apply(String s, Integer integer) throws Exception {
                return s+integer;
            }
        }).subscribe(s -> System.out.println("value=" + s));
    }
```

log

```java
    value=A1
    value=B2
    value=C3

```

结论：

- zip 组合事件的过程就是分别从发射器A和发射器B各取出一个事件来组合，并且一个事件只能被使用一次，组合的顺序是严格按照事件发送的顺序来进行的，所以上面截图中，可以看到，1永远是和A 结合的，2永远是和B结合的。


- 最终接收器收到的事件数量是和发送器发送事件最少的那个发送器的发送事件数目相同，所以代码中，4和5是没有配对的，所以也就无法发射合并事件

### combineLatest操作符

先看一张图，我想你应该已经大致明白意思了

![combineLatest](https://upload-images.jianshu.io/upload_images/1931185-1e60a8bf25b31e91.png?imageMogr2/auto-orient/)

[使用场景，表单验证](https://blog.csdn.net/jdsjlzx/article/details/53040293)

combineLatest是RxJava本身提供的一个常用的操作符，它接受两个或以上的Observable和一个FuncX闭包。当传入的Observable中任意的一个发射数据时，combineLatest将每个Observable的最近值(Lastest)联合起来（combine）传给FuncX闭包进行处理。要点在于：

1. combineLatest是会存储每个Observable的最近的值的
2. 任意一个Observable发射新值时都会触发操作->“combine all the Observable’s lastest value together and send to Function”

看一个例子，可以说明

```java
@Test
    public void testRxJava() {
        final String[] aStrings = {"A1", "A2", "A3", "A4"};
        final String[] bStrings = {"B1", "B2", "B3"};

        final Observable<String> aObservable = Observable.fromArray(aStrings).delay(1,TimeUnit.MILLISECONDS);
        final Observable<String> bObservable = Observable.fromArray(bStrings);

        Observable.combineLatest(aObservable, bObservable, new BiFunction<String, String, String>() {
            @Override
            public String apply(String s1, String s2) throws Exception {
                return s1+s2;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {
                System.out.println(s);
            }
        });
    }
//log
//A1B3
//A2B3
//A3B3
//A4B3
可见，发送A1的时候B1 B2 B3都已经发射完毕
所以，存储每个Observable的最近的值的，也就是B3
当B1，B2，B3发射的 A被观察者一个都没发射，所以得至少两个才会合并，这个看图可以解释
```

##过滤操作
### take等操作符

```java
//take 取前n个数据
Observable.just(1, 2, 3,4,5,6,7)
        .take(3)
       .subscribe(System.out::println);
//log
1
2
3
//takelast 取后n个数据
 Observable.just(1, 2, 3, 4, 5, 6, 7)
                .takeLast(3)
                .subscribe(System.out::println);
//log
5
6
7
//first 只发送第一个数据  
//last 只发送最后一个数据
//skip() 跳过前n个数据发送后面的数据
//skipLast() 跳过最后n个数据，发送前面的数据  
```

### filter操作符

顾名思义，过滤发送的事件

```java
 @Test
    public void testRxJava() {
        Observable.just(1, 2, 3)
                .filter(integer -> {
                    if (integer==2) {
                        return false;
                    }
                    return true;
                }).subscribe(System.out::println);
    }
//log
1
3
```

### ofType

过滤指定类型的数据，与filter类似，

```java
@Test
public void rxjava() {
    Observable.just(1,"1",-1)
            .ofType(Integer.class)
            .subscribe(new Consumer<Integer>() {
                @Override
                public void accept(Integer integer) throws Exception {
                    System.out.println(integer);
                }
//log
1
-1

```

### first和last

```
Observable.just(1, 2, 3)
        .first(2)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);
            }
        });
//1
 Observable.just(1, 2, 3)
                .last(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        System.out.println(integer);
                    }
                });
//3                
```

## 未完待续

## 参考：

[手把手教你使用 RxJava 2.0（一）](https://www.jianshu.com/p/d149043d103a)

[RxJava combineLatest操作符处理复杂表单验证问题](https://blog.csdn.net/jdsjlzx/article/details/53040293)

[Android - RxJava2.0 操作符整理归纳](https://www.jianshu.com/p/b30de498c3cc)