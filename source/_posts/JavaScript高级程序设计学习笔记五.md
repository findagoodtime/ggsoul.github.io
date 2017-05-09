---
title: JavaScript高级程序设计学习笔记五
date: 2016-05-06 22:26:17
tags: javascript
categories: onebook
---

这一章主要涉及的知识点，更多的描述了ECMAScript不同于其他语言的地方，我的体会就是："存在即合理"，上一节讲到函数的参数传递的是值，从这里出发，ECMAScript这门以函数为"第一等公民"的语言围绕函数究竟提供了哪些编程方式

<!--more-->

## 传递的参数不是引用

传递的参数不是引用，那想要传递对象要怎么办呢？

可以直接传递对象内存，不过ECMAScript不能直接操作对象的内存空间。ECMAScript给出的方式是，创建对象实例的时候将它的值保存到堆内存中，而将指向这个值得引用保存到变量中，于是，引用类型的变量仅保存对象的一个引用，这个变量虽然是一个引用，但是它有实际的值，它可以用来当做参数传递给函数。

```javascript
function setName(obj) {
	obj.name = "Nicholas";
}

var person = new Object();
setName(person);
alert(person.name);	//"Nicholas"
```

这个例子就是传递对象的方式，对象的引用作为变量的实际的值进行参数传递，在函数内操作这个引用就操作了这个对象，在函数外面对象的属性被更改就是证明。

除了引用类型，ECMAScript还有5种基本类型，Undefined、Null、Boolean、Number和String，这几种类型的变量中保存的就是实际的值，可以直接作为参数传递。它们的变量是相互独立的，这是和引用类型不同的地方。

typeof用来检测基本类型，instanceof用来检测引用类型，后者检测原理是检测构造函数。

```javascript
typeof(null);	//object
typeof(new Object());	//object

alert(person instanceof Object);	//变量person是Object吗？
alert(colors instanceof Array);	//变量colors是Array吗？
alert(pattern instanceof RegExp);	//变量pattern是RegExp吗？
```

## 不想传递参数就靠作用域

函数不光能够使用传递进来的参数，它能对其执行环境有权访问的所有变量和函数的有序访问，靠的是作用域链，作用域链是由全局执行环境到当前执行环境分别创建的变量对象组成的(这个变量对象保存该环境中定义下所有变量和函数)，离得越近越接近作用域链的前端。

这个作用域链是一个导向，标识符解析是沿着这个导向一级一级地搜索标识符的过程。搜索过程始终从作用域链的前端开始，然后逐级地向后回溯，直至找到标识符为止。

```javascript
var color = "blue";

function changeColor() {
	var anotherColor = "red";

	function swapColors() {
		var tempColor = anotherColor;
		anotherColor = color;
		color = tempColor;

		//这里可以访问color、another和tempColor
	}

	//这里可以访问color和anotherColor，但不能访问tempColor
	swapColors();
}

//这里只能访问color
changeColor();
```

关于动态作用域：

```javascript
var x = 10;

function foo() {
	console.log(x);	//20
}

function bar() {
	var x = 20;
	foo();
}

bar();
```

首先`x:10`入栈，`foo:<function>`入栈，`bar:<function>`入栈，然后执行bar()，进入bar函数遇到`var x = 20;`，意味着`x:20`入栈，执行foo()，遇到`console.log(x);`，在栈中查找x这个标识符，遇到`x:20`，于是输出20：

![note4](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/note4.png)

JavaScript是静态作用域，不过可以通过eval和with构成动态作用域。

## 不一样的作用域

ECMAScript没有块级作用域，就是没有由花括号封闭的代码块形成的作用域。这是个挺不一样的地方，为什么？应该和以函数为"第一等公民"有关系，这是刚开始学习时挺大的转变的地方，另一个地方是length，在Java中数组使用的是length属性，而String使用的是length()方法，在ECMAScript用的都是length属性，这两个地方我改了好久，经常出错。

声明变量，使用var声明的变量会自动被添加到最接近的环境中，而且，还有一个声明提前的事，无论在函数的何处声明变量，在声明之前的位置可以使用这个变量，只是这时的变量是没有赋值的undefined，因此，声明变量应该放在函数的最开始部分，避免不必要的错误，而且便于代码阅读。

```javascript
var foo = 1;
(function main() {
	console.log(foo);	//undefiend
	var foo = 2;
	console.log(foo);	//2
})();
console.log(foo);	//1
```

上面的例子同时还证明，标识符解析是从内向外一级一级进行，如果检索到变量，搜索即行停止。

## 垃圾收集

引自[Mozilla developer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management#.E5.9E.83.E5.9C.BE.E5.9B.9E.E6.94.B6)

### 引用计数

此算法把"对象是否不再需要"简化定义为"对象有没有其他对象引用到它"。如果没有引用指向该对象(零引用)，对象将被垃圾回收机制回收。

### 标记清除

这个算法把"对象是否不再需要"简化定义为"对象是否可以获得"。

这个算法假定设置一个叫做根的对象(在Javascript里，根是全局对象)。定期的，垃圾回收器将从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和所有不能获得的对象。

## 总结

ECMAScript的设计是以直接性和简洁性来完成它的合理性，入门还是很容易的，不过在使用的过程中需要有严谨的态度。