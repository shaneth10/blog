# Vue.js 源码目录设计

```
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```

## compiler

compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。

## core

core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。

## platform

Vue.js 是一个跨平台的 MVVM 框架，platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。

## server

Vue.js 2.0 支持了服务端渲染，相关的逻辑是放在这个目录下的。服务端渲染主要的工作是把组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。

## sfc

这个目录下的代码逻辑会把 .vue 文件内容解析成一个 JavaScript 的对象。

## shared

Vue.js 会定义一些工具方法，这里定义的工具方法都是会被浏览器端的 Vue.js 和服务端的 Vue.js 所共享的。

# Vue.js源码构建

## 构建脚本

基于 NPM 托管的项目都会有一个 package.json 文件，它是对项目的描述文件，我们通常会配置 script 字段作为 NPM 的执行脚本，Vue.js 源码构建的脚本如下：
```
{
  "script": {
    "build": "node scripts/build.js",
    "build:ssr": "npm run build -- web-runtime-cjs,web-server-renderer",
    "build:weex": "npm run build -- weex"
  }
}
```

## 构建过程

在``` script/build.js ```中有这样几行代码：
```
let builds = require('./config').getAllBuilds()

// filter builds via command line arg
if (process.argv[2]) {
  const filters = process.argv[2].split(',')
  builds = builds.filter(b => {
    return filters.some(f => b.output.file.indexOf(f) > -1 || b._name.indexOf(f) > -1)
  })
} else {
  // filter out weex builds by default
  builds = builds.filter(b => {
    return b.output.file.indexOf('weex') === -1
  })
}

build(builds)
```
先从配置文件读取配置，在通过命令行参数对构建配置做过滤

## Runtime Only VS Runtime + Compiler