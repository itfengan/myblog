---
title: ReactNative碎片整理之样式
date: 2018-07-31 15:38:55
tags:
- RN
categories: RN
password:
---

本文整理React Native中的布局方式，FlexBox弹性布局的使用

<!--more-->

# 宽高

RN中，尺寸是没有单位的，它已经代表了设备独立像素

```
<View style={ {width:100,height:100,margin:40,backgroundColor:'gray'}}>
        <Text style={ {fontSize:16,margin:20}}>尺寸</Text>
</View>
```

运行在Android上时，View的长和宽被解释成：100dp 100dp单位是dp，字体被解释成16sp 单位是sp，运行在iOS上时尺寸单位被解释称了pt，这些单位确保了布局在任何不同dpi的手机屏幕上显示不会发生改变；

# RN和Web css上的FlexBox的区别

- flexDirection: React Native中默认为`flexDirection:'column'`，在Web CSS中默认为`flex-direction:'row'`
- alignItems: React Native中默认为`alignItems:'stretch'`，在Web CSS中默认`align-items:'flex-start'`
- flex: 相比Web CSS的flex接受多参数，如:`flex: 2 2 10%;`，但在 React Native中flex只接受一个参数
- 不支持属性：align-content，flex-basis，order，flex-basis，flex-flow，flex-grow，flex-shrink

以上是React Native中的FlexBox 和Web CSSS上FlexBox的不同之处，记住这几点，你可以像在Web CSSS上使用FlexBox一样，在React Native中使用FlexBox。

# Layout Props

## 父视图属性（容器属性）：

- flexDirection enum(‘**row**’, ‘**column**’,’**row-reverse**’,’**column-reverse**’)
- flexWrap enum(‘**wrap**’, ‘**nowrap**’)
- justifyContent enum(‘**flex-start**’, ‘**flex-end**’, ‘**center**’, ‘**space-between**’, ‘**space-around**’)
- alignItems enum(‘**flex-start**’, ‘**flex-end’**, ‘**center**’, ‘**stretch**’)

学习父容器属性之前，先学习下基本概念

<img src="https://mdn.mozillademos.org/files/12998/flexbox.png" width='400'>

和Web的Css不同的是，RN的Flexbox默认的主轴是 竖直的（column）

### flexDirection

`flexDirection enum('row', 'column','row-reverse','column-reverse')`
`flexDirection`属性定义了父视图中的子元素沿横轴或侧轴方片的排列方式。

- row: 从左向右依次排列
- row-reverse: 从右向左依次排列
- column(default): 默认的排列方式，从上向下排列
- column-reverse: 从下向上排列

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/flexDirection.jpg" width='400'/>

### flexWrap

`flexWrap enum('wrap', 'nowrap')`
`flexWrap`属性定义了子元素在父视图内是否允许多行排列，默认为nowrap。

- nowrap flex的元素只排列在一行上，可能导致溢出。
- wrap flex的元素在一行排列不下时，就进行多行排列。

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/flexWrap.jpg" width='400'/>

### justifyContent

`justifyContent enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around') `
`justifyContent`属性定义了浏览器如何分配顺着父容器主轴的弹性（flex）元素之间及其周围的空间，默认为flex-start。

- flex-start(default) 从行首开始排列。每行第一个弹性元素与行首对齐，同时所有后续的弹性元素与前一个对齐。
- flex-end 从行尾开始排列。每行最后一个弹性元素与行尾对齐，其他元素将与后一个对齐。
- center 伸缩元素向每行中点排列。每行第一个元素到行首的距离将与每行最后一个元素到行尾的距离相同。
- space-between 在每行上均匀分配弹性元素。相邻元素间距离相同。每行第一个元素与行首对齐，每行最后一个元素与行尾对齐。
- space-around 在每行上均匀分配弹性元素。相邻元素间距离相同。每行第一个元素到行首的距离和每行最后一个元素到行尾的距离将会是相邻元素之间距离的一半。

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/justifyContent.jpg" width='400'/>



### alignItems

`alignItems enum('flex-start', 'flex-end', 'center', 'stretch')`
`alignItems`属性以与justify-content相同的方式在侧轴方向上将当前行上的弹性元素对齐，默认为stretch

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/alignItems.jpg" width='400'/>

## 子视图属性（规范自己的）

### alignSelf

`alignSelf enum('auto', 'flex-start', 'flex-end', 'center', 'stretch')`
`alignSelf`属性以属性定义了flex容器内被选中项目的对齐方式。注意：alignSelf 属性可重写灵活容器的 alignItems 属性。

- **auto(default) 元素继承了它的父容器的 align-items 属性。如果没有父容器则为 “stretch”。**
- stretch	元素被拉伸以适应容器。
- center	元素位于容器的中心。
- flex-start	元素位于容器的开头。
- flex-end	元素位于容器的结尾。

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/alignSelf.jpg" width='400'/>

### flex

`flex number`
`flex` 属性定义了一个可伸缩元素的能力，默认为0。

<img src="https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/develop/React%20Native%E5%B8%83%E5%B1%80/React%20Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/images/flex.jpg" width='400'/>



# 其他属性

以下属性是React Native所支持的除Flex以外的其它布局属性。

## 视图边框

## 尺寸

## 内部边距

## 外边距

## 边缘

## 定位(position)

position enum(‘absolute’, ‘relative’)属性设置元素的定位方式，为将要定位的元素定义定位规则。

- absolute：生成绝对定位的元素，元素的位置通过 “left”, “top”, “right” 以及 “bottom” 属性进行规定。(相对于父布局的位置)（注意：忽略父组件的padding，默认直接在父组件的左上角，除非设置当前组件的left，top，right，bottom,或者margin之类的）
- relative：生成相对定位的元素，相对于其正常位置进行定位。因此，”left:20” 会向元素的 LEFT 位置添加 20 像素。（相对于自身本应该在的位置）

**例子1:**

```javascript
render() {
        return (
            <View style={{
                flex: 1,
                justifyContent: 'center',
            }}>
                <TouchableOpacity onPress={this.loadData}>
                    <View style={styles.container}>
                        <Image style={styles.header}/>
                        <Text
                            style={styles.name}>absolute</Text>
                    </View>
                </TouchableOpacity>
            </View>
        );
    }    
const styles = StyleSheet.create({
    container: {
        flexDirection: 'row',
        paddingLeft: 15,
        paddingRight: 15,
        paddingTop: 10,
        paddingBottom: 10,
        backgroundColor: '#999999'
    },
    header: {
        width: 100,
        height: 100,
        backgroundColor: '#00a056'
    },
    name: {
        position: 'relative',
        // left: 15,
        alignSelf: 'flex-start',
        fontSize: 13,
        backgroundColor: '#ffff00',
        color: '#999999'
    }
});
```

<img src="https://fenganblogimgs.oss-cn-beijing.aliyuncs.com/blog/position.png" width="400"/>

# 参考：

[React Native布局详细指南](http://www.devio.org/2016/08/01/Reac-Native%E5%B8%83%E5%B1%80%E8%AF%A6%E7%BB%86%E6%8C%87%E5%8D%97/)



