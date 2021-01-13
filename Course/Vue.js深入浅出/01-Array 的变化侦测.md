# 第三章 Array 的变化侦测

Object 的侦测方式是通过 getter/setter 实现的，但类似 push 方法来改变数组，并不会触发 getter/setter 。

正因为我们可以通过 Array 原型上的方法来改变数组的内容，所以 Object 那种通过 getter/setter 的实现方式就行不通了。

## 如何追踪变化

我们可以用一个拦截器覆盖 Array.prototype。之后，每当使用 Array 原型上的方法操作数组时，其实执行的都是拦截器中提供的方法。然后再拦截器中使用原生 Array 的原型方法去操作数组。

