---
title: SharedPreferences的缺陷
date: 2018-07-19 13:44:44
tags:
- Android
categories: Android
password:
---

SharedPreferences在Android使用非常普遍，本文记录一下它的不足之处和使用注意事项

<!--more-->

#### 前言

`SharedPreferences`是Android SDK提供的工具，可以存储应用的一些配置信息，这些信息会以键值对的形式保存在`/sdcard/data/data/packageName/shared_prefs/`路径下的一个`xml`文件中。它提供了多种数据类型的存储，包括：`int`、`long`、`boolean`、`float`、`String`以及`Set<String>`。

#### 使用方式

- `context.getSharedPreference(name, mode);`
- `PreferenceManager.getDefaultSharedPreferences(context);`

这两种方式其实具体实现是一样的，只不过一个是开发者自己定义名字，另一个是使用包名+"_preference"作为存储文件名。

#### 问题

若我们大量使用`PreferenceManager.getDefaultSharedPreferences(context);`，将各种配置项全部存储到一个sp中，就可能会导致一个问题：**该文件过大，读取配置项过慢**

所以推荐，根据情况，将不同的配置文件保存在不同的sp中，而不是全部使用默认的sp，导致同一个sp文件过大

1. 第一次从sp中获取值的时候，有可能阻塞主线程，使界面卡顿、掉帧。
2. 解析sp的时候会产生大量的临时对象，导致频繁GC，引起界面卡顿。
3. 这些key和value会永远存在于内存之中，占用大量内存。

#### 其他注意

- 被加载进来的这些大对象，会永远存在于内存之中，不会被释放。

ContextImpl这个类，在getSharedPreference的时候会把所有的sp放到一个静态变量里面缓存起来：

**static**的sSharedPrefsCache，它保存了你所有使用的sp

```java
@GuardedBy("ContextImpl.class")
private static ArrayMap<String, ArrayMap<File, SharedPreferencesImpl>>sSharedPrefsCache;

private ArrayMap<File, SharedPreferencesImpl> getSharedPreferencesCacheLocked() {
    if (sSharedPrefsCache == null) {
        sSharedPrefsCache = new ArrayMap<>();
    }

    final String packageName = getPackageName();
    ArrayMap<File, SharedPreferencesImpl> packagePrefs = sSharedPrefsCache.get(packageName);
    if (packagePrefs == null) {
        packagePrefs = new ArrayMap<>();
        sSharedPrefsCache.put(packageName, packagePrefs);
    }

    return packagePrefs;
}
```

- 存储JSON等特殊符号很多的value

在sp里面存json或者HTML；这么做不是不可以，但是，如果这个json相对较大，那么也会引起sp读取速度的急剧下降。

JSON或者HTML格式存放在sp里面的时候，需要转义，这样会带来很多 & 这种特殊符号，sp在解析碰到这个特殊符号的时候会进行特殊的处理，引发额外的字符串拼接以及函数调用开销。而JSON本来就是可以用来做配置文件的，你干嘛又把它放在sp里面呢？

- 多次edit多次apply

```java
SharedPreferences sp = getSharedPreferences("test", MODE_PRIVATE);
sp.edit().putString("test1", "sss").apply();
sp.edit().putString("test2", "sss").apply();
sp.edit().putString("test3", "sss").apply();
sp.edit().putString("test4", "sss").apply();
```

每次edit都会创建一个Editor对象，额外占用内存；当然多创建几个对象也影响不了多少；但是，多次apply也会卡界面你造吗？

有童鞋会说，apply不是在别的线程些磁盘的吗，怎么可能卡界面？我带你仔细看一下源码。

```java
public void apply() {
    final MemoryCommitResult mcr = commitToMemory();
    final Runnable awaitCommit = new Runnable() {
            public void run() {
                try {
                    mcr.writtenToDiskLatch.await();
                } catch (InterruptedException ignored) {
                }
            }
        };

    QueuedWork.add(awaitCommit);

    Runnable postWriteRunnable = new Runnable() {
            public void run() {
                awaitCommit.run();
                QueuedWork.remove(awaitCommit);
            }
        };

    SharedPreferencesImpl.this.enqueueDiskWrite(mcr, postWriteRunnable);
    notifyListeners(mcr);
}
```

注意两点，第一，把一个带有await的runnable添加进了QueueWork类的一个队列；第二，把这个写入任务通过enqueueDiskWrite丢给了一个**只有单个线程**的线程池执行。

到这里一切都OK，在子线程里面写入不会卡UI。但是，你去ActivityThread类的handleStopActivity里看一看：

```java
private void handleStopActivity(IBinder token, boolean show, int configChanges, int seq) {

    // 省略无关。。
    // Make sure any pending writes are now committed.（确保所有延迟写入完成）
    if (!r.isPreHoneycomb()) {
        QueuedWork.waitToFinish();
    }

    // 省略无关。。
}
```

如果在Activity Stop的时候，已经写入完毕了，那么万事大吉，不会有任何等待，这个函数会立马返回。但是，如果你使用了太多次的apply，那么意味着写入队列会有很多写入任务，而那里就只有一个线程在写。当App规模很大的时候，这种情况简直就太常见了

虽然apply是在子线程执行的，但是请不要无节制地apply；commit我就不多说了吧？直接在当前线程写入，如果你在主线程干这个

- SP多进程不可靠

sp有一个貌似可以提供「跨进程」功能的FLAG——MODE_MULTI_PROCESS

文档也说了，这玩意在某些Android版本上不可靠，并且未来也不会提供任何支持，要是用跨进程数据传输需要使用类似ContentProvider的东西。而且，SharedPreference的文档也特别说明：

`Note: This class does not support use across multiple processes.`

```java
@Override
public SharedPreferences getSharedPreferences(File file, int mode) {
    checkMode(mode);
    SharedPreferencesImpl sp;
    synchronized (ContextImpl.class) {
        final ArrayMap<File, SharedPreferencesImpl> cache = getSharedPreferencesCacheLocked();
        sp = cache.get(file);
        if (sp == null) {
            sp = new SharedPreferencesImpl(file, mode);
            cache.put(file, sp);
            return sp;
        }
    }
    if ((mode & Context.MODE_MULTI_PROCESS) != 0 ||
        getApplicationInfo().targetSdkVersion < android.os.Build.VERSION_CODES.HONEYCOMB) {
        // If somebody else (some other process) changed the prefs
        // file behind our back, we reload it.  This has been the
        // historical (if undocumented) behavior.
        sp.startReloadIfChangedUnexpectedly();
    }
    return sp;
}
```

这个flag保证了啥？保证了**在API 11以前**的系统上，如果sp已经被读取进内存，再次获取这个sp的时候，如果有这个flag，会重新读一遍文件，仅此而已

#### 总结

1. 不要存放大的key和value！会引起界面卡、频繁GC、占用内存等等
2. 毫不相关的配置项就不要丢在一起了！文件越大读取越慢，防止全部放进defalut的sp！
3. 读取频繁的key和不易变动的key尽量不要放在一起，影响速度。（如果整个文件很小，那么忽略吧，为了这点性能添加维护成本得不偿失）
4. 不要乱edit和apply，每次edit会创建新的EditorImpl对象，尽量批量修改一次提交！
5. 尽量不要存放JSON和HTML，这种场景请直接使用json（文件存取）！
6. Commit发生在UI线程中，apply发生在工作线程中，对于数据的提交最好是批量操作统一提交。虽然apply发生在工作线程（不会因为IO阻塞UI线程）但是如果添加任务较多也有可能带来其他严重后果（参照ActivityThread源码中handleStopActivity方法实现）。
7. 跨进程通信不可靠

[参考1]: http://www.cnblogs.com/mingfeng002/p/5970221.html	"不要滥用SharedPreference"
[参考2]: https://blog.csdn.net/andy_jiangbin/article/details/55045577	"Android数据存储之SharedPreferences及如何安全存储"
[参考3]: https://www.jianshu.com/p/8eb2147c328b	"Android之不要滥用SharedPreference"

