# this和对象原型

this关键词是JavaScript中最复杂的机制之一

## 关于this

- 改变this的指向 学习call()、apply()和bind()

```
let name = '小王', age = 17
let db = {
  name: '德玛',
  age: 35
}
let obj = {
  name: '小张',
  objAge: this.age,
  myFun: function(fm, t) {
    console.log( this.name + '年龄' + this.age, '来自' + fm + '去往' + t)
  }
}

obj.myFun.call(db, '成都', '上海') // 德玛 年龄 99  来自 成都去往上海
obj.myFun.apply(db, ['成都', '上海']) // 德玛 年龄 99  来自 成都去往上海
obj.myFun.bind(db, '成都', '上海')() // 德玛 年龄 99  来自 成都去往上海
obj.myFun.bind(db, ['成都', '上海'])() // 德玛 年龄 99  来自 成都, 上海去往 undefined
```

从以上结果可以看出几个函数的相同与不同：
> - 以上除了bind方法后面多了个()外，结果返回都一致，所以bind返回的是函数，必须调用它才会被执行
> - 这三个函数的第一个参数都是this的指向对象，第二个参数就有了差别，call的参数是直接放在括号内用逗号隔开的，apply是放在数组中再传入的，而bind除了返回是函数以外，它的参数和call是一样的

## this全面解析

### 调用位置
首先要理解调用位置：调用位置就是函数在代码中被调用的位置，然而并不那么简单，因为某些编程模式可能会隐藏真正的调用位置。所以，最重要的是要分析调用栈，因为它决定了this的绑定。

```
function baz() {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域

  console.log('baz')
  bar() // <-- bar的调用位置
}

function bar() {
  // 当前调用栈是baz -> bar
  // 因此，当前调用位置在baz中

  console.log('bar')
  foo() // <-- foo的调用位置
}

function foo() {
  // 当前调用栈是baz -> bar -> foo
  // 因此，当前调用位置在bar中

  console.log('foo')
}

baz() // <-- baz的调用位置
```

### 绑定规则

找到调用位置，判断根据优先级，需要应用下面四条规则中的哪一条。

- 默认绑定

> 最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则。

```
function foo() {
  console.log( this.a )
}

var a = 2
foo()
```
> 在代码中，foo()是直接使用不带任何修饰的函数引用进行调用的，由于this指向全局对象，所以函数在调用时应用了this的默认绑定。

如果使用严格模式，则不能将全局对象用于默认绑定，因此this会绑定到undefined。通常情况下，你不应该在代码中混用严格和非严格模式。然而，有时候你可能会用到第三方库，其严格程度和你的代码有所不同，因此要注意这类兼容性细节。这里给出一种在严格模式下调用foo()不影响默认绑定的方法：
```
function foo() {
  cosnole.log( this.a )
}

var a = 2

(function() {
  "use strict"

  foo()
})()
```

- 隐式绑定

```
function foo() {
  console.log( this.a )
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo() // 2
```
> 当foo()被调用时，他的前面加上了对obj的引用。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。因为调用foo()时this被绑定到obj，因此this.a和obj.a是一样的。

对象属性引用链中只有上一层或者说最后一层在调用位置中起作用。比如：
```
function foo() {
  console.log( this.a )
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo() // 42
```

```
function foo() {
  console.log( this.a )
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo // 函数别名
var a = 'oops, global' // a是全局对象属性
bar() // 'oops, global'
```
> 虽然bar是obj.foo的一个引用，但是实际上，它引用的是foo函数本身,因此此时的bar()其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

- 显式绑定

#### call(...)&apply(...)
```
var bar = function() {
  foo.call(obj)
}
```
> 在bar函数内部调用foo.call(obj)强制把foo的this绑定到了obj。这种绑定是一种显式的强制绑定，我们称之为硬绑定。

#### API调用的“上下文”

```
function foo(el) {
  console.log(el, this.id)
}

var obj = {
  id: 'awesome'
}

[1,2,3].forEach(foo,obj) //1 awesome 2 awesome 3 awesome
```

- new绑定

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。
1、创建（或者说构造）一个全新的对象；
2、这个新对象会被执行Prototype链接；
3、这个新对象会绑定到函数调用的this；
4、如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
```
function foo(a) {
  this.a = a
}
var bar = new foo(2)
console.log(bar.a) // 2
```
> 使用new来调用foo(...)时，我们会构造一个新对象并把它绑定到foo(...)调用中的this上。new是最后一种可以影响函数调用的this绑定行为的方法，我们称之为new绑定。

### 优先级

毫无疑问，默认绑定的优先级是四条规则中最低的，我们可以先不考虑它。

- 隐式绑定和显示绑定
```
function foo() {
  console.log(this.a)
}

var obj1 = {
  a: 2,
  foo: foo
}

var obj2 = {
  a: 3,
  foo: foo
}

obj1.foo() // 2
obj2.foo() // 3

obj1.foo.call(obj2) // 3
obj2.foo.call(obj1) // 2
```
可以看到，显示绑定优先级更高。

- new绑定和隐式绑定
```
function foo(something) {
  this.a = something
}
var obj1 = {
  foo: foo
}
var bar = new obj1.foo(4)
console.log(obj1.a) // 2
console.log(bar.a) // 4
```
可以看到，new绑定比隐式绑定优先级高

- new绑定和显示绑定 

new绑定优先级大于显示绑定

> 所以四种绑定方式的优先级为new绑定 > 显示绑定 > 隐式绑定 > 默认绑定
我们可以通过优先级来判断函数应用的是哪条规则：
1、函数在new中调用（new绑定）？如果是 this绑定的就是新创建的对象 ```var bar = new foo()```
2、函数通过call、apply（显示绑定）或者硬绑定调用？是的话 this绑定的是指定对象 ```var bar = foo.call(obj2)```
3、函数在某个上下文对象中调用（隐式绑定）？如果是 this绑定的是那个上下文对象```var bar = obj1.foo()```
4、如果都不是，使用默认绑定，在严格模式下，就是绑定到undefined，否则绑定到全局对象```var bar = foo()```

### 绑定例外

如果你把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值再被调用时会被忽略，实际应用的是默认绑定规则。
然而总是使用null来忽略this绑定可能产生一些副作用。如果某个函数确实使用了this(第三方库中的一个函数)，那默认绑定规则会把this绑定到全局对象。
- 更安全的this
> 把this绑定到空对象上，就什么都不用担心了，因为任何对于this的使用都会被限制在这个空对象中，不会对全局对象产生影响。```var ø = Object.create(null)```这个创建空对象的方法，并不会创建Object.prototype这个委托，所以它比{}“更空”

### this词法
ES6中的箭头函数并不是通过function关键词定义的，而是使用被称为“胖箭头”的操作符=>定义的。箭头函数不使用this的四种标准规则，二十根据外层作用于来决定this的。
```
function foo() {
  return (a) => {
    console.log(this.a)
  }
}
var obj1 = {
  a: 2
}
var obj2 = {
  a: 3
}
var bar = foo.call(obj1)
bar.call(obj2) // 2,不是3
```
> foo()内部创建的箭头函数会捕获调用时foo()的this。由于foo()的this绑定到obj1，bar（引用箭头函数）的this也会绑定到obj1，箭头函数的绑定无法被修改。