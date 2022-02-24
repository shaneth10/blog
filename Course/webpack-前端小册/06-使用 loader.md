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

不同的模块类型类似于配置了不同的 loader，webpack 会有针对性地进行处理，现阶段实现了以下 5 种模块类型。

- `javascript/auto`：即 webpack 3 默认的类型，支持现有的各种 JS 代码模块类型 —— CommonJS、AMD、ESM
- `javascript/esm`：ECMAScript modules，其他模块系统，例如 CommonJS 或者 AMD 等不支持，是 .mjs 文件的默认类型
- `javascript/dynamic`：CommonJS 和 AMD，排除 ESM
- `javascript/json`：JSON 格式数据，require 或者 import 都可以引入，是 .json 文件的默认类型
- `webassembly/experimental`：WebAssembly modules，当前还处于试验阶段，是 .wasm 文件的默认类型

如果不希望使用默认的类型的话，在确定好匹配规则条件时，我们可以使用 type 字段来指定模块类型，例如把所有的 JS 代码文件都设置为强制使用 ESM 类型：

```
{
  test: /\.js/,
  include: [
    path.resolve(__dirname, 'src'),
  ],
  type: 'javascript/esm', // 这里指定模块类型
},
```

上述做法是可以帮助你规范整个项目的模块系统，但是如果遗留太多不同类型的模块代码时，建议还是直接使用默认的 `javascript/auto`。

## 使用 loader 配置

在当前版本的 webpack 中，`module.rules` 的匹配规则最重要的还是用于配置 loader，我们可以使用 `use` 字段：

```
rules: [
  {
    test: /\.less/,
    use: [
      'style-loader', // 直接使用字符串表示 loader
      {
        loader: 'css-loader',
        options: {
          importLoaders: 1
        },
      }, // 用对象表示 loader，可以传递 loader 配置等
      {
        loader: 'less-loader',
        options: {
          noIeCompat: true  
        }, // 传递 loader 配置
      },
    ],
  },
],
```

## loader 应用顺序

前面提到，一个匹配规则中可以配置使用多个 loader，即一个模块文件可以经过多个 loader 的转换处理，执行顺序是从最后配置的 loader 开始，一步步往前。例如，对于上面的 less 规则配置，一个 style.less 文件会途径 less-loader、css-loader、style-loader 处理，成为一个可以打包的模块。

loader 的应用顺序在配置多个 loader 一起工作时很重要，例如在处理 CSS 的配置上，除了 style-loader 和 css-loader，你可能还要配置 less-loader 然后再加个 postcss 的 autoprefixer 等。

上述从后到前的顺序是在同一个 rule 中进行的，那如果多个 rule 匹配了同一个模块文件，loader 的应用顺序又是怎样的呢？看一份这样的配置：

```
rules: [
  {
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "eslint-loader",
  },
  {
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "babel-loader",
  },
],
```

这样无法法保证 eslint-loader 在 babel-loader 应用前执行。webpack 在 `rules` 中提供了一个 `enforce` 的字段来配置当前 rule 的 loader 类型，没配置的话是普通类型，我们可以配置 `pre` 或 `post`，分别对应前置类型或后置类型的 loader。

> eslint-loader 要检查的是人工编写的代码，如果在 babel-loader 之后使用，那么检查的是 Babel 转换后的代码，所以必须在 babel-loader 处理之前使用。

还有一种行内 loader，即我们在应用代码中引用依赖时直接声明使用的 loader，如 `const json = require('json-loader!./file.json')` 这种。不建议在应用开发中使用这种 loader，后续我们还会再提到。

顾名思义，所有的 loader 按照**前置 -> 行内 -> 普通 -> 后置**的顺序执行。所以当我们要确保 eslint-loader 在 babel-loader 之前执行时，可以如下添加 enforce 配置：

```
rules: [
  {
    enforce: 'pre', // 指定为前置类型
    test: /\.js$/,
    exclude: /node_modules/,
    loader: "eslint-loader",
  },
]
```

## 使用`noParse`

在 webpack 中，我们需要使用的 loader 是在 `module.rules` 下配置的，webpack 配置中的 module 用于控制如何处理项目中不同类型的模块。

除了 `module.rules` 字段用于配置 loader 之外，还有一个 `module.noParse` 字段，可以用于配置哪些模块文件的内容不需要进行解析。对于一些不需要解析依赖（即无依赖） 的第三方大型类库等，可以通过这个字段来配置，以提高整体的构建速度。

使用 `noParse` 进行忽略的模块文件中不能使用 `import、require、define` 等导入机制。

```
module.exports = {
  // ...
  module: {
    noParse: /jquery|lodash/, // 正则表达式

    // 或者使用 function
    noParse(content) {
      return /jquery|lodash/.test(content)
    },
  }
}
```