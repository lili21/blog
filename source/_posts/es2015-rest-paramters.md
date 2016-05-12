---
title: es2015-rest-paramters
date: 2016-05-01 20:32:19
tags:
---

### Rest parameters
在写API的时候一个常见需求是可变函数（接受任意参数的函数）。`String.prototype.concat`就是一个可变函数，它可以接受任意个字符串参数。在ES6之前，是通过arguments来实现的。现在，rest parameters提供了一种更好的方法。
为了达到演示效果，来看一个简单的可变函数`containsAll`，该函数判断一个字符串是否包含多个子字符串。例如，`containsAll('banana', 'b', 'nan')`返回`true`，而`containsAll('banana', 'c', 'nan')`返回`false`。

```
function containsAll(haystack) {
  for (var i = 1; i < arguments.length; i++) {
    var needle = arguments[i];
    if (haystack.indexOf(needle) === -1) {
      return false;
    }
  }
  return true;
}
```

<!-- more -->

这种实现确实能够实现我们的功能，但他的可读性并不是很好。函数的只有一个参数`haystack`，所以你没法一眼看出这个函数接受多个参数。同时，我们在`arguments`上的遍历也是从1而不是0开始，如果我们需要添加一个参数，那么我们同时也需要更改循环代码。rest parameters解决了这两方面的问题。以下是ES6的实现：

```
function containsAll(haystack, ...needles) {
  for (var needle of needles) {
    if (haystack.indexOf(needle) === -1) {
      return false;
    }
  }
  return true;
}
```

当我们调用`containsAll('banana', 'b', 'nan')`时，`haystack`就是第一个被传入的参数，即`banana`。`needles`前面的省略号即表明该参数为rest参数，所有剩下的参数会放入一个数组内并赋给`needles`。即`needles`被设为`['b', 'nan']`。

注意，只有最后一个参数可以被标记为rest参数。当函数调用时，rest参数前的参数都正常赋值，多余的参数组成数组赋给rest参数。如果没有多余的参数，那么rest参数就是一个空的数组，而不是`undefined`。

[原文](https://hacks.mozilla.org/2015/05/es6-in-depth-rest-parameters-and-defaults/)

