---
title: React组件
date: 2017-04-25 19:26:07
tags: react
categories: reprint
---


Components用来把UI拆分成独立的，可重复使用的部分，然后做不同的考量。`React.Component`由`React`提供。

<!--more-->

## 概述

`React.Component`是一个抽象的基类，没有直接有意义的参考。通常用来定义有一个`render`方法的子类。

一个简洁的组件:

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

如果没有使用ES6，可以使用`create-react-class`模块来代替。

### 组件生命周期

每一个组件都有一些生命周期方法，可以重写这些方法来实现在特定时间点的处理。有 **`will`** 前缀的方法是在某事发生前，有 **`did`** 前缀的方法是在某事发生后。

#### 装载

组件实例创建并插入到`DOM`中时会调用这些方法：

- `constructor()`
- `componentWillMount()`
- `render()`
- `componentDidMount()`

#### 更新

`props`和`state`的改变会导致更新。组件被重绘的时候会调用这些方法：
- `componentWillReceiveProps()`
- `shouldComponentUpdate()`
- `componentWillUpdate()`
- `render()`
- `componentDidUpdate()`

#### 挂载

从`DOM`中移除组件时会调用这个方法：

- `componentWillUnmount()`

### 其它APIs

每个组件也提供了一些其它APIs：

  - `setState()`
  - `forceUpdate()`

### 类属性

  - `defaultProps`
  - `displayName`

### 实例属性

  - `props`
  - `state`

* * *

## 参考

### `render()`

```javascript
render()
```

组件中一定要有`render()`方法。

被调用时会检查`this.props`和`this.state`来返回一个`React`原件。这个原件也可以是一个原生`DOM`组件的画像，比如`<div />`，或者自定义的综合组件。

如果不想要渲染任何东西，可以直接返回`null`或者`false`。这时调用`ReactDOM.findDOMNode(this)`也会返回`null`。

`render()`方法应该是纯净的，每次被调用是都应该返回相同的结果，不应该在方法里修改组件状态，或者与浏览器有直接交互。如果想要和浏览器交互，在`componentDidMount()`或者其它声明周期中完成你的工作。保持`render()`的纯净使组件更容易理解。

> Note
>
> 如果`shouldComponentUpdate()`返回false，`render()`不会被执行

* * *

### `constructor()`

```javascript
constructor(props)
```

`React`组件装载之前会执行构造方法。在子类中执行构造方法时，应该在其它声明的上方执行`super(props)`。需要注意的是，构造方法中的`this.props`会变成`undefined`。

在构造方法中初始化`state`很合适。如果不初始化`state`也不绑定方法，那不需要在组件中实现构造方法。

基于`props`初始化`state`是可以的。这些初始有效的`props`，可以直接设置到`state`中。下面是一个例子：

```js
constructor(props) {
  super(props);
  this.state = {
    color: props.initialColor
  };
}
```

需要注意的是，`state`不会随着`props`及时更新。可以用`lift the state up`来代替同步更新`props`给`state`。

如果想要使用`props`设置`state`的值，可以使用`componentWillReceiveProps(nextProps)`来同步更新`state`。但是`lift the state up`更简单而且能避免错误。

`lift the state up`指的是把几个子类的`state`提升到他们最临近的父级里面，然后通过父级接收子类事件统一处理后，用`props`的方式再分发给子类。

* * *

### `componentWillMount()`

```javascript
componentWillMount()
```

`componentWillMount()`是在首次装载发生之前立即调用。在`render()`之前调用，因此在这个方法里同步设置`state`不会触发重绘。避免在这个方法里引入任何副作用或者订阅。

这只是一个声明周期钩子。通常推荐使用`constructor()`。

* * *

### `componentDidMount()`

```javascript
componentDidMount()
```

`componentDidMount()`是在首次装载完成之后立即被调用。这里获取`DOM`元素最合适。如果有从远端请求数据的需求，可以在这里初始化网络请求。在这里设置`state`会触发重绘。

* * *

### `componentWillReceiveProps()`

```javascript
componentWillReceiveProps(nextProps)
```

`componentWillReceiveProps()`是在更新属性准备重绘之前调用。如果你需要根据新的`props`来修改`state`，你可以拿比较`this.props`和`nextProps`后的结果交给`this.setState()`来完成`state`的转换。

注意`props`没有改变的时候也会触发该方法，所以如果只是想处理改变，那请确保比较了当前值和最新值。父组件可能会导致组件重绘。

首次装载时不会调用`componentWillReceiveProps`。如果组件的`props`更新会调用这个方法。通常调用`this.setState`不会触发`componentWillReceiveProps`。

* * *

### `shouldComponentUpdate()`

```javascript
shouldComponentUpdate(nextProps, nextState)
```

使用`shouldComponentUpdate()`是为了让`React`知道当前`props`或`state`的更新是否影响到组件的输出。默认是每次`state`的更新都会重绘，多数情况下应该依靠默认行为。

`shouldComponentUpdate()`是在新的`props`或`state`被接收准备重绘之前被调用。默认返回`true`。首次装载或使用`forceUpdate()`时，该方法不被调用。


如果子组件的`state`改变了，那即便父组件的该方法返回`false`，子组件依旧会重绘。

如果`shouldComponentUpdate()`返回`false`，那么`componentWillUpdate()`、`render()`和`componentDidUpdate()`都不会被调用。需要注意的是，`React`在未来可能会对待`shouldComponentUpdate()`作为一种暗示，为非一个严格的指令，返回`false`也会重绘组件。

如果你确定特定组件在解析时很慢，可以换用继承`React.PureComponent`，`React.PureComponent`没有实现浅比较`props`和`state`的`shouldComponentUpdate()`。如果你有信心自己手写，可以比较`this.props`与`nextProps`和`this.state`与`nextState`来判断更新可以跳过。

对于复杂的数据的比较是非常耗时的，而且可能无法比较，通过使用`Immutable.js`能够很好地解决这个问题，`Immutable.js`的基本原则是对于不变的对象返回相同的引用，而对于变化的对象，返回新的引用。

* * *

### `componentWillUpdate()`

```javascript
componentWillUpdate(nextProps, nextState)
```
`componentWillUpdate()`是在更新`props`或`state`准备重绘之前被调用。这是在发生更新之前做准备的时机。首次装载的时候没有调用这个方法。

注意不能在这里调用`this.setState()`，应该使用`componentWillReceiveProps()`来根据`props`更新`state`。

> Note
>
> 如果`shouldComponentUpdate()`返回false，那么`componentWillUpdate()`不会被调用

* * *

### `componentDidUpdate()`

```javascript
componentDidUpdate(prevProps, prevState)
```

`componentDidUpdate()`是在重绘之后执行，首次装载时不会触发。

可以在这个时机操作组件更新完成后的`DOM`。可以比较`props`的改变在这里做网络请求。（`props`如果没有改变，可能不需要发起网络请求）。

> Note
>
> 如果`shouldComponentUpdate()`返回false，那么`componentDidUpdate()`不会被调用

* * *

### `componentWillUnmount()`

```javascript
componentWillUnmount()
```

`componentWillUnmount()`在组件被卸载和销毁之前会被调用。在这个方法里执行必要的清理工作，比如没用的计时器，取消网络请求，或者是清除在`componentDidMount`执行时创建的`DOM`元素。

* * *

### `setState()`

```javascript
setState(updater, [callback])
```
`setState()`更新组件`state`，并且告诉`React`这个组件需要根据`state`的更新来重绘，但只是加入到执行队列。这是一个更新用户操作和处理服务端数据的主要方法。

请把`setState()`当做一个请求而非立即执行方法。在单线程执行多个组件时，给了更好的性能，`React`可能会延迟执行它。`React`不能保证`state`更新被立即执行。

`setState()`不总是立即更新组件。它会把`JavaScript`执行进度中调用的`setState()`合并在一起，延迟执行。在`setState()`之后读取`this.state`是有问题的。用来代替的是，使用`componentDidUpdate`或者一个`setState`回调(`setState(updater, callback)`)，或者是更新完成后保证可以被触发的方法。如果你需要前一个`state`来设置当前`state`，请看下面的`updater`。

`setState()`总是会触发重绘，除非`shouldComponentUpdate()`返回`false`。如果由于对象很复杂没办法使用`shouldComponentUpdate()`来判断是否重绘，可以执行`setState()`在新的`state`与原有`state`存在变更的时候来避免不必要的重绘。

第一个参数可以是像这样的`updater`方法：

```javascript
(prevState, props) => nextState
```

`prevState`是对之前`state`的引用。不应该直接改变。应该通过根据`prevState`和`props`的输入构建一个新的状态对象来表示更改。例如，假设我们想根据`props.step`增加一个变量值：

```javascript
this.setState((prevState, props) => {
  return {counter: prevState.counter + props.step};
});
```

`updater`收到的`prevState`和`props`都是最新的。

`setState()`第二个参数是一个可选的回调方法，一旦`setState`执行完并且组件重绘完时执行。通常推荐使用`componentDidUpdate()`。

也可以通过第一个参数为对象给`setState()`代替方法：

```javascript
setState(stateChange, [callback])
```

会执行一个浅复制`stateChange`到新的`state`，例如调整购物车商品数量：
```javascript
this.setState({quantity: 2})
```

这种格式的`setState()`也是异步的，并且同一个生命周期的多次请求会合并在一起。比如想在同一个生命周期加入增加数量，结果相当于：

```javaScript
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

在同一生命周期中后续请求会覆盖之前请求，所以数量只会增加一次。如果后面的`state`依赖前面的`state`，推荐使用`updater`方法代替：

```js
this.setState((prevState) => {
  return {counter: prevState.quantity + 1};
});
```

* * *

### `forceUpdate()`

```javascript
component.forceUpdate(callback)
```

默认，如果组件里面的`state`或者`props`更新，组件会重绘。但是如果`render()`调用依赖其它的数据，可以通过`forceUpdate()`来触发组件渲染。

调用`forceUpdate()`会导致组件`render()`执行，并且跳过`shouldComponentUpdate()`。这将触发子组件的正常生命周期方法，包括每个子组件的`shouldComponentUpdate()`。如果标记更新，`React`会仅仅修改`DOM`。

通常应该避免使用`forceUpdate()`并且只使用`this.props`和`this.state`来读取。

* * *

## 类属性

### `defaultProps`

`defaultProps`可以用来定义组件自己的默认属性，设置类的默认`props`。这是使用于未声明的`props`，但不使用于空`props`。例如：

```js
class CustomButton extends React.Component {
  // ...
}

CustomButton.defaultProps = {
  color: 'blue'
};
```

如果`props.color`没有被提供，它会被默认设置为`'blue'`：

```js
  render() {
    return <CustomButton /> ; // props.color会被设置为blue
  }
```

如果`props.color`被设置为`null`，那它就是`null`：

```js
  render() {
    return <CustomButton color={null} /> ; // props.color会被设置为null
  }
```

* * *

### `displayName`

`displayName`被用作`debug`信息。

* * *

## 实例属性

### `props`

`this.props`包含组件调用者定义的`props`。尤其是，`this.props.children`是一个特殊的属性，通常由JSX表达式中的子标签定义，而不是标记本身。

### `state`

`state`包含特定于该组件的状态，随时间而改变。`state`是用户自定义的，应该是一个清晰的`JavaScript`对象。

不要直接改变`this.state`，因为`setState()`可能会替换你的改变。把`this.state`当做不可变的。
