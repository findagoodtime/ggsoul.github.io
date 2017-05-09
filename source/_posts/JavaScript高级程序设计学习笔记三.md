---
title: JavaScript高级程序设计学习笔记三
date: 2016-05-06 22:17:21
tags: javascript
categories: onebook
---

JavaScript借鉴了C语言的基本语法和Java语言的数据类型

<!--more-->

## 语法

在JavaScript中标识符和操作符都是区分大小写的

```javascript
"use strict"//严格模式

//单行注释

/*
 *	这是一个多行
 *	（块级）注释
 */

//下面这几种都是标识符
var variableValue;//变量

function doSomething(name) {//函数及其参数
	this.myName = name;//属性
}
```

在ECMAScript中的语句以一个分号结尾，省略分号是有效的语句，不过不推荐。JavaScript有一个自动修复机制，它试图通过自动插入分号来修正有缺损的程序。但是千万不要指望它，它可能掩盖更为严重的错误

```javascript
function getStatus() {
	return 
	{
		status: true
	};
}
```

将返回的是undefined，自动插入分号导致程序被误解，确没有任何警告提醒，变成这样：

```javascript
return;
{
	status: true
};
```


所以不要省略分号结尾

## 变量

ECMAScript的变量是松散类型的，所谓松散类型就是可以用来保存任何类型的数据。这是一个很宽松的用法，从Java到JavaScript的学习过程中，变量的改变是非常大的转变，在Java中常常为了声明变量而写一大列的变量类型和变量名。而在JavaScript中，可以一条语句定义多个变量，不用关注它的类型，当然这需要开发者十分清楚这些变量到底是用来干嘛的，或者开发者为变量起的名字间接表明自身类型，所以JavaScript本身是不适用作为编程新手的入门语言

```javascript
var message = "hi",
	found = false,
	age = 29;
```

## 数据类型

5种基本类型：Undefined、Null、Boolean、Number和String

1种复杂类型：Object

### typeof

typeof是判断数据类型的操作符

* "undefined"：如果这个值未定义
* "boolean"：如果这个值是布尔值
* "string"：如果这个值是字符串
* "number"：如果这个值是数值
* "object"：如果这个值是对象或null
* "function"：如果这个值是函数

### Undefined类型

使用var声明变量但未对其初始化，这个变量就是undefined。其实更多的使用来判断，变量还没有被声明还是尚未初始化

### Null类型

表示一个空对象指针，typeof检测null值时会返回"object"。

### Boolean类型

只有两个字面量：true和false。

### Number类型

使用IEEE754格式来表示整数和浮点数值，是的只有这两种数值，遥想Java的数值类型，byte、short、int、long、float和double，对按照使用范围对数值进行划分，更好的分配内存，或许真的是设计者觉得没那个必要，所以JavaScript只有两种数值类型

Number类型有3个特别的值：

* Infinity：正无穷，超出数值范围上限的值
* -Infinity：负无穷，超出数值范围下限的值
* NaN：非数值，表示一个本来要返回数值的操作数未返回数值的情况，它不等于任何值包括本身

### String类型

表示由零或多个16位Unicode字符组成的字符序列，字符串可以由双引号或单引号表示。

字符串是不可变的，也就是说，字符串一旦创建，它们的值就不能改变。这点和Java是一致的，对此Java提供了StringBuffer来改善字符串拼接的效率问题，而ECMAScript是靠浏览器在新版本中的实现解决的这个问题。我遇到字符拼接这种活，通常都交给数组处理，毕竟浏览器的内存空间是很宽泛的

### Object类型

对象其实就是一组数据和功能的集合，对象可以通过执行new操作符后跟要创建的对象类型的名称来创建。嗯，这一点应该就是根据Java来的，不过Java对不写new这种情况会及时的报错提醒，不过ECMAScript确不会这样

如果在调用构造函数时忘记了在前面加new，构造函数内部的this将不会绑定到新对象上，而会绑定到全局对象上，所以不但没有扩充新对象，反而破坏了全局变量环境。创建对象最好使用Object.create()

Object类型是它所有实例的基础，每个实例具有如下属性和方法：

* constructor：保存着用于创建当前对象的函数
* hasOwnProperty：用于检查给定属性在当前对象实例中是否存在
* isProperty：用于检查传入的对象是否是传入对象的原型
* propertyIsEnumerable：用于检查给定属性是否能够使用for-in语句枚举
* toLocaleString：返回对象的字符串表示，该字符串与执行环境的地区对应
* toString：返回对象的字符串表示
* valueOf：返回对象的字符串、数值或布尔值表示

## 数据转换

Boolean、Number和String类型分别通过Boolean()、Number()和String()函数实现转换

| 数据类型 | Boolean()转换为true | Boolean()转换为false | Number() | String() |
|:-----:|:---:|:---:|:---:|:---:|
| Boolean | true | false | 1和0 | "true"和"false" |
| Number | 任何非零数字值(包括无穷大) | 0和NaN | 简单的传入传出 | 字符串数字 |
| Null | 不适用 | null | 0 | "null" |
| Undefined | 不适用 | undefined | NaN | "undefined" |
| String | 任何非空字符串 | "" | 有更好的解决方案 | 简单的传入传出 |
| Object | 任何对象 | null | 调用valueOf，转换结果为NaN，调用toString | "[object Object]" |

parseInt()会忽略字符串前面的空格，直到找到第一个非空字符。如果第一个字符不是数字或者负号，就会返回NaN。如果第一个字符是数字字符，parseInt()会继续解析第二个字符，知道解析完所有的后续字符或者遇到了一个非数字字符

parseInt()第二个参数和toString()第一个参数的使用

我曾经遇到这样一个题目，将一段IP转换成数字，规则是192.168.1.1先分别转换为二进制得11000000.10101000.00000001.00000001，在拼接在一起成为11000000101010000000000100000001，最后转换为十进制得3232235777

这是我的解法：

```javascript
function ipToNum(ip) {
  return parseInt(ip.split(".").map(function(val, index) {
  	//parseInt(val, 10)，这是一个十进制数，转换成十进制
  	//toString(2)，转换成二进制数的字符串
    val = parseInt(val, 10).toString(2);
    var len = val.length;
    if(len === 8 ) {
      return val;
    } else {
      var zeros = "0";
      for(var i = 1; i < 8 - len; i++) {
        zeros += "0";
      }
      return zeros + val;
    }
    //parseInt(XXX, 2)，这是一个二进制数，转换成十进制
  }).join(""), 2);
}
```

反向求取：

```javascript
function numToIp(num) {
  var ipArr = [];
  //toString(2)，转换成二进制数的字符串
  var numArr = num.toString(2).split("");
  
  while(numArr.length !== 32)
  {
      numArr.unshift("0");
  }
  
  numArr.reduce(function(preVal, curVal, index) {
    var result = preVal + curVal;
    if(index%8 === 7) {
      //parseInt(result, 2)，这是一个二进制数，转换成十进制
      ipArr.push(parseInt(result, 2));
      return "";
    }
    return result;
  });
  return ipArr.join(".");
}
```

这里面充分运用了parseInt()和toString()两个函数的基数参数来实现，下部分将用二进制来实现它