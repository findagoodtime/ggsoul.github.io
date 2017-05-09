---
title: JavaScript高级程序设计学习笔记二
date: 2016-05-06 21:39:36
tags: javascript
categories: onebook
---

`<script>`元素是为了给HTML页面中引入JavaScript脚本而产生的

<!--more-->

## script元素

### 六个属性

* async：异步加载(仅适用于与外部脚本)
* charset：表示通过src属性指定的代码的字符集
* defer：延迟加载
* language：已弃用，原本表示代码使用的脚本语言
* src：表示包含要执行的外部文件
* type：表示编写代码使用的脚本语言，可以看成language的替代属性

### 两种使用方式

#### 直接嵌入HTML文件中

```javascript
<script type="text/javascript">
	function sayHi() {
		alert("Hi!");
	}
</script>
```

这种方式需要注意的地方是不要在代码中出现`</script>`字符串，会被当做
`<script>`元素的结束，解决办法是使用转义字符`</script>`。

#### 使用src属性包含外部文件

```javascript
<script type="text/javascript" src="examplr.js"></script>
```

## 关于JavaScript引入

### 内联引入

```html
 <!DOCTYPE html>
 <html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <link href="css/style.css" rel="stylesheet">
    <title>Critical Path: Script</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="../assets/images/header.jpg"></div>
    <script>
      var span = document.getElementsByTagName('span')[0];
      span.textContent = 'interactive'; // change DOM text content
      span.style.display = 'inline';  // change CSSOM property
      // create a new element, style it, and append it to the DOM
      var loadTime = document.createElement('div');
      loadTime.textContent = 'You loaded this page on: ' + new Date();
      loadTime.style.color = 'blue';
      document.body.appendChild(loadTime);
    </script>
  </body>
</html>
```

这是一段例子，引自[Google developers](http://developers.guge.io/web/fundamentals/performance/critical-rendering-path/adding-interactivity-with-javascript?hl=zh-cn)

这段代码里面的内联脚本靠近页面底部，如果我们将脚本移至span元素上方，您会发现脚本不起作用，并提示无法再文档中找到任何span元素的引用。这表明一个重要的特性：脚本会在脚本中插入的确切点执行。HTML解析器遇到脚本代码时，它会暂停构建DOM的流程，并对JavaScript引擎进行控制；JavaScript引擎运行完后，浏览器就会从断开的地方继续运行并恢复DOM构建。

总结一句话就是：*执行内联脚本会阻止 DOM 构建，也会使首次呈现出现延迟*

内联脚本即使放在靠近页面底部的位置也还是会有首次呈现出现延迟的问题，不过编写其他代码可以推迟它们的执行。

### defer属性

defer是HTML4.0中定义的，作用是等文档完成解析完成后按照它们在文档中出现的顺序再去下载解析。

#### defer在浏览器支持程度不同

```javascript
//defer测试代码，可将代码复制到本地自己测试，外部脚本src引入，内联脚本直接粘帖
<script type="text/javascript" defer>
    alert('defer')
</script>
<script type="text/javascript">
    alert('script')
</script>
<script type="text/javascript">
    window.onload = function(){
        alert('onload')
    }
</script>
```

* 外部JS在各个浏览器里运行结果跟定义的执行顺序正常，alert信息会按照 script->defer->onload顺序弹出
* 内联脚本，如果脚本都是IE9/8/7/6按照定义的顺序弹出信息，其他浏览器则按照 defer->script->onload 顺序弹出信息，表示defer失效
* 而如果有多个内联defer脚本、在body和head都有分布或者在iframe中也有内联defer脚本，则在IE6中表现一致。

(IE6我没有装虚拟机，所以没有测试)

#### defer的使用

__外部引用加defer__

![note1](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/note1.png)

__内联不加defer(chrome)__

![note2](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/note2.png)

感觉没啥差别，而且defer浏览器表现不同的问题，所以如果是内联JavaScript的话，将脚本放在body底部比给脚本增加defer属性让脚本延迟加载更好。

### async属性

async是HTML5的新增属性，IE10和主流浏览器都支持该属性，作用是让js文件和浏览器加载css一样是异步加载的。

async实现的引入与上面两者的比较中，优势很明显

![note3](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/note3.png)

一种添加额外代码实现的异步加载，在DOM中创建script标签：

```javascript
function delay_js(src) {
	var objScript = document.createElement('script');
	objScript.setAttribute('src', src);
	objScript.setAttribute('type', 'text/javascript');
	document.body.appendChild(objScript);
	return objScript;
}
```

以上代码异步加载的js下载时跟其他一样是并行的，但是执行阶段还是会阻止页面渲染，延长了window.onload的事件。

```javascript
function loadjs(src, succ) {
    var elem = delay_js(src);
    if ((navigator.userAgent.indexOf('MSIE') == -1) ? false: true) {
        elem.onreadystatechange = function() {
            if (/loaded|complete/.test(this.readyState)){
                succ()
            }
        };
    }else{
        elem.onload = function(){
            succ();
        }
    }
    elem.onerror = function() {};
}
```

这种异步方式，引自[html5jscss](http://www.html5jscss.com/js_async.html)

我曾经写过delay\_js这样的方法，然后把它放在window.onload里面去执行，不过显然让它并行的下载会更快。

我在这里还有一些有误区的地方:

* 在做图片预加载的时候，1.创建对象 2.设置src属性 3.设置onload方法
* 在做JavaScript的异步引入时，1.创建对象 2.设置src属性 3.设置type属性 4.添加到body里面 5.设置onload方法

这之间明显的区别是创建的img标签不用添加到body里面就可以触发onload，而script标签不添加到body里面将不会触发onload。我询问了下微信，他告诉我`<script>`元素插入body后是执行，而下载只是执行的一部分，下载完成后JavaScript会立即执行，而图片预加载是创建出了Image对象，直接就可以下载内容了。

下图为normal、defer和async的比较，引自[Peter Beverloo](http://peter.sh/experiments/asynchronous-and-deferred-javascript-execution-explained/)

![execution2](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/execution2.jpg)

推荐使用引入外部js文件的方式，它有几点优势：

* 可维护性：遍及不同HTML页面的JavaScript会造成维护问题。但把所有JavaScript文件都放在一个文件夹中，维护起来就轻松多了。
* 可缓存：浏览器能够根据具体的设置缓存链接的所有外部JavaScript文件。如果有两个页面都使用一个文件，那么这个文件只需要下载一次。
* 可以使用async这个属性，并且没有副作用的使用defer属性。达到的目的是告诉浏览器我这个文件是需要下载的，就不要让DOM构建停止下来了。

## noscript元素

这个元素是为了当浏览器不支持JavaScript时，让页面平稳退化的，现在不支持JavaScript的浏览器很少，不过会有禁止脚本的情况。这两中情况都可以使用`<noscript>`元素。

```html
<body>
	<p>本页面需要浏览器支持(启用)JavaScript。</p>
</body>
```

## 总结

JavaScript在实现DOM和BOM操作的同时，确实也带来了一点干扰，不过只要认真的去分析，就能够将这点干扰降低到最小，接下来将进入JavaScript语言部分。