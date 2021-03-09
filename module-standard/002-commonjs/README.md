

# 002-理解CommonJS

<motto></motto>

![](./img/commonjs.png)

## 一、什么是CommonJS

CommonJS是一种规范，我们关心的是它的模块化规范。

该规范的目标是使JavaScript能够不仅在浏览器中运行，并且可以在Web服务器，桌面和命令行应用程序中建立JavaScript生态系统（原来的JS只能够构建基于浏览器的应用程序）。

该规范对模块的定义有：

1. 模块的定义

2. 模块的引用

3. 模块的标识

也就是`module`, `exports`, `require`

模块必须通过 `module.exports` 导出对外的变量或接口，通过 `require()` 来导入其他模块的导出到当前模块的作用域中。

CommonJS 不参与具体的代码实现，它就像是大家自觉选择遵守的规则一样，具体的实现最火的有Nodejs。

## 二、CommonJS的作用

CommonJS 规范出现的原因就是由于JavaScript语言本身并没有一种模块机制来保证不同模块可以使用相同的变量名，难以做到模块化和包管理，所以JS很难作为后端语言来进行大型项目的开发(当然ES6已经有了模块化规范:`export`, `import`, `export default`)。

所以为了解决 JavaScript 的作用域问题，需要定义一种模块化形式，可以使每个模块在它自身的命名空间中执行。这就是CommonJS的作用。

## 三、NodeJS中CommonJS的实现

下面主要来理解下我们的Node中的CommonJS的导入模块的实现，[Module源码](https://github.com/nodejs/node/blob/b04de23afa6da18d7b81b70c1a4bb53476f125c7/lib/internal/modules/cjs/loader.js)。

### 1. 模块的导入导出的应用栗子

#### 导出

导出在node中有两种方式：

1. 将需要导出的变量或函数挂载到 exports 对象上

```js
let foo = 'foo'
let bar = 'bar'
exports.foo = 'foo'
exports.bar = 'bar'
// 但是你不可以直接对exports赋值：下面的代码虽然可以执行，但是模块并不会导出foo和bar。
// 在后面会详细解释
exports = {
    foo: foo,
    bar: bar
};
```

2. 使用 `module.exports` 对象整体导出一个变量对象或者函数

```js
let foo = "foo"
let bar = "bar"
module.exports = {
    foo, 
    bar
}
//或者函数的导出
module.exports = function(){
    console.log("foo")
}
```

#### 导入

下面有三种来源不同的包引入，主要差异在于路径上的差异(自己编写的模块需要加上相对路径)。

```js
// Node核心模块的引入 
let fs = require('fs')
// 第三方，npm上用户发布的模块
let Router = require('vue-router');
// 用户自己编写的模块引入
let fooModule = require('./foo.js')
```

### 2. 模块加载源码理解

上面我们说到在Node中使用导出的时候有使用到`exports`但是却不可以直接对其进行赋值，而应该将其当作一个对象，给里面的属性进行赋值。

下面我们来简单分析理解一下Node的加载机制，[Modules源码](https://github.com/nodejs/node/blob/b04de23afa6da18d7b81b70c1a4bb53476f125c7/lib/internal/modules/cjs/loader.js)。

Nodejs并没有定义额外的语法来实现模块化，我们知道Node.js 每一个文件都是一个模块。Node内置了一些对象和方法来实现模块化。

![](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/NodeModuleSrc.png)


#### (1)Module对象

Node在每个模块中都内置了一个module对象来表示自身模块。

```js
// foo.js
console.log(module)
```

```shell
> node foo.js
Module {
  id: '.',
  exports: {},
  parent: null,
  filename: 'C:\\Users\\DELL\\Desktop\\module-learn\\foo.js',
  loaded: false,
  children: [],
  paths:
   [ 'C:\\Users\\DELL\\Desktop\\module-learn\\node_modules',
     'C:\\Users\\DELL\\Desktop\\node_modules',
     'C:\\Users\\DELL\\node_modules',
     'C:\\Users\\node_modules',
     'C:\\node_modules' ] 
}

```

```js
// 源码里的Module对象
function Module(id = '', parent) {
  this.id = id;
  this.path = path.dirname(id);
  this.exports = {};
  this.parent = parent;
  updateChildren(parent, this, false);
  this.filename = null;
  this.loaded = false;
  this.children = [];
}
```

| 属性              | 描述                                                 |
| ----------------- | ---------------------------------------------------- |
| `module.id`       | 字符串，模块的识别符，通常是带有绝对路径的模块文件名 |
| `module.exports`  | 对象，模块对外输出的值                               |
| `module.parent`   | 对象，调用该模块的模块                               |
| `module.filename` | 字符串，模块的绝对路径                               |
| `module.loaded`   | 布尔值，模块是否已经完成加载                         |
| `module.children` | 数组，该模块要用到的其他模块                         |
| `module.paths`    | 数组，模块可能存在的绝对路径                         |

#### (2)Module.prototype.require

```js
// Loads a module at the given file path. Returns that module's
// `exports` property.
Module.prototype.require = function(id) {
  validateString(id, 'id');
  if (id === '') {
    throw new ERR_INVALID_ARG_VALUE('id', id,
                                    'must be a non-empty string');
  }
  requireDepth++;
  try {
    return Module._load(id, this, /* isMain */ false);
  } finally {
    requireDepth--;
  }
};
```

可以从代码中看到，`require`方法主要检查了一下id和它为空的错误处理，然后主要就是调用了`Module._load`函数。

#### (3)Module._load

```js
// Check the cache for the requested file.
// 1. If a module already exists in the cache: return its exports object.
// 2. If the module is native: call
//    `NativeModule.prototype.compileForPublicLoader()` and return the exports.
// 3. Otherwise, create a new module for the file and save it to the cache.
//    Then have it load  the file contents before returning its exports
//    object.
// 检查请求文件的缓存。
// 1. 如果一个模块已经在缓存中存在:返回它的导出对象。
// 2. 如果模块是本机的:调用' NativeModule.prototype.compileForPublicLoader() '并返回导出。
// 3. 否则，为文件创建一个新模块并将其保存到缓存中。然后让它在返回导出之前加载模块对象。
Module._load = function(request, parent, isMain) {
  let relResolveCacheIdentifier;
  if (parent) {
    // ... ...省略部分代码
  }

  // 调用_resolveFilename 拼接出文件的绝对路径
  const filename = Module._resolveFilename(request, parent, isMain);

  // 调用_cache查看是否存在缓存，若存在则不需要再次执行，返回缓存即可
  const cachedModule = Module._cache[filename];
  if (cachedModule !== undefined) {
    updateChildren(parent, cachedModule, true);
    return cachedModule.exports;
  }

  // 调用loadNativeModule加载Node原生模块  
  // 如果Node原生模块有该模块且能被用户引用，返回mod.exports
  const mod = loadNativeModule(filename, request, experimentalModules);
  if (mod && mod.canBeRequiredByUsers) return mod.exports;

  // 创建一个模块
  // Don't call updateChildren(), Module constructor already does.
  const module = new Module(filename, parent);

  if (isMain) {
    process.mainModule = module;
    module.id = '.';
  }

  // 将该模块添加到缓存中
  Module._cache[filename] = module;
  if (parent !== undefined) {
    relativeResolveCache[relResolveCacheIdentifier] = filename;
  }

  let threw = true;
  try {
    // 加载执行新的模块
    module.load(filename);
    threw = false;
  } finally {
    if (threw) {
      delete Module._cache[filename];
      if (parent !== undefined) {
        delete relativeResolveCache[relResolveCacheIdentifier];
      }
    }
  }

  // 输出模块的exports属性
  return module.exports;
};
```

`Module._cache`就是个对象, `Module._cache = Object.create(null);`

所以，根据上面的源码，`Module._load`的流程:

1. 调用_resolveFilename 拼接出文件的绝对路径
2. 调用_cache查看是否存在缓存，若存在则不需要再次执行，返回缓存即可
3. 调用loadNativeModule加载Node原生模块，如果Node原生模块有该模块且能被用户引用，就返回该模块导出的内容
4. 创建一个模块
5. 将该模块添加到缓存中
6. 调用prototype中的load方法加载执行新的模块
7. 输出模块的exports属性

> 下面对上述流程中出现的方法调用进行简单的分析：

#### (4)Module._resolveFilename确定模块的绝对路径

```js
Module._resolveFilename = function(request, parent) {

 // 第一步：如果是内置模块，不含路径返回
 if (NativeModule.exists(request)) {
   return request;
 }

 // 第二步：确定所有可能的路径
 var resolvedModule = Module._resolveLookupPaths(request, parent);
 var id = resolvedModule[0];
 var paths = resolvedModule[1];

 // 第三步：确定哪一个路径为真
 var filename = Module._findPath(request, paths);
 if (!filename) {
   var err = new Error("Cannot find module '" + request + "'");
   err.code = 'MODULE_NOT_FOUND';
   throw err;
 }
 return filename;
};
```

可以看到`Module._resolveFilename`又调用了`Module._resolveLookupPaths`和`Module._findPath`

#####  Module._findPath()

```js
Module._findPath = function(request, paths) {

 // 列出所有可能的后缀名：.js，.json, .node
 var exts = Object.keys(Module._extensions);

 // 如果是绝对路径，就不再搜索
 if (request.charAt(0) === '/') {
   paths = [''];
 }

 // 是否有后缀的目录斜杠
 var trailingSlash = (request.slice(-1) === '/');

 // 第一步：如果当前路径已在缓存中，就直接返回缓存
 var cacheKey = JSON.stringify({request: request, paths: paths});
 if (Module._pathCache[cacheKey]) {
   return Module._pathCache[cacheKey];
 }

 // 第二步：依次遍历所有路径
 for (var i = 0, PL = paths.length; i < PL; i++) {
   var basePath = path.resolve(paths[i], request);
   var filename;

   if (!trailingSlash) {
     // 第三步：是否存在该模块文件
     filename = tryFile(basePath);

     if (!filename && !trailingSlash) {
       // 第四步：该模块文件加上后缀名，是否存在
       filename = tryExtensions(basePath, exts);
     }
   }

   // 第五步：目录中是否存在 package.json 
   if (!filename) {
     filename = tryPackage(basePath, exts);
   }

   if (!filename) {
     // 第六步：是否存在目录名 + index + 后缀名 
     filename = tryExtensions(path.resolve(basePath, 'index'), exts);
   }

   // 第七步：将找到的文件路径存入返回缓存，然后返回
   if (filename) {
     Module._pathCache[cacheKey] = filename;
     return filename;
   }
 }

 // 第八步：没有找到文件，返回false 
 return false;
};


```

#### (5)Module.prototype.load

上面拥有了模块的绝对路径了，开始加载模块

```js
// Given a file name, pass it to the proper extension handler.
Module.prototype.load = function(filename) {
  const extension = findLongestRegisteredExtension(filename);
  // 不同的扩展名，对应不同的读取方法
  Module._extensions[extension](this, filename);
  this.loaded = true;
};
```

下面是对应的四种不同模块文件的加载方式

```js
// Native extension for .js
Module._extensions['.js'] = function(module, filename) {
  const content = fs.readFileSync(filename, 'utf8');
  module._compile(content, filename);
};

// Native extension for .json
Module._extensions['.json'] = function(module, filename) {
  const content = fs.readFileSync(filename, 'utf8');

  if (manifest) {
    const moduleURL = pathToFileURL(filename);
    manifest.assertIntegrity(moduleURL, content);
  }

  try {
    module.exports = JSON.parse(stripBOM(content));
  } catch (err) {
    err.message = filename + ': ' + err.message;
    throw err;
  }
};

// Native extension for .node
Module._extensions['.node'] = function(module, filename) {
  if (manifest) {
    const content = fs.readFileSync(filename);
    const moduleURL = pathToFileURL(filename);
    manifest.assertIntegrity(moduleURL, content);
  }
  // Be aware this doesn't use `content`
  return process.dlopen(module, path.toNamespacedPath(filename));
};

Module._extensions['.mjs'] = function(module, filename) {
  throw new ERR_REQUIRE_ESM(filename);
};
```

##### Module.prototype._compile 

```js
Module.prototype._compile = function(content, filename) {
  var result;
  const exports = this.exports;
  const thisValue = exports;
  var module = this;
  return compiledWrapper.call(thisValue, exports, require, module,
                                  filename, dirname);
};
```

实际上，上面的代码由于对象调用，this即当前调用的模块，则上面的代码等同于

```js
(function (exports, require, module, __filename, __dirname) {
  // 模块源码
});
```

### 3. 解惑

#### 3.1 从未自己声明过require、export、module变量

从上面的`Module.prototype._compile`我们可以看出来，模块的实质还是一个函数作用域，就是使用 IIFE 立即调用函数表达式将其包裹了一下，并传入`exports, require, module, __filename, __dirname`这几个在 Node.js 源码内定义了的变量

也就是说，模块的加载实质上就是，注入`exports`、`require`、`module`三个全局变量，然后执行模块的源码，然后将模块的 exports 变量的值输出。

#### 3.2 exports和module.exports的引用是一样的

还是看上面的`Module.prototype._compile`，我们看到`  const exports = this.exports;   var module = this;`，所以，现在，我们总算是明白了为什么既可以使用 `exports.xxx = function(){...}`也可以使用`module.exports={xxx}`了，因为他们根本就是一样的。

> 虽然我们说Node是CommonJs的实现，但实际上，2013年Node Modules就不再和CommonJS纠缠了。
>
> Ryan在2011年也说到过'Consider CommonJS extinct - not worth thinking further about. That was a 2009 thing.'
>
> CommonJS想要在尽可能多的操作系统和解释器上工作，但是他对浏览器却很不友好，不支持异步加载。。。
>
> 而且Node是后端Javascript。Ryan Dahl: "Forget CommonJS. It's dead. We are server side JavaScript."

## THANK

[Wiki-Modules](http://wiki.commonjs.org/wiki/Modules)

[廖雪峰-NodeJS-模块](https://www.liaoxuefeng.com/wiki/1022910821149312/1023027697415616)

[node 的模块运行机制](https://juejin.im/post/5e174b815188254c075d0d8d)

[来来来，探究一下CommonJs的实现原理](https://juejin.im/post/5b8396ae51882542d02ccb89)