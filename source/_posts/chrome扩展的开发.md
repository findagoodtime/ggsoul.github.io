---
title: chrome扩展的开发
date: 2016-05-06 23:06:56
tags: [chrome,javascript]
categories: speedy
---

这是本人写的第一个chrome扩展，这个扩展的普遍适用性不强，但是确实很方便，具体的开发流程写在这里，感兴趣的看官可以试着自己动手写一写

<!--more-->

这个扩展的作用是`change query`，它的适用场景是更换百度搜索页的关键词，并且跳转，如果你理解了这个意思，那你一定会想『这能有什么用？』，是的，这在具体生活和工作中一点用处都没有，它仅仅对笔者和笔者身边的产品与测试有一点用处，笔者这两个月的开发任务是一组query下的百度搜索结果页卡片。在这里，笔者想说自己开发chrome扩展更多的是满足自己的切身需要，因地制宜

__下面主要介绍具体开发流程__

## manifest.json配置文件

第一步就是创建```manifest.json```配置文件：

* ```manifest_version```、```name```和```version```为必选，其它为可选
* 这个文件中```manifest_version```默认为2
* ```name```、```version```和```description```很明显，其中```version```要书写规范，且递增
* ```icons```是一个对象，key是像素值，value是图片地址，chrome会选取合适像素的图片在合适的位置（右上角还是扩展程序页面）当做logo
* ```background```指后台执行环境，指定js文件就可以，因为后台基本没有展现页面的需要
* ```permissions```指都用到了哪些权限，本地保存的权限，操作tab页的权限等，这些权限要在这里声明
* ```browser_action```指左键点击右上角logo弹出的页面，这个页面在点开的时候加载出来，收回的时候被销毁
* ```options_page```指右键点击右上角logo弹出列表中的```选项```是否可点，与可点时左键点击打开的页面
* ```content_scripts```指可以在chrome窗口页运行的js文件，matches用来匹配哪些url的窗口页运行

```json
{
	"manifest_version": 2,
	"name": "Change query",
	"version": "1.0",
	"description": "快速切换导入列表中的query",
	"icons": {
        "48": "img/icon48.png"
    },
	"background": { "scripts": ["./js/background.js"] },
	"permissions": [
		"storage",
		"tabs"
	],
	"browser_action": {
		 "default_icon": {
	        "38": "img/icon38.png"
	    },
		"default_popup": "popup.html"
	},
	"options_page": "options.html",
	"content_scripts": [
        {
            "matches": [
            	"http://*.baidu.com/",
				"https://*.baidu.com/"
            ],
            "js": ["js/open.js"]
        }
    ]
}
```

chrome主要提供了三个运行环境，```background```后台持续运行环境，```browser_action```logo弹出页短暂运行环境，```content_scripts```用户正在浏览页面的操作环境，这个环境里可以操作页面内的元素，但是与页面内的原始js是各自独立的，这三个环境可以通过chrome提供的runtime接口来实现通信，通过runtime接口还可以在不同扩展间通信

## 开发browser_action页面

笔者开发的这个chrome拓展，功能很小，只用到了```browser_action```页面，本文也将只介绍```browser_action```页面的开发，下面是html代码：

```html
<!DOCTYPE html>
<html>
<head>
<link rel="stylesheet" type="text/css" href="./css/style.css">
</head>
<body>
    <textarea class="query-area"></textarea>
    <ul class="query-list"></ul>
	<button id="btn">提交</button>
	<button id="prev">上一个</button>
	<button id="next">下一个</button>
</body>
<script type="text/javascript" src="./js/popup.js"></script>
</html>
```

这里需要注意的是:

* 不可以在html页面里面直接写js代码，只能引用js文件
* 上文提到的，这个页面点开创建页面，收回销毁页面，不会保存变量信息

## Change query拓展的用处

html上面的元素很简单，一个textarea，一个ul，三个button。本拓展的逻辑是在textarea中粘贴进query列表，点击『提交』按钮，接下来通过点击『上一个』或『下一个』来切换相邻query，跳转到相应的结果页面

这个拓展开发的目的很简单，在开发完成后，要对所有的搜索query进行确认，需要在编辑器上复制query，粘贴到输入框回车，切换起来很繁琐，所以开发了这个一次性复制粘贴query，然后在拓展上点击就可以轻松切换query，节省测试时间

第一步，点开popup页：

![change-query02](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/change-query01.jpeg)

第二步，复制query列表，粘贴进textarea：

![change-query02](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/change-query02.jpeg)

第三步，提交：

![change-query02](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/change-query03.jpeg)

第四步，点击下一页：

![change-query02](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/change-query04.png)

可以看到飘红的query是当前搜索的query：

![change-query02](http://7xir4w.com1.z0.glb.clouddn.com/blog/images/change-query05.png)

## Change query拓展用到的API

只用到了两个API，chrome.storage.local与chrome.tabs，使用这两个API需要在```manifest.json```文件的```permissions```中添加『storage』和『tabs』

chrome.storage.local用来本地存储数据，具体使用的两个方法：

```javascript
chrome.storage.local.set({});
chrome.storage.local.get(null, function(data) {});
```

chrome.tabs用来操作tab页，具体使用的方法：

```javascript
// 获取当前用户正在浏览的tab页的url
chrome.tabs.query({active: true}, function(tabs) {
	self.url = tabs[0].url;
});
// 监听当用户切换tab页时，获取切换到的tab页的url
chrome.tabs.onUpdated.addListener(function(tabId, changeInfo, tab) {
	self.url = tab.url;
});
// 操作当前tab页跳转url
chrome.tabs.update(null, {url:nextUrl});
```

本项目的下载地址：[点击下载](http://7xir4w.com1.z0.glb.clouddn.com/blog/zip/query.zip)

## 总结

chrome拓展商店里有很多优秀的拓展可以方便我们的生活与工作
chrome拓展开发很简单，多多动手，科技改变生活

对想学习更详细chrome拓展的同学，推荐这里学习：

[官网网站](https://developer.chrome.com/extensions/getstarted.html)

[Chrome扩展及应用开发](http://www.ituring.com.cn/book/1421)

[中文API](https://crxdoc-zh.appspot.com/apps/api_other)
