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

###