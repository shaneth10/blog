# 强制类型转换

## 值类型转换

将值从一种类型转换为另一种类型通常为类型转换，这是显式的情况，隐式的情况称为强制类型转换。  
类型转换发生在静态类型语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时，然而在 JavaScript 中通常将它们统称为强制类型转换。  
```
var a = 42
var b = a + '' // 隐式强制类型转换
var c = String(a) // 显示强制类型转换
```

## 抽象值操作

### ToString

### ToNumber
有时我们需要将非数字值当作数字来使用，其中 true 转换为 1，false 转换为 0。 undefined 转换为 NaN，null 转换为 0。  
如果 valueOf() 和 toString() 均不返回基本类型值，会产生 TypeError 错误。  
从 ES5 开始，使用 Object.create(null) 创建的对象 Prototype 属性为 null，并且没有 valueOf() 和 toString() 方法，因此无法进行强制类型转换。
```
Number('') // 0
Number([]) // 0
Number(['abc']) // NaN
```

### ToBoolean
JavaScript 有两个关键词 true 和 false ，分别代表布尔类型中的真和假。

- 假值
undefined、null、false、+0 -0 和 NaN、""

- 假值对象
浏览器在某些特定的情况下，在常规 JavaScript 语法基础上自己创建了一些外来值，这些就是 “假值对象”。  
假值对象看起来和普通对象并无二致，但将它们强制类型转换为布尔值时结果为 false 。

- 真值
真值就是假值列表之外的值。

## 显式强制类型转换

### 字符串和数字之间的显式转换
字符串和数字之间的转换是通过 String(..) 和 Number(..) 这两个内建函数来实现的，它们前面没有 new 关键字，并不创建封装对象。  
+c 是 + 运算符的一元形式。+ 运算符显式地将 c 转换为数字，而非数字加法运算。
- 日期显式转换为数字
```
var timestamp = +new Date()
var timestamp = new Date().getTime()
var timestamp = Date.now() // ES5中新加入的静态方法

// 老版本浏览器提供 Date.now() 的 polifill 也很简单：
if (!Date.now) {
  Date.now = function() {
    return +new Date()
  }
}
```

- 奇特的~运算符
- 字位截除

### 显示转换为布尔值
Boolean(..) 是显式的 ToBoolean 强制类型转换。

## 隐式强制类型转换 

- || 和 && 
&& 和 || 运算符的返回值并不一定是布尔类型，而是两个操作数其中一个的值。
```
var a = 42
var b = 'abc'
var c = null

a || b // 42
a && b // 'abc'

c || b // 'abc'
c && b // null
```

|| 和 && 首先会对第一个操作数执行条件判断，如果岂不是布尔值就先进行 ToBoolean 强制类型转换，然后再执行条件判断。  
对于 || 来说，如果条件判断结果为 true 就返回第一个操作数的值，如果为 false 就返回第二个操作数的值。  
&& 则相反，如果条件判断结果为 true 就返回第二个操作数的值，如果为 false 就返回第一个操作数的值。

- 符号的强制类型转换
符号不能被强制类型转换为数字，但可以被强制类型转换为布尔值。
```
var s1 = Symbol('cool')
String(s1) // 'Symbol(cool)'

var s2 = Symbol('not cool')
s2 + '' // TypeError
```

## 宽松相等和严格相等

宽松相等 == 和严格相等 === 都用来判断两个值是否相等，但是它们之前有一个很重要的区别，特别是在判断条件上。  
正确的解释是：” == 允许在相等比较中进行强制类型转换，而 === 不允许。“

### 相等比较操作的性能
== 和 === 都会检查操作数的类型。区别在于操作数类型不同时它们的处理方式不同。但对于性能来说，差别上都是微不足道的，所以不需要去考虑性能方面的问题。只要考虑有没有强制类型转换的必要，有就用 == ，没有就用 === ，不用在乎性能。

### 抽象相等
- 字符串和数字之间的相等比较
如果 Type(x) 是数字，Type(y) 是字符串，则返回 x == ToNumber(y) 的结果，即是将字符串转成数字类型后再进行比较。

- 其他类型和布尔类型之间的相等比较
```
var a = '42'
var b = true

a == b // false
```
为什么会这样呢？规范是这样说的：
> 如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y 的结果，即会将布尔类型的值转成数字类型再比较，布尔类型最终会被转成1或者0.
```
// 这样的显式用法更好：
if (!!a) {}
if (Boolean(a)) {}
```

- null 和 undefined 之间的相等比较
在 == 中 null 和 undefined 是一回事，可以相互进行隐式强制类型转换

- 对象和非对象之间的相等比较
如果 Type(x) 是字符串或者数字，Type(y) 是对象，则返回 x == ToPrimitive(y) 的结果  
但有一些值不这样，原因是 == 算法中其他优先级更高的规则。例如：
```
var a = null
var b = Object(a) // 和Object()一样
a == b // false

var c = undefined
var d = Object(c) 
c == d // false

var e = NaN
var f = Object(e)
e == f // false
```
因为没有对应的封装对象，所以 null 和 undefined 不能够被封装，Object(null) 和 Object() 均返回一个常规对象。  
NaN 能够被封装为数字封装对象，但拆封之后 NaN == NaN 返回 false，因为 NaN 不等于 NaN。

### 比较少见的情况
- 返回其他数字
```
Number.prototype.valueOf = function() {
  return 3
}

new NUmber(2) == 3 // true
```
> 2 == 3 不会有这样的问题，因为 2 和 3 都是数字基本类型值，不会调用 Number.prototype.valueOf() 方法。而 Number(2) 涉及 ToPrimitive 强制类型转换，因此会调用 valueOf()。

- 假值的相等比较
```
"0" == false // true
"0" == 0 // true

false == 0 // true
false == "" // true
false == [] // true

"" == 0 // true 
"" == [] // true

0 == [] // true
```

- 极端情况
```
[] == ![] // true

2 == [2] // true
"" == [null] // true

"true" == true // false
42 == "42" // true
"foo" == ["foo"] // true
```

- 安全运用隐式强制类型转换
如果两边的值中有 true 或者 false，千万不要使用 == ;如果两边的值中有 []、"" 或者 0，尽量不要使用 ==。  
这时最好用 === 来避免不经意的强制类型转换。这两个原则可以让我们避开几乎所有强制类型转换的坑。