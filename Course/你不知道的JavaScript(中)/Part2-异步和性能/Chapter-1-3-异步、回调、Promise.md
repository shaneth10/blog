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