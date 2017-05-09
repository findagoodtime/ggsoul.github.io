---
title: JavaScript高级程序设计学习笔记四
date: 2016-05-06 22:21:59
tags: javascript
categories: onebook
---

关于JavaScript的操作符和语句的介绍

<!--more-->

## 操作符

### 一元操作符

前置递增和递减操作时，变量的值都是在语句被求值以前改变的；后置递增和递减操作时，在包含它们的语句被求值之后才执行的。前置、后置与执行优先级相等

```javascript
var age = 29;
var anotherAge = --age + 2; //30
console.log(age) //28
//类似前置操作的实现
anotherAge = (function () { return age = age - 1; })() + 2;

age = 29;
anotherAge = age-- + 2; //31
console.log(age); //28
//类似后置操作的实现
anotherAge = (function () { setTimeout(function() { age = age - 1; }, 0); return age; })() + 2;
//实现的方式不好，不过大概这个意思
setTimeout(function() {
  console.log(anotherAge);
  console.log(age);
}, 0);
```

对非数值应用一元加或一元减操作符时，该操作会像Number()一样对这个值执行转换。

### 位操作符

我在上一节中使用parseInt()和toString()实现的两个方法，重新用位操作的实现方式：

```javascript
function ipToNum(ip) {
  //左移操作，空出8位，按位或操作补上8位，无符号右移0为无符号整数
  return ip.split('.').reduce(function (sum, x) { return sum << 8 | x }, 0) >>> 0;
}

function numToIp(num) {
  //无符号右移，将右边8位之外的按位与来去掉
  return [num >>> 24, num >> 16 & 255, num >> 8 & 255, num & 255].join('.'); 
}
```
### 布尔操作符

逻辑与`&&`和逻辑或`||`可以做短路操作。

### 相等操作符

相等操作符`==`和全等操作符`===`。

* 相等与不相等  先转换再比较  [相等比较算法](http://ecma-international.org/ecma-262/5.1/#sec-11.9.3)
* 全等与不全等  仅比较不转换  [全等比较算法](http://ecma-international.org/ecma-262/5.1/#sec-11.9.6)

补充一点：比较如果两边都是字符串，比较字符串对应的字符编码值，而大小写字符编码值不同。

全等比较其实就是相等比较中类型一致的那一部分比较算法，想要使用全等比较的话就应该使用全等比较，这样能够清晰地表明意图。

### 逗号操作符

逗号操作符(,)对多个操作数进行求值并返回最后一个操作数的值。它常常用在for循环中，在每次循环时对多个变量进行更新。[逗号操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Expressions_and_Operators#comma_operator)

```javascript
for (var i = 0, j = 9; i <= 9; i++, j--)
  document.writeln("a[" + i + "][" + j + "]= " + a[i][j]);
```

在JavaScript中，逗号运算符的优先级比赋值运算符要底

```javascript
alert(2*5, 2*4); // 输出10
//修改一下
alert((2*5, 2*4)); // 输出8
```

## 语句

不存在块级作用域，循环内的变量，外部可以访问到。

### label语句

```javascript
var num = 0;

outermost:
for(var i = 0; i < 10; i++) {
  for(var j = 0; j < 10; j++) {
    if(i == 5 && j == 5) {
      break outermost;
    }
    num++;
  }
}

console.log(num);
```

### switch语句

```javascript
var num = 25;
switch (true) {
  case num < 0:
    console.log("Less than 0.");
    break;
  case num >= 0 && num <= 10:
    console.log("Between 0 and 10.");
    break;
  case num > 10 && num <=20:
    console.log("Between 10 and 20.");
    break;
  default:
    console.log("More than 20.");
}
```

需要注意的两点：

* 如果需要混合几种情形，不要忘记在代码中添加注释，说明是有意省略了break关键字
* switch语句在比较值时使用的是全等操作符

## 函数

ECMAScript中的函数使用function关键字来声明，后面跟一组参数以及函数体。更为方便的地方是，ECMAScript函数不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型，而Java和C都是需要指明参数个数的，并且C语言的输入输出是要通过占位符指明数据类型的。

ECMAScript之所以这样随便传递参数，原因是每一个函数体内部都有一个arguments对象，它和数组类似，不过不是Array实例。

```javascript
//参数可以重写，但是不要这样做
function doAdd(num1 ,num2) {
  arguments[1] = 10;
  console.log(arguments[0] + num2);
}
doAdd(1,2);//11
```