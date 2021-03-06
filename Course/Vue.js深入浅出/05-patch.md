# patch

虚拟 DOM 最核心的部分就是 patch，它可以将 vnode 渲染成真实的 DOM。

patch 也可以叫做 patching 算法，通过它渲染真实 DOM 时，并不是包里覆盖原有 DOM，而是比对新旧两个 vnode 之间有哪些不同，然后根据对比结果找出需要更新的节点进行更新。

之所以要这么做，主要是因为 DOM 操作的执行速度远不如 JavaScript 的运算速度快。    

## patch 介绍

patch 的目的其实是修改 DOM 节点，也可以理解为渲染视图。patch 不是暴力替换节点，而是在现有 DOM 上进行修改来达到渲染视图的目的。对现有 DOM 进行修改需要做三件事：
- 创建新增的节点；
- 删除已经废弃的节点；
- 修改需要更新的节点；

由于我们的最终目的是渲染视图，所以可以发现渲染视图的标准是以 vnode 来渲染而不是 oldValue 。

### 新增节点

### 删除节点

### 更新节点

### 小结

整个 patch 的过程并不复杂。当 oldValue 不存在时，直接使用 vnode 渲染视图；当 oldValue 和 vnode 都存在但并不是同一个节点时，使用 vnode 创建的 DOM 元素替换旧的 DOM 元素；当 oldValue 和 vnode 是同一个节点时，使用更详细的对比操作对真实的 DOM 节点进行更新。

## 创建节点

