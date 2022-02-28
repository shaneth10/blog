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

### DefinePlugin

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
