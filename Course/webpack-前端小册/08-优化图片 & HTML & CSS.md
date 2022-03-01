# 优化图片 & HTML & CSS

webpack mode 为 production 时会启动很多 plugin 以及很多优化配置，来尽可能减少构建后生成文件的体积大小。

我们总是希望浏览器在加载页面时用的时间越短越好，所以构建出来的文件应该越少越小越好，一来减少浏览器需要发起请求的数量，二来减少下载静态资源的时间。

模块化开发会把业务代码拆成一个个模块，代码文件数量多，webpack 本身的模块化打包，把多个代码模块打包成几个必要的文件，本身已经很大程度减少了静态资源请求数量了，接下来的内容都是有关如何使用 webpack 来实现更多前端资源加载优化的需求的。

- 图片资源相关的优化，以及 HTML 和 CSS 的代码压缩
- 深入 JS 代码优化部分，探讨如何使用 webpack 减少生成的 JS 代码量
- 拆分 JS 代码文件，探讨如何使用 webpack 来实现按需异步加载代码模块

## 图片压缩

我们之前提及使用 file-loader 来处理图片文件，在此基础上，我们再添加一个 image-webpack-loader 来压缩图片文件。简单的配置如下：

```
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /.*\.(gif|png|jpe?g|svg|webp)$/i,
        use: [
          {
            loader: 'file-loader',
            options: {}
          },
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: { // 压缩 jpeg 的配置
                progressive: true,
                quality: 65
              },
              optipng: { // 使用 imagemin-optipng 压缩 png，enable: false 为关闭
                enabled: false,
              },
              pngquant: { // 使用 imagemin-pngquant 压缩 png
                quality: '65-90',
                speed: 4
              },
              gifsicle: { // 压缩 gif 的配置
                interlaced: false,
              },
              webp: { // 开启 webp，会把 jpg 和 png 图片压缩为 webp 格式
                quality: 75
              },
          },
        ],
      },
    ],
  },
}
```

image-webpack-loader 的压缩是使用 imagemin 提供的一系列图片压缩类库来处理的。

## 使用 DataURL

前端以前很流行一种优化大量小图片加载效率的方法，叫 CSS Sprites，利用工具将多个小图片合并成一张，然后利用 CSS background position 的方式来引用到对应的图片资源，这种方式受到 CSS background 的限制，并且 position 的值都由工具生成，有时候不便于维护。

现在更为方便的方式是直接将小图片转换为 base64 编码，使用 DataURL 来引用它，将图片变成编码和代码文件打包到一起，同样可以起到减少小图片请求数量的效果。

url-loader 和 file-loader 的功能类似，但是在处理文件的时候，可以通过配置指定一个大小，当文件小于这个配置值时，url-loader 会将其转换为一个 base64 编码的 DataURL，配置如下：

```
module.exports = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192, // 单位是 Byte，当文件小于 8KB 时作为 DataURL 处理
              fallback: 'file-loader',
            },
          },
        ],
      },
    ],
  },
}
```

## 代码压缩

对于 HTML 文件，之前介绍的 html-webpack-plugin 插件可以帮助我们生成需要的 HTML 并对其进行压缩：

```
module.exports = {
  // ...
  plugins: [
    new HtmlWebpackPlugin({
      template: 'assets/index.html', // 配置文件模板
      minify: { // 压缩 HTML 的配置
        minifyCSS: true, // 压缩 HTML 中出现的 CSS 代码
        minifyJS: true, // 压缩 HTML 中出现的 JS 代码
        collapseInlineTagWhitespace: true, 
        collapseWhitespace: true, // 和上一个配置配合，移除无用的空格和换行
      }
    }),
  ],
}
```

对于 CSS 文件，我们之前介绍过用来处理 CSS 文件的 css-loader，在 1.0 版本之后移除了 cssnano 的 CSS 压缩功能，所以我们需要压缩 CSS 代码的话，得使用 postcss-loader，在它的基础上来使用 cssnano：

```
module.exports = {
  module: {
    rules: {
      // ...
      {
        test: /\.css/,
        include: [
          path.resolve(__dirname, 'src'),
        ],
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              plugins: () => [ // 返回 postcss 的插件列表
                require('cssnano')(), // 使用 cssnano
              ],
            },
          },
       ],
      },
    },
  }
}
```

`postcss` 是个强大的 css 处理器，它还有很多优秀的插件可以使用，例如之前我们提到的 `autoprefixer`，具体插件参考官方文档来使用即可。




