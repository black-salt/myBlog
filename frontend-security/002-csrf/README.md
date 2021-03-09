# 002:CSRF(Cross Site Request Forgery)跨站请求伪造攻击

<motto></motto>

## 概念

攻击者盗用了用户的身份，以用户的名义发送恶意请求，对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。 

## CSRF 漏洞检测

将原来可正常响应的请求去掉 `Referer `字段后再重新发起请求，若该请求仍然有效，那么基本上可以确定存在 CSRF 漏洞。

## 防御 CSRF 攻击

### 1. 验证 HTTP 的 Referer 字段

我们可以通过验证 Referer 来判断该请求是否为第三方网站发起的。

在后端接收到请求的时候，可以通过请求头中的 Referer 请求头来判断请求来源，然后判断请求来源是否在白名单列表中。

### 2. SameSite

Chrome 可以对 Cookie 设置 `SameSite` 属性。

`SameSite` 可以让 Cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）。

`SameSite `可以有下面三种值：

1. **Strict** ：浏览器将只发送同一站点(需要协议和顶级域名以及二级域名完全相同)请求的 Cookie。
2. **Lax** ：只允许在 `get方法提交表单` 和 `a标签发送get请求` 请求携带 Cookie。

3. **None** ：无论是否跨站，请求都会自动携带上 Cookie。

之前默认是 `None` 的，Chrome80 后默认是 `Lax` 。

该属性可以很大程度减少 CSRF 的攻击。

### 3. 在请求地址中添加 token 并验证

服务器在收到登录的请求并验证用户登录成功后，下发一个随机 `Token`，每次发起请求时将 `Token` 携带上，服务器验证 `Token` 是否有效。

#### Token 生成方式：

- 生成过程: token=salt-hash, hash=哈希函数(salt+密码）
- 解密过程: const [salt, hash] = token.split('-'), 重新计算 hash = 哈希函数(salt+密码), 然后比较

#### Token 暴露给客户端的方式

这个 Token 需要向客户端公开，通常是将其包含在初始页面内容中。

一种可能是将它存储在一个 HTML 的标签中，在这个标签中，可以在请求的时候通过 JavaScript 查询到它的值。

方式一：

```html
<form action="/form" method="POST">
  <input type="hidden" name="_csrf" value="csrfToken"> // 注意这里隐藏了csrfToken
  
  <input type="text" name="balabala">
  <button type="submit">Submit</button>
</form>
```

方式二：

```html
<meta name="csrf-token" content="{{csrfToken}}">
...
<script>
    // Read the CSRF token from the <meta> tag
	var token = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

	// Make a request using the Fetch API
	ajax('/form',{
    	headers:{
        	'CSRF-Token': token // is the csrf token as a header
    	},
    	method:'POST',
    	body:{
        	data:'data'
    	}
	})
</script>
```

### 4. 防抓包

使用 HTTPS 协议，当请求的信息被抓包工具抓包后,也无法修改提交的数据.

