# DOM 和 BOM 的总结

## 定义

* DOM
  + 文档对象模型(Document Object Model), 对html文档页面进行编程控制的系列API接口
  + 提供对文档的结构化描述, 定义方式对结构进行访问并进行修改结构, 样式和内容
  + 顶级对象时是document
* Bom
  + 浏览器对象模型(Browser Object Model)
  + 提供系列与浏览器相关api
  + 顶级对象是window
* Bom+Dom结构

![img](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/bomdomcc.88bbad64.png)

* Dom继承关系图



![img](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/nodetree.9e0ddeaf.png)

## DOM

### dom树

![img](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/domtree.fba945b8.jpg)

* 文档: 一个页面就是一个文档, DOM中使用document表示
* 元素: 页面中的所有标签都是元素, DOM中使用element表示
* 节点: 网页中的所有内容都是节点(标签, 属性, 文本, 注释等), DOM中使用node表示

### document 对象

> TIP
>
> document对象对应整个文档, 整个文档相关的操作和编程的api都是通过document对象进行实现

* 常用属性

  + URL: 获取当前文档的URL地址, 只读



``` js
    document.URL
```

  + title: 获取当前文档Head中的title的文字内容, 可写入修改



``` js
    document.title
```

* dom树相关操作

  + getElementById(id) => 根据id, 返回Element✅
  + getElementsByTagName(tageName) => 根据标签名(如果是*, 返回所有), 返回HTMLCollection集合
  + getElementsByName(name) => 根据name属性, 返回NodeList集合
  + querySelector(selectors) => 根据css选择器✅
  + querySelectorAll(selectors) => 根据css选择器✅, 多个选择器以逗号分隔querySelectorAll( 'p, a')

* 获取body, html



``` js
  // 获取body元素
  const bodyElem = document.body;
  // 获取Html元素
  const htmlElem = document.documentElement;
```

### 元素对象

* Element



``` js
  // Element对象是所有标签元素的基础对象,封装了所有标签元素的公共方法与属性
  // 常用属性
  Element.attributes // 获得所有属性key-value集合
  Element.className // 获得类名(可读写)=>idea:elem.className = 'pink',然后再css定义
  // 注意:此方法会覆盖原有类名
  Element.classList // 返回一个元素的类属性的实时 DOMTokenList 集合(只读),通过其内部方法修改
  // ie10+,
  Element.id // 获取元素id
  Element.innerHTML // 获取元素内部包裹的html标签字符串(可读写)
  Element.innerText // 获取元素内部文本,标签不识别会直接进行显示(可读写)
  Element.tagName // (只读)获取元素的标签名字符串
  Element.style // 获取/修改style样式,如Element.style.backgroundColor
  // 常用方法
  Element.getAttribute(attrName) // 返回属性的字符串值,适合获取自定义属性data-
  Element.removeAttribute(attrName) // 从指定的元素中删除一个属性
  Element.setAttribute(attrName, value) // 设置属性
  Element.hasAttribute(attrName) // 检测属性是否存在
  Element.getElementsByClassName() // 获取后代元素根据className
  Element.getElementsByTagName() // 获取后代元素根据tagName
      ...

      Element.dataset.index | Element.dataset['index'] // 用于获取data-自定义属性(h5新增),支持驼峰命名获取(ie11+)
```

* HTMLCollection



``` js
  // HTMLCollection对象,是伪数组。元素的动态集合,提供了用来从该集合选择元素的方法和属性,当其所包含的文档结构发生改变时,会自动更新.
  // 常用属性
  HTMLCollection.length // 返回集合中子元素的数组
  // 常用方法
  HTMLCollection.item() // 根据给定的索引(从0开始)，返回具体的节点
  HTMLCollection.namedItem() // 根据 Id 返回指定节点，或者作为备用，根据字符串所表示的 name 属性来匹配
```

### 节点操作

> WARNING
>
> 一般的, 节点至少拥有nodeType(节点类型), nodeName(节点名称), nodeValue(节点值)

* 元素节点 nodeType为 1
* 属性节点 nodeType为 2
* 文本节点 nodeType为 3(文本几点包含文字, 空格, 换行等)

#### 节点层级

* 父级节点



``` js
  node.parentNode // 找不到返回null
```

* 子节点



``` js
  parentNode.childNodes // 注意: 会返回3种子节点=>如果指向某种节点,可通过nodeType判断
  parentNode.firstChild // 返回第一个子节点(会包括3种节点)
  parentNode.lastChild // 获取最后一个子节点(会包括3种节点)

  parentNode.children // 只获取子元素节点
  parentNode.firstElementChild // 获取第一个子元素节点(ie9+)
  parentNode.lastElementChild // 获取最后一个子元素节点(ie9+)
```

* 兄弟节点



``` js
  node.nextSibling // 当前元素的下一个兄弟节点(会包含3种节点)
  node.previousSibling // 当前元素的上一个兄弟节点(会包含3种节点)

  node.nextElementSibling // 当前元素的下一个兄弟元素节点(ie9+)
  node.previousElementSibling // 当前元素的上一个兄弟元素节点(ie9+)
```

* 节点创建与添加



``` js
  // 创建节点
  let li = document.createElement('li');
  // 在已有子节点后面追加节点(不会覆盖原有节点)
  parentNode.appendChild(li);
  // 在指定子节点前面追加节点(不会覆盖原有节点)
  parentNode.insertBefore(child, 指定子元素)
```

* 节点移除



``` js
  parentNode.removeChild(child节点) // 删除父元素的某个子节点
```

* 节点复制



``` js
  node.cloneNode() // 拷贝node节点=>浅拷贝,只拷贝标签,不拷贝里面内容
  node.cloneNode(true) // 拷贝node节点=>深度拷贝,拷贝标签以及里面内容
```

* 三种动态创建元素的区别



``` js
  document.write('<div>123</div>') // 注意:当页面文档流加载完毕,会进行重绘
  element.innerHTML = '<div>123</div>' // 不要重复拼接字符串,内存消耗过大
  document.createElement()
```

### DOM事件

#### 鼠标事件

| 鼠标事件      | 触发条件                                 |
| ------------- | ---------------------------------------- |
| onclick       | 鼠标点击左键触发                         |
| onmouseover   | 鼠标经过触发                             |
| onmouseout    | 鼠标离开触发                             |
| onfocus       | 获得鼠标焦点触发                         |
| onblur        | 失去鼠标焦点触发                         |
| onmousemove   | 鼠标移动触发（京东商品正方形放大镜效果） |
| onmouseup     | 鼠标弹起触发                             |
| onmousedown   | 鼠标按下触发                             |
| oncontextmenu | 阻止鼠标右键菜单(阻止默认行为)           |
| onselectstart | 禁止鼠标选取内容(阻止默认行为)           |
| onselect      | 禁止复制(阻止默认行为)                   |

#### mouseenter | mouseover区别

> WARNING
>
> mouseover经过自身会触发, 经过子级也会再次触发(支持事件冒泡).mouseenter只有经过自身盒子才触发(不支持事件冒泡)

> TIP
>
> mouseover搭配mouseout; mouseenter搭配mouseleave

#### 其它鼠标事件

1. 禁止鼠标右键菜单

contextmenu主要控制应该何时显示上下文菜单，主要用于程序员取消默认的上下文菜单

``` js
document.addEventListener('contextmenu', function(e) {
    e.preventDefault();
})
```

2. 禁止鼠标选中(selectstart 开始选中)

``` js
document.addEventListener('selectstart', function(e) {
    e.preventDefault();
})
```

#### 键盘事件

> TIP
>
> 三个事件的执行顺序: keydown > keypress > keyup

| 键盘事件 | 触发条件                                                     |
| -------- | ------------------------------------------------------------ |
| keyup    | 某个键盘按键松开触发                                         |
| keydown  | 某个键盘按键按下时触发, 只要不松开会一直触发                 |
| keypress | 某个键盘按键按下时触发, 不识别功能键(ctrl, 上下箭头等), 只要不松开会一直触发 |

#### 注册事件

> TIP
>
> 注册事件有两种方式: **传统方式**和**监听方式**

* 传统方式: 使用on前缀 => 事件注册具有唯一性, 后者会覆盖前者



``` html
  <p onclick="alert(123)">
      点击
  </p>
```



``` js
  const elem = document.querySelector('p')
  elem.onclick = function() {
      alert('点击段落');
  }
  // 解绑事件
  elem.onclick = null
```

* 监听方式



``` js
  // true:事件句柄在捕获阶段执行 false(默认):事件句柄在冒泡阶段执行
  // 注意:解除绑定与绑定的事件内存地址需要相同
  let div = document.getElementById('div')

  function listener(event) {

  }
  div.addEventListener('click', listener, true)
  div.removeEventListener('click', listener, true)

  // ie9以前使用attachEvent,处理兼容性再用吧💩
  div.attachEvent('onclick', method) // 绑定
  div.detachEvent('onclick', method) // 移除
```

#### 触屏事件(移动端)

> TIP
>
> 触屏事件touch, 存在于安卓与ios, touch事件可响应用户手指(或触控笔)对屏幕或者触控板操作

| 触屏touch事件 | 说明                        |
| ------------- | --------------------------- |
| touchstart    | 手指触摸到一个dom元素时触发 |
| touchmove     | 手指在一个dom元素上滑触发   |
| touchend      | 手指从一个dom元素上移开触发 |

触屏事件对象(touchEvent)

> TIP
>
> touchEvent是一类描述手指在触摸平面(触摸屏等)的状态变化的事件对象, 这类事件用于描述一个或多个触点, 使开发者可以检测触点的移动, 触点的增加和减少等

| touchEvent对象       | 说明                                                     |
| -------------------- | -------------------------------------------------------- |
| event.touches        | 正在触摸屏幕的所有手指的一个列表(可判断双指还是单指触屏) |
| event.targetTouches  | 正在触摸当前dom元素上的手指的一个列表                    |
| event.changedTouches | 手指状态发生了改变的列表, 从无到有, 从有到无变化           |

#### 事件流

> TIP
>
> 事件分为捕获阶段与冒泡阶段. 捕获阶段就是事件信息从顶层向下层传递的过程, 冒泡时事件反应处理从底层向上层反馈的过程。

js可以通过addEventListener来实现捕获阶段或者冒泡阶段的事件响应方法注册

![img](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/eventpool.7883f708.jpg)

#### 事件对象

> TIP
>
> 事件对象鼠标事件(MouseEvent), 则记录鼠标相关信息, 如坐标等; 键盘事件(KeyboardEvent), 则记录键盘相关信息, 如点击的哪个按键

``` js
elem.onclick = function(event) { // 此处的event就是事件对象
    event.target // 返回触发事件的元素,不同于this返回的是绑定事件的元素
    event.srcElement // 同event.target,ie兼容处理使用(ie6,7,8),非标准
    event.relatedTarget // 返回绑定事件的元素,类似于this
    event.type // 获取事件触发类型,如click
    event.pageX | event.pageY // 获取光标相对于页面的x坐标和y坐标 ie9+
    event.clientX | event.clientY // 获取光标相对可视区域的x坐标与y坐标(不受滚动条影响)
    event.screenX | event.screenY // 获取光标相对电脑屏幕的x坐标与y坐标
    event.which // 代表获取到的鼠标或键盘的输出的值,鼠标分别为0,1,2
    event.key // 获取按下的按键值
    event.keyCode // 获取按下按键的ascii码值
}
```

#### 默认行为与冒泡阻止

* 常见html默认行为

  + Submit按钮: 在form表单中的, 提交form表单中的数据到服务器
  + Button: 在手机浏览器中, 若是在form中, 则是submit
  + a标签: 默认将当前页面跳转为a标签中href的地址

* 阻止事件默认行为



``` html
  <!-- 方式4 -->
  <a onclick="javascript:;">点击</a>

  <script>
      const a = document.querySelector('a');
      a.onclick = function(event) {
          // 方式1
          event.preventDefault()
          // 方式2,低版本ie使用
          event.returnValue()
          // 方式3,addEventListener注册方式无效
          return false
      }
  </script>
```

* 阻止冒泡



``` html
  <div class="parent">
      <div class="child"></div>
  </div>
  <script>
      const pElem = document.querySelector('.parent')
      const cElem = document.querySelector('.child')
      cElem.onclick = function(event) {
          // 标准: 阻止冒泡
          event.stopPropagation()
          // 非标准: 阻止冒泡(ie678)
          event.cancalBabel = true
      }
  </script>
```

#### 事件委托

> TIP
>
> 事件委托又叫事件代理, Jquery里面称为事件委派

* 原理: 不再每个子节点单独设置事件监听器(提高了性能), 而是将事件监听器设置再父级元素上, 然后利用冒泡原理影响每个子节点

``` html
<ul>
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>

<script>
    const ul = document.querySelector('ul');
    ul.addEventListener('click', event => {
        // 不用单独每个li绑定事件
        console.log(event.target);
    });
</script>
```

#### 鼠标事件对象

event 对象代表事件的状态，跟事件相关的一系列信息的集合。现阶段我们主要是用鼠标事件对象 MouseEvent 和键盘事件对象 KeyboardEvent。

| 鼠标事件对象 | 说明                                  |
| ------------ | ------------------------------------- |
| e.clientX    | 返回鼠标相对于浏览器窗口可视区的X坐标 |
| e.clientY    | 返回鼠标相对于浏览器窗口可视区的Y坐标 |
| e.pageX      | 返回鼠标相对于文档页面的X坐标IE9+支持 |
| e.pageY      | 返回鼠标相对于文档页面的Y坐标IE9+支持 |
| e.screenX    | 返回鼠标相对于电脑屏幕的X坐标         |
| e.screenY    | 返回鼠标相对于电脑屏幕的Y坐标         |

``` js
//鼠标事件对象 MouseEvent
document.addEventListener('click'，function(e) {
    //1.client鼠标在可视区的x和y坐标
    console.log(e.clientx);
    console.log(e.clientY);
    console.log("--");
    //2.page鼠标在页面文档的x和y坐标
    console.log(e.pagex);
    console.log(e.pageY);
    console.log("--");
    //3.screen鼠标在电脑屏幕的x和y坐标
    console.log(e.screenx);
    console.log(e.screenY);
})
```

#### 键盘事件对象

键盘事件对象 KeyboardEvent

虽然可以直接用 e.key 来获取 KeyboardEvent.key 的按键是多少，但是它不兼容，所以使用 keyCode 更好些，keyCode 是ASCII码值。

1. 我们的 keyup 和 keydown 事件不区分字母大小写 a 和 A 得到的都是 65
2. 我们的 keypress 事件区分字母大小写 a 97 和 A 得到的是 65

Keypress 不识别功能键，但是keyCode属性能区分大小写，返回不同的ASCl值

我们可以利用 keycode 返回的 ASCII 码值来判断用户按下了那个键

> 注意：
>
> keydown 和 keypress 在文本框里面的特点：他们两个事件触发的时候，文字还没有落入文本框中。
>
> keyup 事件触发的时候，文字已经落入文本框里面了

## BOM

> TIP
>
> BOM没有标准组织, ECMA有ECMA组织, DOM有W3C组织制定, 而BOM还是在延用网景的标准, 兼容较差
>
> - 它是js访问浏览器窗口的一个接口
> - 它是一个全局对象, 定义在全局作用域的变量, 函数都会变成window对象的属性和方法 => this指向问题产生

### Window对象

#### 属性

``` js
window.innerWidth // 获取当前屏幕宽度,tip 可用来做响应式布局,还是推荐媒体查询,flex做响应式布局
window.innerHeight // 获取当前屏幕高度
```

#### 方法

``` js
window.scroll(x, y) // 滚动窗口至文档中的特定位置 => such: window.scroll(0,0) 滚动到页面顶部
```

#### 事件

* 窗口加载事件



``` js
  window.onload = function() {}
  // 或
  window.addEventListener('load', function() {})

  // window.onload是窗口加载事件,当文档内容完全加载会触发该事件(包括图像,脚本文件,css文件等)
  // window.onload传统注册事件只能写一次,如果多次会以最后一个为准,addEventListener没有此问题

  window.addEventListener('DOMContentLoaded', function() {}) // 只要dom文档加载完就会触发监听事件
```

* 窗口大小调整事件



``` js
  // 窗口大小发生变化,就会触发
  window.onresize = function() {}
  window.addEventListener('resize', function() {})
```

### location对象

#### 属性

``` js
location.href // 获取或设置整个URL
location.host // 返回主机域名
location.port // 返回端口号
location.pathname // 返回路径
location.search // 返回参数
location.hash // 返回锚点
```

#### 方法

``` js
location.assign() // 跟href相同,实现跳转页面
location.replace() // 替换当前页面,因为不记录历史,所以不能后退页面
location.reload() // 重新加载页面,参数为true会强制刷新
```

#### URL(同一资源定位符)

``` text
// url的一般语法格式为
scheme://host[:port]/path/[?jquery]#fragment
https://www.baidu.com/index.html?name=andy&age=18#link
```

| 组成     | 说明                                        |
| -------- | ------------------------------------------- |
| protocol | 通信协议, 如http, https, ftp, maito             |
| host     | 某个键盘按键按下时触发, 只要不松开会一直触发 |
| port     | 端口(默认80)                                |
| path     | 资源路径                                    |
| query    | 参数, 以键值对形式                           |
| fragment | 锚点                                        |

### navigator对象

> TIP
>
> navigator对象包含了很多设备相关属性例如操作系统, 版本号等信息

``` js
// case: 使用userAgent识别手机pc
if (navigator.userAgent.match(/(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|BlackBerry|IEMobile|MQQBrowser|JUC|Fennec|wOSBrowser|BrowserNG|WebOS|Symbian|Windows Phone)/i)) {
    window.location.href = ''; //手机
} else {
    window.location.href = ''; //电脑
}
```

### history对象

> TIP
>
> window对象为我们提供了一个history对象, 与浏览器历史记录进行交互, 该对象包含用户(在浏览器窗口中)访问过的URL

| history对象方法 | 作用                                               |
| --------------- | -------------------------------------------------- |
| back()          | 实现后退                                           |
| forward()       | 实现前进                                           |
| go(参数)        | 前进后退功能, 参数是1: 前进一个页面, -1: 后退一个页面 |

### 定时器

* setInterval(重复执行, 每次调用有时间延迟)



``` js
  let inervalId = setInterval(code, milliseconds);
  let inervalId = setInterval(function, milliseconds, param1, param2, ...)
```

* setTimeOut(只执行一次, 延迟delay毫秒后执行)



``` js
  let interValId = setTimeOut(fn, delay)
```

* 清除延迟



``` js
  clearInterVal(inervalId) // 对应setInterval
  clearTimeOut(inervalId) // 对应setTimeOut
```

* 标题跑马灯效果实现



``` js
  // 数组方式实现跑马灯效果
  setInterval(() => {
      let title = [...document.title];
      title.unshift(title.pop());
      document.title = title.join('');
  }, 500);

  // 字符串方式实现跑马灯
  setInterval(() => {
      let oldTitle = document.title;
      let newTitle = oldTitle.slice(-1).concat(oldTitle.slice(0, -1));
      document.title = newTitle;
  }, 500);
```

### JS执行机制

#### JS是单线程

> WARNING
>
> javascript就是为了处理页面中用户的交互以及操作DOM而产生, js是单线程 => 这就意味着所有任务都需要排队, 前一个任务执行结束才能执行下一个任务。导致某个js流程执行时间过长, 页面渲染就不连贯, 造成页面阻塞

#### 同步和异步

> TIP
>
> 为了解决js的单线程阻塞问题, 利用多核CPU的计算能力, HTML5提出Web Worker标准, 允许Javascript脚本创建多个线程。

* 同步: 前一个任务执行完毕才进行下一任务

  > 注意: 同步任务都在主线程上执行, 形成一个执行栈

* 异步: 前一任务执行过程种, 同时执行另一任务

  > JS的异步任务都是通过回调函数来处理, 如监听事件, 点击事件等

#### 执行机制

* 先执行执行栈的同步任务
* 异步任务(回调函数)放到任务队列中
* 一旦执行栈中的同步任务执行完毕, 系统就会依次执行任务队列中的异步任务

[关于JS事件循环详情请移步另一篇](http://blog.lskreno.top/interview/001-eventloop/)

## 本地存储

> 本地存储特性:
>
> - 数据存储在用户浏览器中
> - 设置, 读取方便, 甚至页面刷新不丢失数据
> - 容量较大, sessionStorage约5M, localStorage约20M
> - 只能存储字符串, 对象转json后再存储

### window.sessionStroage

> - 生命周期为关闭浏览器窗口
> - 在同一窗口(页面)下数据可以共享
> - 以键值对的形式存储使用

``` js
const user = {
    name: 'RENO',
    age: 18
}

// sessionStorage
// 存储数据
sessionStorage.setItem('user', JSON.stringify(user, null, 2))
// 修改数据 => 重新存储即可
// 读取数据
sessionStorage.getItem('user')
// 删除数据
sessionStorage.removeItem('user')
// 清空数据
sessionStorage.clear()
```

### window.localStorage

> - 生命周期永久生效, 除非手动删除否则关闭页面也会存在
> - 可以多窗口(页面)共享(同一浏览器可以共享数据)
> - 以键值对的形式存储使用

``` js
const user = {
    name: 'RENO',
    age: 18
}

// localStorage
// 存储数据
localStorage.setItem('user', JSON.stringify(user, null, 2))
// 修改数据 => 重新存储即可
// 读取数据
localStorage.getItem('user')
// 删除数据
localStorage.removeItem('user')
// 清空数据
localStorage.clear()
```

------

## THANK

[MDN Web API](https://developer.mozilla.org/zh-CN/docs/Web/API)

[MDN-DOM standard](https://dom.spec.whatwg.org/)

[MDN Document对象api](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)

[MDN Element元素对象api](https://developer.mozilla.org/zh-CN/docs/Web/API/Element)

[MDN HTMLCollection对象api](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCollection)

[MDN NodeList api](https://developer.mozilla.org/zh-CN/docs/Web/API/NodeList)

[MDN ClassList](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/classList)

[scroll图解](https://www.cnblogs.com/wenruo/p/9754576.html)
