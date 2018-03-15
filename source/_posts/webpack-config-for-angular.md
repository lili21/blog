---
title: 那些年坑过我的webpack配置，angular+jQuery项目
date: 2016-06-24 14:17:45
tags: webpack
---

最近在死磕webpack，之前我们的项目有用grunt的，也有用gulp+webpack的。因为想简化与统一下构建工具，所以就计划全部用webpack。记录下踩过的坑。

## angular相关的坑

我们用的是angular@1.3.4，这个版本的angular是无法直接用`CommonJS`或`ES2015 module`语法的。如果想要这样用

```javascript
import angular from 'angular'

// or
const angular = require('angular')
```

<!-- more -->

需要用exports-loader处理一下

```javascript
// webpack config file
  ...
  module: {
    loaders: {
      {
        test: /[\/]angular\.js$/,
        loader: 'exports?angular'
      }
    }
  }
  ...
```

## jQuery + angular 的坑
angular内置了一个简化版的jQuery，想要让angular使用jQuery的话，必须要在angular前引入jQuery同时暴露`$`全局变量。

```javascript
import jquery form 'jquery' // 先引入jquery
import angular from 'angular'
```

使用用expose-loader暴露`$`全局变量

```javascript
// webpack config file
  ...
  module: {
    loaders: {
      {
        test: require.resolve('jquery'),
        loader: 'expose?$!expose?jQuery'
      }
    }
  }
  ...
```

你也可以直接在代码中声明全局变量，这样就不需要expose-loader了

```javascript
window.$ = window.jQuery = require('jquery')
const angular = require('angular')
```

## webpack自己的坑
我们代码在发布上线的时候都会使用long-term-caching，我们会把经常变动的业务代码(app.js)与不怎么变动的第三方依赖代码(vendor.js)分开打包，保证每次发布上线只有业务代码需要重新加载。但即使我们只是改动了业务代码，vendor.js的hash值还是会变动，导致long-term-caching无效。具体的原因可以看这个[issue](https://github.com/webpack/webpack/issues/1315#issuecomment-234206979)

完整的配置可以看这个项目[webpack-angular](https://github.com/lili21/ml-template-angular)

