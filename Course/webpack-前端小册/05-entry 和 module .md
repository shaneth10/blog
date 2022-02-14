# entry 和 module

## 入口和模块

从这章节开始，我们会从构建的起点，即 entry 开始，逐步深入 webpack 配置的细节，从更多层面了解 webpack 的配置以及应用。

## entry

先复习一下单一入口的配置：
```
module.exports = {
  entry: './src/index.js' 
}

// 上述配置等同于
module.exports = {
  entry: {
    main: './src/index.js' // 你可以使用其他名称来替代 main，例如 index、app 等
  }
}
```

如果我们有多个页面，需要有多个 js 作为构建入口，可以是这样配置：
```
// 多个入口生成不同文件
module.exports = {
  entry: {
    // 按需取名，通常是业务名称
    foo: './src/foo.js',
    bar: './src/bar.js',
  },
}
```

还有一种场景比较少用到，即是多个文件作为一个入口来配置，webpack 会解析多个文件的依赖然后打包到一起：
```
// 使用数组来对多个文件进行打包
module.exports = {
  entry: {
    main: [
      './src/foo.js',
      './src/bar.js'
    ]
  }
}
```

上述的多种配置方式相对都是固定的，在业务页面稳定或者纯粹的单页应用上使用很方便，但是面对大型项目或者多页面项目的构建，我们需要更加灵活的方式，下边介绍一下如何利用 Node 脚本来给 webpack 配置动态的入口。

## 动态 entry

前边我们已经提过，webpack 配置文件本质上是一个 Node 脚本，所以我们在里边是可以用 JS 代码来编写逻辑的，所以我们可以使用 Node 来实现根据目录文件夹来创建 webpack 构建入口。

我们选择在项目的「src/」目录下创建一个新的文件夹名为「pages」，然后在「src/pages」下创建新的文件夹来作为入口存放的路径，例如「src/pages/foo/index.js」为一个新的页面入口，然后我们在配置里边可以这样来创建入口：

```
const path = require('path');
const fs = require('fs');

// src/pages 目录为页面入口的根目录
const pagesRoot = path.resolve(__dirname, './src/pages');
// fs 读取 pages 下的所有文件夹来作为入口，使用 entries 对象记录下来
const entries = fs.readdirSync(pagesRoot).reduce((entries, page) => {
  // 文件夹名称作为入口名称，值为对应的路径，可以省略 `index.js`，webpack 默认会寻找目录下的 index.js 文件
  entries[page] = path.resolve(pagesRoot, page);
  return entries;
}, {});

module.exports = {
  // 将 entries 对象作为入口配置
  entry: entries,

  // ...
};
```
上述做法可以支持你在「src/pages」下添加多个页面入口，而无需每次都修改 webpack 的配置文件，方便多页面或者大型项目使用。

## module

webpack 的初衷是让 js 支持模块化管理，并且将前端中的各种资源都纳入到对应的模块管理中来，所以在 webpack 的设计中，最重要的部分就是管理模块和模块之间的关系。

在 webpack 支持的前端代码模块化中，我们可以使用类似 `import * as m from './index.js'` 来引用代码模块 index.js。

引用第三方类库则是像这样：`import React from 'react'`。webpack 构建的时候，会解析依赖后，然后再去加载依赖的模块文件，而前边我们详细讲述的 entry，所谓 webpack 构建的起点，本质上也是一个 module，而我们在设置好 webpack 后，开发的过程亦是在写一个个的业务 module。

## 路径解析

当我们写一个 import 语句来引用一个模块时，webpack 是如何获取到对应模块的文件路径的呢？这其中有十分复杂的实现逻辑和相对繁琐的配置选择。

webpack 中有一个很关键的模块 **enhanced-resolve** 就是处理依赖模块路径的解析的，这个模块可以说是 Node.js 那一套模块路径解析的增强版本，有很多可以自定义的解析配置。

我们简单整理一下基本的模块解析规则，以便更好地理解后续 webpack 的一些配置会产生的影响。

- 解析相对路径
  1. 查找相对当前模块的路径下是否有对应文件或文件夹
  2. 是文件则直接加载
  3. 是文件夹则继续查找文件夹下的 package.json 文件
  4. 有 package.json 文件则按照文件中 `main` 字段的文件名来查找文件
  5. 无 package.json 或者无 `main` 字段则查找 `index.js` 文件
- 解析模块名

  查找当前文件目录下，父级目录及以上目录下的 node_modules 文件夹，看是否有对应名称的模块

- 解析绝对路径（不建议使用）

  直接查找对应路径的文件


在 webpack 配置中，和模块路径解析相关的配置都在 resolve 字段下：
```
module.exports = {
  resolve: {
    // ...
  }
}
```
## resolve 配置

- **resolve.alias**

假设我们有个 `utils` 模块极其常用，经常编写相对路径很麻烦，希望可以直接 `import 'utils'` 来引用，那么我们可以配置某个模块的别名，如：

```
alias: {
  utils: path.resolve(__dirname, 'src/utils') // 这里使用 path.resolve 和 __dirname 来获取绝对路径
}
```

上述的配置是模糊匹配，意味着只要模块路径中携带了 utils 就可以被替换掉，如：

```
import 'utils/query.js' // 等同于 import '[项目绝对路径]/src/utils/query.js'
```

如果需要进行精确匹配可以使用：
```
alias: {
  utils$: path.resolve(__dirname, 'src/utils') // 只会匹配 import 'utils'
}
```

如果需要进行精确匹配可以使用：
```
alias: {
  utils$: path.resolve(__dirname, 'src/utils') // 只会匹配 import 'utils'
}
```

- **resolve.extensions**

我们在引用模块时，其实可以直接这样：

```
import * as common from './src/utils/common'
```

webpack 会自行补全文件后缀，而这个补全的行为，也是可以配置的。

```
extensions: ['.wasm', '.mjs', '.js', '.json', '.jsx'],
// 这里的顺序代表匹配后缀的优先级，例如对于 index.js 和 index.jsx，会优先选择 index.js
```

webpack 会尝试给你依赖的路径添加上 `extensions` 字段所配置的后缀，然后进行依赖路径查找，所以可以命中 src/utils/common.js 文件。

但如果你是引用 src/styles 目录下的 common.css 文件时，如 `import './src/styles/common'`，webpack 构建时则会报无法解析模块的错误。

你可以在引用时添加后缀，`import './src/styles/common.css'` 来解决，或者在 `extensions` 添加一个 `.css` 的配置：

```
extensions: ['.wasm', '.mjs', '.js', '.json', '.jsx', '.css'],
```
