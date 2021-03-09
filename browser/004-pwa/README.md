# 004-PWA

<motto></motto>

PWA（Progressive web apps，渐进式 Web 应用）运用现代的 Web API 以及传统的渐进式增强策略来创建跨平台 Web 应用程序。（来自 MDN）

先看看 PWA 有哪些核心技术，就知道它有哪些优势了

## App Shell
App Shell 架构是构建 Progressive Web App 的一种方式，这种应用能可靠且即时地加载到您的用户屏幕上，与本机应用相似。

这个模型包含界面所需要的最小资源文件，如果离线缓存，可以确保重复访问都有快速响应的特性，页面可快速渲染，网络仅仅获取数据。

或者这么理解， App Shell 就类似于原生app，没网络也可以本地启动。

## ServiceWork
PWA 的核心，上面说到缓存可以让页面尽快加载，但必须有网络的情况下才行，没网络下还想加载网页咋办？

ServiceWork 持久的离线缓存的能力就可以实现。

Service Worker 有以下功能和特性：

- 一个独立的 worker 线程，独立于当前网页进程，有自己独立的 worker context

- 一旦被 install，就永远存在，除非被手动 unregister

- 用到的时候可以直接唤醒，不用的时候自动睡眠

- 可编程拦截代理请求和返回，缓存文件，缓存的文件可以被网页进程取到（包括网络离线状态）

- 离线内容开发者可控

- 能向客户端推送消息

- 不能直接操作 DOM

- 必须在 HTTPS 环境下才能工作

- 异步实现，内部大都是通过 Promise 实现

js 是单线程的，ServiceWork 独立线程意味着不会阻塞js执行；可编程拦截代理请求和返回，可自定义文件缓存策略。

这些特点意味着开发者有足够的权限去操作缓存，让缓存做到优雅，效率达到极致

接下来核心是如何让设计缓存策略，

1. 缓存优先，先查询缓存，若存在，直接返回，不存在，请求服务，更新缓存

2. 服务端优先，不查询缓存，直接请求服务端，服务端失败才会查询缓存

3. 稳定优先，先查询缓存，有就读取，同时请求服务端更新资源

推荐大家看看开源的 wordbox 封装的缓存策略，策略更加丰富。

代码不复杂，主要是声明周期、与js线程间通信、api调用，就不贴上来了。

参考文档：

https://lavas.baidu.com/pwa/offline-and-cache-loading/service-worker/service-worker-introduction

https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps