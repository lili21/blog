---
title: webpack系列 - hash, chunkhash, contenthash
date: 2018-07-14 20:26:43
tags: webpack
---

大家在使用webpack的时候，肯定有用到类似这样的配置

```js
// webpack.config.js
module.exports = {
  ...
  output: {
    filename: '[name].[chunkhash].js'
  },
  ...
}
```

webpack在打包的时候，会将`[name]`, `[chunkhash]`这样的占位符替换成对应的值，最终生成的文件名会是`main.cd54a69986890c0f0ba4.js`这样的形式.

webpack在`4.3.0`版本里添加了对`[contenthash]`占位符的支持

```js
// webpack.config.js
module.exports = {
  ...
  output: {
    filename: '[name].[contenthash].js'
  },
  ...
}
```

从官方的文档与介绍性文章来看，官方是推荐使用`[contenthash]`的。最近在使用的过程中踩了一个大坑，所以下定决心研究一下这些hash值到底是怎么生成的。


## 背景知识

webpack默认使用`md4`算法生成hash值，（文档上写的是`md5`，应该是还没有来得及更新）

[webpack默认配置源码](https://github.com/webpack/webpack/blob/18d33c630f799e99e5e43fc0b6931713dc976529/lib/WebpackOptionsDefaulter.js#L164)

```js
...
this.set("output.hashFunction", "md4");
this.set("output.hashDigest", "hex");
this.set("output.hashDigestLength", 20);
...
```

具体一点，webpack生成各种hash值的步骤如下所示

```js
const hash = require('crypto').createHash('md4')

hash.update('content 1')
hash.update('content 2')
hash.update('content 3')

const result = hash.digest('hex')
```

我们用下面这个简单的项目做示例

文件结构

```
- src
  - index.js
- webpack.config.js
```

`src/index.js`

```js
console.log('Hello, Webpack')
```

`webpack.config.js`

```js
module.exports = {
  mode: 'none',
  output: {
    filename: '[name].[contenthash].js'
  },
  optimization: {
    runtimeChunk: true
  }
}
```

## hash
> The hash of the compilation

`[hash]`占位符大家肯定很少用到，因为`[hash]`是以*compilation*为粒度的。也就是说每次打包都会生成一个hash值，用`[hash]`占位符的话，所有文件的hash都会是一样的

![](/images/webpack-hash.jpg)

[webpack源码](https://github.com/webpack/webpack/blob/18d33c630f799e99e5e43fc0b6931713dc976529/lib/Compilation.js#L2188)

简化版

```js

```