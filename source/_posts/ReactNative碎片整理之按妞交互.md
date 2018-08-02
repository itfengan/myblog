---
title: ReactNative碎片整理之按妞交互
date: 2018-08-02 13:48:22
tags:
- RN
categories: RN
password:
---

ReactNative的Button是一个简单的跨平台按钮组件，可以进行一些简单的定制，但是支持的props有限，一般我们会使用TouchableXXX一系列的组件来定制我们需要的按钮

<!--more-->

# Button

```javascript
render() {
    console.log('render')
    return (
        <View style={styles.content}>
            <Button
                //注意 没有style属性，设置无卵用
                style={{
                    backgroundColor:'#ffffff',
                    color:"#00aaaa"
                }}
                onPress={() => Alert.alert('hello')}
                // disabled={true}
                title="click"
                color="#00a056"
            />
        </View>
    )
}
```

查看Props

| props                | 描述                           | 必填   |
| -------------------- | ---------------------------- | ---- |
| `onPress`            | 用户点击此按钮时所调用的处理函数             | 是    |
| `title`              | 按钮内显示的文本                     | 是    |
| `color`              | 文本的颜色(iOS)，或是按钮的背景色(Android) | 否    |
| `disabled`           | 设置为 true 时此按钮将不可点击。          | 否    |
| `accessibilityLabel` | 用于给残障人士显示的文本                 | 否    |
| `testID`             | 用来在端到端测试中定位此视图。              | 否    |

通过上面的属性表格，并没有style属性（可以外部包一层View来定制），可以知道这个Button的组件的样式是有局限的，并且color属性，两端并不统一（一个是文本颜色，一个是背景色），使用非常简单，不再代码演示了。

# TouchableXXX系列组件

TouchableXXX系列组件有以下四种：

| 名称                       | 描述                                       |
| ------------------------ | ---------------------------------------- |
| TouchableWithoutFeedback | 响应用户的点击事件，如果你想在处理点击事件的同时不显示任何视觉反馈，使用它是个不错的选择 |
| TouchableHighlight       | 在TouchableWithoutFeedback的基础上添加了当按下时背景会变暗的效果，可以指定按下抬起的颜色等。 |
| TouchableOpacity         | 相比TouchableHighlight在按下去会使背景变暗的效果，TouchableOpacity会在用户手指按下时降低按钮的透明度，而不会改变背景的颜色。 |
| TouchableNativeFeedback  | 在Android上还可以使用TouchableNativeFeedback，它会在用户手指按下时形成类似水波纹的视觉效果。**注意，此组件只支持Android。** |

**四者的关系？**

```javascript
//TouchableHighlight
var TouchableHighlight = React.createClass({
  propTypes: {
    ...TouchableWithoutFeedback.propTypes,
//TouchableOpacity
var TouchableOpacity = React.createClass({
  mixins: [TimerMixin, Touchable.Mixin, NativeMethodsMixin],

  propTypes: {
    ...TouchableWithoutFeedback.propTypes,
//TouchableNativeFeedback
var TouchableNativeFeedback = React.createClass({
  propTypes: {
    ...TouchableWithoutFeedback.propTypes,
```

> **可以看出：**

因为TouchableWithoutFeedback有其它三个组件的共同属性，所以我们先来学习一下TouchableWithoutFeedback。

接下来分别记录的具体使用

> **注意一点：**

无论是TouchableWithoutFeedback还是其他三种Touchable组件，都是在根节点都是只支持一个组件，如果你需要多个组件同时相应单击事件，可以用一个View将它们包裹着，它的这种根节点只支持一个组件的特性和ScrollView很类似。



> TouchableWithoutFeedback一个Touchable系列组件中最基本的一个组价，只响应用户的点击事件不会做任何UI上的改变，在使用的过程中需要特别留意。

## 使用详情

[详情点击查看：中文网TouchableWithoutFeedback](https://reactnative.cn/docs/touchablewithoutfeedback.html)

[React Native按钮详解|Touchable系列组件使用详解](http://www.devio.org/2017/01/10/React-Native%E6%8C%89%E9%92%AE%E8%AF%A6%E8%A7%A3-Touchable%E7%B3%BB%E5%88%97%E7%BB%84%E4%BB%B6%E4%BD%BF%E7%94%A8%E8%AF%A6%E8%A7%A3/)

整体来说

TouchableWithoutFeedback：无ui变化

TouchableHighlight：定制程度较高，透明度，按下抬起颜色等

TouchableOpacity：只有透明度反馈

TouchableNativeFeedback：支持Android 5.0以上的Ripper效果，仅支持Android设备

# 自定义Button

[Github地址](https://github.com/itfengan/CustomButton)

```
import React, {Component} from 'react'
import {
    View,
    Text,
    StyleSheet,
    TouchableHighlight,
} from 'react-native'
// default props
const backgroundColor = '#000000';
const pressBackgroundColor = backgroundColor;
const textColor = '#FFFFFF';
const pressTextColor = textColor;

export default class CommonButton extends Component {

    // 构造
    constructor(props) {
        super(props);
        // 初始状态
        this.state = {
            //默认文字
            text: this.props.text,
            //是否不可用
            disabled: false,
            //是否按下
            pressed: false,
        };
        this._onPressIn = this._onPressIn.bind(this)
        this._onPressOut = this._onPressOut.bind(this)
    }


    static defaultProps = {
        //背景颜色
        backgroundColor: backgroundColor,
        //按下背景色
        pressBackgroundColor: pressBackgroundColor,
        //文字色
        textColor: textColor,
        //按下文字色
        pressTextColor: pressTextColor,
        //文字大小
        fontSize: 17,
        //圆角
        radius: 0,
        // onPress
        onPressFunc: null,
        //onLongPress
        onLongPressFunc: null,
        //padding
        paddingLeft: 0,
        paddingRight: 0,
        paddingTop: 0,
        paddingBottom: 0,
        //text
        text: '没有设定',
        width: undefined,
        height: undefined
    }

    setDisabled(isDisabled) {
        this.setState({
            disabled: true,
        })
    }

    setButtonText(newText) {
        this.setState({
            text: newText,
        })
    }

    _onPressIn() {
        this.setState({
            pressed: true,
        })
    }

    _onPressOut() {
        this.setState({
            pressed: false,
        })
    }

    render() {
        return (
            <View>
                <TouchableHighlight
                    style={{
                        width: this.props.width,
                        height: this.props.height,
                        justifyContent: 'center',
                        alignItems: 'center',
                        backgroundColor: this.props.backgroundColor,
                        paddingLeft: this.props.paddingLeft,
                        paddingRight: this.props.paddingRight,
                        paddingTop: this.props.paddingTop,
                        paddingBottom: this.props.paddingBottom,
                        borderRadius: 8,
                    }}
                    activeOpacity={1}
                    // 底层的颜色被隐藏的时候调用。
                    onHideUnderlay={null}
                    //当底层的颜色被显示的时候调用。
                    onShowUnderlay={null}
                    // 有触摸操作时显示出来的底层的颜色。
                    underlayColor={this.props.pressBackgroundColor}
                    //disable
                    disabled={this.state.disabled}
                    //onPressIn
                    onPressIn={this._onPressIn}
                    //onPressOut
                    onPressOut={this._onPressOut}
                    //onPress
                    onPress={this.props.onPressFunc}
                    //onLongPress
                    onLongPress={this.props.onLongPressFunc}
                >
                    <View>
                        <Text style={{
                            //文字颜色
                            color: this.state.pressed ? this.props.pressTextColor : this.props.textColor,
                            fontSize: this.props.fontSize
                        }}>
                            {this.state.text}
                        </Text>
                    </View>
                </TouchableHighlight>
            </View>
        )
    }
}

const styls = StyleSheet.create({
    content: {}
})

```

嘻嘻(o^^o)