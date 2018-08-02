---
title: ReactNative碎片整理之State
date: 2018-07-28 17:51:46
tags:
- RN
categories: RN
password:
---

我们使用两种数据来控制一个组件：props和state。props是在父组件中指定，而且一经指定，在被指定的组件的生命周期中则不再改变。 对于需要改变的数据，我们需要使用state。

<!--more-->

state属性主要用来存储组件自身需要的数据，是组件自己私有的，我们一般通过修改 state 属性的值来更新数据，React 内部会监听 `state` 的变化，一旦发生变化就会主动触发组件的 render() 方法来更新 Dom 结构

### state的使用

一般来说，你需要在 constructor()方法中初始化 state（这是ES6的写法，ES5 中一般在 `getInitialState()` 方法中来初始化 `state`），然后在需要修改时调用 setState() 方法。

不要使用 `this.state` 来修改 `state` 属性值，应该调用 `setState()` 方法，`this.state` **是无效的**。

### setState（）方法

`setState()` 的完整表达式：

`setState(updater, [callback])`

`setState()` 方法会把对组件 state 的改变加入到队列中，并且告诉 React 这个组件及其子组件需要重新渲染。

#### 第一个参数：`updater` 函数

`setState(updater, callback)` 方法的第一个参数是一个固定格式的 `updater` 函数：

```javascript
(prevState, props) => stateChange
```

`prevState` 是一个对之前状态（previous state）的引用，我们是不能直接修改这个参数的值，要想修改 `state` 的值，我们应该根据 `prevState` 和 `props` 参数来创建一个新的 JavaScript 对象。

**例子1：**

```javascript
this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
});
```

你也可以传一个对象而不是函数，来作为`setState(updater, callback)` 方法的第一个参数，React 会将该参数 merge 到 state 中。

**例子2:**

```
this.setState({quantity: 2});
```

#### 第二个参数：callback

`setState(updater, callback)` 方法的第二个参数 `callback` 是一个可选参数。（基本是空的，一般不这么使用）

为了更好的性能表现，React 并不能保证 `setState()` 一被调用 state 就能更新。所以，如果在调用 `setState()` 之后，马上就读取 `this.state` 的值的话，可能会出现误差。

因此，这种情况下，推荐使用 `componentDidUpdate` 或者 `setState(updater, callback)` 方法的 `callback` 来获取最新的状态。

**React 官方更推荐**使用 `componentDidUpdate()`，而不是 `callback` 来监听 update 事件（注： 除非 `shouldComponentUpdate()` 方法返回 `false`，`setState()` 将永远都会引发重新渲染）。

**例子1:**

```javascript
import React, {Component} from 'react';
import {
    View,
    StyleSheet,
    TouchableOpacity,
    ToastAndroid,
    Platform,
    Text,
} from 'react-native';
export default class App extends Component<Props> {
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            num: 1
        };
    }

    shouldComponentUpdate() {
  		//返回 false 则setState不刷新
        return false;
    }

    static defaultProps = {
        add: 100
    }

    componentDidUpdate() {
        if(Platform.OS=='ios'){
            return;
        }
        ToastAndroid.show(""+this.state.num,ToastAndroid.SHORT)
    }

    render() {
        return (
            <View style={{flex:1,justifyContent:'center',alignItems: 'center'}}>
                <TouchableOpacity onPress={() => {
                    this.setState((prevState, props) => {
                        return {num: prevState.num + props.add};
                    });
                }} style={styles.container}>
                    <Text style={styles.text}>{this.state.num}</Text>
                </TouchableOpacity>
            </View>
        )
    }
}
const styles = StyleSheet.create({

    container: {
        backgroundColor: '#00a056',
        paddingRight: 70,
        paddingTop: 10,
        paddingBottom: 10,
        paddingLeft: 70,
        shadowColor: '#00a056',
        shadowOffset: {width: 3, height: 3},
        shadowOpacity: 0.5,
        shadowRadius: 4
    },
    text: {
        color: '#ffffff',
        fontSize: 30,

    }
})

```

<img src="https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/state.gif" width="600">

**如上图效果：**

- shouldComponentUpdate 不重写或者返回true，则象左侧ios模拟器一样，可以修改状态
- 返回false，则不重写
- 官方不推荐我们使用setState的第二个参数Callback的形式获取，我们可以在componentDidUpdate回调中，获取最新的State

### 如何正确操作state

#### 不要使用`this.state`来修改`state`

```
// Wrong
this.state.comment = 'Hello';
```

记住，`this.state` 是不可变的。

```javascript
//事实证明，并改变不了
render() {
    return (
        <View style={{flex: 1, justifyContent: 'center', alignItems: 'center'}}>
            <TouchableOpacity onPress={() => {
                this.state.num = 3
            }} style={styles.container}>
                <Text style={styles.text}>{this.state.num}</Text>
            </TouchableOpacity>
        </View>
    )
```

应该调用 `setState()` 方法：

```javascript
// Correct
this.setState({comment: 'Hello'});
//或者
this.setState((prevState, props) => {
                        return {num: prevState.num + props.add};
                    });
//prevState 可以获取上个state 数值
//props 可以获取当前props所有数值，可以做一些操作


//不要这么使用
this.setState((prevState, props) => {
                        return {num: this.state.num+1};
                    });
//考虑到setState可能是异步调用的，所以推荐使用prevState（上一个值），而不是依赖this.state来计算最新的state，这样可能会有误差
```

#### state的改变是一个覆盖（合并）过程

当你调用 `setState()` 方法时，React 会将将你当前所提供的对象合并到当前的状态中。

**例子1:**

```javascript
constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }

componentDidMount() {

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

⬆面的栗子，state更新至替换了comments，posts不受影响

### state传递？

**注意：**

- 父组件和子组件之间不能通过 state 来交互，父组件只能将自己的 state 值传给子组件的 props。


- 这种数据传递的方式通常被称为 “自顶向下（top-down）”或者“单向（unidirectional）”数据流。
- 任何state都是由一个特定的组件所拥有的，任何state数据修改默认只会影响当前组件，
- 可以通过回调传值（后面会整理）
- **React Native建议由顶层的父组件定义state值，并将state值作为子组件的props属性值传递给子组件，这样可以保持单一的数据传递。**

### 参考

[[【1】React Native] React 中的状态（State）](https://www.jianshu.com/p/583473c37db9)





<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=455304006&auto=1&height=32"></iframe>

