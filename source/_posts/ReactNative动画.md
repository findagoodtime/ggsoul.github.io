---
title: ReactNative动画
date: 2017-04-22 16:07:31
tags: [react]
categories: create
---

一直蛮热衷于动画效果开发的，前段时间用ReactNative开发了一个动画效果，个人挺满意的，在这里分享一下

<!--more-->

## ReactNative动画API

__LayoutAnimation__

当布局变化时，自动将视图运动到它们新的位置上。只对布局的创建和更新时才起作用，删除时不起作用，动画属性也比较少，只有透明度和缩放两种，优点是性能好，使用简单，全局作用。

```javascript
import React, { Component } from 'react';
import {
  Text,
  View,
  UIManager,
  LayoutAnimation
} from 'react-native';

const CustomLayoutAnimation = {
    duration: 150,
    create: {
        type: LayoutAnimation.Types.easeIn,
        property: LayoutAnimation.Properties.opacity
    }
};

export default class LayoutAnimationView extends Component {

    constructor(props: Object) {
        super(props);
		// android使用LayoutAnimation需要开启动画设置，ios默认打开
		UIManager && UIManager.setLayoutAnimationEnabledExperimental(true);
    }

    componentWillMount() {
		/*
		 * 这个例子的使用场景是：首次渲染页面，一边异步请求数据，一边页面显示加载中
		 * 拿到数据后使用渐变的效果显示完整页面
		*/
        LayoutAnimation.configureNext(CustomLayoutAnimation);
    }

    render() {
        return <View></View>;
    }
}
```
