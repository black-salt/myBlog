# 003-简单请求和复杂请求

<motto></motto>

> 真是尴尬，昨晚的一个笔试，问到了简单请求和复杂请求，没记全乎，此题凉凉
>
> 在这儿好好记录一下

我们在使用 CORS 的时候，HTTP 请求会被划分为两类，简单请求和复杂请求，而这两种请求的区别主要在于是否会触发 CORS 预检请求。

## CORS 的原理

> 跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，**对那些可能对服务器数据产生副作用的 HTTP 请求方法**（特别是 `GET` 以外的 HTTP 请求，或者搭配某些 MIME 类型的 `POST` 请求），浏览器必须首先使用 `OPTIONS` 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。**在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据**）
>
> ​																																																						——MDN

所以我们可以知道，

1、跨域资源共享标准给我们新增了一组 HTTP 首部字段，服务器可以通过这些字段来控制浏览器有权访问哪些资源。

2、为了安全起见，将请求方式分为两类，一类不会预先发送 `OPTIONS 请求`，一些会预先发送 `OPTIONS 请求`。

3、 `GET` 以外的 HTTP 请求，或者搭配某些 MIME 类型的 `POST` 请求会触发 `OPTIONS 请求`。

4、服务器验证 `OPTIONS 请求` 完成后才会允许发送剩下的 HTTP 请求。

## 简单请求

> 简单请求就是满足以下两个条件的请求：

##### 请求方法是以下三种之一：

- HEAD
- GET
- POST

##### HTTP的头信息不超出以下几种字段:

- Accept
- Accept-Language
- Content-Language
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain



简单请求的部分响应头及解释如下：

> **Access-Control-Allow-Origin（必含）**- 不可省略，否则请求按失败处理。该项控制数据的可见范围，如果希望数据对任何人都可见，可以填写"*"。  
>
> **Access-Control-Allow-Credentials（可选）** – 该项标志着请求当中是否包含cookies信息，只有一个可选值：true（必为小写）。如果不包含cookies，请略去该项，而不是填写false。这一项与 XmlHttpRequest 对象当中的 withCredentials 属性应保持一致，即 withCredentials 为true时该项也为true；withCredentials为false时，省略该项不写。反之则导致请求失败。  （注：跨域请求要想带上 cookie，必须要在 ajax 请求里加上: `xhr.withCredentials = true;`）
>
> **Access-Control-Max-Age（可选）** – 过期前，复杂请求无需再发 OPTIONS请求

## 复杂请求：

> 除了简单请求剩下的就都是复杂请求。

复杂请求在正式请求前都会有预检请求，在浏览器中都能看到有OPTIONS请求，用于向服务器请求权限信息的。