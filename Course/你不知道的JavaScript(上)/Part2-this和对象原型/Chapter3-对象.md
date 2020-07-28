# 对象

## 语法

对象可以通过两种形式定义：声明（文字）形式和构造形式。
对象的文字语法大概是这样：
```
var myObj = {
  key: value
  // ...
}
```
构造形式大概是这样：
```
var myObj = new Object()
myObj.key = value
```

## 类型

对象时JavaScript的基础。简单基本类型（string、boolean、number、null和undefined）本身并不是对象。

### 内置对象
JavaScript中海油一些对象子类型，通常被称为内置对象：String、Number、Boolean、Object、Function、Array、Date、RegExp、Error。

## 内容

对象的内容是由一些存储在特定命名位置的值组成的，我们称之为属性。
```
var myObject = {
  a: 2
}
myObject.a // 2 属性访问
myObject['a'] // 2 键访问
```

### 可计算属性名
ES6增加了可计算属性名，可以在文字形式中使用[]包裹一个表达式来当做属性名：
```
var prefix = 'foo'
var myObject = {
  [profix + 'bar']: "hello",
  [profix + 'baz']: "world",
}
myObject['foobar'] // hello
myObject['foobaz'] // world
```

### 复制对象

- 深拷贝
```
var newObj = JSON.parse( JSON.stringify(someObj))
```
这种方法需要保证对象时JSON安全的，所以只适用于部分情况。
- 浅拷贝
相比较深拷贝，浅拷贝非常易懂并且问题要少得多，所以ES6定义了Object.assign(..)方法来实现浅拷贝。这个方法的第一个参数是目标对象，之后还可以跟一个或多个浅拷贝。

### 属性描述符

JavaScript语言本身并没有提供可以直接检测属性特性的方法，比如判断属性是够只读。但是从ES5开始，所有的属性都具备了属性描述符。
思考下面的代码：
```
var myObject = {
  a: 2
}
Object.getOwnPropertyDescriptor('myObject', 'a')
// {
//   value: 2,
//   writable: true,
//   enumerable: true,
//   configurable: true
// }
```
这个普通的对象属性对应的属性描述符还包含另外三个特性：writable(可写)、enumerable(可枚举)、configurable(可配置)。
在创建普通属性时属性描述符会使用默认值，我们也可以使用Object.defineProperty(..)来添加一个新属性或者修改一个已有属性并对特性进行设置。
比如：
```
var myObject = {}
Object.defineProperty('myObject', 'a', {
  value: 2,
  writable: true,
  enumerable: true,
  configurable: true
})
```
即使属性是configurable，我们还是可以把writable的状态由true改为false，但是无法由false改为true
把enumerable设置成false可以保证属性不会出现在对象的属性枚举中。

### 不变性

- 对象常量
```
var myObject = {}
Object.defineProperty(myObject, 'x', {
  value: 42,
  writable: false,
  configurable: false
})
```

- 禁止扩展
可以用Object.preventExtebsions(..)禁止一个对象添加新属性
```
var myObject = {
  a: 2
}
Object.preventExtensions(myObject)
myObject.b = 3
myObject.b // undefined 严格模式下就会抛出TypeError错误
```

- 密封
Object.seal(..)会创建一个密封的对象，这个方法实际上会在一个现有对象上调用Object.preventExtensions(..)并把所有属性标记为configurable: false

- 冻结
Object.freeze(..)会创建一个冻结对象，这个方法实际上会在一个现有对象上调用Object.seal(..)并把所有“数据访问”属性标记为writable: false，这样就无法修改他们的值。

### [[Get]]

```
var myObject = {
  a: 2
}
myObject.a = 2 // 2
```
myObject.a是一次属性访问，但是这条语句并不仅仅是在myObject中查找名字为a的属性。实际上实现了[[Get]]操作（有点像函数调用：```[[Get]]()```）
