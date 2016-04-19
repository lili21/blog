---
title: 视窗单位 vs 百分比单位
date: 2015-07-20 21:40:47
tags:
---

[原文](http://bitsofco.de/2015/viewport-vs-percentage-units/)
### 视窗单位
视窗单位是相对值，他们的大小由浏览器可视窗口的大小决定。下表列出了相对于视窗的4个单位值。

名称 | 单位 | 描述
------ | ------ | ------
视窗宽 | vw | 视窗宽度的 1/100
视窗高 | vh | 视窗高度的 1/100
最小视窗 | vmin | 视窗高度，宽度中小值的 1/100
最大视窗 | vmax | 视窗高度，宽度中大值的 1/100

<!-- more -->

后两种单位几乎很难用到，所以这里只关注`vh`和`vw`。

视窗单位与百分比在功能上是存在部分重叠的，但它们有各自的有缺点。tl;dr
> 百分比在宽度上更实用，而`vh`在高度上更好用

### 满屏宽度 － % > vw
`vw`单位基于可视窗口的宽度，然后浏览器计算的可视窗口包含了浏览器的滚动条。
![image](http://bitsofco.de/img/blog/14/viewportvsbody.jpg)
这也就意味着，如果存在滚动条，那么视窗的宽度将会大于内容区域的宽度
> viewport > html

所以，如果你把一个元素的宽度设为100vw，这个元素将会超出html的范围。如下面这个例子，我给html设置了红色的边框，给section设置了背景色
![viewport>html](http://bitsofco.de/img/blog/14/viewportvsbody2.png)
所以如果要达到满屏宽度的效果，百分比效果要好于视窗单位

### 满屏高度 － vh > %
我们知道，百分比单位是依赖于父元素的尺寸的。如果我们要某一个元素一百分比单位设置满屏高度，那么父元素也得满屏高度。这同时意味着我们需要把元素的`position`设置为`fixed`，让html元素成为父元素，或者使用一些`hack`代码。
但如果用`vh`的话，就这么简单
```
.example {
  height: 100vh;
}
```
但也要注意的是，在部分浏览器上，`vh`的表现会有些[诡异](http://nicolas-hoizey.com/2015/02/viewport-height-is-taller-than-the-visible-part-of-the-document-in-some-mobile-browsers.html)

