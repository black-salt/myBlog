# 001-前端模块化

## 一、什么是模块化

ES6之前，JavaScript语言本身并没有一种模块机制来保证不同模块可以使用相同的变量名。

所以，当使用多个js文件时，会出现命名冲突等问题。

模块化是一种编程方式，一种能够解耦的解决方案。

一个模块就是一个实现特定功能的文件，有了模块，我们就可以更方便地复用别人家的代码，开发效率将大大提高。

但正是因为，我们的模块会提供给大家进行使用，所以我们需要遵循一定的规范才能够使得大家开发出来的模块都能够被正常地使用。否则我们每加载一个模块都需要去学习一下他自己定义的模块规范，也太难受了。

这就催生出了CommonJS, AMD, CMD等规范。

## 二、为什么要模块化

在模块化出现之前，我们的脚本长这样：

``` html
<script src="module1.js"></script>
<script src="module2.js"></script>
<script src="module3.js"></script>
<script src="module4.js"></script>
... ...
```

但我们引入多个js文件时，会存在一些问题：

* 命名空间：命名冲突
* 依赖：严格的依赖顺序
* 可维护性： `script` 太多，难以维护。
* 加载时间：网页加载响应时间过长
* ...

## 三、JavaScript 模块发展简史

> 本部分引用自 [Webpack 前端工程化入门](https://gitbook.cn/gitchat/column/59e065f64f7fbe555e479204)

让我们先来回顾一下 JavaScript 模块的历史是怎样演化的。

### 1. script 标签

最原始的模块实现方式是通过在页面中插入多个 script 标签。

``` html
<script src="//mycdn.com/moduleA.js"></script>
<script src="//mycdn.com/moduleB.js"></script>
<script src="//mycdn.com/main.js"></script>
```

这种方法的缺陷是所有这些脚本都是共用全局的作用域。在任何一个 JavaScript 文件中进行顶层作用域的变量或函数声明，都会暴露在全局中，使得其它脚本也相应获取。当应用的规模和复杂度上升，这些脚本之间很容易发生命名冲突，从而导致不可预知的问题。

### 2. 立即执行函数表达式

后来随着 Web 的发展，人们发现通过立即执行函数表达式（ IIFE ）可以解决命名空间的问题。IIFE 的实现原理是把代码包裹在一个函数中，并且在声明这个函数之后立即执行它，这样相当于为代码单独创建了一个作用域。比如在下面的代码中我们创建了一个立即执行函数表达式，变量的声明处于它自己的函数作用域内，与其它的模块作用域隔离开。

``` javascript
(function() {
    // foobar 并不会暴露在全局作用域
    var foobar = 'Hello IIFE!';
    console.log(foobar);
})()
```

如果通过这种方法封装的模块需要与别的模块发生交互，则可以将特定的对象绑定在全局来允许其它模块通过全局对象获取，比如下面的形式：

``` javascript
// calculator.js
(function() {
    // 将 calculator 绑定在全局对象上，使其它模块调用
    window.calculator = {
        add: function() {
            //...
        },
        sub: function() {
            //...
        },
        //...
    }
})()

// app.js
calculator.add(1, 2);
```

立即执行函数表达式的缺点在于，如果模块之间有着依赖关系，必须将它们按照特定的顺序引入到页面中。比如上面的 calculator，必须使定义 calculator 的 JavaScript 先执行，再执行调用的模块。由于这种依赖关系是隐式的，当所有这些模块都互相依赖时，文件的引入顺序将变得难以维护。

### 3. AMD

AMD 是 Asynchronous Module Definition（异步模块定义）的缩写。下面的代码使用 AMD 规范定义了一个模块：

``` javascript
// 定义一个求和的模块
define('getSum', ['math'], function(math) {
    return function(a, b) {
        console.log('sum: ' + math.sum(a, b));
    }
});
```

在 AMD 中定义模块是使用 `define` 函数，它可以接受三个参数。第一个参数是当前模块的 ID，相当于给这个模块起一个名字；第二个参数是当前模块的依赖，比如上面我们定义的 `getSum` 模块需要 `math` 模块的依赖；第三个参数可以是函数或者对象。如果是函数，可以利用函数的返回值将定义的模块接口导出；如果是对象，则代表它为当前模块的导出值。

通过这种形式定义模块的好处在于，它 **显式** 地表达出了每个模块所依赖的其它模块。并且模块定义也不再绑定到全局对象上，不必担心其在别的地方被篡改。

### 4. CommonJS 与 Node.js 模块系统

CommonJS 是于 2009 年提出的 JavaScript 规范，它最开始是为了定义服务端标准，而非用于浏览器环境。之后 Node.js 采用并实现了它的部分规范，在模块系统上进行了一些调整。一般来说，我们不会严格区分 CommonJS 与 Node.js 的模块标准，详述两种标准的区别超出了本文的范围，在下文中我会直接使用 CommonJS 来进行表述。

Browserify 的出现带来了浏览器环境模块的变革。它是一个运行在 Node 环境下的模块打包工具，可以把模块按照 Node.js 的模块规则合并为浏览器支持的形式，这使得浏览器端的框架类库也可以按照 CommonJS 的形式编写。随着 Node.js 以及 npm 流行，近两年来对于开发者来说遵循 CommonJS 标准来编写和使用模块已经成为了一个基本通识。

在 CommonJS 中每个文件是一个模块，并且拥有属于自己的作用域和上下文。模块的依赖通过 `require` 函数来引入。

``` javascript
const math = require('./math');
```

如果想把模块的接口暴露给外部，则要通过 `exports` 将其导出，如：

``` javascript
exports.getSum = function(a, b) {
    return a + b;
}
```

AMD 和 CommonJS 具有同样的特性——模块的依赖必须显式引入，这样就解决了之前维护复杂模块引入时的顺序问题。

### 5. ES6 Module

ES6 Module 是目前比较推荐开发者使用的模块标准。之所以在过去我们有各种不同的模块化标准是因为 JavaScript 这门语言本身不具备模块化的特性，而现在 ES6 中已经具备了。ES6 Module 的模块语法和 CommonJS 很像，它通过 `import` 和 `export` 来进行模块的导入和导出。

``` JavaScript
import math from './math';

export function sum(a, b) {
    return a + b;
}
```

在 ES6 Module 中也是每个文件作为一个模块。和 CommonJS 不同的是，ES6 Module 的模块的依赖是静态的，或者说是在编译时确定的，而不是运行时确定的。

举个例子，我们可以在 CommonJS 中的 `if` 语句中 require 模块，根据代码运行时 `if` 的判断条件决定是否要引入该模块。

``` JavaScript
// 根据运行时条件确定是否引入
if (Date.now() > new Date('2019-01-01')) {
    require('./my_module');
}
```

而在 ES6 Module 中则不允许这样做， `import` 必须在代码的顶层作用域，这意味着你不能把它放在 `if` 等代码块中。ES6 Module 这样规定的原因在于可以使编译器在编译阶段就可以获取到整个依赖树，从而进行代码静态分析层面的优化，比如检测出哪些模块是从来没有被使用过的，然后从打包结果中优化掉等等。

### 6. 动态加载模块

有些场景下我们希望能够动态地去加载一些模块，在 CommonJS 中可以直接使用 `require` 实现。

```JavaScript
if(condition) {
    require('moduleA');
} else {
    require('moduleB');
}
```

但是在 ES6 Module 中，由于上面我们提到的 `import` 是在编译时被处理而非运行时，因此无法实现动态加载的特性。

```JavaScript
// 报错
if(condition) {
    require('moduleA');
}

// 报错
var foo = 'foo';
var bar = 'bar';
import foobar from (foo + bar);
```

为了解决这个问题，tc39 提出了一个 `import()` 函数提案。它可以接受一个参数，指定所加载的模块，并且返回一个 Promise 对象。

```JavaScript
var foo = 'foo';
var bar = 'bar';
import(foo + bar).then(module => {
    console.log('foobar loaded:', module);
}).catch(err => {
    console.log(err);
});
```

目前 Webpack 从 2.0.0 版本开始已经支持该动态加载形式，在后面的文章中会更加详细地进行介绍。

## 四、NPM包概念

`NPM` 实践了 `CommonJS` 的包规范。

#### 1. 包规范

关于包规范，类比于 `git` 仓库，我们可以这么理解:

* `git init` 在当前文件夹中生成了隐藏文件 `.git` ，我们把它叫做 `git仓库` 。
* `npm init` 命令在当前文件夹中生成了配置文件 `package.json` ，它描述了当前这个包，我们管这个文件叫做包（概念不准确，可以这么理解）。

#### 2. 包结构

严格按照 `CommonJS` 规范来的话，包的目录应当包含以下文件或目录。

* `package.json` ：包描述文件，存在于包顶级目录下
* `bin` ：存放可执行二进制文件的目录
* `lib` ：存放js代码的目录
* `doc` ：存放文档的目录
* `test` ：存放单元测试用例代码的目录

而 `package.json` 则是一个配置文件，它描述了包的相关信息。

#### 3. npm init 建立包

`npm init` , 会建立一个 `package.json` ，这是就相当于建立了一个包

#### 4. npm install 命令总结

``` shell
/*模块依赖会写入dependencies 节点*/
npm install moduleName
npm install -save moduleName
npm install -S moduleName

/*模块依赖会写入devDependencies节点*/
npm install -save-dev moduleName
npm install -D moduleName

/*全局安装模块包*/
npm install -g moduleName

/*安装特定版本的包*/
npm install包名@版本号
```

#### 5. npm 依赖包版本号

npm 所有node包都使用语义化版本号，规则要求如下：  

* 每个版本号都形如1.2.3，由三个部分组成，依次叫做“主版本号(major)”、“次版本号(minor)”和“修订号(patch)” 。
* 当新版本无法兼容基于前一版本的代码时，则提高主版本号 。
* 当新版本新增了功能与特性，但仍兼容前一版本的代码时，则提高次版本号 。
* 当新版本仅仅修正漏洞或者增强效率，仍然兼容前一版本代码，则提高修订号。

## 五、服务端规范

服务器端规范主要是 `CommonJS` ：NodeJS

### 1. Node中模块的分类

* 核心模块
  + 核心模块指的是那些被编译进Node的二进制模块
  + 预置在Node中，提供Node的基本功能，如fs、http、https等。
  + 核心模块使用C/C++实现，通过Node binding使用JS封装
* 第三方模块
  + `Node` 使用 `NPM` ( `Node Package Manager` )安装第三方模块
  + `NPM` 会将模块安装(可以说是下载到)到应用根目录下的 `node_modules` 文件夹中
  + 模块加载时， `node` 会先在核心模块文件夹中进行搜索，然后再到 `node_modules` 文件夹中进行搜索
* 文件模块
  + 文件可放在任何位置
  + 加载模块文件时加上路径即可
* 文件夹模块
  + `Node` 首先会在该文件夹中搜索 `package.json` 文件，
    - 存在，Node便尝试解析它，并加载main属性指定的模块文件
    - 不存在(或者 `package.json` 没有定义 `main` 属性)，Node默认加载该文件夹下的index.js文件（ `main` 属性其实 `NodeJS` 的一个拓展， `CommonJS` 标准定义中其实并不包括此字段）

估计大家对于文件夹模块概念都比较模糊，它其实相当于一个自定义模块，给大家举一个栗子：

在根目录下的 `/main.js` 中，我们需要使用一个自定义文件夹模块。我们将所有的自定义文件夹模块存放在根目录下的 `/module` 下，其中有一个 `/module/demo` 文件夹，是我们需要引入的文件夹模块; 

``` 
|—— main.js
|—— module
    |—— demo
        |—— package.json
        |—— demo.js
```

`package.json` 文件的信息如下:

``` 
{
    "name": "demo",
    "version": "1.0.0",
    "main": "./demo.js"
}
```

在 `/main.js` 中:

``` 
let demo = require("./modules/demo");
```

此时， `Node` 将会根据 `package.json` 中指定的 `main` 属性，去加载 `./modules/demo/demo.js` ; 

这就是一个最简单的包，以一个文件夹作为一个模块。

## 六、客户端规范

客户端规范主要有：

* `AMD` （异步模块定义，推崇依赖前置）
* `CMD` （通用模块定义，推崇依赖就近）

AMD规范的实现主要有 `RequireJS` ，

CMD规范的主要实现有 `SeaJS` 。

> ES6中已经有了模块化的实现，随着ES6的普及，第三方的模块化实现将会慢慢的淘汰。
