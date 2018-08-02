---
title: 浅析Dalvik和ART的区别
date: 2018-03-05 15:00:14
tags: 
- Android
categories: Android
password: 
---

Dalvik是Google公司自己设计用于Android平台的虚拟机，Dalvik经过优化，更加适合Android平台（具体优点见下面和JVM比较）。

2014年6月谷歌I/O大会上，Android L(5.0)改动幅度比较大，删除了Dalvik，而是用ART代替。

之前对本块的知识理解比较零散，再此本文总结下Dalvik和ART的原理，和二者的区别，以及Jvm和Dalvik的区别，以及Android的Apk编译打包的流程

<!--more-->

### Dalvik的相关知识

- **Google**公司自己设计**用于Android平台**的**java虚拟机**
- 支持.dex（即Dalvik Executable）格式的java应用程序运行
- 基于寄存器，寄存器CPU的一部分（适合内存和处理器有限的系统）
- 允许有限的内存同时运行多个Dalvik虚拟机的实例
- 每个Dalvik应用作为一个独立的Linux进程执行，防止一个程序崩溃导致所有程序崩溃

![](https://ws1.sinaimg.cn/large/006tNc79ly1fp1zktmabmj30hm0r6780.jpg)

### Jvm的相关知识

- 基于栈（内存的一部分）
- javac把程序源码编译成JAVA字节码后，JVM通过逐条解释字节码翻译成机器指令

### Dalvik和Jvm的区别与联系

#### 图表区别

|        |  本质   |   字节码文件    |  效率  |
| :----: | :---: | :--------: | :--: |
| Dalvik | 基于寄存器 |  一个.Dex文件  |  高   |
|  Jvm   |  基于栈  | 多个.class文件 |  低   |

#### 首要区别

- DVM基于寄存器，Jvm基于栈，基于寄存器的编译花费时间更短（在.dex字节码中，变量会赋值给65535个可用寄存器中的任何一个，Dalvik指令直接操作这些寄存器，而不是访问堆栈中的元素。）
- dex字节码更适合于内存和处理器速度有限的系统
- 基于寄存器的Dalvik实现虽然牺牲了一些**平台无关性**，但是它在代码的执行效率上要更胜一筹。
- 每一个Android 的App是独立跑在一个VM中的。因此一个App crash只会影响到自身的VM，不会影响到其他。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个 Dalvik应用作为一个独立的Linux进程执行。

#### 字节码区别

- JVM字节码由.class组成，每个java文件对应一个.class
- DVM字节码只包含一个.dex文件，这个文件包含了程序中所有的类

![图解.class和.dex文件生成过程](https://ws1.sinaimg.cn/large/006tNc79ly1fp1zfxzxttj30jq0o8tau.jpg)

**Dalvik可执行文件体积小。Android SDK中有一个叫dx的工具负责将Java字节码转换为Dalvik字节码。**

简单来讲，dex格式文件就是将多个class文件中公有的部分统一存放，去除冗余信息。

### ART的相关知识（Android Runtime）

Android Runtime（缩写为 ART），是一种在Android操作系统上的运行环境，由Google公司研发，并在2013年作为Android 4.4系统中的一项测试功能正式对外发布，在Android 5.0及后续Android版本中作为正式的运行时库取代了以往的Dalvik虚拟机。ART能够把应用程序的字节码转换为机器码，是Android所使用的一种新的虚拟机。它与Dalvik的主要不同在于：Dalvik采用的是JIT技术，而ART采用Ahead-of-time（AOT）技术。 ART同时也改善了性能、垃圾回收(Garbage Collection)、应用程序除错以及性能分析。

JIT最早在Android 2.2系统中引进到Dalvik虚拟机中，在应用程序启动时，JIT通过进行连续的性能分析来优化程序代码的执行，在程序运行的过程中，Dalvik虚拟机在不断的进行将字节码编译成机器码的工作。 与Dalvik虚拟机不同的是，ART引入了AOT这种预编译技术，在应用程序安装的过程中，ART就已经将所有的字节码重新编译成了机器码。应用程序运行过程中无需进行实时的编译工作，只需要进行直接调用。因此，ART极大的提高了应用程序的运行效率，同时也减少了手机的电量消耗，提高了移动设备的续航能力，在垃圾回收等机制上也有了较大的提升。 为了保证向下兼容，ART使用了相同的Dalvik字节码文件（dex），即在应用程序目录下保留了dex文件供旧程序调用然而.odex文件则替换成了可执行与可链接格式（ELF）可执行文件。一旦一个程序被ART的dex2oat命令编译，那么这个程序将会指通过ELF可执行文件来运行。因此，相对于Dalvik虚拟机模式，ART模式下Android应用程序的安装需要消耗更多的时间，同时也会占用更大的储存空间（指内部储存，用于储存编译后的代码）,但节省了很多Dalvik虚拟机用于实时编译的时间。

Google公司在Android 4.4中带来的ART模式仅仅是ART的一个预览版，系统默认仍然使用的是Dalvik虚拟机，4.4上面提供的预览版ART相对于Android 5.0以后的ART运行时库有较大的不同，尤其体现在兼容性上。

总结一下上诉内容：

- Android 4.4系统后出现（预览版），系统默认仍然使用的是Dalvik虚拟机，5.0以后是正式版取代了Dalvik虚拟机
- ART能够把应用程序的字节码转换为机器码，是Android所使用的一种新的虚拟机
- 为了保证向下兼容，ART使用了相同的Dalvik字节码文件（dex）（而在安装过程中，会通过dex2oat工具生成OAT文件，具体见下面分析）

### Android运行时ART加载OAT文件过程分析

[查看其他博客分析](http://blog.csdn.net/luoshengyang/article/details/39307813)

- ART核心是OAT文件
- 是APK在安装的过程中，会通过dex2oat工具生成一个OAT文件
- APK安装过程中生成的OAT文件的输入只有一个DEX文件，也就是来自于打包在要安装的APK文件里面的classes.dex文件（实际上，一个OAT文件是可以由若干个DEX生成的）
- OAT文件是一种Android私有ELF
- 它不仅包含有从DEX文件翻译而来的本地机器指令，还包含有原来的DEX文件内容

### Dalvik和ART的区别与联系

- Dalvik和ART使用的的都是.dex字节码，事实上我们把apk解压后确实只有classes.dex文件，但是在ART虚拟机在安装过程中通过dex2oat工具将一个或者诺干个dex生成一个OAT文件
- ART（Ahead-of-time   AOT预编译技术）：应用程序在**安装过程**中，ART将所有的字节码重新编译成了机器码，所以应用程序**运行中**无需进行实时编译工作，只需要进行直接调用，因此，ART极大的提高了应用程序的运行效率，同时也减少了手机的电量消耗，提高了移动设备的续航能力，在垃圾回收等机制上也有了较大的提升，但是安装需要更多的内存空间（存储编译后的代码）和时间，节省了很多Dalvik虚拟机用于实时编译的时间
- Dalvik（Just-in-time  JIT即时编译技术）：（jvm也是JIT即时编译），在Dalvik下，**应用每次运行**都需要通过即时编译器（JIT）将**字节码转换为机器码**，即每次都要编译加运行。虽然安装过程比较快，但是拖慢了应用每次启动的速度

|        |    编译技术    | （时机）字节码编译机器码 |   占用空间    | 安装时间 | 运行效率（） |
| :----: | :--------: | :----------: | :-------: | ---- | :----: |
| Dalvik | JIT（即时编译）  |    首次安装时     |     小     | 快    |   慢    |
|  ART   | AOT（预编译技术） |   应用每次启动时    | 大（10-20%） | 满    |   快    |

### 几张截图总结

[查看原文](http://blog.csdn.net/jason0539/article/details/50440669)

![](https://ws1.sinaimg.cn/large/006tNc79ly1fp230bcp69j31ie11m4e4.jpg)

![](https://ws3.sinaimg.cn/large/006tKfTcgy1fp230q7envj31hw0l277h.jpg)