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