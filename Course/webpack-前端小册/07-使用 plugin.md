# 使用 plugin

本章节我们先来看一下 mode 对 plugin 配置的影响，其中涉及到一些 plugin 的应用。

## mode 和 plugin

mode 不同值会影响 webpack 构建配置，其中有一个就是会启用 DefinePlugin 来设置 `process.env.NODE_ENV` 的值，方便代码中判断构建环境。

除此之外，development 和 production 两个不同的 mode 之间还有其他 plugin 使用上的区别，这里详细介绍一下：

### development

development 下会启用 NamedChunksPlugin 和 NamedModulesPlugin，这两个 plugin 官方文档并没有详细的介绍，主要作用是在 Hot Module Replacement（热模块替换，后续简称 HMR）开启时，模块变化时的提示内容显示 chunk 或者 module 名称，而不是 ID。

### production

production 下会启动多个 plugins，分别是：

- FlagDependencyUsagePlugin 在构建时给使用的依赖添加标识，用于减少构建生成的代码量。
- FlagIncludedChunksPlugin 在构建时给 chunk 中所包含的所有 chunk 添加 id，用于减少不必要的 chunk。
- ModuleConcatenationPlugin 构建时添加作用域提升的处理，用于减少构建生成的代码量，详细参考：module-concatenation-plugin。
- NoEmitOnErrorsPlugin 编译时出错的代码不生成，避免构建出来的代码异常。
- OccurrenceOrderPlugin 按使用的次数来对模块进行排序，可以进一步减少构建代码量。
- SideEffectsFlagPlugin 在构建时给带有 Side Effects 的代码模块添加标识，用于优化代码量时使用。
- TerserPlugin 压缩 JS 代码

production mode 下启用的大量 plugin 都是为了优化生成代码而使用的，和配置的 `optimization` 的内容息息相关。

## DefinePlugin

DefinePlugin 是 webpack 内置的插件，可以使用 `webpack.DefinePlugin` 直接获取，前边也提过，在不同的 mode 中，会使用 DefinePlugin 来设置运行时的 `process.env.NODE_ENV` 常量。DefinePlugin 用于创建一些在编译时可以配置值，在运行时可以使用的常量，我们来看下如何使用它。

```
module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      PRODUCTION: JSON.stringify(true), // const PRODUCTION = true
      VERSION: JSON.stringify('5fa3b9'), // const VERSION = '5fa3b9'
      BROWSER_SUPPORTS_HTML5: true, // const BROWSER_SUPPORTS_HTML5 = 'true'
      TWO: '1+1', // const TWO = 1 + 1,
      CONSTANTS: {
        APP_VERSION: JSON.stringify('1.1.2') // const CONSTANTS = { APP_VERSION: '1.1.2' }
      }
    }),
  ],
}
```

上面配置的注释已经简单说明了这些配置的效果，这里再简述一下整个配置规则。

- 如果配置的值是字符串，那么整个字符串会被当成代码片段来执行，其结果作为最终变量的值，如上面的 "1+1"，最后的结果是 2
- 如果配置的值不是字符串，也不是一个对象字面量，那么该值会被转为一个字符串，如 true，最后的结果是 'true'
- 如果配置的是一个对象字面量，那么该对象的所有 key 会以同样的方式去定义


## TerserPlugin

我们已经提过，webpack mode 为 production 时会启用 TerserPlugin 来压缩 JS 代码，我们看一下如何使用的：

```
module.exports = {
  // ...
  // TerserPlugin 的使用比较特别，需要配置在 optimization 字段中，属于构建代码优化的一部分
  optimization: {
    minimize: true, // 启用代码压缩
    minimizer: [new TerserPlugin({
      test: /\.js(\?.*)?$/i, // 只处理 .js 文件
      cache: true, // 启用缓存，可以加速压缩处理
    })], // 配置代码压缩工具
  },
}
```

> 在以前的版本 webpack 是使用 UglifyWebpackPlugin 来压缩 JS 代码，后边更换为 TerserPlugin 了，可以更好地处理新的 JS 代码语法。

## IgnorePlugin

这个插件用于忽略某些特定的模块，让 webpack 不把这些指定的模块打包进去。例如我们使用 `moment.js`，直接引用后，里边有大量的 i18n 的代码，导致最后打包出来的文件比较大，而实际场景并不需要这些 i18n 的代码，这时我们可以使用 IgnorePlugin 来忽略掉这些代码文件，配置如下：

```
module.exports = {
  // ...
  plugins: [
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
  ]
}
```

IgnorePlugin 配置的参数有两个，第一个是匹配引入模块路径的正则表达式，第二个是匹配模块的对应上下文，即所在目录名。

## webpack-bundle-analyzer

这个 plugin 可以用于分析 webpack 构建打包的内容，用于查看各个模块的依赖关系和各个模块的代码内容多少，便于开发者做性能优化。

webpack-bundle-analyzer 是第三方的包，使用前需要安装，配置上很简单，仅仅引入 plugin 即可，在构建时可以在浏览器中查看分析结果：

```
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // ...
  plugins: [
    new BundleAnalyzerPlugin(),
  ],
}
```

使用这个可以配合 IgnorePlugin 来过滤掉部分大而无用的第三方模块。后续的代码加载优化部分内容会再使用到 webpack-bundle-analyzer。


## 小结

本章节讲解了 webpack 的 mode 分别为 development 和 production 时会自动开启的 plugin 是什么，有哪些作用，同时介绍了几个相对常见的 webpack plugin 的使用：

- DefinePlugin
- TerserPlugin
- IgnorePlugin
- webpack-bundle-analyzer