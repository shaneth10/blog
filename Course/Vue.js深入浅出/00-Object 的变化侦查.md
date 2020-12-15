# 第二章 Object 的变化侦查

Vue.js 会自动通过状态生成 DOM，并将其输出到页面上显示出来，这个过程叫渲染。Vue.js 的渲染过程是声明式的，我们通过模板来描述状态与 DOM 之间的映射关系。

变化侦测就是用来解决这个问题的，它分为两种类型：一种是“推”，另一种是“拉”。

Vue.js 的变化侦测属于“推”。当状态发生变化时，Vue.js 立刻就知道了，而且在一定程度上知道哪些状态变了。因此，它知道的信息更多，也就可以进行更细粒度的更新。

所谓更细粒度的更新，就是说：加入有一个状态绑定着好多个依赖，每个依赖表示一个具体的 DOM 节点，那么当这个状态发生变化时，向这个状态的所有依赖发送通知，让它们进行 DOM 更新操作。相比较而言，“拉”的粒度是最粗的。

## 如何追踪变化

学过 JavaScript 的人都知道，有两种方法可以侦测到变化：使用 Object.defineProperty 和 ES6 的 proxy 。但是，由于 ES6 在浏览器中的支持度并不理想，到目前为止 Vue.js 还是使用 Object.defineProperty 来实现的。我们可以写出这样的代码：

```
function defineReactive(data, key, val) {
  Object.defineProperty(data, key, {
    enumerable: true,
    configurable: true,
    get: function() {
      return val
    },
    set: function(newVal) {
      if (val === newVal) {
        return
      }
      val = newVal
    }
  })
}
```
定义一个响应式数据，在函数中进行变化追踪。每当从 data 的 key 中读取数据时，get 函数被触发；每当往 data 的 key 中设置数据时， set 函数被触发。

## 如何收集依赖