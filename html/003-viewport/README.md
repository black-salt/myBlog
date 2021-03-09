# 003 - viewport

<motto></motto>

```html
<meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no" />
```

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| width         | 设置viewport宽度，为一个正整数，或字符串‘device-width’       |
| device-width  | 设备宽度                                                     |
| height        | 设置viewport高度，一般设置了宽度，会自动解析出高度，可以不用设置 |
| initial-scale | 默认缩放比例（初始缩放比例），为一个数字，可以带小数         |
| minimum-scale | 允许用户最小缩放比例，为一个数字，可以带小数                 |
| maximum-scale | 允许用户最大缩放比例，为一个数字，可以带小数                 |
| user-scalable | 是否允许手动缩放                                             |