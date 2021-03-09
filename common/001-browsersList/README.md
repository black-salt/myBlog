# 001-BrowsersList配置

<motto></motto>

> 如今的工程化可以很好地帮助我们进行浏览器上的兼容，
>
> 我们可以通过配置 browserslist 一步到位。

我们可以在 `package.json` 中配置 `browserslist` ，配置好我们对浏览器的相关要求，就可以完成相关的兼容问题。

在`package.json`中添加的 `browserslist` 会被多个和浏览器兼容性相关的包共享使用：

如果 package.json 配置了 browserslist 对象, 那么需要兼容浏览器的包将自动匹配到 browserslist 并使用。

| 组件名               | 功能                        |
| -------------------- | --------------------------- |
| Autoprefixer postcss | 添加 css 浏览器厂商前缀     |
| bable - preset - env | 编译预设环境，添加 polyfill |
| eslint               |                             |

``` json
{ // package.json
	"browserslist": [ // 数组
		"> 1%",
		"last 2 versions"
	]
}
```

| 例子                       | 说明                                                  |
| -------------------------- | ----------------------------------------------------- |
| 1%                         | 全球超过1%人使用的浏览器                              |
| 58 in Us                   | 指定国家使用率覆盖                                    |
| last 2 versions            | 所有浏览器兼容到最后两个版本根据CanlUse.com追踪的版本 |
| Firefox ESR                | 火狐最新版本                                          |
| Firefox>20                 | 指定浏览器的版本范围                                  |
| not ie<=8                  | 方向排除部分版本                                      |
| Firefox 12.1               | 指定浏览器的兼容到指定版本                            |
| unreleased versions        | 所有浏览器的beta测试版本                              |
| unreleased Chrome versions | 指定浏览器的测试版本                                  |
| since 2013                 | 2013年之后发布的所有版本                              |