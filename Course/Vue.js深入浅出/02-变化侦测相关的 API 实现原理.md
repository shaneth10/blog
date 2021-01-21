# 变化侦测相关的 API 实现原理

## vm.$watch

## vm.$set

vm.$set(target, key, value)

如果 target 身上没有 _ob_ 属性，说明它并不是响应式的，并不需要做什么特殊处理，只需要通过 key 和 val 在 target 上设置就行了。

如果前面的所有判断条件都不满足，那么说明用户是在响应式数据上新增了一个属性，这种情况下需要追踪这个新增属性的变化，即使用 defineReactive 将新增属性转换成 getter/setter 的形式即可。

## vm.$delete