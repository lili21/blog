---
title: 严格模式严格在哪里
date: 2015-10-19 20:45:36
tags:
---

[source](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)

## 避免偶然创建全局变量

在正常模式下，如果在赋值的时候输错了变量名，会在当前作用域下创建一个变量。在严格模式下会抛出错误。
```
'use strict';
mistypedVaraible = 17;
```
<!-- more -->
## 在正常模式下默默失败的赋值，在严格模式下会抛出异常

  例如，`NaN`是不可写的全局变量。在正常模式下，赋值给`NaN`是无效的，也不会有任何反馈。在严格模式下，将会抛出异常。赋值给不可写变量，只读变量，或者给不可扩展对象添加新的属性在严格模式下都会抛出异常。
  ```
  'use strict'
  //赋值给不可写变量
  var obj1 = {};
  Object.defineProperty(obj1, 'x', {value: 42, writable: false});
  obj1.x = 9; // 报错
  //赋值给只读变量
  var obj2 = {get x() {return 17;}};
  obj2.x = 5; // 报错
  //给不可扩展对象添加新属性
  var fixed = {};
  Object.preventExtensions(fixed);
  fixed.newProp = 'ohai'; //报错
  ```
## 删除不可删的属性会报错

  ```
  'use strict';
  delete Object.prorotype; //报错
  ```

## 对象属性名必须唯一（es6下可以不唯一）

  ```
  'use strict';
  var o = {p : 1, p: 2}; //报错
  ```

## 函数参数名必须唯一

  ```
  'use strict';
  function sum(a, a, c) { //报错
    'use strict';
    return a + b + c;
  }
  ```
## 八进制不可用

## with不可用

  ```
  'use strict';
  var x = 17;
  with (obj) //报错
  {
    x;
  }
  ```

## eval不会影响外部作用域

  ```
  var x = 17;
  var evalX = eval("'use strict'; var x = 42; x");
  console.assert(x === 17);
  console.assert(evalX === 42);
  ```
  需要注意的是，eval的调用方式也会影响到eval是否在严格模式下
  ```
  function strict1(str) {
    'use strict';
    return eval(str);
  }
  function strict2(f, str) {
    'use strict';
    return f(str);
  }
  function nonstrict(str) {
    return eval(str);
  }
  strict1("'Strict mode code!'");//严格模式
  strict1("'use strict'; 'Strict mode code!'");//严格模式
  strict2(eval, "'Non-strict code.'");//非严格模式
  strict2(eval, "'use strict'; 'Strict mode code!'");//严格模式
  nonstrict("'Non-strict code.'");//非严格模式
  nonstrict("'use strict'; 'Strict mode code!'");//严格模式
  ```

## 禁止删除原始类型变量

  ```
  'use strict';
  var x;
  delete x;//报错

  eval("var y; delete y;");//报错
  ```

## `eval`和`arguments`不可修改，以下操作都是语法错误

  ```
  'use strict';
  eval = 17;
  arguments++;
  ++eval;
  var obj = {set p(arguments) {} };
  var eval;
  try {} catch(arguments) {}
  function x(eval) {}
  function arguments() {}
  var y = function eval() {}
  var f = new Function('arguments', "'use strict'; return 17;");
  ```

## 严格模式下修改参数值不会影响`arguments`，修改`arguments`也不会影响参数

  ```
  function f(a) {
    'use strict';
    a = 42;
    return [a, arguments[0]];
  }
  var pair = f(17);
  console.assert(pair[0] === 42);
  console.assert(pair[1] === 17);
  ```

## `arguments.callee`不可用

  ```
  'use strict';
  var f = function() {return arguments.callee;};
  f(); //报错
  ```

## `this`可以是任意类型的值（在正模式下，`this`是对象）,而且如果没有指定`this`，`this`为`undefined`

  ```
  'use strict';
  function fun() {return this;}
  console.assert(fun() === undefined);
  console.assert(fun.call(2) === 2);
  console.assert(fun.apply(null) === null);
  console.assert(fun.call(undefined) === undefined);
  console.assert(fun.bind(true)() === true);
  ```

