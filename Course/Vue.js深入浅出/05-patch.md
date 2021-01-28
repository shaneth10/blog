# patch

虚拟 DOM 最核心的部分就是 patch，它可以将 vnode 渲染成真实的 DOM。

patch 也可以叫做 patching 算法，通过它渲染真实 DOM 时，并不是包里覆盖原有 DOM，而是比对新旧两个 vnode 之间有哪些不同，然后根据对比结果找出需要更新的节点进行更新。

之所以要这么做，主要是因为 DOM 操作的执行速度远不如 JavaScript 的运算速度快。