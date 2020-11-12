# requestAnimationFrame 

```window.requestAnimationFrame()``` 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画（即根据传入的回调函数，需再次调用 ```window.requestAnimationFrame()``` ）。  
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