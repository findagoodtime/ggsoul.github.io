---
title: JavaScript高级程序设计学习笔记八
date: 2016-05-06 22:40:29
tags: javascript
categories: onebook
---

ECMA-262把对象定义为："无序属性的集合，某属性可以包含基本值、对象或者函数。"严格来讲，这就相当于说对象是一组没有特定顺序的值

<!--more-->

## 理解对象

ECMAScript的对象都是基于一个引用类型创建的，用来保存一组键值对，这些值是对象的属性，这些属性决定了对象的用处，开发者操作这些属性来使用对象，对象本身有一个封装的意思。

ECMAScript中有两种属性：

数据属性

* Configurable：可否配置，默认true
* Enumerable：可否枚举，默认true
* Writable：可否写，默认true
* Value：包含这个属性的数据值

访问器属性

* Configurable：可否配置，默认true
* Enumerable：可否配置，默认true
* Get：在读取属性时调用的函数，默认undefined
* Set：在写入属性时调用的函数，默认undefined

要修改属性默认的特性，必须使用ECMAScript5的Object.defineProperty()方法，这个方法接收三个参数：属性所在的对象、属性的名字和一个描述符对象。

要读取属性的特性，可以使用ECMAScript5的Object.getOwnPropertyDescriptor()方法，这个方法接收两个参数：属性所在的对象和要读取其描述符的属性名称。

对象字面量创建的对象的属性都是数据属性，访问器属性只能通过Object.defineProperty()方法来创建。数据属性可以再外界直接调用，而访问器属性更多是用来设置一个属性的值来修改其他属性的值。

## 创建对象

目前普遍使用的创建对象方式：

```javascript
function Person(name, age ,job) {
	this.name = name;
	this.age = age;
	this.job = job;
	this.friends = ["Shelby", "Court"];
}

Person.prototype.sayName = function() {
	console.log(this.name);
}

var person1 = new Person("Sefa", 23, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

这种创建对象方式是用构造函数和原型来实现的。

### new操作符

构造函数本身是不立即执行的，且不返回对象，需要借助于new操作符，new操作符实际的效果只这样的：

1. 一个新对象被创建。
2. 构造函数被执行。执行的时候，相应的传参会被传入，同时上下文(this)会被指定为这个新实例
3. 如果构造函数返回了一个"对象"，那么这个对象会取代整个new出来的结果。如果构造函数没有返回对象，那么new出来的结果为步骤1创建的对象

相对应的类似代码：

```javascript
var obj = {};
obj.prototype = Person.prototype;
Person.call(obj);
return obj;
```

### 构造函数

构造函数被当做创建对象实例的模板，所实现的是类的一些工作，用来表明要创建的对象有哪些属性，通过传递参数来给属性赋值。不用来表明方法，因为在执行代码的时候，方法都重新创建了一遍，这是没有必要的。

### 原型

无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。由构造函数创建出来的对象实例都指向这个原型，每个构造函数的原型相对于它的对象实例都是共享的。构造函数和它的实例没有直接关系，可是和构造函数的原型有关系，它的实例能够在原型中拿到属性。

由于以上原因，把方法放在原型中共享可以解决构造函数创建多余方法实例的问题。

### in操作符

有两种方式使用in操作符：

* 单独使用：in操作符能够访问给定属性时返回true，无论该属性存在于实例中还是原型中
* 在for-in循环使用：for-in能够返回所有能够通过对象访问的、可枚举的属性

### 原型的使用

更简单的原型语法：

```javascript
function Person() {}

Person.prototype = {
	name: "Sefa",
	age: 23
}

var friend = new Person();
```

这种方式重写了prototype对象，而这种重写实际上是重新创建了一个对象，重写之前创建的实例的prototype属性指向旧的原型对象，重写之后创建的实例的prototype属性将指向新创建的原型对象。这种方式会破坏原型链，在需要实现继承时最好不要给子级构造函数使用。而且这种方式会导致修改constructor属性，不过这个属性保存指向构造函数的指针，没有太大的用，可以忽略。

原型中的属性或方法，不属于任何一个实例，只是可以通过实例查找到，因此原型具有动态性，我们对原型对象所做的任何修改都能够立即从实例上反映出来。

## 继承

继承范式，引自[Mozilla developer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript#Inheritance)

```javascript
// 定义Person构造器
function Person(firstName) {
  this.firstName = firstName;
}

// 在Person.prototype中加入方法
Person.prototype.walk = function(){
  console.log("I am walking!");
};
Person.prototype.sayHello = function(){
  console.log("Hello, I'm " + this.firstName);
};

// 定义Student构造器
function Student(firstName, subject) {
  // 调用父类构造器, 确保"this" 在调用过程中设置正确
  Person.call(this, firstName);

  // 初始化Student类特有属性
  this.subject = subject;
};

// 建立一个由Person.prototype继承而来的Student.prototype对象.
Student.prototype = Object.create(Person.prototype);

// 设置"constructor" 属性指向Student
Student.prototype.constructor = Student;

// 更换"sayHello" 方法
Student.prototype.sayHello = function(){
  console.log("Hello, I'm " + this.firstName + ". I'm studying " + this.subject + ".");
};

// 加入"sayGoodBye" 方法
Student.prototype.sayGoodBye = function(){
  console.log("Goodbye!");
};

// 测试实例:
var student1 = new Student("Janet", "Applied Physics");
student1.sayHello();   // "Hello, I'm Janet. I'm studying Applied Physics."
student1.walk();       // "I am walking!"
student1.sayGoodBye(); // "Goodbye!"

// Check that instanceof works correctly
console.log(student1 instanceof Person);  // true 
console.log(student1 instanceof Student); // true
```

### 原型链

实例包含指向原型对象的内部指针，让原型对象等于另一个对象的实例，这时，这个原型对象将包含指向另一个原型对象的指针，如果原型对象一个一个的相互关联起来，就构成了一条实例与原型的链条。用在原型链上的原型相关的构造函数创建出来的实例，拥有查找它的原型在原型链上的上级的原型对象的属性和方法的能力，原型链依次来实现继承。

不过在子类型的原型上包含超类型的全部实例属性，这些属性不应该全部共享出来。而且在创建子类型的时候，不能向超类型的构造函数中传递参数。

### 借用构造函数

在子类型的构造函数中执行超类型的构造函数，可以在执行的时候传入相应参数，同时要将this指向子类型。这种方式可以不共享实例属性，同时传递了参数给超类型构造函数。实现的话用call或apply都可以。

### Object.create()

ECMAScript5新增Object.create()方法，实现类似：

```javascript
function object(o) {
	function F(){}
	F.prototype = o;
	return new F();
}
```

这个方法接收两个参数：一个用来作新对象原型的对象和一个为新对象定义额外属性的对象。

Object.create()创建出来的对象，需要通过后期增强和原型共享来实现多样性的实例，这些实例是继承自模板对象的子类型。

Object.create()在这里是对原型链上不需要共享属性被共享出来的问题的进一步解决，借用构造函数保证了实例属性不共享，但是在原型中不需要共享的属性还是存在的，解决办法是：

```javascript
Student.prototype = Object.create(Person.prototype);
```

子类型原型将不再赋值为超类型的实例，而是赋值为一个仅仅拿到超类型原型对象的一个引用的对象。

## 总结

在理解使用原型链继承的同时，还要清楚代码中原型链的长度，并在必要时结束原型链，以避免可能存在的性能问题。