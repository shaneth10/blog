# 使用 loader

webpack 的 loader 用于处理不同的文件类型,loader 基本上都是第三方类库，使用时需要安装，有一些 loader 还需要安装额外的类库，例如 less-loader 需要 less，babel-loader 需要 babel 等。

## loader 匹配规则

由于 loader 处理的是我们代码模块的内容转换，所以 loader 的配置是放在 `module` 字段下的，当我们需要配置 loader 时，都是在 `module.rules` 中添加新的配置项，在该字段中，每一项被视为一条匹配使用 loader 的规则。如下代码模块所示：

```
module.exports = {
  // ...
  module: {
    rules: [ 
      {
        test: /\.jsx?/,
        include: [
          path.resolve(__dirname, 'src'), // 指定哪些路径下的文件需要经过 loader 处理
        ],
        use: {
          loader: 'babel-loader', // 指定使用的 loader
          options: {
            presets: ['@babel/preset-env'],
          },
        },
      }, // 一个 object 即一条规则
      // ...
    ],
  },
}
```

loader 的匹配规则中有两个最关键的因素：一个是匹配条件，一个是匹配规则后的应用。

匹配条件通常都使用请求资源文件的绝对路径来进行匹配，在官方文档中称为 `resource`，除此之外还有比较少用到的 `issuer`，则是声明依赖请求的源文件的绝对路径。

举个例子：在 /path/to/app.js 中声明引入 `import './src/style.scss'`，`resource` 是「/path/to/src/style.scss」，`issuer` 是「/path/to/app.js」，规则条件会对这两个值来尝试匹配。

上述代码中的 `test` 和 `include` 都用于匹配 `resource` 路径，是 `resource.test` 和 `resource.include` 的简写，你也可以这么配置：

```
module.exports = {
  // ...
  rules: [ 
      {
        resource: { // resource 的匹配条件
          test: /\.jsx?/, 
          include: [ 
            path.resolve(__dirname, 'src'),
          ],
        },
        // 如果要使用 issuer 匹配，便是 issuer: { test: ... }
        // ...
      },
      // ...
    ], 
}
```

> issuer 规则匹配的场景比较少见，你可以用它来尝试约束某些类型的文件中只能引用某些类型的文件。


## 规则条件配置

webpack 的规则提供了多种配置形式：
- `{ test: ... }` 匹配特定条件
- `{ include: ... }` 匹配特定路径
- `{ exclude: ... }` 排除特定路径
- `{ and: [...] }` 必须匹配数组中所有条件
- `{ or: [...] }` 匹配数组中任意一个条件
- `{ not: [...] }` 排除匹配数组中所有条件

上述的所谓条件的值可以是：

- 字符串：必须以提供的字符串开始，所以是字符串的话，这里我们需要提供绝对路径
- 正则表达式：调用正则的 test 方法来判断匹配
- 函数：(path) => boolean，返回 true 表示匹配
- 数组：至少包含一个条件的数组
- 对象：匹配所有属性值的条件

以下面的例子为例：
```
rules: [
  {
    test: /\.jsx?/, // 正则
    include: [
      path.resolve(__dirname, 'src'), // 字符串，注意是绝对路径
    ], // 数组
    // ...
  },
  {
    resource: {
      test: {
        js: /\.js/,
        jsx: /\.jsx/,
      }, // 对象，不建议使用
      not: [
        (value) => { /* ... */ return true; }, // 函数，通常需要高度自定义时才会使用
      ],
    }
  },
],
```

`test/include/exclude` 是 `resource.(test/include/exclude)` 的简写，`and/or/not` 这些则需要放到 `resource` 中进行配置。

## module type