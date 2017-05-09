---
title: JavaScript高级程序设计学习笔记六
date: 2016-05-06 22:29:37
tags: javascript
categories: onebook
---

关于JavaScript的几种基本类型的介绍

<!--more-->

## Object类型

创建对象有两种方式：

### new操作符

```javascript
var person = new Object();
person.name = "Sefa";
person.age = 29;
```

### 对象字面量

```javascript
var person = {
	name: "Sefa",
	age: 29
}
//这里{}表示的是一个值，因为跟在赋值操作符后面
//{}还可以表示语句块，比如跟在if语句条件的后面
```

两种方式更倾向于后者，看着简洁，而且字面量表示法一定是比调用函数方法更高效，后面会讲到的，var array = []比var array = new Array()高效。不仅于此，对象字面量是向函数传递大量可选参数的首选方式，或者修改已有参数对象，类似这样：

```javascript
function setInfo(args) {
	var def = {
		name: "null",
		age: 0
	}
	for(var param in args) {
		//方括号表示法，可以容忍属性名包含会导致错误的字符，或关键字和保留字
		//或者像现在这种情况，只能使用方括号
		def[param] = args[param];

		//def.param = args.param;如果是这样的话，最后def将是：
		//Object {name: "null", age: 0, param: undefined}
	}
}
setInfo({
	name: "Nicholas",
	age: 29
});
setInfo({
	name: "Greg",
	like: "book"
});
```

## Array类型

ECMAScript数组有两点不同寻常之处：

* 数组中的每一项可以保存任何类型的数据
* length是可以设置的，设置小于当前length则移除后面差项，大于当前length则给增加差项并填充undefined值

### 检测数组

```javascript
function isArray(o) {
    return Object.prototype.toString.call(o) === '[object Array]';
}
```

### 数组方法

这是我自己试着实现的一些数组方法：

栈方法

```javascript
//后进，在数组尾部添加元素
Array.prototype.sefaPush = function() {
	var self = this,
		args = this.slice.call(arguments);
	args.forEach(function(v, i) {
		self[self.length] = v;
	})
	return self;
}
//后出，通过length-1去掉末尾元素
Array.prototype.sefaPop = function() {
	if(this.length == 0) {
		return;
	}
	var last = this[this.length - 1];
	this.length = this.length - 1;
	return last;
}
```

队列方法

```javascript
//前出，移除第一个后面向前移动一位
Array.prototype.sefaShift = function() {
	if(this.length == 0) {
		return;
	}
	var first = this[0];
	this.forEach(function(v, i, self) {
		self[i] = self[i + 1];
	});
	this.pop();
	return first;
}
//前进，拼接数组
Array.prototype.sefaUnshift = function() {
	var args = this.slice.call(arguments);
	args.reverse();
	return args.concat(this);
}
```

重排序方法

```javascript
//反转，浅复制然后依次替换
Array.prototype.sefaReverse = function() {
	var self = this,
		tArr = this.slice(),
		len = this.length - 1;
	this.forEach(function(v, i) {
		self[i] = tArr[len - i];
	});
}
//排序，快速排序
Array.prototype.sefaSort = function() {
	var self = this,
		arr = [],
		compare;
	//如果传入方法，按照方法比较
	if(arguments.length != 0 && typeof arguments[0] == "function") {
		compare = arguments[0];
		arr = quickSort(this);
		arr.forEach(function(v, i){
			self[i] = v;
		})
		return;
	}
	//如果没有传入方法，全部转换成字符串比较
	arr = this.map(function(v) {
		return (typeof v === "object") ? v.toString() : "" + v;
	})
	compare = function(prev, next) {
		if(prev < next) {
			return -1;
		} else if(prev > next) {
			return 1;
		} else {
			return 0;
		}
	}
	arr = quickSort(arr);
	arr.forEach(function(v, i){
		self[i] = v;
	})
	//快速排序方法
	function quickSort(arr) {
		if (arr.length <= 1) { return arr; }
		var pivotIndex = Math.floor(arr.length / 2);
		var pivot = arr.splice(pivotIndex, 1)[0];
		var left = [];
		var right = [];
		for (var i = 0; i < arr.length; i++){
			if (compare(arr[i], pivot) !== 1) {
			left.push(arr[i]);
			} else {
			right.push(arr[i]);
			}
		}
		return quickSort(left).concat([pivot], quickSort(right));
	}
}
```

操作方法

```javascript
//连接，依次替换
Array.prototype.sefaConcat = function() {
	var args = this.slice.call(arguments),
		tArr = [];
	this.forEach(function(v) {
		tArr.push(v);
	});
	args.forEach(function(v) {
		if(isArray(v)) {
			v.forEach(function(value) {
				tArr.push(value);
			});
		} else {
			tArr.push(v);
		}
	});
	return tArr;
}
//浅复制
Array.prototype.sefaSlice = function(start, end) {
	var start = start == undefined ? 0 : start >= 0 ? start : start + this.length,
		end = end == undefined ? this.length : end >= 0 ? end : end + this.length,
		tArr = [],
		i = start;
	if(end < start) {
		return [];
	}
	for( ; i < end; i++) {
		tArr.push(this[i]);
	}
	return tArr;
}
//增删改
Array.prototype.sefaSplice = function() {
	var self = this,
		args = this.slice.call(arguments),
		tArr = [],
		sArr = this.slice(),
		len = args.length;
	this.length = 0;
	switch(true) {
		case len < 2:
			break;
		case len === 2:
			//删除操作，指定两个参数，要删除的第一项的位置和要删除的项数
			sArr.forEach(function(v, i) {
				if(args[0] === i && args[1] !== 0) {
					tArr.push(v);
					args[0]++;
					args[1]--;
				} else {
					self.push(v);
				}
			});
			break;
		case args[1] === 0:
			//插入操作，指定三个或更多参数，起始位置，0和要插入的项
			sArr.forEach(function(v, i) {
				if(args[0] === i){
					args.slice(2).forEach(function(value) {
						self.push(value);
					});
				}
				self.push(v);
			});
			break;
		default:
			//替换操作，指定三个或更多参数，起始位置、要删除的项数和要插入的项
			sArr.forEach(function(v, i) {
				if(args[0] === i && args[1] !== 0) {
					tArr.push(v);
					args[0]++;
					args[1]--;
					if(args[1] === 0 || args[0] === len) {
						args.slice(2).forEach(function(value) {
							self.push(value);
						});
					}
				} else {
					self.push(v);
				}
			});
	}
	//返回，删除的项
	return tArr;
}
```

位置方法

```javascript
//查找
Array.prototype.sefaIndexOf = function(value, index) {
	var i = index || 0,
		len = this.length;
	for( ; i < len; i++) {
		if(this[i] === value) {
			return i;
		}
	}
	return -1;
}
```

迭代方法(使用时注意循环内this指向的是数组)

```javascript
//全部
Array.prototype.sefaEvery = function(callback, that) {
	var i = 0,
		len = this.length,
		isAll = true,
		that = that || this;
	for( ; i < len; i++) {
		if(!callback.call(that, this[i], i, this)) {
			isAll = false;
		}
	}
	return isAll;
}
//过滤
Array.prototype.sefaFilter = function(callback, that) {
	var i = 0,
		len = this.length,
		tArr = [],
		that = that || this;
	for( ; i < len; i++) {
		if(callback.call(that, this[i], i, this)) {
			tArr.push(this[i]);
		}
	}
	return tArr;
}
```

归并方法

```javascript
//归并
Array.prototype.sefaReduce = function(callback, first) {
	var i = 1,
		len = this.length,
		tArr = [];
	if(first != undefined) {
		tArr[0] = callback.call(this, first, this[0], 0, this);
		for( ; i < len; i++) {
			tArr[i] = callback.call(this, tArr[i-1], this[i], i, this);
		}
	} else {
		tArr[0] = callback.call(this, this[0], this[1], 1, this);
		for( ; i < len - 1; i++) {
			tArr[i] = callback.call(this, tArr[i-1], this[i+1], i+1, this);
		}
	}
	return tArr[tArr.length - 1];
}
```

Array数组的方法还有很多，这里就不一一实现了，可能有的地方没有验证参数格式，仅供方法功能参考。

试着添加的一些方法

```javascript
//返回数组的一个副本，包含所有值的平方，原始数组不改变
Array.prototype.square = function(){
  return this.map(function(x){ return Math.pow(x, 2); });
}
//返回数组的一个副本，包含所有值的立方，原始数组不改变
Array.prototype.cube = function(){
  return this.map(function(x){ return Math.pow(x, 3); });
}
//返回所有数组值的平均值
Array.prototype.average = function(){
  return !this.length ? NaN : this.reduce(function(prev, now){ return prev + now; })/this.length;
}
//返回所有数组值的总和
Array.prototype.sum = function(){
  return this.reduce(function(prev, now){ return prev + now; });
}
//返回所有偶数编号的值组成的数组，原始数组不改变
Array.prototype.even = function(){
  return this.filter(function(x){ return x%2 == 0; });
}
//返回所有奇数编号的值组成的数组，原始数组不改变
Array.prototype.odd = function(){
  return this.filter(function(x){ return x%2 != 0; });
}
```