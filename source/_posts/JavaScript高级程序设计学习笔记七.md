---
title: JavaScript高级程序设计学习笔记七
date: 2016-05-06 22:35:23
tags: javascript
categories: onebook
---

关于JavaScript的几种类型的介绍

<!--more-->

## Date类型

获取当前日期对象：
	
```javascript
var num = Math.floor(Math.random() * 10 + 1);
```

指定时间日期对象：
	
```javascript
//Tue May 25 2004 00:00:00 GMT+0800 (中国标准时间)
var someData = new Date(Date.parse("May 25, 2004"));

//Fri May 06 2005 01:55:55 GMT+0800 (中国标准时间)
var allFives = new Date(Date.UTC(2005, 4, 5, 17, 55, 55));
```

更多方法详见[Mozilla developer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date#Date_instances)


## RegExp类型

用来支持正则表达式的类型。

`var expression = / pattern / flags ;`

模式(pattern)部分是正则表达式，后面可以带一个或多个标志(flags)。

* g：表示全局模式，即模式将被应用于所有的字符串
* i：表示不区分大小写模式，即在确定匹配项时忽略模式与字符串的大小写
* m：表示多行模式，即在达到一行文本末尾时还继续查找下一行

正则表达是中的元字符，使用时需要转义：

`( [ { \ ^ $ | ) ? * + . ] }`


实例方法：

* exec()：接受一个参数，即要应用模式的字符串，然后返回包含第一个匹配项信息的数组。这个数组包含额外的属性，index和input，其中index表示匹配项在字符串中的位置，而input表示应用正则的字符串
* test()：接受一个字符串参数，在模式与该参数匹配的情况下返回true，否则false

ECMAScript中的正则表达式不支持一些高级特性。

## Function类型

ECMAScript中的函数实际上是对象，"函数是对象，函数名是指针"，Java说万事万物皆对象，于是ECMAScript的函数直接就搞成了对象。函数变成对象的好处就是，语言结构简单了，函数可以随便传递了，没有重载也无所谓，可以直接分析参数做出不同的处理。

函数声明与函数表达式是有区别的。声明的函数在代码执行之前有一个函数声明提升的过程，会将声明的函数放在源代码数的顶端。也就是函数声明可以卸载函数执行后面，函数表达式就不行啦，因为有一个赋值的过程，在赋值操作执行之前，变量是没有引用到函数体的。

```javascript
//函数声明，可以
console.log(sum(10, 10));
function sum(num1, num2) {
  return num1 + num2;
}

//函数表达式，不可以
console.log(sum(10, 10));
var sum = function(num1, num2) {
  return num1 + num2;
}
```

函数内部有两个特殊对象：arguments和this。函数对象有一个叫callee的属性，是指向拥有这个arguments对象函数的指针。ECMAScript 5也规范化了另一个函数对象的属性：caller，这个属性中保存着调用当前函数的函数的引用。

函数有两个属性：length和prototype。length指的是函数希望接收的命名参数的个数，prototype是保存它们所有实例方法的真正所在。

函数内部包含两个非继承而来的方法：apply()和call()。它们的区别在于传递的参数不同，apply接收两个参数，一个是在其中运行函数的作用域，另一个是参数数组，call第一个参数与之相同，而其余的参数都直接传递给函数。它们真正强大的地方是能够扩充函数赖以运行的作用域。

bind()方法，会创建函数的实例：

```javascript
window.color = "red";
var o = { color: "blue" };

function sayColor() {
	console.log(this.color);
}
var objectSayColor = sayColor.bind(o);
objectSayColor(); //blue
```

## 基本包装类

为了便于操作基本类型值，提供的3个特殊的引用类型：Boolean、Number和String，每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类的对象，从而能够调用一些方法来操作这些数据。

正常代码：

```javascript
var s1 = "some text";
var s2 = s1.substring(2);
```

后台完成如下步骤：

```javascript
var s1 = new String("some text");
var s2 = s1.substring(2);
s1 = null;
```

Boolean类型重写了object的valueOf()方法，所以调用valueOf()能返回true和false不调用这个方法就不能返回true和false了，只有在能触发valueOf方法的场景才是对的。

关于Number包装类的一些方法：

```javascript
var num = 10.005;
console.log(num.toPrecision(1));   // 1e+1
console.log(num.toPrecision(2));  // 10
console.log(num.toPrecision(3)); // 10.0
console.log(num.toExponential(1));     // 1.0e+1
console.log(num.toFixed(1));     // 10.0
```

Boolean和Number的包装类都重写了valueOf()、toLocaleString()和toString()方法，感觉没有特别大的用，String的包装类用的很多，这个基本包装类更多的用处是来实现将String是基本类型这一设定的而带来的更多需求。

String类型的很多方法和数组操作类似，我个人觉得比较有用的是模式匹配方法：

* match()：本质与调用RegExp的exec方法相同
* search()：返回字符串中第一个匹配项的索引
* replace(): 用于替换子字符串
* split()：基于指定分隔符将一个字符串分割成多个子字符串，并将结果放在一个数组里

replace()的例子：

```javascript
function htmlEscape(text) {
	return text.replace(/[<>"&]/g, function(match, pos, originalText) {
		switch(match) {
			case "<":
				return "&lt;";
			case ">":
				return "&gt;";
			case "&":
				return "&amp;";
			case "\"":
				return "&quot;";
		}
	});
}
console.log(htmlEscape("<p class=\"greeting\">Hello world!</p>"));
//<p class="greeting">Hello world!</p>;
```


## Global对象

诸如isNaN()、isFinite()、parseInt()和parseFloat()，实际上都是Global对象的方法。

URI编码方法，有效的URI不包含某些字符，比如说空格，需要URI编码方法对URI编码，用特殊的UTF-8编码替换所有无效的字符，从而让浏览器能够理解。encodeURI()主要用于整个URI，encodeURIComponent()主要用于对URI中的某一段，比如说后缀参数。

## Math对象

Math对象提供计算功能。

Math对象的方法:

* min()：确定一组数值中的最小值
* max()：确定一组数值中的最大值
* ceil()：执行向上舍入
* floor()：执行向下舍入
* round()：执行标准舍入，四舍五入
* random()：返回大于等于0小于1的一个随机数

关于获取随机数：

__值 = Math.floor(Math.random() * 可能值的总数 + 第一个可能的值)__

选择一个1到10的数：

```javascript
var num = Math.floor(Math.random() * 10 + 1);
```

求一个范围之间的值：

```javascript
function selectFrom(low, up) {
  var choices = up - low + 1;
  return Math.floor(Math.random() * choices + low);
}
var num = selectFrom(1, 10);
console.log(num); //介于1和10之间的一个数值(包括1和10)
```

## 总结

引用类型这一章内容很多，看得有点匆忙。