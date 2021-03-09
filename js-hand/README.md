# 手写JS代码

<motto></motto>

## 实现async

``` js
function asyncToGenerator(generatorFunc) {
    return function() {
        const gen = generatorFunc.apply(this, arguments)
        return new Promise((resolve, reject) => {
            function step(key, arg) {
                let generatorResult
                try {
                    generatorResult = gen[key](arg)
                } catch (error) {
                    return reject(error)
                }
                // gen.next() 得到的结果是一个 { value, done } 的结构
                const {
                    value,
                    done
                } = generatorResult
                if (done) {
                    return resolve(value)
                } else {
                    return Promise.resolve(
                        value
                    ).then(
                        function onResolve(val) {
                            step("next", val)
                        }

                        function onReject(err) {
                            step("throw", err)
                        },
                    )
                }
            }
            step("next")
        })
    }
}
```

``` js
// 注释版
function asyncToGenerator(generatorFunc) {
    // 返回的是一个新的函数
    return function() {
        // 先调用generator函数 生成迭代器
        // 对应 var gen = testG()
        const gen = generatorFunc.apply(this, arguments)

        // 返回一个promise 因为外部是用.then的方式 或者await的方式去使用这个函数的返回值的
        // var test = asyncToGenerator(testG)
        // test().then(res => console.log(res))
        return new Promise((resolve, reject) => {

            // 内部定义一个step函数 用来一步一步的跨过yield的阻碍
            // key有next和throw两种取值，分别对应了gen的next和throw方法
            // arg参数则是用来把promise resolve出来的值交给下一个yield
            function step(key, arg) {
                let generatorResult
                // 这个方法需要包裹在try catch中
                // 如果报错了 就把promise给reject掉 外部通过.catch可以获取到错误
                try {
                    generatorResult = gen[key](arg)
                } catch (error) {
                    return reject(error)
                }
                // gen.next() 得到的结果是一个 { value, done } 的结构
                const {
                    value,
                    done
                } = generatorResult
                if (done) {
                    // 如果已经完成了 就直接resolve这个promise
                    // 这个done是在最后一次调用next后才会为true
                    // 以本文的例子来说 此时的结果是 { done: true, value: 'success' }
                    // 这个value也就是generator函数最后的返回值
                    return resolve(value)
                } else {
                    // 除了最后结束的时候外，每次调用gen.next()
                    // 其实是返回 { value: Promise, done: false } 的结构，
                    // 这里要注意的是Promise.resolve可以接受一个promise为参数
                    // 并且这个promise参数被resolve的时候，这个then才会被调用
                    return Promise.resolve(
                        // 这个value对应的是yield后面的promise
                        value
                    ).then(
                        // value这个promise被resove的时候，就会执行next
                        // 并且只要done不是true的时候 就会递归的往下解开promise
                        // 对应gen.next().value.then(value => {
                        //    gen.next(value).value.then(value2 => {
                        //       gen.next()
                        //
                        //      // 此时done为true了 整个promise被resolve了
                        //      // 最外部的test().then(res => console.log(res))的then就开始执行了
                        //    })
                        // })
                        function onResolve(val) {
                            step("next", val)
                        },
                        // 如果promise被reject了 就再次进入step函数
                        // 不同的是，这次的try catch中调用的是gen.throw(err)
                        // 那么自然就被catch到 然后把promise给reject掉啦
                        function onReject(err) {
                            step("throw", err)
                        },
                    )
                }
            }
            step("next")
        })
    }
}
```

## 函数柯里化

> 例如，add(1)(2, 3)

``` js
// 参数长度固定
// 原理是利用闭包把传入参数保存起来，当传入参数的数量足够执行函数时，就开始执行函数
const curry = function(fn) {
    return function judge(...args) {
        return args.length === fn.length ?
            fn(...args) : function(...arg) {
                return judge(...args, ...arg)
            }
    }
}

const add = (a, b, c) => a + b + c;
const curryAdd = curry(add);
console.log(curryAdd(1)(2)(3)); // 6
console.log(curryAdd(1, 2)(3)); // 6
console.log(curryAdd(1)(2, 3)); // 6

// 参数不固定
function add(...rest) {
    return rest.reduce((prev, cur) => {
        return prev + cur
    })
}

function currying(fn) {
    let args = []
    return function temp(...newArgs) {
        if (newArgs.length) {
            args = [...args, ...newArgs]
            return temp
        } else {
            let val = fn.apply(this, args)
            args = [] //保证再次调用时清空
            return val
        }
    }
}

let curryAdd = currying(add)
console.log(curryAdd(1, 2)(3)())
```

## 实现链式调用

``` js
var obj = {
    a: function() {
        console.log("a");
        return this;
    },
    b: function() {
        console.log("b");
        return this;
    },
};
obj.a().b();
```

## 类数组转数组

> 转换方法

* 使用 Array.from()
* 使用 Array.prototype.slice.call()
* 使用 Array.prototype.forEach() 进行属性遍历并组成新的数组

> 转换后的数组长度由 length 属性决定。索引不连续时转换结果是连续的，会自动补位。

``` js
let al1 = {
    length: 4,
    0: 0,
    1: 1,
    3: 3,
    4: 4,
    5: 5,
};
console.log(Array.from(al1)) // [0, 1, undefined, 3]

// 仅考虑 0 或正整数 的索引
let al2 = {
    length: 4,
    '-1': -1,
    '0': 0,
    a: 'a',
    1: 1
};
console.log(Array.from(al2)); // [0, 1, undefined, undefined]

// 使用slice转换产生稀疏数组
let al2 = {
    length: 4,
    '-1': -1,
    '0': 0,
    a: 'a',
    1: 1
};
console.log(Array.prototype.slice.call(al2)); //[0, 1, empty × 2]

// 4） 使用数组方法操作类数组注意地方
let arrayLike2 = {
    2: 3,
    3: 4,
    length: 2,
    push: Array.prototype.push
}

// push 操作的是索引值为 length 的位置
arrayLike2.push(1);
console.log(arrayLike2); // {2: 1, 3: 4, length: 3, push: ƒ}
arrayLike2.push(2);
console.log(arrayLike2); // {2: 1, 3: 2, length: 4, push: ƒ}
```

## 封装Node中间件（实现简易Koa）

``` js
const http = require('http')

function compose(middlewareList) {
    return function(ctx) {
        function dispatch(i) {
            const fn = middlewareList[i]
            try {
                return Promise.resolve(fn(ctx, dispatch.bind(null, i + 1)))
            } catch (err) {
                Promise.reject(err)
            }
        }
        return dispatch(0)
    }
}

class App {
    constructor() {
        this.middlewares = []
    }
    use(fn) {
        this.middlewares.push(fn)
        return this
    }
    handleRequest(ctx, middleware) {
        return middleware(ctx)
    }
    createContext(req, res) {
        const ctx = {
            req,
            res
        }
        return ctx
    }
    callback() {
        const fn = compose(this.middlewares)
        return (req, res) => {
            const ctx = this.createContext(req, res)
            return this.handleRequest(ctx, fn)
        }
    }
    listen(...args) {
        const server = http.createServer(this.callback())
        return server.listen(...args)
    }
}
module.exports = App
```

## 实现Promise

### Promise基本特性

1、Promise有三种状态：pending(进行中)、fulfilled(已成功)、rejected(已失败)

2、Promise对象接受一个回调函数作为参数, 该回调函数接受两个参数，分别是成功时的回调resolve和失败时的回调reject；另外resolve的参数除了正常值以外， 还可能是一个Promise对象的实例；reject的参数通常是一个Error对象的实例。

3、then方法返回一个新的Promise实例，并接收两个参数onResolved(fulfilled状态的回调)；onRejected(rejected状态的回调，该参数可选)

4、catch方法返回一个新的Promise实例

5、finally方法不管Promise状态如何都会执行，该方法的回调函数不接受任何参数

6、Promise.all()方法将多个多个Promise实例，包装成一个新的Promise实例，该方法接受一个由Promise对象组成的数组作为参数(Promise.all()方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例)，注意参数中只要有一个实例触发catch方法，都会触发Promise.all()方法返回的新的实例的catch方法，如果参数中的某个实例本身调用了catch方法，将不会触发Promise.all()方法返回的新实例的catch方法

7、Promise.race()方法的参数与Promise.all方法一样，参数中的实例只要有一个率先改变状态就会将该实例的状态传给Promise.race()方法，并将返回值作为Promise.race()方法产生的Promise实例的返回值

8、Promise.resolve()将现有对象转为Promise对象，如果该方法的参数为一个Promise对象，Promise.resolve()将不做任何处理；如果参数thenable对象(即具有then方法)，Promise.resolve()将该对象转为Promise对象并立即执行then方法；如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为fulfilled，其参数将会作为then方法中onResolved回调函数的参数，如果Promise.resolve方法不带参数，会直接返回一个fulfilled状态的 Promise 对象。需要注意的是，立即resolve()的 Promise 对象，是在本轮“事件循环”（event loop）的结束时执行，而不是在下一轮“事件循环”的开始时。

9、Promise.reject()同样返回一个新的Promise对象，状态为rejected，无论传入任何参数都将作为reject()的参数

### Promise优点

1、统一异步 API
Promise 的一个重要优点是它将逐渐被用作浏览器的异步 API ，统一现在各种各样的 API ，以及不兼容的模式和手法。

2、Promise 与事件对比
和事件相比较， Promise 更适合处理一次性的结果。在结果计算出来之前或之后注册回调函数都是可以的，都可以拿到正确的值。 Promise 的这个优点很自然。但是，不能使用 Promise 处理多次触发的事件。链式处理是 Promise 的又一优点，但是事件却不能这样链式处理。

3、Promise 与回调对比
解决了回调地狱的问题，将异步操作以同步操作的流程表达出来。

4、Promise 带来的额外好处是包含了更好的错误处理方式（包含了异常处理），并且写起来很轻松（因为可以重用一些同步的工具，比如 Array.prototype.map() ）。

### Promise缺点

1、无法取消Promise，一旦新建它就会立即执行，无法中途取消。

2、如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。

3、当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

4、Promise 真正执行回调的时候，定义 Promise 那部分实际上已经走完了，所以 Promise 的报错堆栈上下文不太友好。

``` js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function Promise(excutor) {
    let that = this; // 缓存当前promise实例例对象
    that.status = PENDING; // 初始状态
    that.value = undefined; // fulfilled状态时 返回的信息
    that.reason = undefined; // rejected状态时 拒绝的原因
    that.onFulfilledCallbacks = []; // 存储fulfilled状态对应的onFulfilled函数
    that.onRejectedCallbacks = []; // 存储rejected状态对应的onRejected函数
    function resolve(value) { // value成功态时接收的终值
        if (value instanceof Promise) {
            return value.then(resolve, reject);
        }
        // 实践中要确保 onFulfilled 和 onRejected ⽅方法异步执⾏行行，且应该在 then ⽅方法被调⽤用的那⼀一轮事件循环之后的新执⾏行行栈中执⾏行行。
        setTimeout(() => {
            // 调⽤用resolve 回调对应onFulfilled函数
            if (that.status === PENDING) {
                // 只能由pending状态 => fulfilled状态 (避免调⽤用多次resolve reject)
                that.status = FULFILLED;
                that.value = value;
                that.onFulfilledCallbacks.forEach(cb => cb(that.value));
            }
        });
    }

    function reject(reason) { // reason失败态时接收的拒因
        setTimeout(() => {
            // 调⽤用reject 回调对应onRejected函数
            if (that.status === PENDING) {
                // 只能由pending状态 => rejected状态 (避免调⽤用多次resolve reject)
                that.status = REJECTED;
                that.reason = reason;
                that.onRejectedCallbacks.forEach(cb => cb(that.reason));
            }
        });
    }

    // 捕获在excutor执⾏行行器器中抛出的异常
    // new Promise((resolve, reject) => {
    //     throw new Error('error in excutor')
    // })
    try {
        excutor(resolve, reject);
    } catch (e) {
        reject(e);
    }
}
Promise.prototype.then = function(onFulfilled, onRejected) {
    const that = this;
    let newPromise;
    // 处理理参数默认值 保证参数后续能够继续执⾏行行
    onFulfilled = typeof onFulfilled === "function" ? onFulfilled : value => value;
    onRejected = typeof onRejected === "function" ? onRejected : reason => {
        throw reason;
    };
    if (that.status === FULFILLED) { // 成功态
        return newPromise = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onFulfilled(that.value);
                    resolvePromise(newPromise, x, resolve, reject); //新的promise resolve 上⼀一个onFulfilled的返回值
                } catch (e) {
                    reject(e); // 捕获前⾯面onFulfilled中抛出的异常then(onFulfilled, onRejected);
                }
            });
        })
    }
    if (that.status === REJECTED) { // 失败态
        return newPromise = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onRejected(that.reason);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
        });
    }
    if (that.status === PENDING) { // 等待态
        // 当异步调⽤用resolve/rejected时 将onFulfilled/onRejected收集暂存到集合中
        return newPromise = new Promise((resolve, reject) => {
            that.onFulfilledCallbacks.push((value) => {
                try {
                    let x = onFulfilled(value);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
            that.onRejectedCallbacks.push((reason) => {
                try {
                    let x = onRejected(reason);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
        });
    }
};
```

## 实现Promise.all

``` js
Promise.prototype.all = (values) => {
    return new Promise((resolve, reject) => {
        let resultArr = []
        let count = 0
        let resultByKey = (value, index) => {
            resultArr[index] = value
            if (++count === values.length) {
                resolve(resultArr)
            }
        }
        values.forEach((promise, index) => {
            promise.then((value) => {
                resultByKey(value, index)
            }, reject)
        })
    })
}
```

## 模拟call

* 第一个参数为 `null` 或者 `undefined` 时， `this` 指向全局对象 `window` ，值为原始值的指向该原始值的自动包装对象，如 `String` 、 `Number` 、 `Boolean`
* 为了避免函数名与上下文( `context` )的属性发生冲突，使用 `Symbol` 类型作为唯一值
* 将函数作为传入的上下文( `context` )属性执行
* 函数执行完成后删除该属性
* 返回执行结果

``` js
Function.prototype.myCall = function(context, ...args) {
    context = (context ? ? window) || new Object(context)
    const key = Symbol()
    context[key] = this
    const result = context[key](...args)
    delete context[key]
    return result
}
```

注： 代码实现使用了 `ES2020` 新特性 `Null` 判断符 `??` ， 详细参考阮一峰老师的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/object)

**ES3实现**

``` js
function fnFactory(context) {
    var unique_fn = "fn";
    while (context.hasOwnProperty(unique_fn)) {
        unique_fn = "fn" + Math.random();
    }
    return unique_fn;
}
Function.prototype.call2 = function(context) {
    // 1. 若是传入的context是null或者undefined时指向window;
    // 2. 若是传入的是原始数据类型, 原生的call会调用 Object() 转换
    context = (context !== null && context !== undefined) ? Object(context) : window;
    // 3. 创建一个独一无二的fn函数的命名
    var fn = fnFactory(context);
    // 4. 这里的this就是指调用call的那个函数
    // 5. 将调用的这个函数赋值到context中, 这样之后执行context.fn的时候, fn里的this就是指向context了
    context[fn] = this;
    // 6. 定义一个数组用于放arguments的每一项的字符串: ['agruments[1]', 'arguments[2]']
    var args = [];
    // 7. 要从第1项开始, 第0项是context
    for (var i = 1, l = arguments.length; i < l; i++) {
        args.push("arguments[" + i + "]");
    }
    // 8. 使用eval()来执行fn并将args一个个传递进去
    var result = eval("context[fn](" + args + ")");
    // 9. 给context额外附件了一个属性fn, 所以用完之后需要删除
    delete context[fn];
    // 10. 函数fn可能会有返回值, 需要将其返回
    return result;
};
```

## 模拟apply

* 前部分与 `call` 一样
* 第二个参数可以不传，但类型必须为数组或者类数组

``` js
Function.prototype.myApply = function(context) {
    context = (context ? ? window) || new Object(context)
    const key = Symbol()
    const args = arguments[1]
    context[key] = this
    const result = args ? context[key](...args) : context[key]()
    delete context[key]
    return result
}
```

注：代码实现存在缺陷，当第二个参数为类数组时，未作判断（有兴趣可查阅一下如何判断类数组）

**ES3实现**

``` js
function fnFactory(context) {
    var unique_fn = "fn";
    while (context.hasOwnProperty(unique_fn)) {
        unique_fn = "fn" + Math.random();
    }
    return unique_fn;
}
Function.prototype.apply2 = function(context, arr) {
    // 1. 若是传入的context是null或者undefined时指向window;
    // 2. 若是传入的是原始数据类型, 原生的call会调用 Object() 转换
    context = context ? Object(context) : window;
    // 3. 创建一个独一无二的fn函数的命名
    var fn = fnFactory(context);
    // 4. 这里的this就是指调用call的那个函数
    // 5. 将调用的这个函数赋值到context中, 这样之后执行context.fn的时候, fn里的this就是指向context了
    context[fn] = this;

    var result;
    // 6. 判断有没有第二个参数
    if (!arr) {
        result = context[fn]();
    } else {
        // 7. 有的话则用args放每一项的字符串: ['arr[0]', 'arr[1]']
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push("arr[" + i + "]");
        }
        // 8. 使用eval()来执行fn并将args一个个传递进去
        result = eval("context[fn](" + args + ")");
    }
    // 9. 给context额外附件了一个属性fn, 所以用完之后需要删除
    delete context[fn];
    // 10. 函数fn可能会有返回值, 需要将其返回
    return result;
};
```

## 模拟bind

* 使用 `call / apply` 指定 `this`
* 返回一个绑定函数
* 当返回的绑定函数作为构造函数被 `new` 调用，绑定的上下文指向实例对象
* 设置绑定函数的 `prototype` 为原函数的 `prototype`

``` js
Function.prototype.myBind = function(context, ...args) {
    const fn = this
    const bindFn = function(...newFnArgs) {
        return fn.call(
            this instanceof bindFn ? this : context,
            ...args, ...newFnArgs
        )
    }
    bindFn.prototype = Object.create(fn.prototype)
    return bindFn
}
```

## 模拟new

* 创建一个新的空对象
* 把 `this` 绑定到空对象
* 使空对象的 `__proto__` 指向构造函数的原型( `prototype` )
* 执行构造函数，为空对象添加属性
* 判断构造函数的返回值是否为对象，如果是对象，就使用构造函数的返回值，否则返回创建的对象

``` js
const createNew = (Con, ...args) => {
    const obj = {}
    Object.setPrototypeOf(obj, Con.prototype)
    let result = Con.apply(obj, args)
    return result instanceof Object ? result : obj
}
```

## 模拟instanceof

* 遍历左边变量的原型链，直到找到右边变量的 prototype，如果没有找到，返回 `false`

``` js
const myInstanceOf = (left, right) => {
    let leftValue = left.__proto__
    let rightValue = right.prototype
    while (true) {
        if (leftValue === null) return false
        if (leftValue === rightValue) return true
        leftValue = leftValue.__proto__
    }
}
```

## 深拷贝（简单版）

* 判断类型是否为原始类型，如果是，无需拷贝，直接返回
* 为避免出现循环引用，拷贝对象时先判断存储空间中是否存在当前对象，如果有就直接返回
* 开辟一个存储空间，来存储当前对象和拷贝对象的对应关系
* 对引用类型递归拷贝直到属性为原始类型

``` js
const deepClone = (target, cache = new WeakMap()) => {
    if (target === null || typeof target !== 'object') {
        return target
    }
    if (cache.get(target)) {
        return target
    }
    const copy = Array.isArray(target) ? [] : {}
    cache.set(target, copy)
    Object.keys(target).forEach(key => copy[key] = deepClone(target[key], cache))
    return copy
}
```

缺点：无法拷贝函数、 `Map` 、 `Set` 、正则等其他类型

## 深拷贝（复杂版）

[如何写出一个惊艳面试官的深拷贝?](https://github.com/ConardLi/ConardLi.github.io/blob/master/demo/deepClone/src/clone_6.js)

``` js
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';
const argsTag = '[object Arguments]';

const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
const errorTag = '[object Error]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';

const deepTag = [mapTag, setTag, arrayTag, objectTag, argsTag];

function forEach(array, iteratee) {
    let index = -1;
    const length = array.length;
    while (++index < length) {
        iteratee(array[index], index);
    }
    return array;
}

function isObject(target) {
    const type = typeof target;
    return target !== null && (type === 'object' || type === 'function');
}

function getType(target) {
    return Object.prototype.toString.call(target);
}

function getInit(target) {
    const Ctor = target.constructor;
    return new Ctor();
}

function cloneSymbol(targe) {
    return Object(Symbol.prototype.valueOf.call(targe));
}

function cloneReg(targe) {
    const reFlags = /\w*$/;
    const result = new targe.constructor(targe.source, reFlags.exec(targe));
    result.lastIndex = targe.lastIndex;
    return result;
}

function cloneFunction(func) {
    const bodyReg = /(?<={)(.|\n)+(?=})/m;
    const paramReg = /(?<=\().+(?=\)\s+{)/;
    const funcString = func.toString();
    if (func.prototype) {
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            if (param) {
                const paramArr = param[0].split(',');
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        return eval(funcString);
    }
}

function cloneOtherType(targe, type) {
    const Ctor = targe.constructor;
    switch (type) {
        case boolTag:
        case numberTag:
        case stringTag:
        case errorTag:
        case dateTag:
            return new Ctor(targe);
        case regexpTag:
            return cloneReg(targe);
        case symbolTag:
            return cloneSymbol(targe);
        case funcTag:
            return cloneFunction(targe);
        default:
            return null;
    }
}

function clone(target, map = new WeakMap()) {

    // 克隆原始类型
    if (!isObject(target)) {
        return target;
    }

    // 初始化
    const type = getType(target);
    let cloneTarget;
    if (deepTag.includes(type)) {
        cloneTarget = getInit(target, type);
    } else {
        return cloneOtherType(target, type);
    }

    // 防止循环引用
    if (map.get(target)) {
        return map.get(target);
    }
    map.set(target, cloneTarget);

    // 克隆set
    if (type === setTag) {
        target.forEach(value => {
            cloneTarget.add(clone(value, map));
        });
        return cloneTarget;
    }

    // 克隆map
    if (type === mapTag) {
        target.forEach((value, key) => {
            cloneTarget.set(key, clone(value, map));
        });
        return cloneTarget;
    }

    // 克隆对象和数组
    const keys = type === arrayTag ? undefined : Object.keys(target);
    forEach(keys || target, (value, key) => {
        if (keys) {
            key = value;
        }
        cloneTarget[key] = clone(target[key], map);
    });

    return cloneTarget;
}
```

## 函数防抖

函数防抖是在事件被触发 `n` 秒后再执行回调，如果在 `n` 秒内又被触发，则重新计时。 函数防抖多用于 `input` 输入框

* 箭头函数的 `this` 继承自父级上下文，这里指向触发事件的目标元素
* 事件被触发时，传入 `event` 对象
* 传入 `leading` 参数，判断是否可以立即执行回调函数，不必要等到事件停止触发后才开始执行
* 回调函数可以有返回值，需要返回执行结果

``` js
 const debounce = (fn, wait = 300, leading = true) => {
     let timerId, result
     return function(...args) {
         timerId && clearTimeout(timerId)
         if (leading) {
             if (!timerId) result = fn.apply(this, args)
             timerId = setTimeout(() => timerId = null, wait)
         } else {
             timerId = setTimeout(() => result = fn.apply(this, args), wait)
         }
         return result
     }
 }
```

## 函数节流（定时器）

函数节流是指连续触发事件，但是在 n 秒中只执行一次函数，适合应用于动画相关的场景

``` js
const throttle = (fn, wait = 300) => {
    let timerId
    return function(...args) {
        if (!timerId) {
            timerId = setTimeout(() => {
                timerId = null
                return result = fn.apply(this, ...args)
            }, wait)
        }
    }
}
```

## 函数节流（时间戳）

``` js
const throttle = (fn, wait = 300) => {
    let prev = 0
    let result
    return function(...args) {
        let now = +new Date()
        if (now - prev > wait) {
            prev = now
            return result = fn.apply(this, ...args)
        }
    }
}
```

### 函数节流实现方法区别

|    方法    |  使用时间戳  |    使用定时器    |
| :--------: | :----------: | :--------------: |
| 开始触发时 |   立刻执行   |    n秒后执行     |
| 停止触发后 | 不再执行事件 | 继续执行一次事件 |

## 函数节流（双剑合璧版）

``` js
 const throttle = (fn, wait = 300, {
     // 参数解构赋值
     leading = true,
     trailing = true,
 } = {}) => {
     let prev = 0
     let timerId
     const later = function(args) {
         timerId && clearTimeout(timerId)
         timerId = setTimeout(() => {
             timerId = null
             fn.apply(this, args)
         }, wait)
     }
     return function(...args) {
         let now = +new Date()
         if (!leading) return later(args)
         if (now - prev > wait) {
             fn.apply(this, args)
             prev = now
         } else if (trailing) {
             later(args)
         }
     }
 }
```

`leading：false` 表示禁用第一次执行

`trailing: false` 表示禁用停止触发的回调

注意： `leading：false` 和 `trailing: false` 不能同时设置。

## 数组去重

``` js
const uniqBy = (arr, key) => {
    return [...new Map(arr.map(item => [item[key], item])).values()]
}

const singers = [{
        id: 1,
        name: 'Leslie Cheung'
    },
    {
        id: 1,
        name: 'Leslie Cheung'
    },
    {
        id: 2,
        name: 'Eason Chan'
    },
]
console.log(uniqBy(singers, 'id'))

//  [
//    { id: 1, name: 'Leslie Cheung' },
//    { id: 2, name: 'Eason Chan' },
//  ]
```

原理是利用 `Map` 的键不可重复

## 数组去重(filter版)

```js
arr = ['a', 'b', 'c', 'a'];
let arr2 = arr.filter((item,index,self)=>{
    return self.indexOf(item) == index;
});
console.log(arr2);
```

## 数组扁平化（技巧版）

利用 `toString` 把数组变成以逗号分隔的字符串，遍历数组把每一项再变回原来的类型。缺点：数组中元素必须是 `Number` 类型， `String` 类型会被转化成 `Number`

``` js
const str = [0, 1, [2, [3, 4]]].toString()
// '0, 1, 2, 3, 4'
const arr = str.split(',')
// ['0','1','2', '3', '4']
const newArr = arr.map(item => +item)
// [0, 1, 2, 3, 4]

const flatten = (arr) => arr.toString().split(',').map(item => +item)
```

## 数组扁平化

`reduce` + 递归

``` js
const flatten = (arr, deep = 1) => {
    return arr.reduce((cur, next) => {
        return Array.isArray(next) && deep > 1 ? [...cur, ...flatten(next, deep - 1)] : [...cur, next]
    }, [])
}

const arr = [1, [2],
    [3, [4]]
]
flatten(arr, 1) // [1, [2], [3, [4]]]
flatten(arr, 2) // [1，2, [3, 4]]
flatten(arr, 3) // [1，2, 3, 4]
```

## JSON.stringify 和 JSON.parse

```javascript
if (!window.JSON) {
    window.JSON = {
        parse: function(jsonStr) {
            return eval('(' + jsonStr + ')');
        },
        stringify: function(jsonObj) {
            var result = '',
                curVal;
            if (jsonObj === null) {
                return String(jsonObj);
            }
            switch (typeof jsonObj) {
                case 'number':
                case 'boolean':
                    return String(jsonObj);
                case 'string':
                    return '"' + jsonObj + '"';
                case 'undefined':
                case 'function':
                    return undefined;
            }

            switch (Object.prototype.toString.call(jsonObj)) {
                case '[object Array]':
                    result += '[';
                    for (var i = 0, len = jsonObj.length; i < len; i++) {
                        curVal = JSON.stringify(jsonObj[i]);
                        result += (curVal === undefined ? null : curVal) + ",";
                    }
                    if (result !== '[') {
                        result = result.slice(0, -1);
                    }
                    result += ']';
                    return result;
                case '[object Date]':
                    return '"' + (jsonObj.toJSON ? jsonObj.toJSON() : jsonObj.toString()) + '"';
                case '[object RegExp]':
                    return "{}";
                case '[object Object]':
                    result += '{';
                    for (i in jsonObj) {
                        if (jsonObj.hasOwnProperty(i)) {
                            curVal = JSON.stringify(jsonObj[i]);
                            if (curVal !== undefined) {
                                result += '"' + i + '":' + curVal + ',';
                            }
                        }
                    }
                    if (result !== '{') {
                        result = result.slice(0, -1);
                    }
                    result += '}';
                    return result;

                case '[object String]':
                    return '"' + jsonObj.toString() + '"';
                case '[object Number]':
                case '[object Boolean]':
                    return jsonObj.toString();
            }
        }
    };
}
```

## 发布订阅EventEmitter

``` js
class EventEmitter {
    #subs = {}
    emit(event, ...args) {
        if (this.#subs[event] && this.#subs[event].length) {
            this.#subs[event].forEach(cb => cb(...args))
        }
    }
    on(event, cb) {
        (this.#subs[event] || (this.#subs[event] = [])).push(cb)
    }
    off(event, offCb) {
        if (offCb) {
            if (this.#subs[event] && this.#subs[event].length)
                this.#subs[event] = this.#subs[event].filter(cb => cb !== offCb)
        } else {
            this.#subs[event] = []
        }
    }
}
```

`subs` 是 `EventEmitter` 私有属性(最新特性参考阮一峰老师的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/class))，通过 `on` 注册事件， `off` 注销事件， `emit` 触发事件

## 寄生组合继承

``` js
  function Super(foo) {
      this.foo = foo
  }
  Super.prototype.printFoo = function() {
      console.log(this.foo)
  }

  function Sub(bar) {
      this.bar = bar
      Super.call(this)
  }
  Sub.prototype = Object.create(Super.prototype)
  Sub.prototype.constructor = Sub
```

## ES6版继承

``` js
  class Super {
      constructor(foo) {
          this.foo = foo
      }
      printFoo() {
          console.log(this.foo)
      }
  }
  class Sub extends Super {
      constructor(foo, bar) {
          super(foo)
          this.bar = bar
      }
  }
```

`ES5` 的继承，实质是先创造子类的实例对象，然后将再将父类的方法添加到`this `上。

`ES6 `的继承，先**创造父类的实例对象**（所以必须先调用`
super `方法，然后**再用子类的构造函数修改`this`**

## THANK

[【进阶 6-2 期】深入高阶函数应用之柯里化](https://juejin.im/post/5ce288eee51d454f72302450)

[JavaScript专题之跟着 underscore 学节流 ](https://juejin.im/post/5947312a61ff4b006cf66be9)

[如何写出一个惊艳面试官的深拷贝?](https://juejin.im/post/5d6aa4f96fb9a06b112ad5b1#heading-13)

[头条面试官：你知道如何实现高性能版本的深拷贝嘛？](https://juejin.im/post/5df7175fe51d45582512962c)

[JSON.stringify 和 JSON.parse 的实现](https://www.jianshu.com/p/f1c8bcd16f71)
