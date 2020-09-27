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