---
title: ReactNative start
date: 2017-12-25 15:52:32
tags: 
- Android
categories: notes
---

# ReactNative start

<!--more-->

## 1，为什么学习ReactNative

### 好处

- 因为现在许多主流的应用都有React Native的影子，它对比原生开发更为灵活，对比H5体验更为高效，而且跨平台的支持特性。


- 相对ionic这类PhoneGap，它效率更高，和原生之间的交互更方便。


- 多个版本迭代后的今天，它已经拥有了丰富第三方插件支持。


- React Native解决不了的，可以通过各位熟悉的原生来解决。


- 更方便的热更新。

### 坏处

- 尽管是跨平台，但是不同平台Api的特性与显示并不一定一致。
- 相对增大了app的体积。
- 调试’相对‘麻烦。
- Android上的兼容性问题。

### 总的来说

React Native适合作为项目中的补充，而不是作为核心去开发APP。

## 2，搭建开发环境

- Webstrom（号称web开发神器，目前对前端的了解有限，因为和android stuido师出同门，所以不商量直接选用他，当然stom也试了试，但是快捷键不如Webstrom熟悉）

- HomeBrew（Mac系统的包管理器，用于安装NodeJS和一些其他必需的工具软件

- Node（基于 Chrome V8 引擎的 JavaScript 运行环境）

- Androd Studio（安卓老本家没啥说的）

- ReactNative的命令行工具（react-native-cli，React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务

  [详细搭建步骤查看ReactNative中文网](https://reactnative.cn/docs/0.31/getting-started.html#content)

一步一步安装完毕后，没错，你已经起飞了；

## 3，创建一个项目

- 终端cd到你想初始化项目的文件夹

- ```
  react-native init FirstRNApp
  ```


- 然后会发现创建了一个FirstRNApp文件夹，这就是创建的第一个项目

![](https://upload-images.jianshu.io/upload_images/3673902-0affa2b6b71fc650.jpeg?imageMogr2/auto-orient/)

![项目目录](https://ws1.sinaimg.cn/large/006tNc79gy1foabtgnbqjj30bc0k20ua.jpg)

## 几个关键的文件

- **android**文件夹，就是一个可以用android studio打开的android项目。
- **ios**文件夹，是一个可以用xcode打开的ios项目。
- **index.android.js**，这是android的React Native入口文件。
- **index.ios.js**，这是ios的React Native入口文件。
- **package.json**，类似android studio的build.gradle，你依赖的库都写在里面。
- **node_module**文件夹，你依赖的库下载下来都存放在里面，属于git的忽略文件，你要找的依赖库源码也在里面，包括React和React Native。
- **jscode**文件夹，是自己创建的文件夹，用来存放自己写的js文件。

### 需要注意的点

**package.json**

类似于android studio中的build.gradle添加远程依赖，不同的是，package.json大多数时候不需要我们手动添加，我们只需要在根目录通过命令行，`npm install xxxxxx --save` 就可以依赖一个库了。

**install**之后，库的依赖信息，自动被写到package.json里面，对应的库也会被下载到node_module文件夹中，类似android studio依赖后把aar同步到本地。

![package.json](https://ws2.sinaimg.cn/large/006tNc79gy1foabyhxltmj30xe0m6n04.jpg)

------

**node_module**

**node_module**是一个忽略文件，提交的时候不需要提交到git上，类似android studio远程依赖下来的aar，也不会提交到git上。其他人在使用React Native项目时，只需要npm install，工程就会根据package.json，去同步下载各个依赖库到node_module。

**注**：有时候还需要运行`react-native link` 或 `react-native link xxx`，这是因为有些第三方库是通过原生代码加React Native实现的，通过这个命令，可以自动把相关的配置代码，自动添加到android和ios工程中。

------

## 运行这个项目

作为安卓端有两种方式，ios也同样有两种

- cd到项目根目录

- ```
  react-native run-android【react-native run-ios】
  ```


- 或者用android stuido直接打开项目中的android文件夹（上面有说过，这是一个可以独立打开的安卓项目）

### 遇到的小坑

#### 1，**无法从资产的index.android.bundle中加载脚本。确保你的包被正确打包或者你正在运行一个packager服务器**

![bundle无法加载报错](https://ws4.sinaimg.cn/large/006tNc79gy1foac7mc3cuj30re0zowmx.jpg)

**搜索了一波，这个错误很常见，解决的办法也很常见，如下**

- 打开自己的项目文件夹,在Android/app/src/main目录下创建一个空的assets文件夹
- cd到项目根目录，执行

```
react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
```

- 事实证明并没有解决我的问题
- 解决我的问题的是下面这行

```
react-native bundle --platform android --dev false --entry-file index.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res
```

![两者的区别](https://ws4.sinaimg.cn/large/006tNc79gy1foacbagifbj31dc0b8wk6.jpg)

**我们可以回头看上面创建完项目的截图，老版的是分别有index.android.js和index.ios.js**

**而我创建的却没有这两个文件，只有一个index.js,修改完之后，我们创建的assets目录下会生成bundle的两个文件**

## 运行成功（android端）

![运行成功截图](https://ws1.sinaimg.cn/large/006tNc79gy1foacebj37wj30n014oqco.jpg)

相关资料

[恋猫月亮博客]https://www.jianshu.com/p/97692b1c451d)

[中文网](https://reactnative.cn/docs/0.31/getting-started.html#content)

[报错](http://blog.csdn.net/DavisCZ/article/details/79072062)

本文仅作个人学习总结，如有出入不够严谨的地方，请联系更改。

**ReactNative才刚刚起步学习，麻烦和坑后续还会经常碰到。但是，既然选择了学习，就要坚持下来！**

