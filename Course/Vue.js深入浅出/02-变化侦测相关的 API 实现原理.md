# 变化侦测相关的 API 实现原理

## vm.$watch

## vm.$set

vm.$set(target, key, value)

如果 target 身上没有 _ob_ 属性，说明它并不是响应式的，并不需要做什么特殊处理，只需要通过 key 和 val 在 target 上设置就行了。

如果前面的所有判断条件都不满足，那么说明用户是在响应式数据上新增了一个属性，这种情况下需要追踪这个新增属性的变化，即使用 defineReactive 将新增属性转换成 getter/setter 的形式即可。

最后，向 target 的依赖触发变化通知，并返回 val。

## vm.$delete

由于 Vue.js 的变化侦测是使用 Object.defineProperty 实现的，所以如果数据是使用 delete 关键字删除的，那么无法发现数据发生了变化。

vm.$delete(target, key)

使用 vm.$delete 。他帮助我们再删除属性后自动向依赖发送消息，通知 Watcher 数据发生了变化。