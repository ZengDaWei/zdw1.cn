---
title: ReactNative-1
date: 2019-06-28 16:41:13
tags:
- React Native
- 前端
category: React Native
cover: "https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1089520985,1354236843&fm=26&gp=0.jpg"
---

这周学习了一下RN，简单的写一篇博客

<!-- more -->

简介：

React Native 可以同时开发ios和android应用，它看起来非常像React，其基础组件是原生组件而非web组件，需要先了解一下ES6和JSX语法，不过也可以边看边学

React Native 简称 **RN**

更多的我相信搜索引擎说的比我好，所以我就不多说了。

让我们开始吧！



## 搭建环境

官方文档：[点击访问](https://reactnative.cn/docs/getting-started.html)

我们需要先安装开发环境，我的系统为mac，使用Homebrew来安装

```sh
brew install node
brew install watchman
npm install -g yarn react-native-cli
```

然后安装Xcode，app store 里面可以快捷安装，然后打开Xcode，按下**Command + ,** 可以打开设置界面

![](https://reactnative.cn/docs/assets/GettingStartedXcodeCommandLineTools.png)

将Command Line Tools 选成那个唯一的选项。



## Init helloworld

我们可以使用React Navicat 命令行工具来创建一个helloworld项目

```sh
react-navicat init helloworld
```

执行这段命令，会在当前目录下创建一个helloworld项目

> 提示：你可以使用`--version`参数（注意是`两`个杠）创建指定版本的项目。例如`react-native init MyApp --version 0.44.3`。注意版本号必须精确到两个小数点。

```sh
cd hellowrld 
react-navice run-ios
```

执行完成后，就可以看到程序打开了你的虚拟机，正在运行你的项目。

![AwesomeProject on iOS](https://reactnative.cn/docs/assets/GettingStartediOSSuccess.png)

## Props（属性）

大多数组件在创建的时候就可以使用各种参数来进行定制，用于定制的这些参数就被称为`props`，

以常见的基础组件`Image`为例，在创建一个图片的时候，可以传入一个`source`的prop来执行要显示的图片路径，以及使用名为`style`的prop来控制其尺寸。

```react
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

再来看看下面这个例子，请注意`{pic}`外围有一层括号，我们需要使用括号把`pic`这个变量嵌入JSX语句中，括号的意思是括号内部为一个js比那里或者表达式，需要执行后取值。

```react
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class Greeting extends Component {
  render() {
    return (
      <View style={{alignItems: 'center', marginTop: 50}}>
        <Text>Hello {this.props.name}!</Text>
      </View>
    );
  }
}

export default class LotsOfGreetings extends Component {
  render() {
    return (
      <View style={{alignItems: 'center'}}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
  }
}
```

## State（状态）

我们使用两种数据来控制一个组件：`props`和`state`，`props`是在父组件中指定，而且一经指定，在被指定的组件的生命周期中则不再改变，对于需要改变的数据，我们需要使用`state`。

一般来说，你需要在constructor中初始化`state`，这是ES6的写法

```react
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class Blink extends Component {
  constructor(props) {
    super(props);
    this.state = { isShowingText: true };

    // 每1000毫秒对showText状态做一次取反操作
    setInterval(() => {
      this.setState(previousState => {
        return { isShowingText: !previousState.isShowingText };
      });
    }, 1000);
  }

  render() {
    // 根据当前showText的值决定是否显示text内容
    if (!this.state.isShowingText) {
      return null;
    }

    return (
      <Text>{this.props.text}</Text>
    );
  }
}

export default class BlinkApp extends Component {
  render() {
    return (
      <View>
        <Blink text='I love to blink' />
        <Blink text='Yes blinking is so great' />
        <Blink text='Why did they ever take this out of HTML' />
        <Blink text='Look at me look at me look at me' />
      </View>
    );
  }
}
```

实际开发中，我们不会定义一个计时器来操作state，典型的操作是接受后端返回的数据来定义逻辑。

每次调用`setState`时，BlinkApp 都会重新执行 render 方法重新渲染。这里我们使用定时器来不停调用`setState`，于是组件就会随着时间变化不停地重新渲染。

- `state`的修改必须通过`setState()`函数来改变
  - this.state.likes = 100;  ，直接修改赋值是无效的
  - setState是一个merge合并的操作，只修改其属性，不影响其他属性
  - setState是一个异步操作，修改**不会马上生效**

## 样式

在RN中，不需要学习什么特殊的语法来定义样式，任然使用的是JavaScript来写样式，所有的核心组件都接受名为`style`的属性。

```react
import React, { Component } from 'react';
import { AppRegistry, StyleSheet, Text, View } from 'react-native';

export default class LotsOfStyles extends Component {
  render() {
    return (
      <View>
        <Text style={styles.red}>just red</Text>
        <Text style={styles.bigBlue}>just bigBlue</Text>
        <Text style={[styles.bigBlue, styles.red]}>bigBlue, then red</Text>
        <Text style={[styles.red, styles.bigBlue]}>red, then bigBlue</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  bigBlue: {
    color: 'blue',
    fontWeight: 'bold',
    fontSize: 30,
  },
  red: {
    color: 'red',
  },
});

```

本人感觉，和CSS区别不大，改了格式而已，命名为**驼峰命名法**，当然这只是个人**拙见**。

## Flex 宽高

首先我们先看看普通的宽高

```react
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FixedDimensionsBasics extends Component {
  render() {
    return (
      <View>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 100, height: 100, backgroundColor: 'skyblue'}} />
        <View style={{width: 150, height: 150, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
}
```

需要注意的是，**React Native 中的尺寸都是无单位的，表示的是与设备像素密度无关的逻辑像素点。**

在组件中，使用Flex可以让可利用的空间中动态的扩张和收缩。一般我们会使用`flex:1`来指定某一个组件扩张以撑满所有的剩余空间。如果有多个并列的子组件使用了`flex:1`，则这些子组件会平分父容器中剩余的空间。如果这些并列的子组件的`flex`值不一样，则谁的值更大，谁占据剩余空间的比例就更大（即占据剩余空间的比等于并列组件间`flex`值的比）。

```react
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FlexDimensionsBasics extends Component {
  render() {
    return (
      // 试试去掉父View中的`flex: 1`。
      // 则父View不再具有尺寸，因此子组件也无法再撑开。
      // 然后再用`height: 300`来代替父View的`flex: 1`试试看？
      <View style={{flex: 1}}>
        <View style={{flex: 1, backgroundColor: 'powderblue'}} />
        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
}
```

## 使用Flexbox布局

我们在RN中使用flexbox规则来指定某个组件的子元素的布局，我们使用三个样式属性就已经可以满足大多布局需求。

1. flexDirection
2. alignItems
3. justifyContent

### Flex Direction

通俗的来讲，这个属性就是用来定义排列方向的，横着排，竖着排

```react
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class FlexDirectionBasics extends Component {
  render() {
    return (
      // 尝试把`flexDirection`改为`column`看看
      <View style={{flex: 1, flexDirection: 'row'}}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};
```

### Justify Content

这个属性就是用来定义组件的排列方式的，是应该均匀的排，还是靠着末尾排，对应的这些可选项有：`flex-start`、`center`、`flex-end`、`space-around`、`space-between`以及`space-evenly`。

```react
export default class JustifyContentBasics extends Component {
  render() {
    return (
      // 尝试把`justifyContent`改为`center`看看
      // 尝试把`flexDirection`改为`row`看看
      <View style={{
        flex: 1,
        flexDirection: 'row',
        justifyContent: 'space-between',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'skyblue'}} />
        <View style={{width: 50, height: 50, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};
```

### Align Items

在组件的 style 中指定`alignItems`可以决定其子元素沿着**次轴**（与主轴垂直的轴，比如若主轴方向为`row`，则次轴方向为`column`）的**排列方式**。子元素是应该靠近次轴的起始端还是末尾段分布呢？亦或应该均匀分布？对应的这些可选项有：`flex-start`、`center`、`flex-end`以及`stretch`。

> 注意：要使`stretch`选项生效的话，子元素在次轴方向上不能有固定的尺寸。以下面的代码为例：只有将子元素样式中的`width: 50`去掉之后，`alignItems: 'stretch'`才能生效。

```react
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class AlignItemsBasics extends Component {
  render() {
    return (
      // 尝试把`alignItems`改为`flex-start`看看
      // 尝试把`justifyContent`改为`flex-end`看看
      // 尝试把`flexDirection`改为`row`看看
      <View style={{
        flex: 1,
        flexDirection: 'cloumn',
        justifyContent: 'center',
        alignItems: 'stretch',
      }}>
        <View style={{width: 50, height: 50, backgroundColor: 'powderblue'}} />
        <View style={{height: 50, backgroundColor: 'skyblue'}} />
        <View style={{height: 100, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
};
```

## 文本输入

这里引用官方的一个demo

[`TextInput`](https://reactnative.cn/docs/textinput#content)是一个允许用户输入文本的基础组件。它有一个名为`onChangeText`的属性，此属性接受一个函数，而此函数会在文本变化时被调用。另外还有一个名为`onSubmitEditing`的属性，会在文本被提交后（用户按下软键盘上的提交键）调用。

假如我们要实现当用户输入时，实时将其以单词为单位翻译为另一种文字。我们假设这另一种文字来自某个吃货星球，只有一个单词： 🍕。所以"Hello there Bob"将会被翻译为"🍕🍕🍕"。

```react
import React, { Component } from 'react';
import { AppRegistry, Text, TextInput, View } from 'react-native';

export default class PizzaTranslator extends Component {
  constructor(props) {
    super(props);
    this.state = {text: ''};
  }

  render() {
    return (
      <View style={{padding: 10}}>
        <TextInput
          style={{height: 40}}
          placeholder="Type here to translate!"
          onChangeText={(text) => this.setState({text})}
        />
        <Text style={{padding: 10, fontSize: 42}}>
          {this.state.text.split(' ').map((word) => word && '🍕').join(' ')}
        </Text>
      </View>
    );
  }
}
```

map((word) => word && '披萨')，这个我起初也没看懂，后面查了一波，这是ES5的基础语法...

意思就是，如果 word这个参数存在就替换成后面的披萨，不然就无视

**看来我要好好补一补前端知识了！**

这篇博客就说这么多，不是因为我这周只学了这么点，因为我敲的眼睛酸了，撒油拉拉！！

**Happy Coding!!!!**


