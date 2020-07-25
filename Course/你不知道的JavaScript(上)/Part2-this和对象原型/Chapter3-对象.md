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

### 数组