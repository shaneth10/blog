# 行为委托

如果在第一个对象上没有找到需要的属性或者方法引用，引擎就会继续在 Prototype 关联的对象上进行查找。同理，如果在后者中也没有找到需要额引用就会继续查找它的 Prototype ，以此类推。这一系列对象的链接被称为”原型链“。

换句话说，JavaScript 中这个机制的本质就是对象之间的关联关系。

## 面向委托的设计

为了更好地学习如何更直观地使用Prototype，我们必须认识到它代表的是一种不同于类的设计模式。

### 委托理论
```
Task = {
  setID: function(ID) { this.id = ID },
  outputID: function() { console.log( this.id )}
}

// 让 XYZ 委托 Task
XYZ = Object.create( Task )

XYZ.prepareTask = function(ID, Label) {
  this.setID( ID )
  this.label = label
}

XYZ.outputTaskDetails = function() {
  this.outputID()
  console.log( this.label )
}

// ABC = Object.create( Task )
// ABC ... = ...
```
Task 和 XYZ 并不是类，它们是对象。 XYZ 通过 Object.create(..) 创建，它的 Prototype 委托了 Task 对象。

对象关联风格的代码还有一些不同之处：
1、通常来说，在 Prototype 委托中最好把状态保存在委托者（XYZ、ABC）而不是委托目标（Task）上。
2、尽量少使用容易被重写的通用方法名，提倡使用更有描述性的方法名。
3、XYZ中的方法首先会寻找 XYZ 自身是否有 setID(...)，但是 XYZ 中并没有这个方法名，因此会通过 Prototype 委托关联到 Task继续寻找。
 