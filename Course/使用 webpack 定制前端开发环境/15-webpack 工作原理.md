# webpack 工作原理

了解 webpack 实现原理，掌握 webpack 基础的工作流程，在平时使用 webpack 遇见问题时，能够帮助我们洞察问题的根本所在，能够帮助我们理清解决问题的基本思路，同时也有助于我们更好地理解 loader 和 plugin 的使用和意义，在处理一些定制化的构建需求时更加得心应手。

抛开 webpack 复杂的 loader 和 plugin 机制，webpack 本质上就是一个 JS 模块 Bundler，用于将多个代码模块进行打包，所以我们先撇开 webpack 错综复杂的整体实现，来看一下一个相对简单的 JS 模块 Bunlder 的基础工作流程是怎么样的，在了解了 bundler 如何工作的基础上，再进一步去整理 webpack 整个流程，将 loader 和 plugin 的机制弄明白。

## 合并代码

笔者记得比较久远之前，在 js 的模块解决方案出来之前，一些前端类库为了拆分代码文件，会编写简单的合并代码文件的工具，有序得将多个代码文件合并到一起成为最终的 js 文件。举个例子：

```
// A.js
function A() {}

// B.js
function B() {}

// 结果
function A() {}
function B() {
  // 这里可以用 A 方法
}
```

这样的合并脚本很简单，读者们可以尝试用 Node 写一个。这样简单合并的方法有很明显的缺点，在代码库越来越大的时候会变得难以维护：

- 文件合并时的顺序很难确定
- 代码文件内变量和方法命名容易冲突

为了解决上述的问题，js 的模块解决方案就出来了，发展到今日，webpack 就是前端社区里最受欢迎的的 JS 模块 bundler。下边我们看下 webpack 是如何解决上述这两个问题的。

## 模块化

第一个问题，模块文件合并到一起时位置的顺序，简单理解也就是模块代码的执行顺序。CommonJS 规范和 ES Module 规范定义的就是在模块中声明依赖的方式，我们在模块中写下这样的代码：

```
// entry.js
import { bar } from './bar.js'; // 依赖 ./bar.js 模块

// bar.js
const foo = require('./foo.js'); // 依赖 ./foo.js 模块
```

便是在声明当前模块代码（即是入口的 entry.js）要执行时，需要依赖于「bar.js」模块的执行，而「bar.js」这个模块则依赖于「foo.js」。bundler 需要从这个入口代码（第一段）中解析出依赖 bar.js，然后再读取 bar.js 这个代码文件，解析出依赖 foo.js 代码文件，继续解析其依赖，递归下去，直至没有更多的依赖模块，最终形成一颗模块依赖树。

依赖解析和管理便是 webpack 这个 bundler 很重要的一个工作。如果 foo.js 文件没有依赖其他的模块的话，那么这个简单例子的依赖树也就相对简单：`entry.js -> bar.js -> foo.js`，当然，日常开发中遇见的一般都是相当复杂的代码模块依赖关系。

如果放到合并文件的处理上，上述的「foo.js」和「bar.js」模块的代码需要放到我们入口代码的前边。但是 webpack 不是简单地按照依赖的顺序合并，而是采取了一种更加巧妙的方式，顺带解决了前边提到的文件合并的第二个问题。

## webpack 的打包

在已经解析出依赖关系的前提下，webpack 会利用 JavaScript Function 的特性提供一些代码来将各个模块整合到一起，即是将每一个模块包装成一个 JS Function，提供一个引用依赖模块的方法，如下面例子中的 `__webpack__require__`，这样做，既可以避免变量相互干扰，又能够有效控制执行顺序，简单的代码例子如下：

```
// 分别将各个依赖模块的代码用 modules 的方式组织起来打包成一个文件
// entry.js
modules['./entry.js'] = function() {
  const { bar } = __webpack__require__('./bar.js')
}

// bar.js
modules['./bar.js'] = function() {
  const foo = __webpack__require__('./foo.js')
};

// foo.js
modules['./foo.js'] = function() {
  // ...
}

// 已经执行的代码模块结果会保存在这里
const installedModules = {}

function __webpack__require__(id) {
  // ... 
  // 如果 installedModules 中有就直接获取
  // 没有的话从 modules 中获取 function 然后执行，将结果缓存在 installedModules 中然后返回结果
}
```

我们前边介绍 webpack 生成 js 代码体积大小优化时，介绍过有一个配置可以用于移除掉部分 function 代码，因为这种实现方式最大的缺点就是会增大生成的 js 代码体积，当 webpack 可以确定代码执行顺序，以及可以用唯一的模块 id 去调整模块内变量名防止冲突时，这些所谓的胶水代码也就没有必要存在了。

## webpack 的结构

webpack 需要强大的扩展性，尤其是插件实现这一块，webpack 利用了 tapable 这个库（其实也是 webpack 作者开发的库）来协助实现对于整个构建流程各个步骤的控制，最主要的功能就是用来添加各种各样的钩子方法（即 Hook）。

webpack 基于 tapable 定义了主要构建流程后，使用 tapable 这个库添加了各种各样的钩子方法来将 webpack 扩展至功能十分丰富，同时对外提供了相对强大的扩展性，即 plugin 的机制。

在这个基础上，我们来了解一下 webpack 工作的主要流程和其中几个重要的概念。

- Compiler，webpack 的运行入口，实例化时定义 webpack 构建主要流程，同时创建构建时使用的核心对象 compilation
- Compilation，由 Compiler 实例化，存储构建过程中各流程使用到的数据，用于控制这些数据的变化
- Chunk，即用于表示 chunk 的类，即构建流程中的主干，一般情况下一个入口会对应一个 chunk，对于构建时需要的 chunk 对象由 Compilation 创建后保存管理
- Module，用于表示代码模块的类，衍生出很多子类用于处理不同的情况，关于代码模块的所有信息都会存在 Module 实例中，例如 dependencies 记录代码模块的依赖等
- Parser，其中相对复杂的一个部分，基于 acorn 来分析 AST 语法树，解析出代码模块的依赖
- Dependency，解析时用于保存代码模块对应的依赖使用的对象
- Template，生成最终代码要使用到的代码模板，像上述提到的 function 代码就是用对应的 Template 来生成

> 官方对于 Compiler 和 Compilation 的定义是：
>
> compiler 对象代表了完整的 webpack 环境配置。这个对象在启动 webpack 时被一次性建立，并配置好所有可操作的设置，包括 options，loader 和 plugin。当在 webpack 环境中应用一个插件时，插件将收到此 compiler 对象的引用。可以使用它来访问 webpack 的主环境。
>
> compilation 对象代表了一次资源版本构建。当运行 webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。一个 compilation 对象表现了当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。compilation 对象也提供了很多关键步骤的回调，以供插件做自定义处理时选择使用。

上述是 webpack 源码实现中比较重要的几个部分，webpack 运行的大概工作流程是这样的：

```
创建 Compiler -> 
调用 compiler.run 开始构建 ->
创建 Compilation -> 
基于配置开始创建 Chunk -> 
使用 Parser 从 Chunk 开始解析依赖 -> 
使用 Module 和 Dependency 管理代码模块相互关系 -> 
使用 Template 基于 Compilation 的数据生成结果代码 ->
```

## webpack 的源码

webpack 主要的构建处理方法都在 Compilation 中，我们要了解 loader 和 plugin 的机制，就要深入 Compilation 这一部分的内容。

Compilation 的实现也是比较复杂的，lib/Compilation.js 单个文件代码就有近 2000 行之多，我们挑关键的几个部分来介绍一下。