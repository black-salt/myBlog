# 001 - 回流 Reflow 和重绘 Repaint

<motto></motto>

## 一、回流 Reflow

### 1. 定义

`元素`或者`元素的大小`或者`位置`发生了变化（当页面布局和几何信息发生变化的时候），触发了重新布局，导致渲染树重新计算布局和渲染

### 2. 如何引发回流 Reflow

- 添加或删除可见的DOM元素
- 元素的位置发生变化
- 元素的尺寸发生变化
- 内容发生变化（比如文本变化或图片被另一个不同尺寸的图片所替代）
- 页面一开始渲染的时候（无法避免）
- 浏览器的窗口尺寸变化：因为回流是根据`视口的大小`来`计算元素的位置和大小`的，所以也会引发回流

## 二、重绘 Repaint

### 1. 定义

元素样式的改变（但宽高，大小，位置等不变）

### 2. 如何只单纯地引发重绘 Repaint

- outline
- visibility
- color
- background-color

重绘的代价是高昂的，因为浏览器必须验证DOM树上其他节点元素的可见性。

## 三、二者对比

> 1. `重绘不是很消耗性能`，但是`回流很消耗性能`（DOM元素的大小和位置信息都要重新计算一遍），而且一旦发生回流，重新计算完后，还需要重绘
> 2. 回流一定会触发重绘，而重绘不一定会回流

## 四、减少 DOM 回流

- **放弃传统地操作DOM**
  - 基于 Vue/React 开始数据影响视图的模式
  - MVVM/MVC/Virtual DOM/DOM Diff.……
  - Vue/React 数据驱动思想：不操作 DOM，只操作数据，借助框架帮助我们根据数据渲染视图（框架内部本身对于 DOM 的回流和重绘以及其它性能优化做的已经很好了，视图的更新会有将`多步操作合并`等）
  
- **分离读写**：对 DOM 属性的读写要分离

- **样式集中改变**：尽量使用 class 操作样式改变或者 cssText 的方式进行更改，而不是频繁操作 style
  - div.className = "wrapper"
  - div.style.cssText = "width:50px;height:50px;"
  
- **批量修改元素**
  - **文档碎片**： 临时创建的一个存放文档的容器，我们可以把新创建的 li，存放到容器中，当所有的 li 都存储完，我们统一把容器中的内容增加到页面中（只触发一次回流）
    - `let frag = document.createDocumentFragment()；`
    - `frag.appendChild(li)；`
    - `item.appendChild(frag)；`
  - **字符串拼接**：
    - `let str = "<a>xxx</a>" `
    - `item.innerHTML = str `
  
- **避免使用 table 布局**

  - 由于浏览器使用流式布局，对`Render Tree`的计算通常只需要遍历一次就可以完成，**但`table`及其内部元素除外，他们可能需要多次计算，通常要花3倍于同等元素的时间，这也是为什么要避免使用`table`布局的原因之一**。

- **CSS3硬件加速** 做优化

  - 如下几个 css 属性可以触发硬件加速：

    - transform：可以通过 transform 的3D属性强制开启GPU加速：`transform: translateZ(0);` `transform: rotateZ(360deg);`

    - opacity

    - filter

    - will-change：哪一个属性即将发生变化，例：用 will-change : transform，进而进行优化。

    - **原理:**

      - `transform,opacity,filter` 动画由GPU控制，支持硬件加速，并不需要软件方面的渲染。
      - CSS `transform` 创建了一个新的复合图层，可以被GPU直接用来执行 `transform` 操作。在chrome开发者工具中开启“show layer borders”选项后，每个复合图层就会显示一条黄色的边界。
    
    - ##### 注意事项：
    
      - 不能让每个元素都启用硬件加速，这样会暂用很大的内存，使页面会有很强的卡顿感。
      - GPU渲染会影响字体的抗锯齿效果。这是因为GPU和CPU具有不同的渲染机制，即使最终硬件加速停止了，文本还是会在动画期间显示得很模糊。

- **对具有复杂动画的元素使用绝对定位**，使它脱离文档流，否则会引起父元素及后续元素频繁回流。







