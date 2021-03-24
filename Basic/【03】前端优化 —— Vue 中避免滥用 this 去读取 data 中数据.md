# 前端优化 —— Vue 中避免滥用 this 去读取 data 中数据

在 Vue 中， data 选项是个好东西，把数据往里一丢，在一个 Vue 组件中任何一个地方都可以通过this 来读取 data 中数据。但是要避免滥用 this 去读取 data 中数据，至于在哪里要避免滥用，如果滥用会导致什么后果？

## 一、用 this 读取 data 中数据的过程

在 Vue 源码中会把 data 中数据添加 getter 函数和 setter 函数，将其转成响应式的。getter 函数代码如下所示：

```
function reactiveGetter() {
    var value = getter ? getter.call(obj) : val;
    if (Dep.target) {
        dep.depend();
        if (childOb) {
            childOb.dep.depend();
            if (Array.isArray(value)) {
                dependArray(value);
            }
        }
    }
    return value
}
```

用 this 读取 data 中数据时，会触发 getter 函数，在其中通过 ```var value = getter ? getter.call(obj) : val;``` 获取到值后执行 ```return value``` ，实现读取数据的目的。

这其中有一段逻辑代码，只有在 Dep.target 存在的时候才会去执行，一系列的 depend() 其实就是在建立依赖关系，这边就不详细介绍了。

这样其实就能看出一些端倪了，如果是在 Dep.target 存在的时候滥用 this ，就会导致重复收集依赖，从而产生资源上的浪费。那么，Dep.target 什么时候存在呢？

## 二、Dep.target 什么时候存在

Dep.target 是由依赖赋值的。依赖又称为 Watcher （侦听者）或者订阅者。在 Vue 中有三种依赖，其中两种是很常见的，就是 watch （侦听器）和 computed （计算属性）。还有一种隐藏的依赖———渲染 Watcher，在模板首次渲染的过程中创建的。

Dep.target 是在依赖创建时被赋值，依赖是用构造函数 Watcher 创建。

```
function Watcher(vm, expOrFn, cb, options, isRenderWatcher) {
    //...
    if (typeof expOrFn === 'function') {
        this.getter = expOrFn;
    } else {
        this.getter = parsePath(expOrFn);
    }
    this.value = this.lazy ? undefined : this.get();
};
Watcher.prototype.get = function get() {
    pushTarget(this);
    try {
        value = this.getter.call(vm, vm);
    } catch (e) {
        
    }
    return value
};
Dep.target = null;
var targetStack = [];
function pushTarget(target) {
    targetStack.push(target);
    Dep.target = target;
}
```

在构造函数Watcher最后会执行实例方法```get```，在实例方法```get```中执行```pushTarget(this)```中给```Dep.target```赋值的。

**而依赖是在Vue页面或组件初次渲染时创建，所以产生的性能问题应该是首次渲染过慢的问题。**

## 三、在何处滥用 this 去读取 data 中数据

在 Dep.target 存在时去执行这些滥用 this 去读取 data 中数据的代码会产生性能问题，故还要搞清楚这些代码是写在哪里才会被执行到，换句话来说，要搞清楚在哪里滥用 this 去读取 data 中数据会产生性能问题。

在第二小节中介绍了 Dep.target 被赋值后会执行 ```value = this.getter.call(vm, vm)``` ，其中 ```this.getter``` 是一个函数，那么若在其中有用 this 去读取 data 数据，就会去收集依赖，假如滥用的话就会产生性能问题。

```this.getter``` 是在创建依赖过程中赋值的，每种依赖的 ```this.getter``` 都是不相同的。下面来一一介绍。

- watch（侦听器）依赖的 ```this.getter``` 是 ```parsePath``` 函数，其函数参数就是侦听的对象。

```
var bailRE = new RegExp(("[^" + (unicodeRegExp.source) + ".$_\\d]"));
function parsePath(path) {
    if (bailRE.test(path)) {
        return
    }
    var segments = path.split('.');
    return function(obj) {
        for (var i = 0; i < segments.length; i++) {
            if (!obj) {
                return
            }
            obj = obj[segments[i]];
        }
        return obj
    }
}
```

如下所示的代码中的 ```a``` 和 ```a.b.c``` 作为参数传入 parsePath 函数会返回一个函数赋值给 ```this.getter``` ，执行 ```this.getter.call(vm, vm)``` 会得到 ```this.a``` 和 ```this.a.b.c``` 的值。**在这个过程中不会存在滥用 this 去读取 data 中数据的场景。**

```
watch:{
  a:function(newVal, oldVal){
    //做点什么
  }
}
vm.$watch('a.b.c', function (newVal, oldVal) {
  // 做点什么
})
```

- computed（计算属性）依赖的 ```this.getter``` 有两种，如果计算属性的值是个函数，那么  ```this.getter``` 就是这个函数。如果计算属性的值是个对象，那么 ```this.getter``` 就是这个对象的 get 属性值， get 属性值也是个函数。在这个函数可能会存在滥用 this 去读取 data 中数据的场景，举个例子，代码如下所示。

```
computed:{
    d:function(){
        let result = 0;
        for(let key in this.a){
            if(this.a[key].num > 20){
                result += this.a[key].num + this.b + this.c;
            }else{
                result += this.a[key].num + this.e + this.f;
            }
        }
        return result;
    }
}
```

在计算属性 ```d``` 中就存在滥用 this 去读取 data 数据。其中 ```this.a``` 是个数组，此时 Dep.target 的值为计算属性 ```d``` 这个依赖，在循环 ```this.a``` 中使用 this 去获取中 ```a``` 、```b``` 、 ```c``` 、```e``` 、 ```f``` 的数据，使这些数据进行一系列复杂的逻辑运算来重复地收集计算属性 ```d``` 这个依赖。导致获取计算属性 ```d``` 的值的速度变慢，从而产生性能问题。

- 渲染 Watcher 的 ```this.getter``` 是一个函数如下所示：

```
updateComponent = function() {
  vm._update(vm._render(), hydrating);
};
```

其中 ```vm._render()``` 会把 template 模板生成的渲染函数 render 转成虚拟 DOM（VNode） ： ```vnode = render.call(vm._renderProxy, vm.$createElement);```，举一个例子来说明一下渲染函数 render 是什么。

例如template模板：

```
<template>
  <div class="wrap">
    <p>{{a}}<span>{{b}}</span></p>
  </div>
</template>
```

通过 vue-loader 会生成渲染函数 render ，如下所示：

```
(function anonymous() {
    with(this) {
        return _c('div', {
            attrs: {
                "class": "wrap"
            }
        }, [_c('p', [_v(_s(a)), _c('span', [_v(_s(b))])])])
    }
})
```

其中 ```with``` 语句的作用是为一个或一组语句指定默认对象，例 ```with(this){ a + b }``` 等同 ```this.a + this.b```，那么在 template 模板中使用 ```{{ a }}``` 相当使用 this 去读取 data 中的 ```a``` 数据。故在 template 模板生成的渲染函数 render 中也可能存在滥用 this 去读取 data 中数据的场景。举个例子，代码如下所示：

```
<template>
  <div class="wrap">
    <div v-for=item in list>
      <div> {{ arr[item.index]['name'] }} </div>
      <div> {{ obj[item.id]['age'] }} </div>
    </div>
  </div>
</template>
```

其中用 ```v-for``` 循环 ```list``` 数组过程中，不断用 this 去读取 data 中 ```arr```、```obj``` 的数据，使这些数据进行一系列复杂的逻辑运算来重复收集这个依赖，导致初次渲染的速度变慢，从而产生性能问题。

## 四、如何避免滥用this去读取data中数据

综上所述在计算属性和 template 模板中滥用 this 去读取 data 中数据会导致多次重复地收集依赖，从而产生性能问题，那要怎么避免这种情况。

- 计算属性中如何避免

用 ES6 对象解构赋值来避免，计算属性的值是一个函数，其参数是 Vue 的实例化 ```this``` 对象，在上述计算属性中滥用 this 的例子中可以这样优化。

**优化前：**

```
computed:{
    d:function(){
        let result = 0;
        for(let key in this.a){
            if(this.a[key].num > 20){
                result += this.a[key].num + this.b + this.c;
            }else{
                result += this.a[key].num + this.e + this.f;
            }
        }
        return result;
    }
}
```

**优化后：**

```
computed: {
  d({ a, b, c, e, f }) {
    let result = 0;
    for (let key in a) {
      if (a[key].num > 20) {
        result += a[key].num + b + c;
      } else {
        result += a[key].num + e + f;
      }
    }
    return result;
  }
}
```

以上利用解构赋值提前把 data 数据中的 ```a```、```b```、```c```、```e```、```f``` 赋值给对应的变量```a```、```b```、```c```、```e```、```f```，然后在计算属性中可以通过这些变量访问 data 数据的，且不会触发 data 中对应数据的依赖收集。这样只用 ```this``` 读取了一次 data 中的数据，只触发了一次依赖收集，避免了多次重复地依赖收集产生的性能问题。

- template模板中如何避免

提前处理 ```v-for``` 循环所用的数据，不要在 ```v-for``` 循环中去读取数组、对象类型的数据。在上述 template 模板中滥用 this 的例子中可以这样优化。

假设 ```list```、```arr```、```obj``` 皆是服务端返回来的数据，且 ```arr``` 和 ```obj``` 没有用到任何模块渲染中，可以这样优化。

**优化前：**

```
<template>
  <div class="wrap">
    <div v-for=item in list>
      <div> {{ arr[item.index]['name'] }} </div>
      <div> {{ obj[item.id]['age'] }} </div>
    </div>
  </div>
</template>
```

**优化后：**

```
<template>
  <div class="wrap">
    <div v-for=item in listData>
      <div{{item.name}} </div>
        <div>{{item.age}}</div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      list: [],
    }
  },
  created(){
    // 在这里定义arr和obj避免被转成响应式
    this.arr = [];
    this.obj = {};
  },
  computed: {
    listData: function ({list}) {
      list.forEach(item => {
        item.name = this.arr[item.index].name;
        item.age = this.obj[item.id].age;
      })
      return list;
    }
  },
}
</script>
```