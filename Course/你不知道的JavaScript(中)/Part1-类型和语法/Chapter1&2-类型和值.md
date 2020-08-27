# 类型

## 内置类型

JavaScript有七种内置类型，对象及其他几个普通类型：
- 空值（null）
- 未定义（undefined）
- 布尔值（boolean）
- 数字（number）
- 字符串（string）
- 对象（object）
- 符号（symbol，ES6中新增）

我们可以用 typeof 运算符来查看值的类型，它返回的是类型的字符串值。
```
typeof undefined === 'undefined'
typeof true === 'boolean'
typeof 42 === 'number'
typeof '42' === 'string'
typeof { life: 42 } === 'object'

// ES6中新加入的类型
typeof Symbol() === 'symbol'
```
而 null 就比较特殊，按理说返回结果应该是 ’null’ ，但是实际却是 ‘object’ ，这个 bug 存在已久，很有可能不会再修复。因此，我们需要使用复合条件来检测 null 值的类型：```(!a && typeof a === 'object')```。

还有一种情况：``` typeof function a() {} === 'function' ```。这样看来，function 实际上是 object 的一个‘子类型’。具体来说，函数是‘可调用对象’，它有一个内部属性 call ,该属性使其可以被调用。

函数不仅是对象，还可以拥有属性。例如：```function a(b ,c) {}```。函数对象的 length 属性是其声明的参数个数：```a.length // 2```。因为该函数声明了两个命名参数， b 和 c ，所以其 length 值为2。

数组也是 object 的一个子类型，数组的元素按数字顺序来进行索引，其 length 属性是元素的个数。

## 值和类型

### undefined 和 undeclared
变量在未持有值的时候为 undefined 。即已在作用域中声明但还没有赋值的变量，是 undefined 的。相反，还没有在作用域中声明过的变量，是 undeclared 的。

### typeof Undeclared
通过 typeof 的安全防范机制（阻止报错）来检查 undeclared 变量，有时是个不错的办法。
```
// 这样是安全的
if (typeof DEBUG !== "undefined") {
  console.log( "Debugging is starting" )
}
```

# 值

## 数组

和其他强类型语言不同，在 JavaScript 中，数组可以容纳任何类型的值，可以是字符串、数字、对象（object），甚至是其他数组。
使用 delete 运算符可以将单元从数组中删除，但是数组的length不会变化。
在创建”稀疏“数组时要注意：
```
var a = []

a[0] = 1
//此处没有设置a[1]单元
a[2] = [3]

a[1] // undefined

a.length // 3
```
数组通过数字进行索引，但有趣的是它们也是对象，所以也可以包含字符串键值和属性（但这不计算在数组长度内）：
```
var a = []
a[0] =1
a['foobar'] = 2

a.length // 1
a['foobar'] // 2
a.foobar // 2
```
如果说字符串键值能够被强制类型转换为十进制数字的话，它就会被当作数字索引来处理，比如说```a['13'] = 42```这种。

### 类数组
有时需要将类数组转换为真正的数组，这一般通过数组工具函数（如 indexOf()、concat()、forEach()等）来实现。ES6中有个Array.from()能实现这个功能：
```
console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]

console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]
```

## 字符串

字符串经常被当成字符数组，很多方法都能通用，比如说 indexOf()、concat() 等。
```
a.join // undefined
a.map // undefined

var c = Array.prototype.join.call(a, '-')
var d = Array.prototype.map.call(a, function(v) {
  return v.toUpperCase() + '.'
}).join('')

c // 'f-o-o'
d // 'F.O.O'
```
以上这些函数是可以通过借用来使用的，但是也有一些无法借用。
```
a.reverse // undefined
Array.prototype.reverse.call(a)
// 返回值任然是字符串”foo“的一个封装对象
```
但是我们可以用变通的方法先将字符串转换为数组，待处理完后再将结果转换回字符串：
```
var c = a.split('').reverse().join('')
```

## 数字

```
0.1 + 0.2 === 0.3 // false
```
> 二进制浮点数中的0.1和0.2并不是十分精确，它们相加的结果并非刚好等于0.3，而是一个比较接近的数字0.300000...0004，所以结果为false。  
> 最常用的方法是设置一个误差范围值，通常称为“机器精度”，对 JavaScript 的数字来说，这个值通常是2^-52。  
从 ES6 开始，该值定义在 Number.EPSILON 中，我们可以直接拿来用，也可以为 ES6 之前的版本写 polifill:
```
if (!Number.EPSILON) {
  Number.EPSILON = Math.pow(2, -52)
}
```
可以使用 Number.EPSILON 来比较两个数字是否相等（在指定的误差范围内）：
```
function numbersCloseEnoughToEqual(n1, n2) {
  return Math.abs(n1 - n2) < Number.EPSILON
}
var a = 0.1 + 0.2
var b = 0.3

numbersCloseEnoughToEqual(a, b) // true
numbersCloseEnoughToEqual(0.0000001, 0.0000002) // false
```

要检测一个值是否是整数，可以使用 ES6 中的 NUmber.isInteger(..) 方法：
```
Number.isInteger(42) // true
Number.isInteger(42.3) // false
```

polyfill 方法：
```
if (!Number.isInteger) {
  Number.isInteger = function(num) {
    return typeof num == "Number" && num % 1 === 0
  }
}
```

要检测一个值是否是安全的整数，可以使用 ES6 中的 Number.isSafeInteger(..) 方法：
```
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Math.pow(2, 53)) // false
Number.isSafeInteger(Math.pow(2, 53) - 1) // true
```

polifill 方法：
```
if (!Number.isSafeInteger) {
  Number.isSafeInteger = function(num) {
    return Number.isInteger(num) && Math.abs(num) <= Number.MAX_SAFE_INTEGER
  }
}
```

a | 0 可以将变量 a 的数字转换为32位有符号整数，因为数位运算符 | 只适用于32位整数 

## 特殊数值

undefined 和 null ，他们的名称既是类型也是值。
- undefined 指从未赋值
- null 指曾赋过值，但是目前没有值

如果数字运算的操作数不是数字类型，就无法返回一个有效的数字，这种情况下返回值为 NaN 。NaN 是一个特殊值，它和自身不相等，可以用 isNaN() 来判断一个值是否是 NaN ，不过很多 JavaScript 程序都可能存在 NaN 方面的问题，所以我们应该尽量使用 Number.isNaN(...) 这样可靠的方法。

ES6 中新加入了一个工具方法 Object.is(..) 来判断两个值是否绝对相等，可以用来处理上述所有的特殊情况：
```
var a = 2 / 'foo'
var b = -3 * 0

Object.is(a, NaN) // true
Object.is(b, -0) // true
Object.is(b, 0) // false
```