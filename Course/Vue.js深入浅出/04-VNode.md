# VNode

## 什么是 VNode

在 Vue.js 中存在一个 VNode 类，使用它可以实例化不同类型的 vnode 实例，而不同类型的 vnode 实例鸽子表示不同类型的 DOM 元素。

简单地说， vnode 可以理解成节点描述对象，它描述了应该怎样去创建真实的 DOM 节点。

vnode 表示一个真实的 DOM 元素，所有真实的 DOM 节点都使用 vnode 创建并插入到页面中。渲染视图的过程是先创建 vnode，然后再使用 vnode去生成真实的 DOM 元素，最后插入到页面渲染视图。