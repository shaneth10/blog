# webpack-dev-server

在构建代码并部署到生产环境之前，我们需要一个本地环境，用于运行我们开发的代码。这个环境相当于提供了一个简单的服务器，用于访问 webpack 构建好的静态文件，我们日常开发时可以使用它来调试前端代码。

前边我们已经简单介绍过 webpack-dev-server 的使用了。webpack-dev-server 是 webpack 官方提供的一个工具，可以基于当前的 webpack 构建配置快速启动一个静态服务，除此之外，基于 webpack-dev-server 丰富的配置，我们还可以使用它来帮助我们进一步完善本地的前端开发环境。

## 配置

「快速开始」的章节我们已经介绍过 webpack-dev-server 的基本安装和使用，这里不再赘述。

webpack-dev-server 默认使用 8080 端口，如果你使用了 html-webpack-plugin 来构建 HTML 文件，并且有一个 index.html 的构建结果，那么直接访问 http://localhost:8080/ 就可以看到 index.html 页面了。如果没有 HTML 文件的话，那么 webpack-dev-server 会生成一个展示静态资源列表的页面。

接下来我们看一下 webpack-dev-server 的相关配置。

在 webpack 的配置文件中，我们可以通过 devServer 字段来配置 webpack-dev-server，如端口、启动 gzip 压缩等，这里简单讲解几个常用的配置。

`public` 字段用于指定静态服务的域名，默认是 http://localhost:8080/ ，当你使用 Nginx 来做反向代理时，应该就需要使用该配置来指定 Nginx 配置使用的服务域名。

`port` 字段用于指定静态服务的端口，如上，默认是 8080，通常情况下都不需要改动。

`publicPath` 字段用于指定构建好的静态文件在浏览器中用什么路径去访问，默认是 `/`，例如，对于一个构建好的文件 `bundle.js`，完整的访问路径是 `http://localhost:8080/bundle.js`，如果你配置了 `publicPath: 'assets/'`，那么上述 `bundle.js` 的完整访问路径就是 `http://localhost:8080/assets/bundle.js`。可以使用整个 URL 来作为 `publicPath` 的值，如 `publicPath: 'http://localhost:8080/assets/'`。

**如果你使用了 HMR，那么要设置 `publicPath` 就必须使用完整的 URL。**

> 建议将 devServer.publicPath 和 output.publicPath 的值保持一致。

`proxy` 用于配置 webpack-dev-server 将特定 URL 的请求代理到另外一台服务器上。当你有单独的后端开发服务器用于请求 API 时，这个配置相当有用。例如：

`
proxy: {
  '/api': {
    target: "http://localhost:3000", // 将 URL 中带有 /api 的请求代理到本地的 3000 端口的服务上
    pathRewrite: { '^/api': '' }, // 把 URL 中 path 部分的 `api` 移除掉
  },
}
`

webpack-dev-server 的 proxy 功能是使用 http-proxy-middleware 来实现的。

`contentBase` 用于配置提供额外静态文件内容的目录，之前提到的 `publicPath` 是配置构建好的结果以什么样的路径去访问，而 `contentBase` 是配置额外的静态文件内容的访问路径，即那些不经过 webpack 构建，但是需要在 webpack-dev-server 中提供访问的静态资源（如部分图片等）。推荐使用绝对路径：

```
// 使用当前目录下的 public
contentBase: path.join(__dirname, "public") 

// 也可以使用数组提供多个路径
contentBase: [path.join(__dirname, "public"), path.join(__dirname, "assets")]
```

> publicPath 的优先级高于 contentBase。


`before` 和 `after` 配置用于在 webpack-dev-server 定义额外的中间件，如

```
before(app){
  app.get('/some/path', function(req, res) { // 当访问 /some/path 路径时，返回自定义的 json 数据
    res.json({ custom: 'response' })
  })
}
```

`before` 在 webpack-dev-server 静态资源中间件处理之前，可以用于拦截部分请求返回特定内容，或者实现简单的数据 mock。

`after` 在 webpack-dev-server 静态资源中间件处理之后，比较少用到，可以用于打印日志或者做一些额外处理。

webpack-dev-server 的配置项比较多，这里只列举了一些日常比较有用的。

## webpack-dev-middleware

简而言之，中间件就是在 Express 之类的 Web 框架中实现各种各样功能（如静态文件访问）的这一部分函数。多个中间件可以一起协同构建起一个完整的 Web 服务器。

webpack-dev-middleware 就是在 Express 中提供 webpack-dev-server 静态服务能力的一个中间件，我们可以很轻松地将其集成到现有的 Express 代码中去，就像添加一个 Express 中间件那么简单。

首先安装 webpack-dev-middleware 依赖：

`
npm install webpack-dev-middleware --save-dev
`

接着创建一个 Node.js 服务的脚本文件，如 app.js：

`
const webpack = require('webpack')
const middleware = require('webpack-dev-middleware')
const webpackOptions = require('./webpack.config.js') // webpack 配置文件的路径

// 本地的开发环境默认就是使用 development mode
webpackOptions.mode = 'development'

const compiler = webpack(webpackOptions)
const express = require('express')
const app = express()

app.use(middleware(compiler, {
  // webpack-dev-middleware 的配置选项
}))

// 其他 Web 服务中间件
// app.use(...)

app.listen(3000, () => console.log('Example app listening on port 3000!'))
`

然后用 Node.js 运行该文件即可：

`
node app.js # 使用刚才创建的 app.js 文件
`

使用 webpack-dev-server 的好处是相对简单，直接安装依赖后执行命令即可，而使用 webpack-dev-middleware 的好处是可以在既有的 Express 代码基础上快速添加 webpack-dev-server 的功能，同时利用 Express 来根据需要添加更多的功能，如 mock 服务、代理 API 请求等。

其实 webpack-dev-server 也是基于 Express 开发的，前面提及的 webpack-dev-server 中 `before` 或 `after` 的配置字段，也可以用于编写特定的中间件来根据需要添加额外的功能。

## mock

在前端的日常开发工作中，我们本地需要的不仅仅是提供静态内容访问的服务，还需要模拟后端 API 数据来做一些应用测试工作，这个时候我们需要一个 mock 数据的服务，而 webpack-dev-server 的 `before` 或 `proxy` 配置，又或者是 webpack-dev-middleware 结合 Express，都可以帮助我们来实现简单的 mock 服务。

我们先基于 Express app 实现一个简单 mock 功能的方法：
```
module.export = function mock(app) {
  app.get('/some/path', (req, res) => {
    res.json({ data: '' })
  })

  // ... 其他的请求 mock
  // 如果 mock 代码过多，可以将其拆分成多个代码文件，然后 require 进来
}
```

然后应用到配置中的 `before` 字段：

```
const mock = require('./mock')

// ...
before(app) {
  mock(app) // 调用 mock 函数
}
```

这样的 `mock` 函数照样可以应用到 Express 中去，提供与 webpack-dev-middleware 同样的功能。

由于 `app.get('', (req, res) => { ... })` 的 callback 可以拿到 req 请求对象，其实可以根据请求参数来改变返回的结果，即通过参数来模拟多种场景的返回数据来协助测试多种场景下的代码应用。

当你单独实现或者使用一个 mock 服务时，你可以通过 proxy 来配置部分路径代理到对应的 mock 服务上去，从而把 mock 服务集成到当前的开发服务中去，相对来说也很简单。

当你和后端开发进行联调时，亦可使用 proxy 代理到对应联调使用的机器上，从而可以使用本地前端代码的开发环境来进行联调。当然了，连线上环境的异常都可以这样来尝试定位问题。