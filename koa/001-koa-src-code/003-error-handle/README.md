# 003-Koa源码解读：错误处理

在 Node.js 里，我们对于错误处理的方式主要有以下几种：

- throw / try / catch 方法
- callback(err, data) 回调形式
- 通过 EventEmitter 触发一个 error 事件

在Koa里try/catch和EventEmitter发挥了很大的作用。

相信在看完上面的application.js后，你会发现在三个地方有涉及到错误处理：

* 一个是在 `callback()` 中的 `if (!this.listenerCount("error")) this.on("error", this.onerror);` ，是在Application类里实现的方法 `onerror()` 。

* 另一个是 `onFinished(res, onerror);` 

  在context.js里实现的方法 `onerror()` ，并且在 `handleRequest()方法` 中进行了调用，由于使用了 `onFinished()函数` ，上文也有对该函数进行相应的解释，该函数主要是在HTTP响应结束时调用传入的回调函数，在这里使用的回调函数就是 `ctx.onerror()方法` 。

* 还有一个就是在 `return fnMiddleware(ctx).then(handleResponse).catch(onerror);` 

  在执行所有中间件的时候，若有异常，则Promise的状态会变成reject状态，所以会执行 `catch(onerror)` 回调，这里的 `onerror函数` 并非Application类的方法，而是context.js中的onerror，context.js中的onerror的核心是在 this.app.emit('error', err, this); this.app就是对application的引用，当context.js的onerror触发时，会触发application实例的error事件。

在context.js中，可以看到有借助http-error模块的createError()封装的throw()方法，还有onerror方法。

也就是说，Koa 继承了 **Emitter**，有了注册(发布)和监听(订阅)事件的能力后，***为了能够处理可能在任意时间抛出的异常***，所以需要**监听(订阅) error 事件**。

那根据上面我们看过的源码，可以知道在Koa中，error 的处理主要是针对的对象不同，有两个：

* 一个针对的是各个中间件内部发生错误时的**错误日志输出**（在 `compose()函数` 和 `handleRequest()方法` 之间挂载，也就是在 `创建'洋葱模型函数'` 和 `执行'洋葱模型函数'` ）

* 一个是针对的是**一次HTTP请求和响应的错误处理**，在 `handleRequest()方法` 中结合 `onFinished()函数` 后，可以监听一次HTTP的请求的 `关闭` 、 `完成` 或 `出现错误` 的事件，然后调用context的 `onerror()` 这个回调函数。

``` js
// application.js
// 在'创建洋葱模型函数'和'调用洋葱模型函数'之间，
// koa 会挂载一个默认的错误处理【运行时确定异常处理】，其实就是要打印堆栈信息
if (!this.listenerCount("error")) this.on("error", this.onerror); // callback()方法中的代码
// 在这行不知道该写点什么。。。就当做是分割线好啦
onerror(err) {
    if (!(err instanceofError))
        thrownewTypeError(util.format("non-error thrown: %j", err));

    // 如果没有在最外层指定错误事件侦听器
    // (也就是没有app.on('error',(err)=>{}))，
    // 那么将使用 app.onerror，除非 error.expose 为 true 
    // 或 app.silent 为 true 或 error.status 为 404，否则只简单记录错误。
    
    // 如果是状态码是404，则无需打印堆栈信息，直接返回即可
    if (404 == err.status || err.expose) return; 
    if (this.silent) return;

    const msg = err.stack || err.toString();
    console.error();
    console.error(msg.replace(/^/gm, "  "));
    console.error();
}

// -------------------------------------------------------------------------------

// context.js
// 默认的错误处理
// 默认的错误处理程序本质上是中间件链开始时的一个 try-catch。
// 创建你自己的错误处理程序的示例:
// app.use(async (ctx, next) => {
//   try {
//     await next();
//   } catch (err) {
//     // will only respond with JSON
//     ctx.status = err.statusCode || err.status || 500;
//     ctx.body = {
//       message: err.message
//     };
//   }
// })
// 要使用不同的错误处理程序，只需在中间件链的起始处放置另一个 try-catch，并在那里处理错误。
// 但是，默认错误处理程序对于大多数用例来说都是足够好的。
// 它将使用状态代码 err.status，或默认为500。如果 err.expose 是 true，
// 那么 err.message 就是答复。
// 否则，将使用从错误代码生成的消息（例如，对于代码500，将使用消息“内部服务器错误”）。

// 所有消息头将从请求中清除，但是任何在 err.headers 中的消息头将会被设置。
// 你可以使用如上所述的 try-catch 来向此列表添加消息头。
// 但是记得如果使用了try/catch来捕获该错误，
// 那么该错误就将止步于这个try/catch，
// 在app.js里就无法使用app.on('error', (err)=>{...})来捕获了，
// 如果还有需要使用app.on来进行捕获，
// 则需在try/catch中手动释放error事件 ctx.app.emit('error', err, ctx);
onerror(err) {
    // don't do anything if there is no error.
    // this allows you to pass `this.onerror` 
    // to node-style callbacks.
    // 没有错误就直接返回
    if (null == err) return;

    // 先检查下传进来的参数是不是个Error，不是的话就报TypeError这个错
    if (!(err instanceof Error)) err = new Error(util.format('non-error thrown: %j', err));

    let headerSent = false;
    if (this.headerSent || !this.writable) {
        headerSent = err.headerSent = true;
    }

    // delegate
    // emit就是Emitter的一个方法，因为Application类继承了Emitter，
    // 所以app这个实例可以使用emit方法
    
    // emit方法就是，发射event事件，传递若干可选参数到事件监听器的参数表
    
    // this.app就是对application的引用，
    // 当context.js的onerror触发时，会触发application实例的error事件
    this.app.emit('error', err, this); 

    // nothing we can do here other
    // than delegate to the app-level
    // handler and log.
    if (headerSent) {
        return;
    }

    const {
        res
    } = this;

    // first unset all headers
    /* istanbul ignore else */
    // 首先移除所有headers
    if (typeof res.getHeaderNames === 'function') {
        res.getHeaderNames().forEach(name => res.removeHeader(name));
    } else {
        res._headers = {}; // Node < 7.7
    }

    // then set those specified
    // 然后设置那些指定的error的headers
    this.set(err.headers);

    // force text/plain
    // 强制将类型改为text
    this.type = 'text';

    // ENOENT support
    if ('ENOENT' == err.code) err.status = 404;

    // default to 500
    // 默认将status改为500
    if ('number' != typeof err.status || !statuses[err.status]) err.status = 500;

    // respond
    // 响应
    const code = statuses[err.status];
    const msg = err.expose ? err.message : code;
    this.status = err.status;
    this.length = Buffer.byteLength(msg);
    res.end(msg); // 原生Node的响应结束
}
```

## 1. 错误处理类型

### （1）ctx.throw()

koa 提供了 ctx.throw() 方法（该方法在context.js源码中定义，主要使用http-error模块进行createError）用来抛出错误，我们可以直接 ctx.throw(500) 就是抛出500的错误，类似下面例子：

```js
app.use(async (ctx,next) => {
    ctx.throw(500);
});
```

值得注意的是，如果将 ctx.response.status 设置成对应的状态码，效果跟 ctx.throw() 是一样的，比如设置成 404

```js
app.use(async (ctx,next) => {
    ctx.response.status = 404;
    ctx.response.body = 'Page Not Found';
});
```

### （2） try/catch
为了更加方便的处理错误，使用 try/catch 将错误捕获是比较好的，但是每个请求都写感觉太麻烦，其实我们可以定义一个中间件负责对所有的错误进行处理。

```js
app.use(async (ctx,next) => {
    try{
        await next()
    }catch(err){
        ctx.response.status = err.statusCode || err.status || 500;
        ctx.response.body = {
            message: err.message
        };
    }
});

// --------------------------------------------------------------
// 下面是自定义一个中间件对所有错误进行处理

// 这个middleware处理在其它middleware中出现的异常
// 并将异常消息回传给客户端：{ code: '错误代码', msg: '错误信息' }
const errorHandler = (ctx, next) => {
  return next().catch(err => {
    if (err.code == null) {
      logger.error(err.stack)
    }
    ctx.body = {
      code: err.code || -1,
      data: null,
      msg: err.message.trim()
    }

    ctx.status = 200 // 保证返回状态是 200, 这样前端不会抛出异常
    return Promise.resolve()
  })
}
```

### （3）监听 error 事件

其实，koa 还有一个 error 事件监听，如果代码运行发生错误，就会发出这个事件，我们也可以用它来处理错误

```js
app.on('error', (err, ctx) =>
    console.error('server error', err);
);
```

有一点需要注意的是，如果错误已经被 try/catch 捕获，那么就不会出发 error 事件。这是就必须释放 error 事件，
调用 ctx.app.emit() ,手动释放 error 事件，koa 监听函数才会生效，否则监听事件是不会触发的。

```js
app.use(async (ctx,next) => {
    try{
        await next()
    }catch(err){
        ctx.response.status = err.statusCode || err.status || 500;
        ctx.response.body = {
            message: err.message
        };
        // 手动释放error事件
        ctx.app.emit('error', err, ctx);
    }
});
// 继续触发error事件
app.on('error',(err) => {
    console.error('server error', err.message);
    console.error(err);
});
```

## 2. 应用实例

Koa提供了可以进行统一错误处理的办法。

如果要在Koa中统一处理错误的话，只需要让koa实例监听`'error'`事件就可以了。则所有的中间件逻辑错误都会在这里被捕获并处理。如下所示：

```js
app.on('error', err => {
    log.error('server error', err)
});
// -------------------------------------------------------------------------------
// 想好好处理一下的话，可以这样做
var server = http.createServer(app.callback())
server.listen(port)
server.on('error', onError)
function onError(error) {
    if (error.syscall !== 'listen') {
        throw error
    }

    var bind = typeof port === 'string' ?
        'Pipe ' + port :
        'Port ' + port

    // handle specific listen errors with friendly messages
    switch (error.code) {
        case 'EACCES':
            console.error(bind + ' requires elevated privileges')
            process.exit(1)
            break
        case 'EADDRINUSE':
            console.error(bind + ' is already in use')
            process.exit(1)
            break
        default:
            throw error
    }
}
```

## THANK

[读 koa2 源码后的一些思考与实践](https://mp.weixin.qq.com/s?__biz=MzUxNzk1MjQ0Ng==&mid=2247484544&idx=1&sn=3180d250dfa5c76089d3b7bc2ddb1f5a&chksm=f9910251cee68b4773df8f8029682cad877add495c41130071f40fda520e014342c47536cc29&mpshare=1&scene=23&srcid=&sharer_sharetime=1580813890425&sharer_shareid=b12d04dea52958e10ae6bac1a582b72b#rd)

[学习 koa 源码的整体架构，浅析koa洋葱模型原理和co原理](https://www.lxchuan12.cn/koa/#前言)

