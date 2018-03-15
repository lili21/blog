---
title: ng-repeat与ui-sortable一起使用的问题与解决方案
date: 2015-08-12 19:43:23
tags:
---

[demo](http://jsbin.com/fizubi/edit?js,console,output)
### 问题
通过拖拽把0拖至2的位置，再把2拖至3的位置。理应是`[1, 0, 3, 2, 4]`的列表却变成了`[1, 0, 4, 3, 2]`。控制台中输出的是当前的`lists`，但实际的UI与我们的`model`没有对应起来。
### 原因
初始化
```html
<ul sortable="" class="ui-sortable">
  <!-- ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">0</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">1</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">2</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">3</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">4</span>
  </li>
  <!-- end ngRepeat: l in lists -->
</ul>
```
<!-- more -->

第一次拖拽

```html
<ul sortable="" class="ui-sortable">
  <!-- ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">1</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">2</span>
  </li>
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">0</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">3</span>
  </li>
  <!-- end ngRepeat: l in lists -->
  <li ng-repeat="l in lists" class="ng-scope">
    <span class="ng-binding">4</span>
  </li>
  <!-- end ngRepeat: l in lists -->
</ul>
```
`ng-repeat`生成每一个`element`都会带有一个注释，这个注释相当于在页面上的分隔符，便于指令更好处理`lists`变化后相应的ui变化。而在移动后我们可以看到注释乱了，导致指令的处理发生了错乱。
### 解决方案
1. 保存移动前的元素节点（`$('ul').contents()`），在sortable的`stop`事件中取消掉sortable，修改`lists`，通过ng-repeat来实现排序。即只是通过sortable来获取移动的index，然后修改数组，通过angular来实现排序。[ui-sortable](https://github.com/angular-ui/ui-sortable)即是这样实现的。

2. `ng-repeat= "l in lists track by Math.random()"`
这里简单解释一点`ng-repeat`的内部实现，`ng-repeat`在初始化的时候会遍历`lists`并保存数组每一个元素对应的`id`，`scope`，`element`，同时会保存一个map用来判断数组中的元素是否是新元素，如果提供了`track by`，该map的key值即为`track by`对应的值，如果没有提供，key值由angular生成。key值对应的value在初始化的过程中设为`true`。
在`lists`发生变化的时候，`ng-repeat`指令会在遍历`lists`的过程中判断每一个元素是新元素还是旧元素，如果是旧元素，那么直接利用保存好的`scope`与`element`即可，如果是新元素，就重新生成。
而在拖拽的过程中，会导致内部保存的元素与页面实际元素不符，导致错乱。
原因找到，就好解决了。只要每次都生成新的元素就可以了。而`ng-repeat`指令是通过map来判断是否是新值的，只要每一次`track by`的值是不同的，问题就能够解决了。


