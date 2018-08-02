---
title: ReactNative碎片整理之组件生命周期
date: 2018-08-01 17:27:52
tags:
- RN
categories: RN
password:
---

生命周期是一定要知道的，这样才知道整个组件的工作流程，知道哪些操作适合在哪个回调中进行……(o^^o)

<!--more-->

# 一个栗子

```javascript
import React, {Component} from 'react';
import {
    View,
    Text,
    StyleSheet,
} from 'react-native';


export default class App extends Component<Props> {

    /**
     * 构造函数，初始化需要的state
     * @param props
     */
    constructor(props) {
        super(props);

        console.log('constructor')

        // 初始状态
        this.state = {
            result: '点击前',
        };
    }

    /**
     * 描述：控件渲染前触发
     * 次数：一次
     * 推荐：
     */
    componentWillMount() {
        console.log('componentWillMount')
    }

    /**
     * 描述：控件渲染后触发
     * 次数：一次
     * 推荐：
     */
    componentDidMount() {
        console.log('componentDidMount')
    }

    /**
     * 描述：组件接收到新的props时被调用
     * 次数：多次
     * 推荐：
     */
    componentWillReceiveProps() {
        console.log('componentWillReceiveProps')
    }

    /**
     * 描述：当组件接收到新的props和state时被调用
     * 次数：多次
     * 推荐：
     */
    shouldComponentUpdate() {
        console.log('shouldComponentUpdate')
        return true;
    }

    /**
     * 描述：组件重新渲染完成后会调用此方法
     * 次数：多次
     * 推荐：
     */
    componentDidUpdate() {
        console.log('componentDidUpdate')
    }


    /**
     * 描述：控件渲染后触发
     * 次数：1次
     * 推荐：
     */
    componentDidMount() {
        console.log('componentDidMount')
    }


    /**
     * 描述：组件卸载和销毁之前被调用
     * 次数：一次
     * 推荐：用于清理一些无用的内容，比如：定时器清除
     */
    componentWillUnmount() {
        console.log('componentWillUnmount')
    }

    componentWillUpdate() {
        console.log('componentWillUpdate')
    }
    render() {
        console.log('render')
        return (
            <View style={styles.content}>
                <Text style={styles.text} onPress={
                    () => {
                        this.setState({
                            result: '点击后'
                        })
                    }
                }>
                    {this.state.result}
                </Text>
            </View>
        )
    }
}

const styles = StyleSheet.create({
    content: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
    },
    text: {
        padding: 50,
        fontSize: 20,
        color: '#ffffff',
        backgroundColor: '#00a056'
    }
})
```

**console**

```javascript
constructor
App.js:39 componentWillMount
App.js:103 render
App.js:86 componentDidMount
App.js:108 onPress call
App.js:66 shouldComponentUpdate
App.js:100 componentWillUpdate
App.js:103 render
App.js:76 componentDidUpdate
```

# 组件的生命周期

通过上面的栗子，有灵性的小哥哥已经基本猜到大致的生命周期了

下面再看看这张我偷来的帅图

<img src="https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/ReactNativeLifeCycle.png" width="400">

再结合这张酷表

| 生命周期                                     | 调用次数      | 能否使用 setSate() | 描述                                       |
| ---------------------------------------- | --------- | -------------- | ---------------------------------------- |
| getDefaultProps（es6:static defaultProps） | 1(全局调用一次) | 否              | 初始化默认属性                                  |
| getInitialState(es6:constructor(props))  | 1         | 否              | 构造函数，初始化需要的state                         |
| componentWillMount                       | 1         | 是              | 控件渲染前触发                                  |
| render                                   | >=1       | 否              | 渲染控件的方法                                  |
| componentDidMount                        | 1         | 是              | 控件渲染后触发                                  |
| componentWillReceiveProps                | >=0       | 是              | 组件接收到新的props时被调用                         |
| shouldComponentUpdate                    | >=0       | 否              | 当组件接收到新的props和state时被调用                  |
| componentWillUpdate                      | >=0       | 否              | props或者state改变，并且此前的shouldComponentUpdate方法返回为 true会调用该方法 |
| componentDidUpdate                       | >=0       | 否              | 组件重新渲染完成后会调用此方法                          |

另外还有一个场景需要知道的，代码我就不贴了

父组件中有一个子组件，点击父组件调用setState，那么父子组件的生命周期如何回调？

```
//加载过程
constructor
App.js:42 componentWillMount
App.js:107 render
ChildComponent.js:26 ChildComponent,constructor
ChildComponent.js:41 ChildComponent,componentWillMount
ChildComponent.js:106 ChildComponent,render
ChildComponent.js:88 ChildComponent,componentDidMount
App.js:89 componentDidMount
//点击后，调用父组件的setState后
shouldComponentUpdate
App.js:103 componentWillUpdate
App.js:107 render
ChildComponent.js:59 ChildComponent,componentWillReceiveProps
ChildComponent.js:68 ChildComponent,shouldComponentUpdate
ChildComponent.js:102 ChildComponent,componentWillUpdate
ChildComponent.js:106 ChildComponent,render
ChildComponent.js:78 ChildComponent,componentDidUpdate
App.js:79 componentDidUpdate
```

可以看到：

- 组件的加载是从内到外一级一级的，这个和android类似
- 修改父组件state，触发update整个流程，但是不再次触发子组件的constructor

# 推荐的操作

- constructor()方法里初始化state 
- static defaultProps指定默认属性
- componentDidMount()：该方法在render()方法后自动调用，网络请求一般放在这个方法中
- shouldComponentUpdate()：该方法返回一个boolean值，用来决定是否需要重新渲染组件，默认返回true，你可以自己重写此方法，通过条件判断来决定你是否需要更新组件
- componentWillUnmount()：在组件被移除前调用，在该方法中，释放一些不需要的资源，比如停止定时器
- **不要在 constructor 或者 render 里 setState()**
- constructor 已含 this.state={} 
- render 里 setState 会造成setState -> render -> setState -> render 
- 能做的setState，只要是render前，就放在componentWillMount，render后，就放在 componentDidMount。這两个 function 是 react lifecycle 中，最常使用的两个
- 更多以后再添加总结



# 参考

[React-Native生命周期的触发场景和一些小建议](https://blog.csdn.net/ddwhan0123/article/details/78490884)

[React Native 中组件的生命周期](https://blog.csdn.net/sinat_30949835/article/details/79833721)

[React Native之React速学教程(中)](http://www.devio.org/2016/08/10/React-Native%E4%B9%8BReact%E9%80%9F%E5%AD%A6%E6%95%99%E7%A8%8B-(%E4%B8%AD)/)

