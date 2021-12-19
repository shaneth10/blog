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