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
myObject.a是一次属性访问，但是这条语句并不仅仅是在myObject中查找名字为a的属性。实际上实现了[[Get]]操作（有点像函数调用：```[[Get]]()```）。对象默认的内置[[Get]]操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值。然而，如果没有找到名称相同的属性，按照[[Get]]算法的定义就会执行遍历可能存在的[[Property]]链，即原型链。
  
### [[Put]]

[[Put]]被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性。如果已经存在这个属性，[[Put]]算法大致会检查下面这些内容。
1.属性是否是访问描述符？如果是并且存在setter就调用setter。
2.属性的数据描述符中writable是否是false？如果是，在严格模式下静默失败，在严格模式下抛出TypeError异常。
3.如果都不是，将该值设置为属性的值。

### Getter和Setter

在ES5中可以使用getter和setter部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。getter是一个隐藏函数，会在获取属性值时调用。setter也是一个隐藏函数，会在设置属性值时调用。
通常来说getter和setter是成对出现的：
```
var myObject = {
  // 给a定义一个getter
  get a() {
    return this._a_
  }

  // 给a定义一个setter
  set a(val) {
    this._a_ = val * 2
  }
}
myObject.a = 2
myObject.a // 4
```
在这个例子中，实际上我们把赋值操作中的值2存储到了另一个变量_a_中，而如果缺少setter/getter函数的话，赋值/获取值的操作就不会生效。

### 存在性

我们可以在不访问属性值的情况下判断对象中是否存在这个属性：
```
var myObject = {
  a: 2
}
("a" in myObject) // true
("b" in myObject) // false

myObject.hasOwnProperty("a") // true
myObject.hasOwnProperty("b") // false
```
in会检查属性是否在对象及其原型链中。而hasOwnProperty(..)只会检查属性是否在myObject对象中，不会检查原型链。
propertyIsEnumberable(..)会检查给定的属性名是否直接存在于对象中（而不是在原型链上）并且满足enumberable:true。Object.keys(..)会返回一个数组，包含所有可枚举属性，Object.getOwnPropertyNames(..)会返回一个数组，包含所有属性，无论他们是否可枚举。in和hasOwnProperty(..)的区别在于是否查找原型链，然而，Object.keys(..)和Object.getOwnPropertyNames(..)都只会查找对象直接包含的属性。

## 遍历

遍历数组下标时采用的是数字顺序（for循环或者其他迭代器），但是遍历对象属性时的顺序是不确定的，在不同的JavaScript引擎中可能不一样。因此，在不同的环境中需要保证一致性时，一定不要相信任何观察到的顺序，他们是不可靠的。

```
var myArray = [1,2,3]
var it = myArray[Symbol.iterator]()

it.next() // { value: 1, done: false }
it.next() // { value: 2, done: false }
it.next() // { value: 3, done: false }
it.next() // { done: true }
```
> 我们使用ES6中的符号Symbol.iterator来获取对象的@@iterator内部属性。引用类似iterator的特殊属性时要使用符号名，而不是符号包含的值。此外，虽然看起来很像一个对象，但是@@iterator本身并不是一个迭代器对象，而是一个返回迭代器对象的函数。
> 和数组不同，普通的对象没有内置的@@iterator，所以无法完成```for...of```遍历。


# 混合对象“类”

