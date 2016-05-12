---
title: window.postMessage接口里的message参数
date: 2016-04-20 20:35:48
tags:
---

最近刚好在做一些iframe方面的东西，就使用了下postMessage这个API，结果遇到个奇怪的错误。

![](/images/post-message-error.png)

原因是message参数所使用的序列化的算法（结构化克隆算法）对于message的内容与结构有一定的限制。

<!-- more -->

1. `Error`与`Function`对象无法被该算法复制，如果message中包含这样的对象，将会抛一个`DATA_CLONE_ERR`异常
2. message中如果包含`DOM`节点，同样会抛一个`DATA_CLONE_ERR`异常
3. 某些对象的属性不会被复制
    1. `RegExp`对象的`lastIndex`属性不会被复制
    2. 属性描述符，`setters`以及`getters`同样不会被复制，所以如果一个对象通过属性描述符设置为只读的，在复制后将变为可读可写
    3. 对象的原型链也不会被复制

[source](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)
