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