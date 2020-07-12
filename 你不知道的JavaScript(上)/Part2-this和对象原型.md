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
- 这三个函数的第一个参数都是this的指向对象，第二个参数就有了差别，call的参数是直接放在括号内用逗号隔开的，apply是放在数组中再传入的，而bind除了返回是函数以外，它的参数和call是一样的