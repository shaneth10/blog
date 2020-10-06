# 什么是 TypeScript

TypeScript 是 JavaScript 的一个超集，主要提供了**类型系统**和**对 ES6 的支持**。
- TypeScript 增加了代码的可读性和可维护性
- TypeScript 非常包容
- TypeScript 拥有活跃的社区

此外，TypeScript 的弊端在于：
- 有一点的学习成本，需要理解接口Interfaces、泛型Generics、类Classes、枚举类型Enums等概念
- 短期可能会增加一些开发成本，毕竟要多写一些类型的定义，不过对于一个需要长期维护的项目，TypeScript 能够减少其维护成本
- 集成到构建流程需要一些工作量
- 可能和一些库结合的不是很完美

## 安装 TypeScript

```
npm install -g typescript
```
如果在 mac 上安装有报错，大概率是没有管理员权限，在最前面加上 sudo 再次运行命令即可。
```
tsc hello.ts // 运行ts文件
```

## Hello TypeScript
```
function sayHello(person: string) {
    return 'Hello, ' + person;
}

let user = 'Tom';
console.log(sayHello(user));
```
在 TypeScript 中，使用```:```指定变量的类型，前后有没有空格都可以。我们用```:```指定 person 参数类型为 string，TypeScript 只会进行静态检查，如果发现有错误，编译的时候就会报错。