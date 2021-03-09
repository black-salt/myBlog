# 002-文本溢出截断总结

<motto></motto>

## 一、单行文本溢出省略

### 1. 方案一(white-space)

#### CSS代码

``` css
overflow: hidden; //文字长度超出限定宽度, 则隐藏超出的内容
white-space: nowrap; //设置文字不可换行,只能一行显示, 
text-overflow: ellipsis; //规定当文本溢出时, 使用省略号来代表被截断的文本
```

| 特性       | 描述                       |
| ---------- | -------------------------- |
| 兼容性     | 无兼容问题                 |
| 响应性     | 响应式截断省略             |
| 显示       | 只在溢出时显示省略号       |
| 省略号位置 | 省略号位置刚好             |
| 适用场景   | 在单行文本场景下省略，均可 |

## 二、多行文本溢出省略

### 1. 方案一(-webkit-line-clamp)

#### 1.1 CSS代码

``` css
-webkit-line-clamp: 2; //控制文本最多显示2行。为了实现该效果，它需要组合其他的 WebKit 属性
display: -webkit-box; //和1结合使用,将对象作为弹性伸缩盒子模型显示
-webkit-box-orient: vertical; //和1结合使用,设置或检索伸缩盒对象的子元素的排列方式 
overflow: hidden; //文本溢出限定的宽度就隐藏内容
text-overflow: ellipsis; //规定当文本溢出时, 使用省略号来代表被截断的文本
```

| 特性       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 兼容性     | -webkit-line-clamp 属性只有 WebKit 内核的浏览器才支持(Firefox和IE不支持)， |
| 响应性     | 响应式截断省略                                               |
| 显示       | 只在溢出时显示省略号                                         |
| 省略号位置 | 省略号位置刚好                                               |
| 适用场景   | 多适用于移动端页面，因为移动设备浏览器更多是基于 WebKit 内核 |

###  2. 方案二(JS)

#### 2.1 JS代码

``` js
function ellipsizeMultiLineText(id) {
    var element = document.getElementById(id);
    var words = element.innerHTML.split(' ');
    while (element.scrollHeight > element.offsetHeight) {
        words.pop();
        element.innerHTML = words.join(' ') + '...';
    }
}
ellipsizeMultiLineText('block-with-text');
```

###  3. 方案三(JS)

#### 3.1 JS代码

``` js
const text = '这是一段很长的文本这是一段很长的文本这是一段很长的文本这是一段很长的文本这是一段很长的文本';
const totalTextLen = text.length;
const formatStr = () => {
    const ele = document.getElementsByClassName('demo')[0];
    const lineNum = 2;
    const baseWidth = window.getComputedStyle(ele).width;
    const baseFontSize = window.getComputedStyle(ele).fontSize;
    const lineWidth = +baseWidth.slice(0, -2);
    // 所计算的strNum为元素内部一行可容纳的字数(不区分中英文)
    const strNum = Math.floor(lineWidth / +baseFontSize.slice(0, -2));

    let content = '';

    // 多行可容纳总字数
    const totalStrNum = Math.floor(strNum * lineNum);
    const lastIndex = totalStrNum - totalTextLen;
    if (totalTextLen > totalStrNum) {
        content = text.slice(0, lastIndex - 3).concat('...');
    } else {
        content = text;
    }
    ele.innerHTML = content;
}

formatStr();

window.onresize = () => {
    formatStr();
};
```

### 4. 方案四(根据高度)

#### 4.1 CSS代码

``` css
overflow: hidden; //文本超出限定的高度就隐藏内容
line-height: 20px; //结合元素高度，高度固定的情况下，设定行高， 控制显示行数
max-height: 40px; //设定当前元素最大高度
```

| 特性       | 描述                                 |
| ---------- | ------------------------------------ |
| 兼容性     | 无兼容问题                           |
| 响应性     | 响应式截断省略                       |
| 显示       | 单纯截断文字, 不展示省略号           |
| 省略号位置 | 无省略号                             |
| 适用场景   | 文本溢出不需要显示省略号的情况，均可 |

### 5. 方案五(伪元素 + 定位)

#### 5.1 CSS代码

``` css
position: relative; //为伪元素绝对定位
overflow: hidden; //文本超出限定的高度就隐藏内容
position: absolute; //给省略号绝对定位
line-height: 20px; //结合元素高度,高度固定的情况下,设定行高, 控制显示行数
height: 40px; //设定当前元素高度

::after {}

//设置省略号样式
```

| 特性       | 描述                                         |
| ---------- | -------------------------------------------- |
| 兼容性     | 无兼容问题                                   |
| 响应性     | 响应式截断省略                               |
| 显示       | 无论文本是否溢出范围, 一直显示省略号         |
| 省略号位置 | 省略号显示可能不会刚刚好，有时会遮住一半文字 |
| 适用场景   | 对省略效果要求较低，文本一定会溢出元素的情况 |

#### 5.2 栗子

``` html
<style>
    .demo {
        position: relative;
        line-height: 20px;
        height: 40px;
        overflow: hidden;
    }

    .demo::after {
        content: "...";
        position: absolute;
        bottom: 0;
        right: 0;
        padding: 0 20px 0 10px;
    }
</style>

<body>
    <div class='demo'>这是一段很长的文本</div>
</body>
```

### 6. 方案五(利用 Float 特性)

#### 6.1 CSS代码

``` css
line-height: 20px; //结合元素高度,高度固定的情况下,设定行高, 控制显示行数
overflow: hidden; //文本溢出限定的宽度就隐藏内容
float: right/left; //利用元素浮动的特性实现
position: relative; //根据自身位置移动省略号位置, 实现文本溢出显示省略号效果
word-break: break-all; //使一个单词能够在换行时进行拆分
```

| 特性       | 描述                                         |
| ---------- | -------------------------------------------- |
| 兼容性     | 无兼容问题                                   |
| 响应性     | 响应式截断省略                               |
| 显示       | 文本溢出时才显示省略号                       |
| 省略号位置 | 省略号显示可能不会刚刚好，有时会遮住一半文字 |
| 适用场景   | 对省略效果要求较低，多行文本响应式截断的情况 |

#### 6.2 栗子

``` html
<style>
    .demo {
        background: #099;
        max-height: 40px;
        line-height: 20px;
        overflow: hidden;
    }

    .demo::before {
        float: left;
        content: '';
        width: 20px;
        height: 40px;
    }

    .demo .text {
        float: right;
        width: 100%;
        margin-left: -20px;
        word-break: break-all;
    }

    .demo::after {
        float: right;
        content: '...';
        width: 20px;
        height: 20px;
        position: relative;
        left: 100%;
        transform: translate(-100%, -100%);
    }
</style>

<body>
    <div class='demo'>这是一段很长的文本</div>
</body>
```

