---
title: ReactNative的prop-types属性验证问题
date: 2018-08-02 16:12:54
tags:
- RN
categories: RN
password:
---

使用 React Native 创建的组件是可以复用的，所以我们开发的组件可能会给项目组其他同事使用。但别人可能对这个组件不熟悉，常常会忘记使用某些属性，或者某些属性传递的数据类型有误。

<!--more-->

在开发 React Native 自定义组件时，可以通过属性确认来声明这个组件需要哪些属性。这样，如果在调用这个自定义组件时没有提供相应的属性，则会在手机与调试工具中弹出警告信息，告知开发者该组件需要哪些属性。

# 新版本的问题

React Native已经升级到0.51.0了，版本升级很快，但是对老项目也会有一些问题，常见的就是属性找不到的问题。例如：



<img src="https://ws1.sinaimg.cn/large/006tKfTcgy1ftvfe47ylmj30d40nkjt9.jpg" width="200"/>

主要原因是随着React Native的升级，系统废弃了很多的东西，过去我们可以直接使用 React.PropTypes 来进行属性确认，不过这个自 React v15.5 起就被移除了，转而使用prop-types库来进行替换

# 如何使用

## 安装

**需要先安装这个第三方库**

`npm install --save prop-types`

安装成功如下：

<img src="https://ws1.sinaimg.cn/large/006tKfTcgy1ftvfhhy65bj30y40j274r.jpg" width="200"/>

## 导入

`import propTypes from 'prop-types';`

## 验证

```
static propTypes = {
  title: PropTypes.string,
  leftIcon: PropTypes.string,
  rightIcon: PropTypes.string,
  leftPress: PropTypes.func,
  rightPress: PropTypes.func,
  style: PropTypes.object
 }
```

这样，如果在调用这个自定义组件时没有提供相应的属性，则会在手机与调试工具中弹出警告信息，告知开发者该组件需要哪些属性。

# 完整语法

1，要求属性是指定的 JavaScript 基本类型。例如：

属性: PropTypes.array,
属性: PropTypes.bool,
属性: PropTypes.func,
属性: PropTypes.number,
属性: PropTypes.object,
属性: PropTypes.string,

2，要求属性是可渲染节点。例如：

属性: PropTypes.node,

3，要求属性是某个 React 元素。例如：

属性: PropTypes.element,

4，要求属性是某个指定类的实例。例如：

属性: PropTypes.instanceOf(NameOfAClass),

5，要求属性取值为特定的几个值。例如：

属性: PropTypes.oneOf(['value1', 'value2']),

6，要求属性可以为指定类型中的任意一个。例如：

属性: PropTypes.oneOfType([
  PropTypes.bool,
  PropTypes.number,
  PropTypes.instanceOf(NameOfAClass),
])

7，要求属性为指定类型的数组。例如：

属性: PropTypes.arrayOf(PropTypes.number),

8，要求属性是一个有特定成员变量的对象。例如：

属性: PropTypes.objectOf(PropTypes.number),

9，要求属性是一个指定构成方式的对象。例如：

属性: PropTypes.shape({
  color: PropTypes.string,
  fontSize: PropTypes.number,
}),

10，属性可以是任意类型。例如：

属性: PropTypes.any

**将属性声明为必须**

使用关键字 isRequired 声明它是必需的。

属性: PropTypes.array.isRequired,
属性: PropTypes.any.isRequired,
属性: PropTypes.instanceOf(NameOfAClass).isRequired,

## 源码（prop-types.js）

```javascript
/**
 * Copyright (c) 2015-present, Facebook, Inc.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 * @nolint
 * @format
 */

// TODO (bvaughn) Remove this file once flowtype/flow-typed/pull/773 is merged

type $npm$propTypes$ReactPropsCheckType = (
  props: any,
  propName: string,
  componentName: string,
  href?: string,
) => ?Error;

declare module 'prop-types' {
  declare var array: React$PropType$Primitive<Array<any>>;
  declare var bool: React$PropType$Primitive<boolean>;
  declare var func: React$PropType$Primitive<Function>;
  declare var number: React$PropType$Primitive<number>;
  declare var object: React$PropType$Primitive<Object>;
  declare var string: React$PropType$Primitive<string>;
  declare var any: React$PropType$Primitive<any>;
  declare var arrayOf: React$PropType$ArrayOf;
  declare var element: React$PropType$Primitive<any>; /* TODO */
  declare var instanceOf: React$PropType$InstanceOf;
  declare var node: React$PropType$Primitive<any>; /* TODO */
  declare var objectOf: React$PropType$ObjectOf;
  declare var oneOf: React$PropType$OneOf;
  declare var oneOfType: React$PropType$OneOfType;
  declare var shape: React$PropType$Shape;

  declare function checkPropTypes<V>(
    propTypes: $Subtype<{[_: $Keys<V>]: $npm$propTypes$ReactPropsCheckType}>,
    values: V,
    location: string,
    componentName: string,
    getStack: ?() => ?string,
  ): void;
}

```

[参考](https://www.jb51.net/article/130879.htm)