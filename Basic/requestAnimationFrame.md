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
  - setInterval可以手动设定FPS，而requestAnimationFrame则会根据浏览器及设备性能自动设定FPS。