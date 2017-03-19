---
title: 利用decorator解决angular依赖注入问题
date: 2017-01-19 11:18:40
tags: decorator angular
---

依赖注入一直是`angular`为人称道的特性之一，在这里先简单介绍一下依赖注入的实现原理。

```
// controller.js
function ctrl ($scope, ServiceA) {
  ...
}
```

### 依赖注入处理“三部曲”

1. 通过调用`ctrl.toString()`讲函数转为字符串
2. 通过正则获取函数的参数名
3. 根据参数名获取相应的依赖

像我们这样严肃的前端工程师，线上的代码肯定是有代码压缩过的。所有最后我们的代码可能变成

```
// controller.js
function a (b, c) {
  ...
}
```

因为找不到`b`, `c`，`angular`就会报错。解决方式也很简单，

```
// 数组方式
['$scope', 'ServiceA', function ($scope, ServiceA) {
  ...
}]

// 添加$inject属性方式
function ctrl ($scope, ServiceA) {
  ...
}

ctrl.$inject = ['$scope', 'ServiceA']
```

以上两种方式都可以解决代码压缩导致依赖注入不可用的问题。

但在实际的项目中，每个地方都需要这么写有点boring。所以我们一般都会引入相应的工具帮助我们自动化这个过程。像`webpack`里`ng-annotate-loader`，`gulp`里的`gulp-ng-annotate`。FYI，这些工具都是依赖[ng-annotate](https://github.com/olov/ng-annotate)。

工具的问题在于，依赖的更新会导致工具的不可用。比如最近我把`webpack`更新到了`2.0`，导致`ng-annotate-loader`了不可用。另一个问题是，很多时候我们会使用`class`，

```
// controller.js
class Ctrl {
  constructor (ServiceA, ServiceB, ServiceC, ...) {
    this.sa = ServiceA
    this.sb = ServiceB
    this.sc = ServiceC
    ...
  }
}
```

我们需要在`constructor`里将依赖绑在`this`上，当依赖变多的时候，每个地方都要这么写同样也很boring。这就是我们今天需要解决的问题。

### Decorators

`decorator`是ES7中的特性，目前该提案处于`stage-2`。利用`babel`，我们今天就可以用了。废话不多说，直接上代码


```
@Inject('ServiceA', 'ServiceB', 'ServiceC')
class Ctrl {
  fetchData () {
    this.ServiceA.fetch()
  }
}

function Inject (...args) {
  return function (target, key, descriptor) {
    const ctrl = function (..._args) {
      args.forEach((v, i) => {
        this[v] = _args[i]
      })
      return target.apply(this, _args)
    }
    ctrl.prototype = target.prototype
    ctrl.$inject = args
    return ctrl
  }
}
```

利用`decorator`，我们完美的解决这些问题。