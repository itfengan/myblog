---
title: ReactNative碎片整理之Props
date: 2018-04-28 14:28:33
tags:
- RN
categories: RN
password:
---

props的属性都是父组件传递的，所以无法修改它，但是我们可以指定默认prop；

<!--more-->

### 基本介绍

大多数组件创建的时就可以使用各种参数来定制（可以理解为Android自定义View的传值）。但这是不准确的，props在RN中代表着属性，是为了描述一个组件的特征而存在，它是由父组件传递给子组件的（调用方传入）而且一经指定，在被指定的组件的生命周期则不在改变（这与state不同），所以要使用，我们首先在父组件中定义子组件的属性，但不可以添加默认属性（可以在子组件中添加默认属性），props除了传递属性值，还有约束作用...

### 普通使用

以常见的基础组件`Image`为例，在创建一个图片时，可以传入一个名为`source`的 prop 来指定要显示的图片的地址，以及使用名为`style`的 prop 来控制其尺寸。

```javascript
import React, { Component } from 'react';
import { Image } from 'react-native';

export default class Bananas extends Component {
  render() {
    let pic = {
      uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg'
    };
    return (
      <Image source={pic} style={{width: 193, height: 110}} />
    );
  }
}

```

以上就是prop的最简单的理解，基础组件`Image`的`style`和`resource`都是它的`prop`属性

### 自定义Props

```javascript
import React, {Component} from 'react';
import {
    View,
    StyleSheet,
    Alert,
    TouchableOpacity,
    Text,
} from 'react-native';
class MyText extends Component {
    // 构造
    constructor(props) {
        super(props);
    }

    //指定默认props
    static defaultProps = {
        myText: '默认值'
    }

    render() {
        return (
            <TouchableOpacity onPress={() => {
                Alert.alert('Alert')
            }}>
                <Text style={styles.continer}>
                    hello {this.props.myText}!
                </Text>
            </TouchableOpacity>
        )
    }
}

export default class App extends Component<Props> {
    render() {
        return (
            <View style={{alignItems: 'center'}}>
                <MyText myText='测试'/>
                <MyText myText='android'/>
                <MyText myText='ios'/>
                {/*传入未指定的pros*/}
                <MyText test='hello'/>
                {/*未传入pros*/}
                <MyText />
            </View>
        )
    }
}
const styles = StyleSheet.create({
    continer: {
        color: '#ffffff',
        fontSize: 17,
        backgroundColor: '#00a056',
        margin: 20,
        padding: 10,
        shadowColor: '#00a056',
        shadowOffset: {width: 3, height: 3},
        shadowOpacity: 0.5,
        shadowRadius: 4
    }
})
```

<img src="https://ws3.sinaimg.cn/large/006tKfTcgy1ftpkp21prij30bc0m1q2z.jpg" width='200'/>

### props约束

使用PropTypes 已经被取消了

[react native关于 从react中引入PropTypes报错的问题](https://blog.csdn.net/allangold/article/details/78843333)

### 方法中调用this.props的问题

```javascript
import React, {Component} from 'react';
import {
    View,
    StyleSheet,
    Alert,
    TouchableOpacity,
    Text,
} from 'react-native';

class MyText extends Component {
    // 构造
    constructor(props) {
        super(props);
    }
    //指定默认props
    static defaultProps = {
        name: '冯英俊',
        age: 1,
    }

    //点击弹出传入的信息
    _onClickEvent() {
        Alert.alert(this.props.name, "年纪" + this.props.age)
    }
    render() {
        return (
            <TouchableOpacity onPress={this._onClickEvent}>
                <Text style={styles.continer}>
                    hello {this.props.name}你的年纪是{this.props.age}!
                </Text>
            </TouchableOpacity>
        )
    }
}

export default class App extends Component<Props> {
    render() {
        return (
            <View style={{alignItems: 'center'}}>
                <MyText name='刘德华' age={20}/>
                <MyText age={18}/>
                <MyText name='刘德华' age={22}/>
                <MyText name='刘德华' age={30}/>

            </View>
        )
    }
}
const styles = StyleSheet.create({
    continer: {
        color: '#ffffff',
        fontSize: 17,
        backgroundColor: '#00a056',
        margin: 20,
        padding: 10,
        shadowColor: '#00a056',
        shadowOffset: {width: 3, height: 3},
        shadowOpacity: 0.5,
        shadowRadius: 4
    }
})
```

<img src="https://ws3.sinaimg.cn/large/006tKfTcgy1ftplzf9renj30bc0m174y.jpg" width='200'/>

[React Native绑定this(bind(this))](https://www.jianshu.com/p/ce970791aba4)

[【React Native】React Native的bind方法](https://blog.csdn.net/sinat_34693148/article/details/72631956)

```javascript
_onClickEvent() {
    Alert.alert(this.props.name, "年纪" + this.props.age)
}
//修改为：
render() {
    return (
        <TouchableOpacity onPress={() => this._onClickEvent()}>
            <Text style={styles.continer}>
                hello {this.props.name}你的年纪是{this.props.age}!
            </Text>
        </TouchableOpacity>
    )
}
//或者
render() {
        return (
            <TouchableOpacity onPress={() =>{
                Alert.alert(this.props.name, "年纪" + this.props.age)
            }}>
                <Text style={styles.continer}>
                    hello {this.props.name}你的年纪是{this.props.age}!
                </Text>
            </TouchableOpacity>
        )
    }
//或者
render() {
        return (
            <TouchableOpacity onPress={this._onClickEvent.bind(this)}>
                <Text style={styles.continer}>
                    hello {this.props.name}你的年纪是{this.props.age}!
                </Text>
            </TouchableOpacity>
        )
    }
//或者 箭头函数完成bind
 render() {
        return (
            <TouchableOpacity onPress={() => {
                this._onClickEvent('hello')
            }}>
                <Text style={styles.continer}>
                    hello {this.props.name}你的年纪是{this.props.age}!
                </Text>
            </TouchableOpacity>
        )
    }
```

**关于ReactNative的this更多内容后面会再整理**

### 参考：

[ReactNative中文网](https://reactnative.cn/docs/props.html)