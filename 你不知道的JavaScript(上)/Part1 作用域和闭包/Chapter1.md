# 作用域和闭包

## 作用域是什么

- 引擎
- 编译器
  > 分词/词法分析、解析/语法分析、代码生成
- 作用域

变量的赋值操作会执行两个动作，首先编译器会在当前作用域中声明一个变量（如果之前没有声明过），然后在运行时引擎会在作用域中查找该变量，如果能够找到就会对它赋值。

## 词法作用域

```
eval(...) // 可以接受一个字符串参数，并视为在程序中这个位置的代码
with(obj) {} // 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域
```
> eval(...)和with会在运行时修改或创建新的作用域，以此来欺骗其他在书写时定义的词法作用域，在严格模式下会受限制，性能也会下降，所以尽量不要使用。

## 函数作用域和块作用域

- 立即执行函数表达式

```
var a = 2

(function foo(){
  var a = 3
  console.log(a) // 3
})()

console.log(a) // 2
```

> 区分函数声明和表达式最简单的方法是看function关键字出现在声明中的位置。如果function是声明中的第一个词，那么就是一个函数声明，否则就是一个函数表达式。

- 块级作用域 (with、try/catch、let)

```
{
  let j
  for (j=0; j<10; j++) {
    let i = j
    console.log(i)
  }
}
```
> for循环头部的let不仅将i绑定到了for循环的块中，事实上他将其重新绑定到了循环的每一个迭代中，确保使用上一个循环迭代结束时的值重新进行赋值。

## 提升

下面看两个例子，看输出结果是多少
- 案例1
```
a = 2
var a
console.log(a) // 2
```
- 案例2
```
console.log(a) // undefined
var a = 2
```

> 正确的思路是，包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。
当你看到var a = 2时，可能会认为这是一个声明。但JavaScript实际上会将其看成两个声明：var a和a = 2。第一个定义声明是在编一阶段进行的。第二个赋值声明会被留在原地等待执行阶段。

## 作用域闭包

```
function foo() {
  var a = 2

  function bar() {
    console.log(a)
  }

  return bar
}

var baz = foo()

baz() // 2
```

> 函数bar()的词法作用域能够访问foo()的内部作用域。然后我们将bar()函数本身当做一个值类型进行传递。
拜bar()所声明的位置所赐，它拥有涵盖foo()内部作用域的闭包，使得该作用域能够一直存活，以供bar()在之后任何时间进行引用。
bar()依然持有对该作用域的引用，而这个引用就叫做闭包。

- 重返作用域
```
for (var i=1; i<=5; i++) {
  let j = i // 闭包的块作用域
  setTimeout( function timer() {
    console.log(i)
  }, i*1000)
}
```
> 块作用域和闭包共同使用

- 模块

```
function CoolModule() {
  var something = 'cool'
  var another = [1, 2, 3]

  function doSomething() {
    console.log(something)
  }

  function doAnother() {
    console.log(another.join('!'))
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  }
}

var foo = CoolModule

foo.doSomething() // cool
foo.doAnother() // 1!2!3
```

> 这个模式被称为模块，最常见的实现模块模式的方法通常被称为模块暴露。模块模式需要具备两个必要条件
1、必须有外部的封闭函数，该函数必须至少被调用一次。
2、封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。