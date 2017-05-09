---
title: html5的video元素学习手札
date: 2016-05-19 13:23:06
tags: html5
categories: work
---

为了监控移动端视频播放的情况，研究了一下 html5 `<video>` 标签的属性与事件触发，及其在各系统和各个浏览器的兼容情况

<!--more-->

## 属性与事件

理解清楚属性和事件，才能更好的使用 video ，达到预期的效果，更好的检测视频播放的状况来做出分析和调整，这里仅列举了解到且大致理解的，更多相关后续补充

### 属性

`<video>` 标签嵌入到HTML文档中

```html
<video src="" type="video/mp4" autoplay="autoplay" controls="" poster="" preload="none"></video>
```

初始化 `<video>` 标签时主要设置的属性

1. src：要嵌到页面的视频的URL。可选；你也可以使用video块内的 `<source>` 元素来指定需要嵌到页面的视频
2. autoplay：布尔属性；指定后，视频会马上自动开始播放，不会停下来等着数据载入结束
3. controls：加上这个属性，Gecko 会提供用户控制，允许用户控制视频的播放，包括音量，跨帧，暂停/恢复播放
4. poster：一个海报帧的URL，用于在用户播放或者跳帧之前展示。如果属性未指定，那么在第一帧可用之前什么都不会展示；之后第一帧就像海报帧一样展示
5. preload：该枚举属性旨在告诉浏览器作者认为达到最佳的用户体验的方式是什么。可能是下列值之一：
	* none：提示作者认为用户不需要查看该视频，服务器也想要最小化访问流量；换句话说就是提示浏览器该视频不需要缓存
	* metadata：提示尽管作者认为用户不需要查看该视频，不过抓取元数据（比如：长度）还是很合理的
	* auto：用户需要这个视频优先加载；换句话说就是提示：如果需要的话，可以下载整个视频，即使用户并不一定会用它
	* 空字符串：也就代指 auto 值
6. buffered：这个属性可以读取到哪段时间范围内的媒体被缓存了。该属性包含了一个 TimeRanges 对象
7. played：一个 TimeRanges 对象，指明了视频已经播放的所有范围
8. loop：布尔属性；指定后，会在视频结尾的地方，自动返回视频开始的地方
9. muted：布尔属性，指明了视频里的音频的默认设置。设置后，音频会初始化为静音。默认值是 false ,意味着视频播放的时候音频也会播放
10. height：视频展示区域的高度，单位是 CSS 像素
11. width：视频显示区域的宽度，单位是 CSS 像素
12. crossorigin：该枚举属性指明抓取相关图片是否必须用到CORS（跨域资源共享）。 支持CORS的资源 可在 `<canvas> `元素中被重用，而不会被污染。允许的值如下：
	* anonymous：跨域请求会被执行，但是不发送凭证。
	* use-credentials：跨域请求A cross-origin request会被执行，且凭证会被发送。

TimeRanges 对象表示事件段，比如，视频快进的时间段，有一个 length 属性，表示时间段的个数，有两个方法 start() 和 end() ，分别返回时间段开始的时间点和结束的时间点

事件交互中主要使用的属性

1. currentTime：播放进行到的时间点，单位为秒
2. duration：视频总时长，单位为秒

### 事件

还有更多事件api，这里只列举了试过的

1. playing：在媒体开始播放时触发（不论是初次播放、在暂停后恢复、或是在结束后重新开始）
2. ended：播放结束时触发
3. pause：播放暂停时触发
4. waiting：在一个待执行的操作（如回放）因等待另一个操作（如跳跃或下载）被延迟时触发
5. timeupdate：元素的 currentTime 属性表示的时间已经改变
6. seeking：	在跳跃操作开始时触发
7. seeked：在跳跃操作完成时触发
8. error：在发生错误时触发。元素的 error 属性会包含更多信息
9. loadeddata： 媒体的第一帧已经加载完毕

## 监控指标

1. 播放：start：1（首次播放）2（重播）
2. 播放：end：1
3. 播放暂停：pause：1
4. 播放中止：pause：1
5. 快进/快退：Jump：1（快进）2（快退）
6. 错误：fail： 1（取回过程）；2（当下载时发生错误）；3（当解码时发生错误）；4（不支持音频/视频）
7. 播放等待： wait：1
8. 播放时长：totaltime：秒（包含重播）

其中1，2，3，5，6，7都很好监控，对相应事件进行监听就可以了，这里主要讲下是怎么监控播放中止和播放时长的

## 播放中止

具体场景是移动端浏览器切换tab导致的隐藏和用户按home键退出浏览器

html5 提供了 Page Visibility API 来支持监听tab切换，与之对应新增了

* document.hidden 属性，它显示页面是否为用户当前观看的页面，值为 ture 或 false
* document.visibilityState 属性， visible 表示页面被展现， hidden 表示页面未被展现， prerender 表示页面在重新生成，用户不可见
* visibilitychange 事件，监听页面在 visible 与 hidden 之间的切换

visibilitychange事件的具体使用

```javascript
var hidden;
var visibilityChange;
if (typeof document.hidden !== 'undefined') {
    hidden = 'hidden';
    visibilityChange = 'visibilitychange';
} else if (typeof document.mozHidden !== 'undefined') {
    hidden = 'mozHidden';
    visibilityChange = 'mozvisibilitychange';
} else if (typeof document.msHidden !== 'undefined') {
    hidden = 'msHidden';
    visibilityChange = 'msvisibilitychange';
} else if (typeof document.webkitHidden !== 'undefined') {
    hidden = 'webkitHidden';
    visibilityChange = 'webkitvisibilitychange';
}

document.addEventListener(visibilityChange, function () {
    if (document[hidden]) {
        // do something...
    }
}, false);
```

关于 visibilitychange 事件的兼容性，测试了两部手机，华为mt7 和 iphone6 ，兼容情况如下

华为mt7

* qq浏览器：tab切换触发，home键退出触发
* uc浏览器：tab切换触发，home键退出触发（退出后进程继续在跑，其它浏览器进程被暂停）
* 手机百度：tab切换不触发，home键退出不触发

iphone6

* uc浏览器：tab切换触发，home键不触发
* 百度浏览器：同上
* 手机百度：同上
* Safari：tab切换触发，home键退出触发


由于兼容问题，且各系统的各个浏览器基本在tab切换触发，home键退出触发的情况下触发pause事件，所以播放中止的日志依旧打印pause，如果后面没有继续操作则把这个pause日志当做播放中止

页面刷新和浏览器tab被关闭的时候会触发 window.onunload ，也可以做为补充场景

## 播放时长

起初的思路是获取到开始播放到停止播放的事件差，记下时间点使用了 currentTime 属性，主要实现在两方面

1. playing 时记下时间点startT， pause 和 ended 和 seeked 时记下时间点endT，endT - startT 即播放时长
2. seeked 时记下时间点startT， seeking 时记下时间点endT，endT - startT 即播放时长

这个思路在 ios 下是看似没有问题的，但是 android 下确实不行，主要原因是 seeking 事件的监听没理解到位，seeking 事件触发点是用户目标跳跃到的位置，比如：视频播放在 0 秒点时，用户点击到了 60 秒点处，这是取到的 currentTime 就是 60 ，本来以为会是 0 ， ios 下看似没有问题是因为它的全屏播放模式下，进度条是要拖拽的，不能直接点击到某个点

于是，使用 timeupdate 来获取 seeking 触发前的时间点，就可以获取到相对准确的播放时长了

## error事件

监听 error 事件会返回 error.code 来标识错误类型：

* 1 = MEDIA_ERR_ABORTED - 取回过程被用户中止
* 2 = MEDIA_ERR_NETWORK - 当下载时发生错误
* 3 = MEDIA_ERR_DECODE - 当解码时发生错误
* 4 = MEDIA_ERR_SRC_NOT_SUPPORTED - 不支持音频/视频

官网的解释是：

* MEDIA_ERR_ABORTED (numeric value 1)
> The fetching process for the media resource was aborted by the user agent at the user's request.
> 在取回资源过程中，被用户的操作中止（您中止了视频播放）
* MEDIA_ERR_NETWORK (numeric value 2)
> A network error of some description caused the user agent to stop fetching the media resource, after the resource was established to be usable.
> 因为一些网络问题导致的用户无法取回资源，前提是资源被确定为可用（网络问题导致视频下载中断）
* MEDIA_ERR_DECODE (numeric value 3)
> An error of some description occurred while decoding the media resource, after the resource was established to be usable.
> 解码资源时产生的问题，前提是资源被确定为可用（『a corruption problem（翻译不过来）』 或者 所使用的视频功能的浏览器不支持）
* MEDIA_ERR_SRC_NOT_SUPPORTED (numeric value 4)
> The media resource indicated by the src attribute was not suitable.
> 由src属性所指定的媒体资源不适合（视频无法加载，或因为服务器或网络故障，或格式不支持）

总结一下就是，1、2、3是在视频可用情况下因为外在因素导致的播放失败，4是可能因为资源本身有问题导致的播放失败（存在资源可播放也打印 error.code 为4的情况）

## 遇到的一些状况

* 没有 `<source>` 元素且 src 属性为空时播放会触发 error 事件，状态码为4
> 解决：忽略 src 属性为空时的报错
* 播放结束会触发暂停
> 解决：声明状态变量，随着具体操作更新状态，播放状态下才会执行暂停操作，结束状态不执行
* 播放结束后重播会触发 seeking 和 seeked ，一般浏览器触发一次， android 下uc浏览器触发多次
> 解决：同上
* 一些浏览器监听不到 seeking 和 seeked
> 解决：在 timeupdate 里来分析猜测用户行为
* 一些浏览器存在多次连续触发 seeking + seeked 的情况
> 解决：时间戳 + 节流 等待最后一次
*  seeking 和 seeked 与 timeupdate 需要保证不会同时执行
> 解决：监听到 seeking 触发，就不再执行 timeupdate 模拟
> 走过的坑：我曾设想在播放时直接判断出是否支持 seeking ，方式是播放时设置 currentTime 为 0.01 ，然后检测 seeking 属性，后来发现浏览器在这样设置后的 seeking 属性值不一致
* 个别浏览器播放状态下不触发 seeking 和 seeked ，但是在重播的时候触发
> 解决：声明状态变量，随着具体操作更新状态，结束状态不监听 seeking 触发

提示一下，遇到浏览器表现不同的情况，千万不要尝试对各个浏览器适配 hack ，没法根本上解决问题，而我的做法是声明状态变量，随着具体操作更新状态，在正确的状态下才进行操作，通过播放状态来规范事件的触发时机

## 总结

欲先善其事必先利其器，遇到没搞过的技术，一定要先测试一遍 api ，不然太浪费时间，学习内容来源

[媒体相关事件](https://developer.mozilla.org/zh-CN/docs/Web/Guide/Events/Media_events)

希望能多提宝贵建议，帮助笔者继续优化