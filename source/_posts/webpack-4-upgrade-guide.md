---
title: webpack-4-upgrade-guide
date: 2018-03-15 15:53:33
tags: webpack
---

**2018-04-09更新**
[mini-css-extract-plugin@^0.4.0](https://github.com/webpack-contrib/mini-css-extract-plugin)支持了**contenthash**（需配合webpack@^4.3.0使用）。

**2018-03-21更新**

[etwp决定不再支持webpack@4](https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/749)


webpack 4发布也有一段时间了，虽然官方的文档，升级指南都没有完成，部分插件适配webpack 4的开发也还在进行中，但对于一个资深前端配置工程师，大刀早已饥渴难耐，所以就出一份非官方升(cai)级(keng)指南吧。

[我没时间看你废话，给我一份配置](https://github.com/lili21/webpack4-demo)

webpack 4做了很多改进，我就挑一些我比较关心的点来展开

### 0CJS

随着像[parcel](https://github.com/parcel-bundler/parcel)这样零配置，开箱即用打包工具的流行，被人吐槽难用，难懂，难配置的webpack显然感受到了压力，在4.0的版本中，给部分配置加上了默认值，并引入了`mode`概念，为开发，及生产环境提供两种不同的默认配置，极大的简化了webpack的配置。在4.0版本中，你甚至不需要配置，即可使用(虽然没什么用)。

**默认值**

```javascript
module.exports = {
  entry: 'src/',
  outpu: {
    filename: '[name].js',
    path: 'dist/'
  }
}
```

webpack 4中引入的`mode`是必需设置的（如果没有设置，会报警），你可以通过`webpack --mode ...`来设置，也可以通过`webpack.config.js`来设置。

```javascript
module.exports = {
  mode: 'development' // development || production
}
```

`mode`也是为了简化配置项，不同的`mode`会为我们设置不同的默认配置，就不需要我们一个个去分别配置了。

开发模式更关注开发体验：

- 更利于浏览器调试
- 更快的增量编译速度
- 更详细的报错消息

生成模式更关注用户体验：

- 性能（文件大小，运行性能，打包速度）

详细的说明可以参考[这篇文章](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)

### Loader与Plugin

loader的其实没有什么变化，我用到的loader升级到最新版都可以直接用，有些甚至不用升级。变化比较大的是plugin。
webpack 4使用了新的插件体系，导致之前的插件基本都是需要改的，那么这些插件目前对webpack 4的支持如何呢？

- extract-text-webpack-plugin@4.0.0-beta.0

    [issue list](https://github.com/webpack-contrib/extract-text-webpack-plugin/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+4.0.0-beta.0) 可以看到，问题还是比较多的。
    我自己遇到的问题是，在配合`splitChunks`(后面会讲到)时，会生成0kb的css文件，[issue](https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/738)。
    对于这个问题，我已经提了[PR](https://github.com/webpack-contrib/extract-text-webpack-plugin/pull/746)了。

    官方出了一个[mini-css-extract-plugin](github.com/webpack-contrib/mini-css-extract-plugin)作为替代品

- html-webpack-plugin@3.0.0+

    这个插件目前是支持webpack 4的，只是他自己的插件不一定支持，这个在升级的时候需要注意

- uglifyjs-webpack-plugin

    这个插件目前也是支持webpack 4的。其实在`production`模式下，代码是默认会压缩的。当然，如果你有更细致的自定义需求，就可以用到这个插件。

### RIP CommonsChunkPlugin

上面讲到的都是一些第三方的插件改动，而改动最大，影响也最大的就是webpack 4使用`optimization.splitChunks`替代了`CommonsChunkPlugin`，并且直接移除了`CommonsChunkPlugin`，所以这部分迁移起来比较麻烦。

对于各位配置工程师来说，`CommonsChunkPlugin`应该是很熟悉了，我们主要用他来抽取代码中的共用部分，`webpack runtime`之类的代码，结合`chunkhash`，实现最好的缓存策略。而这一部分，也是webpack支持的比较差的，这个几乎三岁的[issue](https://github.com/webpack/webpack/issues/1315)，至今还没有解决。对这个问题比较感兴趣的，可以拜读一下这篇[文章](https://medium.com/webpack/predictable-long-term-caching-with-webpack-d3eee1d3fa31)。我这里就不展开了，直接贴一份`CommonsChunkPlugin`时代解决这个问题的配置

```javascript
module.exports = {
  ...
  plugins: [
    ...
    new webpack.HashedModuleIdsPlugin(),

    new webpack.NamedChunksPlugin(chunk => {
      if (chunk.name) {
        return chunk.name
      } else {
        return 'faceless-chunk' // a chunk has no name
      }
    }),

    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          /node_modules/.test(module.resource)
        )
      }
    }),

    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime',
      minChunks: Infinity
    }),

    // extracts shared chunks from code splitted chunks
    // https://github.com/webpack/webpack/issues/4392
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'async-vendor',
      children: true,
      minChunks: 3
    })

  ]
}
```

升级到webpack 4后的配置

```javascript
module.exports = {
  ...
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendors: {
          test: chunk => (
            chunk.resource &&
            /\.js$/.test(chunk.resource) &&
            /node_modules/.test(chunk.resource)
          ),
          chunks: 'initial',
          name: 'vendors',
        },
        'async-vendors': {
          test: /[\\/]node_modules[\\/]/,
          minChunks: 2,
          chunks: 'async',
          name: 'async-vendors'
        }
      }
    },
    runtimeChunk: { name: 'runtime' }
  },
  plugins: [
    new webpack.HashedModuleIdsPlugin(),
    new webpack.NamedChunksPlugin(chunk => chunk.name || 'faceless-chunk'), // a chunk has no name!!!
    ...
  ]
}
```

这份[gist](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)算是一份关于`optimization.splitChunks`，`optimization.runtimeChunk`的文档了，感兴趣的同学可以看一下


迁移时间其实花的挺长的，一方面给webpack提issue，一方面是给wepback插件提issue跟PR。
wepback对于issue的反馈还是很快的，上次提的issue，几个小时就给我直接修复了。但是插件就比较慢了，可能issue提了几天才有反馈。
总之希望这篇文章能够帮到正在升级和打算升级的同学们。

最后奉上一份完整的[webpack 4配置](https://github.com/lili21/webpack4-demo)，有问题可以直接在这里repo里提issue。
