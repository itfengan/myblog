---
title: ReactNative碎片整理之ref
date: 2018-08-01 10:11:32
tags:
- RN
categories: RN
password:
---

> 概述

ref属性是一个特殊的属性，可以把它挂载到任何组件

它可以是一个回调函数（也可以是一个字符串，基本废弃这种用法）

这个回调函数在组件被挂载后立即被执行，应用到的组件作为参数传递

回调函数可以立即使用组件，也可以将参数的组件引用保存起来，后续使用（调用组件方法或者获取组件参数）

<!--more-->

# ref属性指定回调方法

## 栗子：

```javascript
import React, {Component} from 'react';
import {
    View,
    Text,
    StyleSheet,
} from 'react-native';


export default class App extends Component<Props> {
    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            result: '点击前'
        };
    }

    render() {
        return (
            <View style={styles.content}>
                <Text style={styles.button} onPress={
                    () => {
                        this._Test.print()
                    }
					//() => {
                    //    this.setState((prestate, props) => {
                    //       console.warn("setState")
                    //        return {result: '点击了'}
                    //    })
                    }
                }>{this.state.result}</Text>
                <RefTest ref={
                    //指定ref属性的回调函数
                    (e) => {
                        console.warn("ref的回调执行了！")
                        //参数e则为当前RefTest组件的引用
                        this._Test = e;
                    }
                }>
                </RefTest>
            </View>
        )
    }
}

const styles = StyleSheet.create({
    content: {
        flex: 1,
        backgroundColor: '#00a056',
        flexDirection: 'row'
    },
    button: {
        padding: 30,
        backgroundColor: '#ffffff',
        fontSize: 17,
        alignSelf: 'center'
    },
    text: {
        padding: 30,
        backgroundColor: '#ffffff',
        fontSize: 30,
    }
})

class RefTest extends Component<Props> {

    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            data: 'old'
        };
    }

    print() {
        this.setState((preState, props) => {
            return {
                data: 'new'
            }
        })
        console.warn("print方法执行了：" + this.state.data)
    }

    render() {
        return (
            <View style={{
                alignSelf: 'center',
                left: 10,
                backgroundColor: '#ffff00',
            }}>
                <Text style={styles.text}>
                    当前数据：{this.state.data}
                </Text>
            </View>
        );
    }
}
```

<img src="https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/ref1.gif" width="400">

可以看到：

- 父组件拥有的子组件，子组件有ref属性
- 首次进入的时候，组件初次绑定 ref的回调会执行一次，父组件保存该引用
- 点击后，使用组件引用，调用组件方法print
- 若点击后，父组件setState，则子组件 卸载后，重新调用子组件Rend（）方法，这个过程，子组件的ref回调会被调用两次（卸载一次，挂载一次）

需要知道的：

- 在父组件在子组件指定ref的回调中保存引用对子组件的
- 这个回调在父组件每次setState（父Render（）调用）
- 组件的render方法被调用时，ref才会被调用，组件才会返回ref
- setState涉及先卸载，后挂载，ref回调执行两次

