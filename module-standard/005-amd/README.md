# 005-理解AMD

<motto></motto>

AMD 即Asynchronous Module Definition异步模块定义，它采用异步方式加载模块，模块的加载不影响它后面语句的运行，所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行

一般来说，AMD是 RequireJS 在推广过程中对模块定义的规范化的产出，因为平时在开发中比较常用的是require.js进行模块的定义和加载，一般是使用define来定义模块，使用require来加载模块