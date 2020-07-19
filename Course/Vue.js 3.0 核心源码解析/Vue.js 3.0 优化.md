# Vue.js 3.0 优化

Vue.js 从 1.x 到 2.0 版本，最大的升级就是引入了虚拟 DOM 的概念，它为后续做服务端渲染以及跨端框架 Weex 提供了基础。

2.x有很多需要解决的痛点，比如源码自身的维护性，数据量大后带来的渲染和更新的性能问题，一些想舍弃但为了兼容一直保留的鸡肋 API 等；另外，还希望有更好的 TypeScript 支持、更好的逻辑复用实践等，所以坐着希望能从源码、性能和语法 API 三个大的方面优化框架。

## 源码优化

源码的优化主要体现在使用 monorepo 和 TypeScript 管理和开发源码，这样做的目标是提升自身代码可维护性。

### monorepo代码管理
vue.js 2.x源码托管在src目录:
```
src
  - compiler
  - core
  - platforms
  - server
  - sfc
  - shared
```
vue.js 3.0整个源码是通过monorepo的方式维护的，根据功能将不同的模块拆分到packages目录下面不同的子目录中:
```
packages
  - compiler-core
  - compiler-dom
  - compiler-sfc
  - compiler-ssr
  - reactivity
  - runtime-core
  - runtime-dom
  - runtime-test
  - server-renderer
  - shared
  - size-check
  - template-explorer
  - vue
  global.d.ts
```
> monorepo把这些模块分到不同的package中，每个package有各自的API、类型定义和测试。
> 另外一些package(比如reactivity响应式库)是可以独立于Vue.js使用的，这样用户如果只想使用Vue.js3.0的响应式能力，可以单独以来这个响应式库而不用去依赖整个Vue.js,减小了引用包的体积大小。

### TypeScript

Vue.js 3.0 自身采用了 TypeScript 开发，而1.x没有用到类型语言，2.0的时候用了Flow，因为对于复杂场景类型的检查，支持得并不好，所以换用了 TypeScript。

## 性能优化

### 源码体积优化

- 移除一些冷门的feature
- 引入tree-shaking的基数，减少打包体积(压缩阶段会利用例如 uglify-js、terser 等压缩工具真正地删除这些没有用到的代码)
```
export function square(x) {
  return x * x
}
export function cube(x) {
  return x * x * x
}
```
引入cube
```
import { cube } from './math.js'
// do something with cube
```
math 模块会被 webpack 打包生成如下代码：
```
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
  'use strict';
  /* unused harmony export square */
  /* harmony export (immutable) */ __webpack_exports__['a'] = cube;
  function square(x) {
    return x * x;
  }
  function cube(x) {
    return x * x * x;
  }
});
```

### 数据劫持优化

> DOM 是数据的一种映射，数据发生变化后可以自动更新 DOM，用户只需要专注于数据的修改，没有其余的心智负担。

想实现这个功能，必须劫持数据的访问和更新。也就是说，为了自动更新 DOM，那么就必须劫持数据的更新，也就是说当数据发生改变后能自动执行一些代码去更新 DOM。因为在渲染 DOM 的时候访问了数据，我们可以对它进行访问劫持，这样就在内部建立了依赖关系，也就知道数据对应的 DOM 是什么了。

Vue.js 1.x 和 Vue.js 2.x 内部都是通过 Object.defineProperty 这个 API 去劫持数据的 getter 和 setter。但这个 API 有一些缺陷，它必须预先知道要拦截的 key 是什么，所以它并不能检测对象属性的添加和删除。

另外 Object.defineProperty 的方式还有一个问题，对于一个嵌套层级较深的对象，要劫持它内部深层次的对象变化，就需要递归遍历这个对象，执行 Object.defineProperty 把每一层对象数据都变成响应式的，会有相当大的性能负担。

Vue.js 3.0 使用了 Proxy API 做数据劫持，由于它劫持的是整个对象，那么自然对于对象的属性的增加和删除都能检测到。Vue.js 3.0 的处理方式是在 getter 中去递归响应式，这样的好处是真正访问到的内部对象才会变成响应式，而不是无脑递归。

### 编译优化

new Vue -> init -> $mount -> compile -> render -> vnode -> patch -> DOM

> 响应式过程就发生在图中的init阶段，另外 template compile to render function 的流程是可以借助 vue-loader 在 webpack 编译阶段离线完成，并非一定要在运行时完成。
> 所以想优化整个 Vue.js 的运行时，除了数据劫持部分的优化，也可以通过在编译阶段优化编译的结果，来实现运行时 patch 过程的优化。
> Vue.js 3.0通过编译阶段对静态模板的分析，编译生成了 Block tree，一个将模版基于动态节点指令切割的嵌套区块，每个区块内部的节点结构是固定的，而且每个区块只需要以一个 Array 来追踪自身包含的动态节点。借助 Block tree，Vue.js 将 vnode 更新性能由与模版整体大小相关提升为与动态内容的数量相关。
