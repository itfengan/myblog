---
title: 3.X的gradle依赖implementation
date: 2018-01-24 15:47:00
tags:
- Android
categories: Android
password:
---

依赖方式有很多，我们这里只弄清楚implementation和Compile的区别，就好了

<!--more-->

### implementation和Compile的区别

先看下图

![依赖区别](https://upload-images.jianshu.io/upload_images/2139461-fdab3438f31ddfe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/361)

假设：

**LibraryA—--—--implementation———LibraryC**

App Module 无法使用LibraryC

**LibraryB—--—--compile———LibraryD**

App Module 可以使用LibraryD

### 好处

- 可以加快编译速度
- 隐藏对外不必要的接口

为什么能够加快编译速度呢

这对于大型项目含有多个`Moudle`模块的， 以上图为例，比如我们改动 `LibraryC` 接口的相关代码，这时候编译只需要单独编译`LibraryA`模块就行， 如果使用的是`api`或者旧时代的`compile`，由于`App Moudle` 也可以访问到 `LibraryC`,所以 `App Moudle`部分也需要重新编译。

### 其他几种方式

**首先是2.x版本的依赖方式：**

1. Compile
2. Provided
3. APK
4. Test compile
5. Debug compile
6. Release compile

**再来看看3.0的**

1. Implementation
2. API
3. Compile only
4. Runtime only
5. Unit Test implementation
6. Test implementation
7. Debug implementation
8. Release implementation

**2.x版本依赖的可以看看下面的说明，括号里对应的是3.0版本的依赖方式。**

#### compile（api）

这种是我们最常用的方式，使用该方式依赖的库将会**参与编译和打包**。 
当我们依赖一些第三方的库时，可能会遇到com.android.support冲突的问题，就是因为开发者使用的compile依赖的com.android.support包，而他所依赖的包与我们本地所依赖的com.android.support包版本不一样，所以就会报All com.android.support libraries must use the exact same version specification (mixing versions can lead to runtime crashes这个错误。

[support包冲突](https://blog.csdn.net/yuzhiqiang_1993/article/details/78214812)

#### provided（compileOnly）

**只在编译时有效，不会参与打包** 
可以在自己的moudle中使用该方式依赖一些比如com.android.support，gson这些使用者常用的库，避免冲突。

#### apk（runtimeOnly）

只在生成apk的时候参与打包，编译时不会参与，很少用。

#### testCompile（testImplementation）

testCompile 只在单元测试代码的编译以及最终打包测试apk时有效。

#### debugCompile（debugImplementation）

debugCompile 只在debug模式的编译和最终的debug apk打包时有效

#### releaseCompile（releaseImplementation）

Release compile 仅仅针对Release 模式的编译和最终的Release apk打包。

**参考：**

[Android Studio3.x新的依赖方式（implementation、api、compileOnly）](https://blog.csdn.net/yuzhiqiang_1993/article/details/78366985?locationNum=6&fps=1)



### 