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

在 Vue.js 2.0 中，模板使用数据等同于组件使用数据，所以当数据发生变化时，会将通知发送到组件，然后组件内部在通过虚拟 DOM 重新渲染。
总结起来，其实就一句话，在 getter 中收集依赖，在 setter 中触发依赖。

## 依赖是谁

收集谁，换句话说，就是当属性发生变化后，通知谁。
我们要通知用到数据的地方，而使用这个数据的地方有很多，而且类型还不一样，极有可能是模板，也有可能是用户写的一个 watch ，这是需要抽象出一个能集中处理这些情况的类。然后，我们在依赖收集阶段只收集这个封装好的类的实例进来，通知也只通知它一个。接着，它再负责通知其他地方。所以，我们要抽象的这个东西取名叫 Watcher !

## 什么是 Watcher

只要把这个 watcher 实例添加到 data.a.b.c 属性的 Dep 中就行了。然后，当 data.a.b.c 的值发生变化时，通知 Watcher 。接着，Watcher 再执行参数中的这个回调函数。

在 get 方法中先把 window.target 设置成了 this ，也就是当前 watcher 实例，然后再读一下 data.a.b.c 的值就触发了 getter 。

触发了 getter ，就会触发收集依赖的逻辑，会从 window.target 中读取一个依赖并添加到 Dep 中。

这就导致，只要先在 window.target 赋一个 this ，然后再读一下值去触发 getter ，就可以把 this 主动添加到 keypath 的 Dep 中。

依赖注入到 Dep 中后，每当 data.a.b.c 的值发生变化时，就会让依赖列表中多有的依赖循环触发 update 方法，也就是 Watcher 中的 update 方法。而 update 方法会执行参数中的回调函数，将 value 和 oldValue 传到参数中。

所以，其实不管是用户执行的 vm.$watch('a.b.b', (value, oldValue) => {})，还是模板中用到的data，都是通过 Watcher 来通知自己是否需要发生变化。

## 递归侦测所有 key

将数据中的所有属性都侦测到，所以要封装一个 Observer 类。这个类的作用是将一个数据内的所有属性都转换成 getter/setter 的形式，然后去追踪他们的变化。

定义一个 Observer 类，它用来将一个正常的 object 转换成被侦测的 object。

然后判断数据的类型，只有 Object 类型的数据才会调用 walk 将每一个属性转换成 getter/setter 的形式来侦测变化。

最后，在 defineReactive 中新增 new Observer(val) 来递归紫属性，这样我们就可以把 data 中的所有属性都转换成 getter/setter 的形式来侦测变化。

当 data 中的属性发生变化时，与这个属性对应的依赖就会接收到通知。也就是说，只要我们将一个 object 传到 Observer 中，那么这个 object 就会变成响应式的 object。

