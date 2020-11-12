# requestAnimationFrame 略知一二

```window.requestAnimationFrame()``` 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画（即根据传入的回调函数，需再次调用 ```window.requestAnimationFrame()``` ，实际上就是递归的形式）。    

而这个回调函数的执行次数通常是每秒60次，但在不同浏览器中，回调函数执行次数通常与浏览器屏幕刷新次数相匹配。  

回调函数会传入 ```DOMHighResTimeStamp``` 参数，它指示当前被 ```requestAnimationFrame()``` 排序的回调函数被触发的时间，回调函数会接收到一个时间戳，最小进度为 1ms 。

**使用示例：**  
```
function step(timestamp) {
  console.log(timestamp)
  if ( true ) {
    window.requestAnimationFrame(step);
  } else {
    // ...
  }
}

window.requestAnimationFrame(step);
```

## 与 setInterval 的区别

- 1.FPS
  - setInterval 可以手动设定 FPS，而 requestAnimationFrame 则会根据浏览器及系统自动设定 FPS。
- 2.性能及掉帧问题
  - Javascript 引擎是单线程的，在某一个特定的时间内只能执行一个任务，并阻塞其他任务的执行，所以 setInterval 会在低端机上出现卡顿、抖动现象（刷新频率受屏幕分辨率和屏幕尺寸影响，会引起掉帧现象）。而 requestAnimationFrame 的策略是，当浏览器达不到设定的调用周期时，requestAnimationFrame 的策略是跳过某些帧的方式来表现动画。
- 3.节约资源
  - 页面被隐藏和最小化，定时器 setInterval 仍会运行，而 使用 requestAnimationFrame，当页面处于未激活的状态下，该页面的屏幕刷新任务会被系统暂停

## 移动端兼容性思考

由于 requestAnimationFrame 的刷新机制跟终端的 CPU，设备运行情况，屏幕刷新频率等诸多因素有关，这就导致了在不同终端上，刷新的频率是不一致的，这样就需要我们去考虑兼容性相关的问题。首先对不同浏览器做下兼容处理：
```
var raf = window.requestAnimationFrame || window.webkitRequestAnimationFrame || window.mozRequestAnimationFrame || window.oRequestAnimationFrame || window.msRequestAnimationFrame
```
我们从以下动画入手：
```
let div = document.getElementById("div1")
let count = 0

function raf_ani(timestamp) {
  count++;
  if(count > 300){
    count = 0
  }
  div.style.left = count + 'px'
}

raf(raf_ani)
```
这段实现让 div 移动的动画逻辑非常简单，但是由于 requestAnimationFrame 刷新频率虽终端而异，在不同终端上， div 的移动速度其实是不同的，而在 pc 上几乎无差异，约等于 60 fps。为了保证移动速度的统一，可以做下调整：  
```
let frameStatus = false
let startTime
let speed = 5
let totaltime = 1000 // 一帧的时间需自己设定
function raf_ani(timestamp) {
  if (!frameStatus) {
    frameStatus = true
    startTime = timestamp
  }
  let curr = (timestamp - startTime) / totaltime * speed
  div.style.left = count + 'px'
};

raf(raf_ani);
```
通过时间的判断可以让 div 的移动速度在不同终端上的速度一致。

**解决在移动端速率差异的另一种方式思考**    
除了通过事件来决定动画的步数外，还有一种方法，那就是在动画开始前，我们先通过js来计算执行的频率，在通过频率来决定动画步数的大小。
```
let count = 0
let time

function raf_test() {
  if(count === 0) {
    time = (new Date()).getTime()
  }
  if(count < 100) {
    raf(raf_ani)
  } else {
    let now = (new Date()).getTime()
    let speed = (now-time)/100
    console.log('speed:' + speed)
  }
}

raf_test()
```
比如上面这段代码，我们让raf执行了100次，在执行完后计算出执行依次的间隔时间。这样的操作需要考虑到预处理问题，避免在测试过程中影响用户体验。

**解决抓金牛游戏中遇到的动画问题**  
**背景**：在开发抓金牛活动时，遇到在安卓低版本手机特别是魅族手机中，发现牛的跑动速度非常慢，预期分批从屏幕跑过的牛一股脑全出现在了屏幕上。  

**分析问题**：由于顶部倒计时是准确的，而牛每帧移动的像素也是确定的，可以断定是 requestAnimationFrame 的刷新频率太低了。  

**解决方案**：对出现问题的机型（安卓）及版本（安卓6.1以下）采用降低频率的方法，使用setInterval 方法，降低频率到125ms调用一次，问题解决。  

**反思及优化**：虽然该兼容方法经历住了活动投放十四天的考验，但我总觉得这么处理也不是完美的方法，我还有个方法是将 setTimeout 和 requestAnimationFrame 相结合，但没在项目中使用过。  

**大致思路**：动画用 requestAnimationFrame 执行，并且用间隔为 1000/60 的 setTimeout 执行，如果 requestAnimationFrame 执行了，那么 setTimeout 下一次就不执行。也就是整体是用 setTimeout 来执行动画，到了浏览器页面渲染的刷新时间的时候，用 requestAnimationFrame 替代 setTimeout 执行，保证关键帧不掉帧，之后继续由 setTimeout 执行。这样的话，就能保证动画的执行速率基本一致，并且对于浏览器渲染刷新的关键帧也不掉帧，一定程度上提高了动画的流畅度。

```
let sraf_flag = true

function sraf(callback, speed) {
  raf_fn(callback)
  sraf_any(callback)
}

function raf_fn(callback) {
  // requestAnimationFrame 执行时设置 flag 是 false
  raf(() => {
    sraf_flag = false
    console.log('rAF:' + (new Date()).getTime())
    callback()
    raf_fn(callback)
  })
}

function sraf_any(callback) {
  setTimeout(() => {
    //如果 falg 是 false，则此次 setTimeout 内容不执行，等待下一次执行。
    if(sraf_flag) {
      callback()
    }
    sraf_flag = true
    console.log('setTimeout:' + (new Date()).getTime())
    sraf_any(callback)
  },1000/60)
}

sraf(callback)
```
