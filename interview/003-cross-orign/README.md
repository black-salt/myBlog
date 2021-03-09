# 003:前端跨域问题

<motto></motto>


## 1. 为什么会出现跨域问题？

跨域问题的出现，主要是因为同源策略（same origin policy）。

同源策略会阻止一个域下的JS脚本不可以和另一个域下的内容进行交互，所以就出现了跨域问题。

同源策略是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。

## 2. 同源策略

### 什么是源？

`源 = 协议 + 域名 + 端口号`

如果两个`url`的协议、域名、端口号完全一致，那么这两个`url`就是`同源`的。

我们可以通过`window.origin`或`location.origin`得到当前源。

### 什么是同源策略？

**同源策略即**：不同源之间的页面，不可以互相访问数据。

### 为什么会有同源策略？

那你可能想问了，跨域是由同源策略导致的，那么为什么要有同源策略呢？

之所以需要使用同源策略，就是`为了保护用户的隐私`。

以微信为例，源为 `https://user.weixin.com`，假设当前用户已经登录，并且`AJAX`请求 `/userinfo.json` 可以获取用户信息。

这个时候黑客来了，他把 `https://user-winxin.com`分享给你，实际上这是一个钓鱼网站，你点开这个网页，这个网页也请求你的用户信息`https://user.weixin.com/userinfo.json。`

请问，这个时候你的用户信息是不是就被黑客给偷走了？

### 问题的根源

之所以会出现这个问题，其根源就在于**无法区分发送者**。

微信里面的`JS`和黑客的`JS`发送到请求几乎没有区别（`referer`区别）

但是如果后台的开发者没有检查 `referer`，那么就完全没有区别。

所以，如果没有同源策略，任何页面都能偷走微信里面的数据，甚至是支付宝里面的余额。

### 安全原则

有的小伙伴可能会问，既然`referer`有区别，那检查`referer`不就好了？

**安全原则**：安全链条上的强度取决于安全链条上最弱的一环。

同时，万一这个网站的后端开发者是一个傻叉呢？

所以浏览器应该主动预防这种偷数据的行为。

总之，为了保护用户的隐私，浏览器设置了严格的同源策略。

如果浏览器不限制跨域，一定是这个浏览器出现了`bug`。

## 3. 什么是跨域

### 跨域概念

跨域，即浏览器试图执行其他网站的脚本。但是由于同源策略的限制，导致我们无法实现跨域。

当一个请求url的协议、域名(包括主域名、子域名)、端口三者之间任意一个与当前页面url不同即为跨域。

| 当前页面url                  | 被请求页面url                      | 是否跨域 | 原因                      |
| ---------------------------- | ---------------------------------- | -------- | ------------------------- |
| http://www.baitang.com/      | http://www.baitang.com/index.html  | 否       | 同源                      |
| http://www.baitang.com/      | https://www.baitang.com/index.html | 跨域     | 协议不同（http/https      |
| http://www.baitang.com/      | http://www.baidu.com/              | 跨域     | 主域名不同（baitang/baidu |
| http://www.baitang.com/      | http://blog.baitang.com            | 跨域     | 子域名不同（www/blog      |
| http://www.baitang.com:8080/ | http://www.baitang.com:7001        | 跨域     | 端口号不同                |

### 为什么`www.baitang.com`访问`blog.baitang.com`也算跨域？

因为历史上，出现过**不同的公司共用域名**，`www.baitang.com`和`blog.baitang.com`不一定是同一个网站，浏览器谨慎起见，认为这是不同的源。

### 为什么不同端口也算跨域？

原因同上，一个端口一个公司的情况也不是没有的。

**记住**：安全链条的强度取决于最弱的一环，所有和安全相关的问题都要谨慎对待。

### 为什么两个网站的`IP`一样，也算跨域？

原因同上，因为`IP`也是可以共用的。

### 为什么可以跨域使用`CSS`、`JS`和图片等？

同源策略限制的是`数据访问`，我们引用`CSS`、`JS`和图片的时候，其实并不知道其内容，我们只是在引用。

### 为什么同源是浏览器行为的疑问？

你在看到同源策略是浏览器行为的时候可能会有疑问：

同源限制既然是浏览器行为，为什么服务端设置CORS之后，浏览器就能发出跨域请求了，如果是同源策略是浏览器行为的话，理论上浏览器应该是发不出跨域请求，那么服务器如何根据Origin的值来判断是否接受本次请求呢？

**解答：**

同源策略是浏览器行为，实际上跨域请求是能发送的，服务器也能正常接收，数据也能正常返回，只是浏览器把你跨域请求的响应数据给屏蔽了而已。

## 4、跨域解决方法

1. 设置document.domain解决无法读取非同源网页的 Cookie问题

2. 跨文档通信 API：window.postMessage()

3. JSONP

4. CORS

5. Proxy

作为开发人员，最关心的跨域一般是2种交互的跨域，即`Proxy`和`CORS`，很多开发只图一时方便，使用了Proxy，在打包后就发现又有跨域了，不知道怎么解决，下面我们通过实例一点点给大家解析。

### 跨域解决之Proxy

现在项目一般都使用脚手架，即使用webpack，那可以使用webpack自带的proxy特性来处理跨域。

这种方式是开发最常用的，但是打包后就有问题了，因为打包后就不存在proxy了，跨域还是会存在，那应该怎么解决？

### 跨域解决之CORS(跨域资源共享Cross-Origin Resource Sharing)

实现`CORS`通信的关键是服务器。只要服务器配置了`CORS`接口，前端无需任何处理，就可以跨域通信，无论是**在开发时还是部署时都是OK**的。

下面，我们把proxy注释掉，使用CORS方式处理，如下：

**设置允许所有域名跨域：**

```js
app.all("*",function(req,res,next){
    //设置允许跨域的域名，*代表允许任意域名跨域
    res.header("Access-Control-Allow-Origin","*");
    //允许的header类型
    res.header("Access-Control-Allow-Headers","content-type");
    //跨域允许的请求方式 
    res.header("Access-Control-Allow-Methods","DELETE,PUT,POST,GET,OPTIONS");
    if (req.method.toLowerCase() == 'options')
        res.send(200);  //让options尝试请求快速结束
    else
        next();
}
```

**设置允许指定域名“http://www.baitang.com”跨域：**

```js
app.all("*",function(req,res,next){
    //设置允许跨域的域名，*代表允许任意域名跨域
    res.header("Access-Control-Allow-Origin","http://www.baitang.com");
    //允许的header类型
    res.header("Access-Control-Allow-Headers","content-type");
    //跨域允许的请求方式 
    res.header("Access-Control-Allow-Methods","DELETE,PUT,POST,GET,OPTIONS");
    if (req.method.toLowerCase() == 'options')
        res.send(200);  //让options尝试请求快速结束
    else
        next();
}
```

**设置允许多个域名跨域：**

```js
app.all("*",function(req,res,next){
    if( req.headers.origin.toLowerCase() == "http://www.baitang.com"
        || req.headers.origin.toLowerCase() =="http://127.0.0.1" ) {
        //设置允许跨域的域名，*代表允许任意域名跨域
        res.header("Access-Control-Allow-Origin", req.headers.origin);
    }
    //允许的header类型
    res.header("Access-Control-Allow-Headers", "content-type");
    //跨域允许的请求方式 
    res.header("Access-Control-Allow-Methods", "DELETE,PUT,POST,GET,OPTIONS");
    if (req.method.toLowerCase() == 'options')
        res.send(200);  //让options尝试请求快速结束
    else
        next();    
}
```

**如果允许的域名较多，可以将允许跨域的域名放到数组当中：**

```js
app.all("*",function(req,res,next){
    var orginList=[
        "http://www.baitang.com",
        "http://www.alibaba.com",
        "http://www.qq.com",
        "http://www.baidu.com"
    ]
    if(orginList.includes(req.headers.origin.toLowerCase())){
        //设置允许跨域的域名，*代表允许任意域名跨域
        res.header("Access-Control-Allow-Origin",req.headers.origin);
    }
    //允许的header类型
    res.header("Access-Control-Allow-Headers", "content-type");
    //跨域允许的请求方式
    res.header("Access-Control-Allow-Methods", "DELETE,PUT,POST,GET,OPTIONS");
    if (req.method.toLowerCase() == 'options')
        res.send(200);  //让options尝试请求快速结束
    else
        next();
}
```

配置了cors后，接口就可以随便访问了。

#### 简单请求&复杂请求

`CORS`跨域分为两种请求，一种是**简单请求**，另外一种就是**复杂请求**。

1. 简单请求：
    1): 请求方式只能是：`head`，`get`，`post`
    2): 请求头允许的字段：`Accept`，`Accept-Language`，`Content-Language`，`Last-Event-ID`
    `Content-Type`：application/x-www-form-urlencoded、multipart/form-data、text/plain  三选一

2. 复杂请求：不满足上面的，都是我啦！

##### 简单请求

简单请求的实现具体来说就是在信息头中加入一个`Origin`字段：

```http
GET /cors HTTP/1.1
Origin: http://blog.baitang.com 
Host: api.baitang.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0
...
```

浏览器：你小子要跨域是吧，我得问问服务器大哥肯不肯！往请求头添加`origin`亮一下牌面

服务器：诶，你是谁，我来看看你的origin，嗯嗯，可以，符合我的要求，放行！顺便告诉你，老夫的规矩！

```
Access-Control-Allow-Origin`，标识允许哪个域的请求。
当然，如果服务器不通过，根本没有这个字段，接着触发`XHR`的`onerror`，再接着你就看到浏览器的提示`xxx的服务器没有响应Access-Control-Allow-Origin字段
```

`Origin`的作用就是用来说明本次请求来自哪个源，服务器会根据`Origin`的值来判断是否接受本次请求。

如果`Origin`所表示的源不被服务器接受，即浏览器发现回应的信息头中没有`Access-Control-Allow-Origin`字段，就会自动抛出一个错误。

注意：这种错误是无法通过状态码识别的，这也是通过`CORS`实现跨域请求的一个弊端。

如果`Origin`所表示的源被服务器端所接受，那么服务器就会返回如下响应：

```http
Access-Control-Allow-Origin: http://api.baitang.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: baitang
Content-Type: text/html; charset=utf-8
```

`Access-Control-Allow-Origin` :该字段是必须的。它的值要么是请求时`Origin`字段的值，要么是一个`*`，表示接受任意域名的请求

`Access-Control-Allow-Credentials`: 该字段可选。它的值是一个布尔值，表示是否允许发送`Cookie`。默认情况下，`Cookie`不包括在`CORS`请求之中。设为`true`，即表示服务器明确许可，`Cookie`可以包含在请求中，一起发给服务器。这个值也**只能设为`true`**，如果服务器不要浏览器发送`Cookie`，删除该字段即可。(注意：如果要发送`cookie`，不仅要进行上述的设置，还要在`AJAX`请求中设置`withCredentials`属性）

`Access-Control-Expose-Headers`:该字段可选。`CORS`请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定

##### 复杂请求:

最常见的情况，当我们使用`put`和`delete`请求时，浏览器会先发送`option`（预检）请求，不过有时候，你会发现并没有，这是后面我们会讲到缓存。

**预检请求**

与简单请求不同的是，option请求多了2个字段：
 `Access-Control-Request-Method`：该次请求的请求方式
 `Access-Control-Request-Headers`：该次请求的自定义请求头字段

```js
var url = 'http://api.wang.com/cors';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
```

服务器检查通过后，做出响应：

```http
//指定允许其他域名访问
'Access-Control-Allow-Origin:http://172.20.0.206'//一般用法（*，指定域，动态设置），3是因为*不允许携带认证头和cookies
//是否允许后续请求携带认证信息（cookies）,该值只能是true,否则不返回
'Access-Control-Allow-Credentials:true'
//预检结果缓存时间,也就是上面说到的缓存啦
'Access-Control-Max-Age: 1800'
//允许的请求类型
'Access-Control-Allow-Methods:GET,POST,PUT,POST'
//允许的请求头字段
'Access-Control-Allow-Headers:x-requested-with,content-type'
```

这里有个注意点：`Access-Control-Request-Method`，`Access-Control-Request-Headers`返回的是满足服务器要求的所有请求方式，请求头，不限于该次请求。

服务器收到预检请求之后，检查了`Origin`、`Access-Control-Request-Method`和`Access-Control-Request-Headers`字段以后，确认允许跨源请求，才会做出相应的回应。

```http
Access-Control-Allow-Origin: http://api.wang.com
Content-Type: text/html; charset=utf-8
```

#### CORS存在的问题

不支持`IE8/9`，如果要在`IE8/9`使用`CORS`跨域需要使用`XDomainRequest`对象来支持`CORS`。

### 跨域解决之Proxy

#### 什么是JSONP？

我们在跨域的时候由于当前的浏览器不支持 `CORS` 或者因为某些条件不支持 `CORS`，我们必须使用另外一种方式来跨域，于是我们就请求一个 `JS` 文件，这个 `JS` 文件会执行一个回调，回调里面就有我们需要的数据。

```js
let script = document.createElement('script');
script.src = 'http://www.test.cn/login?username=user1&callback=callback';
document.body.appendChild(script);
function callback(res) {
  console.log(res);
}
```

#### 回调函数的名字是什么？

回调的名字是可以随机生成的的一个随机数，我们把这个名字当成 `callback` 的参数传给后台，后台会把这个函数再次返回给我们并执行

#### JSONP跨域优点

- 兼容`ie`
- 可以跨域

#### JSONP跨域缺点

- 由于是 `script` 标签，所以读不到 `ajax` 那么精确的状态，不知道状态码是什么，也不知道响应头是什么，它只知道成功和失败。
- 不支持`post`（因为是 `script` 标签，所以只支持 `get` 请求）









