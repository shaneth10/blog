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

在第二小节中介绍了 Dep.target 被赋值后会执行 value = this.getter.call(vm, vm) ，其中 this.getter 是一个函数，那么若在其中有用 this 去读取 data 数据，就会去收集依赖，假如滥用的话就会产生性能问题。

this.getter 是在创建依赖过程中赋值的，每种依赖的 this.getter 都是不相同的。下面来一一介绍。

- watch（侦听器）依赖的 this.getter 是 parsePath 函数，其函数参数就是侦听的对象。

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

如下所示的代码中的 a 和 a.b.c 作为参数传入 parsePath 函数会返回一个函数赋值给 this.getter ，执行 this.getter.call(vm, vm) 会得到 this.a 和 this.a.b.c 的值。**在这个过程中不会存在滥用 this 去读取 data 中数据的场景。**