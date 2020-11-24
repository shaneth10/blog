# JavaScript 相关题目汇总

## 01、数据扁平化

数据扁平化就是指将一个多维数组变为一个一维数组
```
const arr = [1, [2, [3, [4, 5]]], 6];
// => [1, 2, 3, 4, 5, 6]
```

### 方法一：使用flat()
```
const res1 = arr.flat(Infinity); // 使用 Infinity，可展开任意深度的嵌套数组
```

### 方法二：使用正则
```
const res2 = JSON.stringify(arr).replace(/\[|\]/g, '').split(','); // 会转成字符串类型
const res3 = JSON.parse('[' + JSON.stringify(arr).replace(/\[|\]/g, '') + ']'); // 改良版本
```

### 方法三：使用reduce
```
const flatten = arr => {
  return arr.reduce((pre, cur) => {
    return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
  }, [])
}
const res4 = flatten(arr);
```

### 方法四：函数递归
```
const res5 = [];
const fn = arr => {
  for (let i = 0; i < arr.length; i++) {
    if (Array.isArray(arr[i])) {
      fn(arr[i]);
    } else {
      res5.push(arr[i]);
    }
  }
}
fn(arr);
```

## 02、数组去重

```
const arr = [1, 1, '1', 17, true, true, false, false, 'true', 'a', {}, {}];
// => [1, '1', 17, true, false, 'true', 'a', {}, {}]
```

### 方法一：利用Set
```
const res1 = Array.from(new Set(arr)); // set对象存储的值总是唯一的
```

### 两层 for 循环+splice
```
const unique1 = arr => {
  let len = arr.length;
  for (let i = 0; i < len; i++) {
    for (let j = i + 1; j < len; j++) {
      if (arr[i] === arr[j]) {
        arr.splice(j, 1);
        // 每删除一个树，j--保证j的值经过自加后不变。同时，len--，减少循环次数提升性能
        len--;
        j--;
      }
    }
  }
  return arr;
}
```

### 利用 indexOf
```
const unique2 = arr => {
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (res.indexOf(arr[i]) === -1) res.push(arr[i]);
  }
  return res;
}
```
当然也可以用include、filter，思路大同小异。
```
const unique3 = arr => {
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (!res.includes(arr[i])) res.push(arr[i]);
  }
  return res;
}

const unique4 = arr => {
  return arr.filter((item, index) => {
    return arr.indexOf(item) === index;
  });
}
```

### 利用 Map
```
const unique5 = arr => {
  const map = new Map();
  const res = [];
  for (let i = 0; i < arr.length; i++) {
    if (!map.has(arr[i])) {
      map.set(arr[i], true)
      res.push(arr[i]);
    }
  }
  return res;
}
```

## 03、类数组转化为数组

类数组是具有length属性，但不具有数组原型上的方法。常见的类数组有arguments、DOM操作方法返回的结果。

### 方法一：Array.from
```
Array.from(document.querySelectorAll('div'))
```

### 方法二：Array.prototype.slice.call()
```
Array.prototype.slice.call(document.querySelectorAll('div'))
```

### 方法三：扩展运算符
```
[...document.querySelectorAll('div')]
```

### 方法四：利用concat
```
Array.prototype.concat.apply([], document.querySelectorAll('div'))
```

## 04.Array.prototype.filter()
```
array.filter(function(currentValue,index,arr), thisValue)
```

```
Array.prototype.filter = function(callback, thisArg) {
  if (this == undefined) {
    throw new TypeError('this is null or not undefined');
  }
  if (typeof callback !== 'function') {
    throw new TypeError(callback + 'is not a function');
  }
  const res = [];
  // 让O成为回调函数的对象传递（强制转换对象）
  const O = Object(this);
  // >>>0 保证len为number，且为正整数
  const len = O.length >>> 0;
  for (let i = 0; i < len; i++) {
    // 检查i是否在O的属性（会检查原型链）
    if (i in O) {
      // 回调函数调用传参
      if (callback.call(thisArg, O[i], i, O)) {
        res.push(O[i]);
      }
    }
  }
  return res;
}
```

## 05.Array.prototype.map()
```
array.map(function(currentValue,index,arr), thisValue)
```

```
Array.prototype.map = function(callback, thisArg) {
  if (this == undefined) {
    throw new TypeError('this is null or not defined');
  }
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  const res = [];
  // 同理
  const O = Object(this);
  const len = O.length >>> 0;
  for (let i = 0; i < len; i++) {
    if (i in O) {
      // 调用回调函数并传入新数组
      res[i] = callback.call(thisArg, O[i], i, this);
    }
  }
  return res;
}
```

## 06.Array.prototype.forEach()
```
array.forEach(function(currentValue, index, arr), thisValue)
```
forEach跟map类似，唯一不同的是forEach是没有返回值的。  
```
Array.prototype.forEach = function(callback, thisArg) {
  if (this == null) {
    throw new TypeError('this is null or not defined');
  }
  if (typeof callback !== "function") {
    throw new TypeError(callback + ' is not a function');
  }
  const O = Object(this);
  const len = O.length >>> 0;
  let k = 0;
  while (k < len) {
    if (k in O) {
      callback.call(thisArg, O[k], k, O);
    }
    k++;
  }
}
```

## 07.Array.prototype.reduce()
```
array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
```

函数累计处理的结果
```
Array.prototype.reduce = function(callback, initialValue) {
  if (this == undefined) {
    throw new TypeError('this is null or not defined');
  }
  if (typeof callback !== 'function') {
    throw new TypeError(callbackfn + ' is not a function');
  }
  const O = Object(this);
  const len = this.length >>> 0;
  let accumulator = initialValue;
  let k = 0;
  // 如果第二个参数为undefined的情况下
  // 则数组的第一个有效值作为累加器的初始值
  if (accumulator === undefined) {
    while (k < len && !(k in O)) {
      k++;
    }
    // 如果超出数组界限还没有找到累加器的初始值，则TypeError
    if (k >= len) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    accumulator = O[k++];
  }
  while (k < len) {
    if (k in O) {
      accumulator = callback.call(undefined, accumulator, O[k], k, O);
    }
    k++;
  }
  return accumulator;
}
```