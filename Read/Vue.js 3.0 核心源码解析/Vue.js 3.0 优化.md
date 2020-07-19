# Vue.js 3.0 优化

Vue.js 从 1.x 到 2.0 版本，最大的升级就是引入了虚拟 DOM 的概念，它为后续做服务端渲染以及跨端框架 Weex 提供了基础。

2.x有很多需要解决的痛点，比如源码自身的维护性，数据量大后带来的渲染和更新的性能问题，一些想舍弃但为了兼容一直保留的鸡肋 API 等；另外，还希望有更好的 TypeScript 支持、更好的逻辑复用实践等，所以坐着希望能从源码、性能和语法 API 三个大的方面优化框架。

## 源码优化

源码的优化主要体现在使用 monorepo 和 TypeScript 管理和开发源码，这样做的目标是提升自身代码可维护性。

![Image text](https://raw.githubusercontent.com/shaneth10/blog/master/Read/Vue.js%203.0%20%E6%A0%B8%E5%BF%83%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/img-folder/Vue2-src.png)