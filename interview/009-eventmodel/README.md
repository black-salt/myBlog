# JavaScript事件模型

<motto></motto>


## JavaScript中的事件

- JavaScript程序使用的是事件驱动的设计模式，为一个元素添加事件监听函数，当这个元素的相应事件被触发那么其添加的事件监听函数就被调用：事件是javascript和HTML交互基础, 任何文档或者浏览器窗口发生的交互, 都要通过绑定事件进行交互。
- 所有浏览器都支持DOM0级事件处理程序，且使用该方式时，事件处理程序是在元素的作用域中运行，因此程序中的this都是指向元素。

## 理解DOM0,DOM2,DOM3

DOM(Document Object Model)文档对象模型，是一种与编程语言及平台无关的API(Application programming Interface)，借助于它，程序能够动态地访问和修改文档内容、结构或显示样式。

W3C协会早在1988年就开始了DOM标准的制定，W3C DOM标准可以分为DOM1,DOM2,DOM3三个版本。

DOM1级主要定义的是HTML和XML文档的底层结构。

DOM2和DOM3级别则在这个结构的基础上引入了更多的交互能力，也支持了更高级的XML特性。为此DOM2和DOM3级分为许多模块（模块之间具有某种关联），分别描述了DOM的某个非常具体的子集。这些模块如下：

1. DOM2级核心（DOM Level 2 Core）：在1级核心的基础上构建，为节点添加了更多方法和属性； 
2. DOM2级视图（DOM Level 2 Views）：为文档定义了基于样式信息的不同视图； 
3. DOM2级事件（DOM Level 2 Style）：定义了如何以编程方式来访问和改变CSS样式信息； 
4. DOM2级遍历和范围（DOM Level 2 Traversal and Range）：引入了遍历DOM文档和选择其特定部分的新接口。 
5. DOM2级HTML（DOM Level 2 HTML）：在1级HTML基础上构建，添加了更多属性、方法和新接口。 
6. DOM3级又增加了XPath模块和加载与保存（Load and Save）模块。 

DOM2级和3级的目的在于扩展DOM API，以满足操作XML的所有需求，同时提供更好的错误处理及特性检测能力。 DOM0就是直接通过 onclick写在html里面的事件;

DOM2是通过addEventListener绑定的事件, 还有IE下的DOM2事件通过attachEvent绑定; DOM3是一些新的事件。









## 常见事件行为

### 鼠标事件

| event      | Description                        |
| ---------- | ---------------------------------- |
| click      | 点击事件 (移动端click被识别为单机) |
| dblclick   | 双击事件                           |
| mousedown  | 鼠标摁下                           |
| mouseup    | 鼠标抬起                           |
| mousemove  | 鼠标移动                           |
| mouseover  | 鼠标滑过                           |
| mouseout   | 鼠标划出                           |
| mouseenter | 鼠标进入                           |
| mouseleave | 鼠标离开                           |
| mousewhell | 鼠标滚轮滚动                       |

### 键盘事件

| event    | Description                                     |
| -------- | ----------------------------------------------- |
| keydown  | 按下某个键                                      |
| keyup    | 抬起某个键                                      |
| keypress | 除shift/Fn/CapsLock 键以外,其他键按住(连续触发) |

### 移动端的单手指事件Touch

| 单手指事件模型  Touch |                                            |
| --------------------- | ------------------------------------------ |
| touchstart            | 手指摁下                                   |
| touchmove             | 手指移动                                   |
| touchend              | 手指松开                                   |
| touchcancel           | 操作取消(一般应用于非正常状态下的操作结束) |

### 移动端的多手指事件模型Gesture

| 多手指事件模型  Gesture         |                                            |
| ------------------------------- | ------------------------------------------ |
| gesturestart                    | 手势摁下                                   |
| gesturechange  /  gestureupdate | 手势移动                                   |
| gestureend                      | 手势松开                                   |
| gesturecancel                   | 操作取消(一般应用于非正常状态下的操作结束) |

### 表单元素常用事件

| event  | Description |
| ------ | ----------- |
| focus  | 获取焦点    |
| blur   | 失去焦点    |
| change | 内容改变    |

### 音视频常用事件

| event          | Description                             |
| -------------- | --------------------------------------- |
| canplay        | 可以播放(资源没有加载完,播放可能会卡顿) |
| canplaythrough | 可以播放(资源已经加载完,播放中不会卡顿) |
| play           | 开始播放                                |
| playing        | 播放中                                  |
| pause          | 暂停                                    |

### 其他常用事件

| event            | Description           |
| ---------------- | --------------------- |
| load             | 加载完                |
| unload           | 资源卸载              |
| beforeunload     | 当前页面关闭之前      |
| error            | 资源加载失败          |
| scroll           | 滚动时间              |
| readystatechange | AJAX 请求状态改变事件 |
| contextmenu      | 鼠标右键触发          |

















