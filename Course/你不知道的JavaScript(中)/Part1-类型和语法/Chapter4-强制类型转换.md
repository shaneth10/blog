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