---
title: transient关键字的作用
date: 2017-03-15 20:51:11
tags: 
- Android
categories: notes
password: 123456
---

**前言**

transient的作用及使用方法，官方解释：

```java
Variables may be marked transient to indicate that they are not part of the persistent state of an object.
```

<!--more-->

**遇到问题**

我们都知道一个对象只要实现了Serializable接口，这个对象就可以被序列化，那么在实际开发过程中，可能会遇到这样的问题，这个类的有些属性需要序列号，而其他属性不需要序列化，比如说：一个用户有一些敏感信息，比如密码或者银行卡号之类，为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

**解决问题**

再次之前，有一个姿势点：

父类序列化

> **一个情景**：一个子类实现了Serializable接口，它的父类都没有实现Serializable接口，序列化该子类对象，然后反序列化后，输出父类定义的某变量的数值，该变量数值与序列化前不同

**解决**：**要想将父类对象也序列化，就需要让父类也实现Serializable 接口**。如果父类不实现的话的，就 **需要有默认的无参的构造函数**。在父类没有实现 Serializable 接口时，虚拟机是不会序列化父对象的，而一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。所以反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。如果你考虑到这种序列化的情况，在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如 int 型的默认是 0，string 型的默认是 null。

Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

**特性使用案例**

我们熟悉使用 Transient 关键字可以使得字段不被序列化，那么还有别的方法吗？根据父类对象序列化的规则，我们可以将不需要被序列化的字段抽取出来放到父类中，子类实现 Serializable 接口，父类不实现，根据父类序列化规则，父类的字段数据将不被序列化

![](http://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/8u215.png)

上图中可以看出，attr1、attr2、attr3、attr5 都不会被序列化，放在父类中的好处在于当有另外一个 Child 类时，attr1、attr2、attr3 依然不会被序列化，不用重复抒写 transient，代码简洁。



**参考：**

[java学习—序列化与Transient关键字](http://blog.csdn.net/janronehoo/article/details/7306900)

[Transient关键字使用场景](http://blog.csdn.net/hushaoxi/article/details/52385614)