---
title: messengerjs iframe 跨域传数据
date: 2016-05-06 23:08:28
tags: [javascript,iframe]
categories: work
---

刚来公司时做得第一个项目是跨部门合作，使用了MessengerJS来做通信，十分简单，MessengerJS代码不长，这里分析一下iframe间通信的实现方式
<!--more-->

源码

```javascript
/**
 *     __  ___
 *    /  |/  /___   _____ _____ ___   ____   ____ _ ___   _____
 *   / /|_/ // _ \ / ___// ___// _ \ / __ \ / __ `// _ \ / ___/
 *  / /  / //  __/(__  )(__  )/  __// / / // /_/ //  __// /
 * /_/  /_/ \___//____//____/ \___//_/ /_/ \__, / \___//_/
 *                                        /____/
 *
 * @description MessengerJS, a common cross-document communicate solution.
 * @author biqing kwok
 * @version 2.0
 * @license release under MIT license
 */

window.Messenger = (function(){

    // 消息前缀, 建议使用自己的项目名, 避免多项目之间的冲突
    // !注意 消息前缀应使用字符串类型
    var prefix = "[PROJECT_NAME]",
        supportPostMessage = 'postMessage' in window;

    // Target 类, 消息对象
    function Target(target, name, prefix){
        var errMsg = '';
        if(arguments.length < 2){
            errMsg = 'target error - target and name are both required';
        } else if (typeof target != 'object'){
            errMsg = 'target error - target itself must be window object';
        } else if (typeof name != 'string'){
            errMsg = 'target error - target name must be string type';
        }
        if(errMsg){
            throw new Error(errMsg);
        }
        this.target = target;
        this.name = name;
        this.prefix = prefix;
    }

    // 往 target 发送消息, 出于安全考虑, 发送消息会带上前缀
    if ( supportPostMessage ){
        // IE8+ 以及现代浏览器支持
        Target.prototype.send = function(msg){
            this.target.postMessage(this.prefix + '|' + this.name + '__Messenger__' + msg, '*');
        };
    } else {
        // 兼容IE 6/7
        Target.prototype.send = function(msg){
            var targetFunc = window.navigator[this.prefix + this.name];
            if ( typeof targetFunc == 'function' ) {
                targetFunc(this.prefix + msg, window);
            } else {
                throw new Error("target callback function is not defined");
            }
        };
    }

    // 信使类
    // 创建Messenger实例时指定, 必须指定Messenger的名字, (可选)指定项目名, 以避免Mashup类应用中的冲突
    // !注意: 父子页面中projectName必须保持一致, 否则无法匹配
    function Messenger(messengerName, projectName){
        this.targets = {};
        this.name = messengerName;
        this.listenFunc = [];
        this.prefix = projectName || prefix;
        this.initListen();
    }

    // 添加一个消息对象
    Messenger.prototype.addTarget = function(target, name){
        var targetObj = new Target(target, name,  this.prefix);
        this.targets[name] = targetObj;
    };

    // 初始化消息监听
    Messenger.prototype.initListen = function(){
        var self = this;
        var generalCallback = function(msg){
            if(typeof msg == 'object' && msg.data){
                msg = msg.data;
            }
            
            var msgPairs = msg.split('__Messenger__');
            var msg = msgPairs[1];
            var pairs = msgPairs[0].split('|');
            var prefix = pairs[0];
            var name = pairs[1];

            for(var i = 0; i < self.listenFunc.length; i++){
                if (prefix + name === self.prefix + self.name) {
                    self.listenFunc[i](msg);
                }
            }
        };

        if ( supportPostMessage ){
            if ( 'addEventListener' in document ) {
                window.addEventListener('message', generalCallback, false);
            } else if ( 'attachEvent' in document ) {
                window.attachEvent('onmessage', generalCallback);
            }
        } else {
            // 兼容IE 6/7
            window.navigator[this.prefix + this.name] = generalCallback;
        }
    };

    // 监听消息
    Messenger.prototype.listen = function(callback){
        var i = 0;
        var len = this.listenFunc.length;
        var cbIsExist = false;
        for (; i < len; i++) {
            if (this.listenFunc[i] == callback) {
                cbIsExist = true;
                break;
            }
        }
        if (!cbIsExist) {
            this.listenFunc.push(callback);
        }
    };
    // 注销监听
    Messenger.prototype.clear = function(){
        this.listenFunc = [];
    };
    // 广播消息
    Messenger.prototype.send = function(msg){
        var targets = this.targets,
            target;
        for(target in targets){
            if(targets.hasOwnProperty(target)){
                targets[target].send(msg);
            }
        }
    };

    return Messenger;
})();
```

__下面主要分析代码结构__

## supportPostMessage变量

用来检测当前浏览器是否支持postMessage

postMessage是HTML5引入的通信API，它可以避开同源策略的限制，实现安全的跨域通信

向外界窗口发送消息

```javascript
otherWindow.postMessage(message, targetOrigin);
```
* otherWindow:  指目标窗口，也就是给哪个window发消息，是 window.frames 属性的成员或者由 window.open 方法创建的窗口
* message:  是要发送的消息，类型为 String、Object (IE8、9 不支持)，一般使用json数据
* targetOrigin:  是限定消息接收范围，协议+主机+端口号[+URL]，URL会被忽略，所以可以不写，不限制请使用 ‘*’

接受信息的message事件

```javascript
var onmessage = function (event) {
    var data = event.data;
    var origin = event.origin;
    //do someing
};
if (typeof window.addEventListener != 'undefined') {
    window.addEventListener('message', onmessage, false);
} else if (typeof window.attachEvent != 'undefined') {
    //for ie
    window.attachEvent('onmessage', onmessage);
}
```

注意：ie6/7不支持postMessage，因此在ie6/7中跨域通信通常使用window.name

window.name的美妙之处：name 值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 name 值（2MB）

window.navigator有与window.name类似的特性，而且可以保存回调方法

MessengerJS的实现思路是高级浏览器使用postMessage，不支持postMessage的使用window.navigator来保存回调方法

## Target类

消息类，发送执行者

```javascript
function Target(target, name){
    this.target = target;
    this.name = name;
}

Target.prototype.send = function(msg){
    // 发送消息
};
```

## Messenger类

信使类，创建多个消息对象，注册多个监听事件，每一个消息对象的广播消息会被这个信使类下面的所有监听事件接收到

```javascript
function Messenger(messengerName, projectName){
    this.targets = {};
    this.name = messengerName;
    this.listenFunc = [];
    this.initListen();
}

// 添加一个消息对象
Messenger.prototype.addTarget = function(target, name){};

// 初始化消息监听
Messenger.prototype.initListen = function(){};

// 监听消息
Messenger.prototype.listen = function(callback){};

// 注销监听
Messenger.prototype.clear = function(){};

// 广播消息
Messenger.prototype.send = function(msg){};
```

实现逻辑是：

* initListen方法初始化，将generalCallback回调方法注册到message监听中
* addTarget将消息对象添加到targets对象中
* listen方法将监听方法添加到listenFunc数组中
* send方法执行每一个target对象的send方法
* target对象的send方法执行，触发了message监听，触发了generalCallback的执行，从而执行了listenFunc数组中的方法

在postMessage的注册回调方法里加了一个回调方法组listenFunc

在postMessage的监听触发方法外加了一层集体触发对象targets

从而达到了广播的效果

postMessage本身可以实现广播的效果，但是MessengerJS为了兼容，限制了postMessage的能力，自行实现了广播

## 使用场景

MessengerJS来做iframe通信解决的最常见的问题是，在主页面为iframe留足高度

parent页面

```javascript
var messenger = new Messenger('parent');
    var iframe = document.getElementById('iframepage');
    messenger.addTarget(iframe.contentWindow, 'iframe');

    messenger.listen(function (msg) {
        var result = parseInt(msg, 10) + 20;

        if (result < mainWindowHeight) {
            result = mainWindowHeight;
        }
        $('#iframepage').height(result);
});
```

iframe页面

```javascript
// iframe跨域传数据
var messenger = new Messenger('iframe');
messenger.addTarget(window.parent, 'parent'); 

// 跨域传main 高度
var height = $('.main').height();
messenger.targets['parent'].send(height);

messenger.listen(function (msg) {

});
```

## 总结

postMessage是一个用于安全的使用跨源通信的方法，帮助web开发回归正轨

MessengerJS实现效果很好，即便做频繁的交互，也不会有明显的卡顿，不过时代在进步，以后可能会很少用到这样的兼容了

官方博文看这里：[MessengerJS](https://github.com/biqing/MessengerJS)