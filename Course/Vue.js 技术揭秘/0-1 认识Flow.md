# 认识 Flow

Vue.js 的源码利用了 Flow 做了静态类型检查。  
类型检查是当前动态类型语言的发展趋势，就是在编译期尽早发现（由类型错误引起的）bug，又不影响代码运行（不需要运行时动态检查类型），使编写 JavaScript 具有和编写 Java 等强类型语言相近的体验。

## Flow 的工作方式

通常类型检查分成 2 种方式：
- 类型推断：  
通过变量的使用上下文来推断出变量类型，然后根据这些推断来检查类型。
- 类型注释：  
事先注释好我们期待的类型，Flow 会基于这些注释来判断。

### 类型推断
```
function split(str) {
  return str.split(' ')
}

split(11)
```
Flow 检查上述代码会报错，因为函数 split 期待的参数是字符串，而我们输入了数字。

### 类型注释
在某些特定的场景下，添加类型注释可以提供更好更明确的检查依据。  
```
function add(x: number, y: number): number {
  return x + y
}   

add('Hello', 11)
```
这样 Flow 就能检查出错误，函数参数的期待类型为数字，而我们提供了字符串。

- 数组  
数组类型注释的格式是 Array<T>，T 表示数组中每项的数据类型，如果我们给这个数组添加了一个字符串，Flow 能检查出错误。
```
var arr: Array<number> = [1, 2, 3]
arr.push('Hello')
```

- 类和对象  
类的类型注释格式如上，可以对类自身的属性做类型检查，也可以对构造函数的参数做类型检查。这里需要注意的是，属性 y 的类型中间用 | 做间隔，表示 y 的类型即可以是字符串也可以是数字。
```
class Bar {
  x: string;           // x 是字符串
  y: string | number;  // y 可以是字符串或者数字
  z: boolean;

  constructor(x: string, y: string | number) {
    this.x = x
    this.y = y
    this.z = false
  }
}

var bar: Bar = new Bar('hello', 4)

var obj: { a: string, b: number, c: Array<string>, d: Bar } = {
  a: 'hello',
  b: 11,
  c: ['hello', 'world'],
  d: new Bar('hello', 3)
}
```

- Null  
若想任意类型 T 可以为 null 或者 undefined，只需类似如下写成 ?T 的格式即可。  
```
var foo: ?string = null
```