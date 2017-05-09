---
title: jquery插件开发相关
date: 2016-05-06 23:25:40
tags: javascript
categories: work
---

打算把我之前写的一个轮播组件封装成jquery插件，一边学一边改，这个轮播组件的设计思路是把图片拼成一排，然后修改整排的left属性，在配上js的animate方法，实现轮播滑动

<!--more-->

这个组件的结构是这样的：

```javascript
function Carousel() {}
Carousel.prototype.afreash = function(){}
Carousel.prototype.listener = function(){}
...
```

主要讲下我修改其为jquery插件，具体都改了哪些地方。

__把全部代码放在闭包里__

此时闭包相当于一个私有作用域，外部无法访问到内部的信息，并且不会存在全局变量的污染。

```javascript
(function($) {
	//在局部作用域中用$引用jQuery
	//...
})(jQuery);
```

__添加 Carousel.DEFAULTS = {} 来保存默认参数__

```javascript
Carousel.DEFAULTS = {
	interval: 5000,
	limit: 0.2,
	mouse: 50,
	toler: 0,
	speed: 500,
	up: false
}
```

__添加Plugin方法__

这个Plugin方法是在bootstrap里面看到的，直接拿来用了。
$.extend(target, [object1], [objectN]) 这个方法主要用来合并两个或更多个对象的内容到第一个对象。
值得注意的是：多个对象合并时，会破坏第一个对象的结构，所以可传递一个空对象作为第一个参数。

```javascript
function Plugin(option) {
	//返回遍历结果是为了保存jquery的链式调用
	
	return this.each(function() {
		var $this   = $(this);
	    var data    = $this.data('xf.carousel');
	    //将默认参数和新修改的参数绑到一起
	    var options = $.extend({}, Carousel.DEFAULTS, $this.data(), typeof option == 'object' && option);
	    //方法名
	    var action  = typeof option == 'string' ? option : false;
	    
	    if(!data) {
	    	//持久化数据
	    	$this.data('xf.carousel', (data = new Carousel(this, options)));
	    	data.init();
	    }
	    
	    if(typeof option == 'number') {
	    	data.go(option);
	    } else if(action) {
	
	    	data[action]();
	    } else if(options.interval) {
	    	data.options = options;
	    	data.afresh();
	    	data.alarm();
	    }
	});
}
```
__把Plugin绑定到jquery原型上，也就是$.fn上__

```javascript
$.fn.carousel = Plugin;
$.fn.carousel.Constructor = Carousel;
```

__使用$.proxy__

在处理内部代码的时候，遇到在setTimeout和setInterval里面执行方法，如何引入正确this的情况
使用$.proxy可以非常干净的实现，$.proxy本身依靠得还是apply

感兴趣的看官点击这里 [github:carousel](http://github.com/ggsoul/carousel)