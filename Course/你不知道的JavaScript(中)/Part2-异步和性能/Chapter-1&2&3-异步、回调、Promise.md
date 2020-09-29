# 异步：现在与将来

## 分块的程序

可以把 JavaScript 程序写在单个 .js 文件中，但是这个程序几乎一定是由多个块构成的。这些块中只有一个是现在执行，其余的则会在将来执行。最常见的块单位是函数。  
```
var data = ajax( "http://some.url.1" )

console.log( data )
// date 通常不会包含 Ajax 结果
```
标准的 Ajax 请求不是同步完成的，这意味着 ajax(..) 函数还没有返回任何值可以赋给变量 data。如果 ajax(..) 能够阻塞到响应返回，那么 data = .. 赋值就会正常工作。  
其实，最简单的方法是使用一个回调函数。

## 事件循环

程序通常分成了很多小块，在事件循环队列中一个接一个地执行。严格地说，和你的程序不直接相关的其他事件也会插入到队列中。

## 并行线程

异步是关于现在和将来的时间间隙，而并行是关于能够同时发生的事情。  
并行计算最常见的工具就是进程和线程。进程和线程独立运行，并可能同时运行。

## 并发

- 非交互
- 交互
- 协作

## 任务

## 语句顺序

以上几个模块由于为经常接触的编程思路及思想，这边就一笔带过不细讲了。

# 回调

到目前为止，回调是编写和处理 JavaScript 程序异步逻辑的最常用方式。确实，回调是这门语言中最基础的异步模式。回调函数是 JavaScript 的异步主力军，并且它们不辱使命地完成了自己的任务。

## continuation

回调函数包裹或者说封装了程序的延续。

## 顺序的大脑

### 执行与计划

### 嵌套回调与链式回调
```
listen('click', function handler(evt) {
  setTimeout(function request() {
    ajax(....)
  })
})
```
以上三个函数进行嵌套构成的链，其中每个函数异步序列中的一个步骤。  
这种代码常常被称为回调函数，有时也被称为毁灭金字塔，得名于嵌套缩进产生的横向三角形状。

## 信任问题

## 省点回调

# Promise

通过回调表达程序异步和管理并发的两个主要缺陷：缺乏顺序性和可信任性。所以，我们就要请出Promise，绝大多数 JavaScript/DOM 平台新增的异步 API 都是基于 Promise 构建的。

## 什么是Promise

### 未来值
- 现在值与将来值
- Promise
```
function add(xPromise, yPromise) {
  // Promise.all([])接受一个promise数组并返回一个新的promise,这个新的promise等待数组中的所有promise完成
  return Promise.all([xPromise, yPromise])

  // 这个promise决议之后，我们取得收到的X和Y值并加在一起
  .then(function(values) {
    // values是来自于之前决议的promisesei的消息数组
    return values[0] + values[1]
  })
}

// fetch()和fetch()返回相应值的promise，可能已经就绪，也可能以后就绪
add(fetchX(), fetchY())

// 我们得到一个这两个数组的和的promise
// 现在链式调用 then(..) 来等待返回promise的决议
.then(function(sum) {
  console.log(sum)
})
```
fetchX()和fetchY()是直接调用的，它们的返回值promise被传给 add(..)。这些 promise 代表的底层值的可用时间可能是现在或将来。   
第二层是 add(..) (通过 Promise.all([ .. ])) 创建并返回的 promise。我们可以通过调用 then(..) 等待这个 promise。add(..) 运算完成后，未来值 sum 就准备好了，可以打印出来。   
通过 Promise，调用 then(..) 实际上可以接受两个函数，第一个用于完成情况，第二个用于拒绝情况：
```
add( fetchX(), fetchY())
.then(
  // 完成处理函数
  function(sum) {
    console.log( sum )
  },

  // 拒绝处理函数
  function(err) {
    console.log( err )
  }
)
```
如果在获取 x 和 y 的过程中出错，或者在加法过程中出错，add(..) 返回的就是一个被拒绝的 promise，传给 then(..) 的第二个错误处理回调就会从这个 promise 中得到拒绝值。  
Promise 是一种封装和组合未来值的易于复用的机制。  

### 完成事件
在典型的 JavaScript 风格中，如果需要侦听某个通知，你可能就会想到事件。因此，可以把对通知的需求重新组织为对 foo(..) 发出的一个完成事件的侦听。  
使用回调的话，通知就是任务（foo(..)）调用的回调。而是用 Promise 的话，我们把这个关系反转了过来，侦听来自 foo(..) 的事件，然后在得到通知的时候，根据情况继续。  

- Promise "事件"
```
function foo(x) {
  // 做一些可能耗时的工作

  // 构造并返回一个 promise
  return new Promise( function(resolve, reject) {
    // 最终调用 resolve(..) 或者 reject(..)
    // 这是这个 promise 的决议回调
  })
}

var p = foo(42)

bar(p)

baz(p)
```
> new Promise(function() {}) 模式通常称为 revealing constructor 。传入的函数会立即执行，它有两个参数，在本例中我们将其分别称为 resolve 和 reject。这些是 promise 的决议函数。resolve(..) 通常标识完成，而 reject(..) 则标识拒绝。

## 具有 then 方法的鸭子类型

识别 Promise 就是定义某种称为 thenable 的东西，将其定义为任何具有 then(..) 方法的对象和函数。我们认为，任何这样的值就是 Promise 一致的 thenable。  
根据一个值的形态对这个值的类型做出一些设定。这种类型检查一般用术语鸭子类型来表示。于是，对 thenable 值的鸭子类型检测就大致类似于：
```
if (
  p !== null && (
    typeof p === "object" ||
    typeof p === "function"
  ) && typeof p.then === "function"
) {
  // 假定这是一个 thenable!
} else {
  // 不是 thenable
}
```

## Promise 信任问题

Promise 的特性就是专门用来为这些问题提供一个有效的可复用的答案

- 调用过早
- 调用过晚
- 回调未调用
- 调用次数过少或过多
- 未能传递参数/环境值
- 吞掉错误或异常
- 是可信任的 Promise 吗
```
var p1 = new Promise(function(resolve, reject) {
  resolve(42)
})
var p1 = Promise.resolve( 42 )
var p2 = Promise.resolve( p1 )

p1 === p2 // true
```
Promise.resolve(..) 可以接受任何 thenable，将其解封为它的非 thenable 值。从 Promise.resolve(..) 得到的是一个真正的Promise，是一个可以信任的值。如果你传入的已经是真正的 Promise，那么你得到的就是它本身，所以通过 Promise.resolve(..) 过滤来获得可信任性完全没有坏处。  
Promise.resolve(..) 提供了可信任的 Promise 封装工具，可以链接使用：
```
// 不要只是这么做：
foo(42)
.then( function(v) {
  console.log(v)
})

// 而要这么做：
Promise.resolve(foo(42))
.then( function(v) {
  console.log(v)
})
```

## 链式流
```
var p = Promise.resolve(21)

p.then(function(v) {
  console.log(v) // 21

  // 用42完成连接的promise
  return v * 2
})

// 这里是链接的promise
.then(function(v) {
  console.log(v) // 42
})
```
现在第一个 then(..) 就是异步序列中的第一步，第二个 then(..) 就是第二步。这可以一直任意扩展下去。只要保持把先前的 then(..) 连到自动创建的每一个 Promise 即可。  
then(null, function(err) { .. }) 这个模式--只处理拒绝，但又把完成值传递下去--有一个缩写形式的API：catch(function(err) {...})  

让我们来总结一下链式流程控制可行的 Promise 固有特性。
- 调用 Promise 的 then(...) 会自动创建一个新的 Promise 从调用返回。
- 在完成或拒绝处理函数内部，如果返回一个值或抛出一个异常，新返回的 Promise 就相应的决议。
- 如果完成或拒绝处理函数返回一个 Promise，它将会展开，这样一来，不管它的决议值是什么，都会成为当前 then(..) 返回的链接 Promise 的决议值。

## 错误处理

对多数开发者来说，错误处理最自然的形式就是同步的 try...catch 结构。遗憾的是，它只能是同步的，无法用于异步代码模式。

## Promise 模式

### Promise.all([ .. ])

all([..]) 这种模式可以实现等待两个或更多并行/并发的任务都完成才能继续。它们的完成顺序并不重要，但是必须都要完成，才能让流程控制继续。  
Promise.all([ .. ]) 需要一个参数，是一个数组，通常由 Promise 实例组成。从 Promise.all([ .. ]) 调用返回的 promise 会收到一个完成消息。这是一个由所有传入 promise 的完成消息组成的数组，与指定的顺序一致。  
> 从 Promise.all([ .. ]) 返回的主 promise 在且仅在所有的成员 promise 都完成后才会完成。如果这些 promise 中有任何一个被拒绝的话，主 Promise.all([ .. ])promise 就会立即被拒绝，并丢弃来自其他所有 promise 的全部结果。所以，永远要记住为每个 promise 关联一个拒绝/错误处理函数，特别是从 Promise.all([ .. ]) 返回的那一个。

### Promise.race([ .. ])

有时你会想只响应“第一个跨过终点线的 Promise ”，而抛弃其他 Promise。这种模式在 Promise 中称为竞态。与 Promise.all([ .. ]) 类似，一旦有任何一个 Promise 
决议为完成，Promise.race([ .. ]) 就会完成；一旦有任何一个 Promise 决议为拒绝，它就会拒绝。而且千万不要传入一个空数组，否则它将永远不会决议。

- 1.超时竞赛
```
// foo() 是一个支持 Promise 的函数
// 前面定义的 timeoutPromise(..) 返回一个 promise，
// 这个promise会在指定延时之后拒绝
// 为 foo 设定延时
Promise.race([
  foo(), // 启动foo()
  timeoutPromise( 3000 ) // 给它 3 秒钟
]).then(
  function() {
    // foo(..)按时完成
  },
  function(err) {
    // 要么foo()被拒绝，要么只是没能够按时完成，
    // 因此要查看err了解具体原因
  }
)
```

- 2.finally
有些开发者提出，Promise 需要一个 finally(...) 回调注册，这个回调在 Promise 决议后总是会被调用，并且允许你执行任何必要的清理工作。类似于：
```
var p = Promise.resolve(42)

p.then(something)
.finally(cleanup)
.then(another)
.finally(cleanup)
```
为了避免出现未处理拒绝的问题，我们可以构建一个静态辅助工具来支持查看 Promise 的决议：
```
// polyfill 安全的 guard 检查
if (!Promise.observe) {
  Promise.observe = function(pr, cb) {
    // 观察 pr 的决议
    pr.then(
      function fullfilled(msg) {
        // 安排异步回调
        Promise.resolve(msg).then(cb)
      },
      function rejected(err) {
        // 安排异步回调
        Promise.resolve(err).then(cb)
      }
    )

    // 返回最初的 Promise
    return pr
  }
}
```

下面是如何使用这个工具：
```
Promise.race([
  Promise.observe(
    foo(),
    function cleanup(msg) {
      // 在 foo 之后清理，即使它没有在超时之前完成
    }
  ),
  timeoutPromise(3000) // 给它3秒钟
])
```

### all([ .. ]) 和 race([ .. ]) 的变体
虽然原生 ES6 Promise 中提供了内建的 Promise.all([ .. ]) 和 Promise.race([ .. ])，但这些语义还有其他几个常见的变体模式。

- none([ .. ])
这个模式类似于 all([ .. ])，不过完成和拒绝的情况互换了。所有的 Promise 都要被拒绝，即拒绝转化为完成值，反之亦然。

- any([ .. ])
这个模式与 all([ .. ]) 类似，但是会忽略拒绝，所以只需要完成一个而不是全部。

- first([ .. ])
这个模式类似于与 any([ .. ]) 的竞争，及只要第一个 Promise 完成。他就会忽略后续的任何拒绝和完成。

- last([ .. ])
这个模式类似于 first([ .. ])，但却是只有最后一个完成胜出。

### 并发迭代
有些时候会需要在一列 Promise 中迭代，并对所有 Promise 都执行某个任务，非常类似于对同步数组可以做的那样。  
在这个 map(..) 实现中，不能发送异步拒绝信号，但如果在映射的回调内出现同步的异常或错误，主 Promise.map(..) 返回的 promise 就会拒绝。

## Promise API 概述

### new Promise(..) 构造器
有启示性的构造器 Promise(..) 必须和 new 一起使用，并且必须提供一个函数回调。这个回调是同步的或立即调用的。这个函数接受两个函数回调，用以支持 promise 的决议。通常我们把这两个函数称为 resolve(..) 和 reject(..):
```
var p = new Promise( function(resolve, reject) {
  // resolve(..) 用于决议/完成这个 promise
  // reject(..) 用于拒绝这个 promise
})
```
reject(..) 就是拒绝这个 promise；但 resolve(..) 既可以完成 promise，也可以拒绝，要根据传入参数而定。如果传给 resolve(..) 的是一个真正的 Promise 或 thenable 值，这个值就会被递归展开，并且（要构造的）promise将取用其最终决议值或状态。

### Promise.resolve(..) 和 Promise.reject(..)
创建一个已被拒绝的 Promise 的快捷方式是使用 Promise.reject(..)，所以以下两个 promise 是等价的：
```
var p1 = new Promise( function(resolve, reject) {
  reject('oops')
})

var p2 = Promise.reject('oops)
```
Promise.resolve(..) 也会展开 thenable 值。在这种情况下，返回的 Promise 采用传入的这个 thenable 的最终决议值，可能是完成，可能是拒绝：
```
var p1 = Promise.resolve( filfilledTh ) // 是完成的 promise
var p2 = Promise.resolve( rejectedTh ) // 是拒绝的 promise
```
> 如果传入的是真正的 Promise，Promise.resolve(..) 什么都不会做，只会直接把这个值返回。

### then(..) 和 catch(..)
每个 Promise 实例都有 then(..) 和catch(..) 方法。通过这两个方法可以为这个 Promise 注册完成和拒绝处理函数。Promise 决议后，会立即调用这两个函数之一，但不会两个都调用，而且总是异步回调。  
then(..) 接受一个或两个参数：第一个用于完成回调，第二个用于拒绝回调。catch(..) 只接受一个拒绝回调作为参数，并自动替换默认完成回调。默认完成回调只是把消息传递下去，而默认拒绝回调则只是重新抛出其接收到的出错原因。
catch(..) 只接受一个拒绝回调作为参数，并自动替换默认完成回调。换句话说，它等价于 then(null, ..):
```
p.then( fulfilled )
p.then( fulfilled, rejected )
p.catch( rejected ) // p.then(null, rejected)
```
如果完成或拒绝回调中抛出异常，返回的 promise 是被拒绝的。如果任意一个回调返回非 promise、非thenable 的立即值，这个值会被用作返回 promise 的完成值。如果完成处理函数返回一个 promise 或 thenable，那么这个值会被展开，并作为返回 promise 的决议值。

### promise.all([ .. ]) 和 Promise.race([ .. ])
这两个函数都会创建一个 Promise 作为它们的返回值。这个 promise 的决议完全由传入的 promise 数组控制。  
若向 Promise.all([ .. ]) 传入空数组，它会立即完成，但 Promise.race([ .. ]) 会挂住，且永远不会决议。

## Promise 局限性（大概了解）

- 顺序错误处理
- 单一值
- 单决议
- 惯性
- 无法取消的 Promise
- Promise 性能