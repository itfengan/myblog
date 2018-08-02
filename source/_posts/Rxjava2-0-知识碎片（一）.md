---
title: Rxjava2.0-知识碎片(一)
date: 2018-07-23 15:02:17
tags:
- Rxjava
categories: Rxjava
password:
---

Observable.create(ObservableOnSubscribe)和subscribe()基本使用

<!--more-->

### Observable.create(ObservableOnSubscribe)的使用

获得一个Observable有很多种方式，通过我们下面的动图可以看到

![获取Observable的方式](https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/Observable.gif)

今天的碎片化整理，就只整理最普通的一种方式`Observable.create(ObservableOnSubscribe)`

```java
/**
 * ObservableEmitter:Emitter是发射器的意思，
 * 它可以发出三种类型的事件，
 * 通过调用emitter的onNext(T value)、onComplete()和onError(Throwable error)就可以分别发出next事件、complete事件和error事件。
 * 发射满足规则：
 * 1，Observable可以发送无限个onNext, Observer也可以接收无限个onNext
 * 2，发送了一个onComplete后, Observable的onComplete或者onError之后的事件将会继续发送, 而Observer收到onComplete/onError事件之后将不再继续接收事件.
 * 3，Observable可以发送无限个onNext可以不发送onComplete或onError.
 * 4，Observable不可以发送多个onComplete和多个onError（onComplete和onError加一起只能发射一次）
 * 5，Observable发射多个onComplete不报错（只收到第一个onComplete就不再接受了）。发射多个onError会报错
 */
@Test
public void rxjava() {
    Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("hello");
            emitter.onNext("rxjava");
            emitter.onComplete();
            emitter.onNext("无法接受");
        }
    }).subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            System.out.println("onSubscribe");
        }

        @Override
        public void onNext(String s) {
            System.out.println("onNext:value=" + s);
        }

        @Override
        public void onError(Throwable e) {
            System.out.println("onError:error" + e.getMessage());
        }

        @Override
        public void onComplete() {
            System.out.println("onComplete");
        }
    });
}

```

**logcat：**

```java
onSubscribe
onNext:value=hello
onNext:value=rxjava
onComplete
```

### 发送规则

 * 发射满足规则：
 * 1，Observable可以发送无限个onNext, Observer也可以接收无限个onNext
 * 2，发送了一个onComplete后, Observable的onComplete或者onError之后的事件将会继续发送, 而Observer收到onComplete/onError事件之后将不再继续接收事件.
 * 3，Observable可以发送无限个onNext可以不发送onComplete或onError.
 * 4，Observable不可以发送多个onComplete和多个onError（onComplete和onError加一起只能发射一次）
 * 5，Observable发射多个onComplete不报错（只收到第一个onComplete就不再接受了）。发射多个onError会报错

![假设事件流图](https://upload-images.jianshu.io/upload_images/1008453-7133ff9a13dd9a59.png?imageMogr2/auto-orient/)

这里的`上游`和`下游`就分别对应着RxJava中的`Observable`和`Observer`，它们之间的连接就对应着`subscribe()`

以上几个规则用示意图表示如下:

|                    | 示意图                                      |
| ------------------ | ---------------------------------------- |
| **只发送onNext事件**    | ![img](https://upload-images.jianshu.io/upload_images/1008453-2526f7824c53a47e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/580)next |
| **发送onComplete事件** | ![img](https://upload-images.jianshu.io/upload_images/1008453-bdb581480fc9379b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/595)complete |
| **发送onError事件**    | ![img](https://upload-images.jianshu.io/upload_images/1008453-b731e6294f342d66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/595)error |

[参考链接](https://www.jianshu.com/p/464fa025229e)

### onSubscribe(Disposable d)的参数Disposable

```java
@Test
    public void rxjava() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                emitter.onNext(1);
                emitter.onNext(2);
                emitter.onNext(3);
                emitter.onNext(4);
                emitter.onNext(5);
                emitter.onComplete();
            }
        }).subscribe(new Observer<Integer>() {
            public Disposable mDisposable;

            @Override
            public void onSubscribe(Disposable d) {
                System.out.println("onSubscribe");
                this.mDisposable = d;
            }

            @Override
            public void onNext(Integer integer) {
                if (integer == 3) {
                    System.out.println("onNext:isDisposed=" + mDisposable.isDisposed());
                    mDisposable.dispose();
                    System.out.println("onNext:isDisposed=" + mDisposable.isDisposed());
                }
                System.out.println("onNext:valuse=" + integer);

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
```

**logcat：**

```java
onSubscribe
onNext:valuse=1
onNext:valuse=2
onNext:isDisposed=false
onNext:isDisposed=true
onNext:valuse=3
```

由代码可看出：

- `onSubscribe`方法最先调用，并且只调用一次
- `Disposable.dispose()`调用后，`onNext`将接受不到发出的4，5事件（**但是emitter.onNext(4);**
  ​                **emitter.onNext(5)确实执行了；**
- Disposable的用处不止这些, 后面讲解到了线程的调度之后, 我们会发现它的重要性.后续补充

### `subscribe()`的多个重载的方法

```java
1,public final Disposable subscribe() {}
2,public final Disposable subscribe(Consumer<? super T> onNext) {}
3,public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> 	onError) {} 
4,public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> 	onError, Action onComplete) {}
5,public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> 	onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
6,public final void subscribe(Observer<? super T> observer) {}
```

- 不带任何参数的`subscribe()` 表示下游不关心任何事件,你上游尽管发你的数据去吧, 下游无法接受
- 带有一个`Consumer`参数的方法表示下游只关心onNext事件, 其他的事件我假装没看见, 因此我们如果只需要onNext事件可以这么写
- `Observer`参数，如例子，比较完整的onSubscribe，onNext，onComplete，onError
- 如果我们不需要其他回调，可选择我们需要的`Consumer` 的具体参数

[参考链接](https://www.jianshu.com/p/464fa025229e)