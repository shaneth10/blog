# 原型

## [[Prototype]]

JavaScript中的对象有一个特殊的Prototype内置属性，其实就是对于其他对象的引用。几乎所有的对象在创建时Prototype属性都会被赋予一个非空的值。
> 注意：对象的Prototype可以为空，但是是很少见的。
```
var myObject = {
  a: 2
}
myObject.a // 2
```
如果a不在myObject中，就需要使用对象的Prototype链了。对于默认的get操作，如果无法在对象本身找到需要的属性，就会继续访问对象的Prototype链。

### Object.prototype

所有普通的Prototype链最终都会指向内置的Object.prototype。由于所有的普通对象都源于这个Object.prototype对象，所以它包含JavaScript中许多通用的功能，比如.toString()和.value()

### 属性设置和屏蔽

```myObject.foo = "bar"```
- 如果对象中包含名为foo的普通数据访问属性，这条赋值语句只会修改已有的属性值。
- 如果foo不是直接存在于对象中，Prototype链就会被遍历。如果原型链上找不到foo，foo就会被添加到该对象上。
- 如果属性名foo及出现在对象中也出现在对象的Prototype链上层，myObject中包含的foo属性会屏蔽原型链上层的所有foo属性，因为这种情况下，总是会选择最底层的属性。
- 如果foo存在于原型链上，则这个行为就会有些不同。一共有三种情况：1、没有被标记为只读（writable:false），那就会直接在myObject中添加一个名为foo的新属性，它是屏蔽属性。2、如果被标记为只读，不会发生屏蔽，这条语句会被忽略，而运行在严格模式下，代码会抛出一个错误。3、如果是一个seeter，那么就一定会调用这个setter。不会添加，也不会重新定义foo这个setter。
```
var anotherObject = {
  a: 2
};
var myObject = Object.create(anotherObject);
anotherObject.a; // 2
myObject.a; // 2
anotherObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("a"); // false
myObject.a++; // 隐式屏蔽!
anotherObject.a; // 2
myObject.a; // 3
myObject.hasOwnProperty("a"); // true
```
myObject.a++分解为myObject.a=myObject.a+1

## “类”

JavaScript中只有对象，根本不存在类。

### “类”函数
“类”函数利用了函数的一种特殊特性：所有的函数默认都会拥有一个名为prototype的公有并且不可枚举的属性，它会指向另一个对象：
```
function Foo() {
  // ...
}

Foo.prototype // []
```
这个对象通常被称为Foo的原型，因为我们通过名为Foo.prototype的属性引用来访问它。
最直接的解释就是，这个对象是在调用new Foo()时创建的，最后会被关联到这个Foo.prototype对象上。
```
function Foo() {
  // ...
}

var a = new Foo()

Object.getPrototypeOf( a ) === Foo.prototype // true
```
调用new Foo()时会创建a，其中一步就是将a内部的Prototype链接到Foo.prototype所指向的对象

### “构造函数”
函数本身并不是构造函数，然而，当你在普通的函数调用前面加上new关键字之后，就会把这个函数调用变成一个“构造函数调用”。实际上，new会劫持所有普通函数并用构造函数的形式来调用它。
```
function NothingSpecial() {
  console.log( "Don't mind me!" )
}
var a = new NothingSpecial()
// Don't mind me

a // {}
```
> NothingSpecial只是一个普通函数，但是使用new函数时，它就会构造一个对象并赋值给a，这看起来就像是new的一个副作用。这个调用是一个构造函数调用，但是NothingSpecial本身并不是一个构造函数。
> 换句话说，在JavaScript中对于“构造函数”最准确的解释是，所有带new的函数调用。
> 函数不是构造函数，但是当且仅当使用new时，函数调用会变成“构造函数调用”。

### 技术
Foo.prototype的.constructor属性只是Foo函数在声明时的默认属性。如果你创建了一个新对象并替换了默认的.prototype对象引用，那么新对象并不会自动获得.constructor属性。
```
function Foo() {}
Foo.prototype = {} //创建一个新原型对象
var a1 = new Foo()
a1.constructor === Foo // false
a1.constructor === Object // true
```
> a1并没有.constructor属性，所以它会委托prototype链上的Foo.prototype。但是这个对象也没有该属性，所以它会继续委托，这次会委托给委托链的顶端Object.prototype。这个对象有.constructor属性，指向内置的Object(...)函数
> .constructor并不是一个不可变属性。它是不可枚举的，但是它的值是可写的。此外，你可以给任意Prototype链中的人以对象添加一个名为constructor的属性或者对其进行修改，你可以任意对其赋值。.constructor是一个非常不可靠并且不安全的引用。

## （原型）继承

```
function Foo(name) {
  this.name = name
}

Foo.prototype.myName = function() {
  return this.name
}

function Bar(name, label) {
  Foo.call(this, name)
  this.label = label
}

// 我们创建了一个新的Bar.prototype对象并关联到Foo.prototype
Bar.prototype = Object.create(Foo.prototype)

// 注意！现在没有Bar.prototype.constructor了
Bar.prototype.myLabel = function() {
  return this.label
}

var a = new Bar('a', 'obj a')

a.myName() // 'a'
a.myLabel // 'obj a'
```
这段代码的核心部分就是语句Bar.prototype = Object.create(Foo.prototype)。调用Object.create(...)回凭空创建一个“新”对象并把新对象内部的Prototype关联到指定的对象。

注意，下面两种方式是常见的错误做法，实际上它们都存在一些问题：
```
// 喝你想要的机制不一样！
Bar.prototype = Foo.prototype

// 基本上满足你的需求，但是可能会产生一些副作用：
Bar.prototype = new Foo()
```
```Bar.prototype = Foo.prototype```并不会创建一个关联到Bar.prototype的新对象，它只是让Bar.prototype直接饮用Foo.prototype对象。所以，可以直接使用Foo就可以了，这样代码也会更简单一些。
```Bar.prototype = new Foo()```的确会创建一个关联到Bar.prototype的新对象。但是它使用了Foo(...)的“构造函数调用”，如果函数Foo有一些副作用的话，就会影响到Bar()的后代。
因此。要创建一个合适的关联对象，我们必须创建使用Object.create(...)而不是使用具有副作用的Foo(...)。ES6添加了辅助函数Object.setPrototypeOf(...)，可以用标准并且可靠的方法来修改关联。
我们来对比一下两种关联方法：
```
// ES6之前需要抛弃默认的Bar.prototype
Bar.prototype = Object.create(Foo.prototype)

// ES6开始可以直接修改现有的Bar.prototype
Object.setPrototypeOf(Bar.prototype, Foo.prototype)       
```

### 检查“类”关系

```Foo.prototype.isPrototypeOf(a) // true```
isPrototypeOf(..)回答的问题是：在a的整条Prototype链中是否出现过Foo.prototype。
我们也可以直接获取一个对象的Prototype链。在Es5中，标准的方法是：
```Object.getPrototypeOf(a)```
可以验证一下，这个对象引用是否和我们想的一样：
```Object.getPrototypeOf(a) === Foo.prototype // true```
绝大多数浏览器也支持一种非标准的方法来访问内部Prototype属性：
```a.__proto__ === Foo.prototype // true```
这个奇怪的.__proto__属性引用了内部的Prototype对象，如果你想查找原型链的话，这个方法非常有用。

此外，.__proto__看起来很像一个属性，但是实际上它更像一个getter/setter。.__proto__的实现大致上是这样的：
```
Object.defineProperty(Object.prototype, "__proto__", {
  get: function() {
    return Object.getPropertyOf(this)
  },
  set: function(o) {
    // ES6中的setPrototypeOf(..)
    Object.setPrototypeOf(this, o)
    return o
  }
})
```
因此，访问a.__proto__时，实际上是调用了a.__proto__()(调用getter函数)。虽然getter函数存在于Object.prototype对象中，但是它的this指向对象a，所以和Object.getPrototypeOf(a)结果相同。

## 对象关联

Prototype机制就是存在于对象中的一个内部链接，它会引用其他对象。通常来说，这个链接的作用是：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在Prototype关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的Prototype，以此类推。这一系列对象的链接被称为“原型链”。

### 创建关联
```
var foo = {
  something: function() {
    console.log("Tell me something good...")
  }
}

var bar = Object.create(foo)

bar.somrthing() // Tell me something good
```
Object.create(..)会创建一个新对象（bar）并把它关联到我们指定的对象（foo）。Object.create(null)会创建一个拥有空连接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以instanceof操作符无法进行判断，总是会返回false。这些对象通常被称作“字典”，它们完全不会受到原型链的干扰，因此非常适合用来存储数据。

**Object.create()的polyfill代码**
Object.create()实在ES5中新增的函数，所以在ES5之前的环境中如果要支持这个功能的话就需要polifill代码，部分实现Object.create()的功能：
if (!Object.create) {
  Object.create = function(o) {
    function F() {}
    F.prototype = o
    return new F()
  }
}
这段polyfill代码使用了一个一次性函数F，我们通过改写它的.prototype属性使其指向想要关联的对象，然后再使用new F()来构造一个新对象进行关联。

### 关联关系是备用
```
var anotherObject = {
  cool: function() {
    console.log("cool!")
  }
}

var myObject = Object.create(anotherObject)
myObject.cool() // "cool!"
```
这段代码是可以正常运行的，但是看起来有点难以理解和维护，我们再看下面一段代码：
```
var anotherObject = {
  cool: function() {
    console.log("cool!")
  }
}

var myObject = Object.create(anotherObject)

myObject.doCool = function() {
  this.cool() // 内部委托
}

myObject.doCool() // ”cool！“
```
这里我们调用的 myObject.doCool() 是实际存在于 myObject 中的，使我们的API设计更加清晰。从内部来说，我们的实现遵循的是委托设计模式，通过Prototype委托到anotherObject.cool()。
