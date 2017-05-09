---
title: JavaScript高级程序设计学习笔记九
date: 2016-05-06 22:43:25
tags: javascript
categories: onebook
---

关于JavaScript函数式和闭包的介绍

<!--more-->

## 函数表达式

函数声明与函数表达式的区别在于，前者会有一个函数提升的过程，即函数声明可以在任何位置，在需要执行的地方就会正确执行，而函数表达式则需要写在执行的地方的前面，否则会报错。不过函数表达式还有很多有用的地方。

## 递归

正常的写法：

```javascript
function factorial(num) {
	if(num < 1) {
		return 1;
	} else {
		return num * factorial(num - 1);
	}
}
```

这种方式在更换方法名后可能会失效，所以我们采用一种更好的解耦的方式，利用arguments.callee这个指向正在执行的函数的指针：

```javascript
function factorial(num) {
	if(num < 1) {
		return 1;
	} else {
		return num * arguments.callee(num - 1);//factorial换成arguments.callee
	}
}
```

由于严格模式不能通过脚本访问arguments.callee，所以使用命名函数表达式：

```javascript
var factorial = (function f(num) {
	if(num < 1) {
		return 1;
	} else {
		return num * f(num - 1);
	}
});
```

这里使用函数表达式能够解决递归更换函数名导致调用错误函数的问题。

## 闭包

闭包是指有权访问另一个函数作用域中的变量的函数。

函数创建到执行完毕的过程：

1. 创建函数：创建初始作用域链，保存在函数的[[Scope]]属性中，里面包含全局变量对象
2. 调用函数：创建执行环境并被推入到环境栈中，里面包含用[[Scope]]属性中的初始作用域链构建的执行作用域链
3. 活动对象：创建活动对象，并将其推入执行环境作用域的前端
4. 标识符解析：从作用域链的前端开始，一级一级地搜索标识符
5. 执行完毕：执行环境被弹出环境栈，作用域链被销毁，响应的局部活动变量也将被销毁
6. 注意：在另一个函数内部定义的函数会将包含函数的活动对象添加到它的作用域链中，也就是1

闭包指的就是上面的6，内部函数的执行环境作用域链上保存着外部函数的活动对象的引用，基于垃圾回收机制，只要内部函数还有从全局环境对象来的引用，那即使外部函数执行完毕，其执行环境的作用域链会被销毁，可外部函数的活动对象却不会被销毁。

由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存，要谨慎使用。

闭包保存机制的问题：

```javascript
function createFunction() {
	var result = new Array();

	for(var i = 0; i < 10; i++) {
		result[i] = function() {
			return i;
		}
	}

	return result;
}

var arr = createFunction();
for(var i = 0, len = arr.length; i < len; i++) {
	console.log(arr[i]());//都是10
}
```

闭包保存的是整个变量对象，每一个闭包保存的都是同一个共有对象，这个共有对象里面的i是在createFunction()返回时的i的值。

稍微更改一下，达到预期效果：

```javascript
function createFunction() {
	var result = new Array();

	for(var i = 0; i < 10; i++) {
		result[i] = (function(num) {
			return function() {
				return num;
			};
		})(i);
	}

	return result;
}

var arr = createFunction();
for(var i = 0, len = arr.length; i < len; i++) {
	console.log(arr[i]());//从0至9
}
```

添加一个立即执行函数，使每一个立即执行函数的活动对象被保存，闭包保存的是外层函数执行完成后的变量对象。

## this对象

闭包不能保存this对象，在使用之前要把this对象赋值给一个变量，然后在闭包中使用这个变量，否则闭包里面的this对象将是闭包执行环境的this对象，原因就是执行环境的作用域链里面的活动变量里面的this对象覆盖了闭包保存的那个外部函数的活动变量里面的this对象。

## 内存泄露

IE使用引用计数垃圾收集，而JavaScript使用标记-清除算法，导致了在使用闭包时会有内存泄露的问题：

```javascript
function assignHandler() {
	var element = document.getElementById("someElement");
	element.onclick = function() {
		console.log(element.id);
	}
}
```

以上代码中闭包保存了element，element至少会有一个引用，所以不会被浏览器回收，element不会被回收导致挂在它上面的闭包也不会被JavaScript回收，创建了一个循环引用。

解决办法：

```javascript
function assignHandler() {
	var element = document.getElementById("someElement");
	var id = element.id;
	element.onclick = function() {
		console.log(id);
	}
	element = null;//制空，保证闭包中不直接引用element
}
```

或者：

```javascript
function assignHandler() {
	//element不在成为变量
	document.getElementById("someElement").onclick = function() {
		console.log(this.id);
	}
}
```

关于内存泄露还有很多种情况，我还在找相关文章。

## 模仿块级作用域

块级作用域：

```javascript
(function() {
	//这是一个块级作用域
})();
```

这是一个立即执行函数，function关键字前面加括号阻止其函数声明(不一定非要用括号)，这个函数内部的变量不会被外界使用，除非使用闭包。

## 私有变量

私有变量函数内部不被外界拿到的变量，不过有时还是要提供访问私有变量的公有方法。

### 在构造函数中定义特权方法：

```javascript
function MyObject() {
	//私有变量和私有函数
	var privteVariable = 10;

	function privateFunction() {
		return false;
	}

	//特有方法
	this.publicMethod = function() {
		privateVariable++;
		return privateFunction();
	}
}
```

* 优点：私有变量在每一个实例中可以不同
* 缺点：每一个实例都会创建同样一组新方法

### 在私有作用域中定义：

```javascript
(function() {

	//私有变量和私有函数
	var privateVariable = 10;

	function privateFunction() {
		return false;
	}

	//构造函数
	//这里前面是没有var的，它是全局变量才不会被当做私有变量
	MyObject = function(){}

	//公有/特权方法
	MyObject.prototype.publicMethod = function() {
		privateVariable++;
		return privateFunction();
	}
})();
```

* 优点：使用原型，代码复用
* 缺点：每一个实例都没有自己的私有变量

以上两种方式除了publicMethod()这一个途径外，没有办法直接访问privteVariable和privateFunction()。

### 模块模式

这种模式在需要对单例进行某些初始化，同时又需要维护其私有变量时是非常有用的。

增强版单例：
	
```javascript
var singleton = function() {

	//私有变量和私有函数
	var privateVariable = 10;

	function privateFunction() {
		return false;
	}
	//公有/特权方法和属性
	return {
		publicProperty: true,

		publicMethod: function() {
			privateVariable++;
			return privateFunction();
		}
	};
}();
```

## 总结

闭包与作用域链息息相关，认清作用域链对理解闭包很重要。