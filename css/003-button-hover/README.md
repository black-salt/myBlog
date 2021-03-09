# 003-Button Hover效果收集

<motto></motto>



## 一、边框效果

### HTML

```html
<div id="draw-border">
  <button>Hover me</button>
</div>
```

### CSS

```css
#draw-border {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

button {
  border: 0;
  background: none;
  text-transform: uppercase;
  color: #4361ee;
  font-weight: bold;
  position: relative;
  outline: none;
  padding: 10px 20px;
  box-sizing: border-box;
}

button::before, button::after {
  box-sizing: inherit;
  position: absolute;
  content: '';
  border: 2px solid transparent;
  width: 0;
  height: 0;
}

button::after {
  bottom: 0;
  right: 0;
}

button::before {
  top: 0;
  left: 0;
}

button:hover::before, button:hover::after {
  width: 100%;
  height: 100%;
}

button:hover::before {
  border-top-color: #4361ee;
  border-right-color: #4361ee;
  transition: width 0.3s ease-out, height 0.3s ease-out 0.3s;
}

button:hover::after {
  border-bottom-color: #4361ee;
  border-left-color: #4361ee;
  transition: border-color 0s ease-out 0.6s, width 0.3s ease-out 0.6s, height 0.3s ease-out 1s;
}

```

### GIF图

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/295e3b962d9b4b08b34226604343d256_tplv-k3u1fbpfcp-zoom-1.gif" style="zoom:80%;" />

## 二、圆形进度效果

### HTML

```html
<div id="circle-btn">
  <div class="btn-container">
    // 这里有一个svg元素
    <button>Hover me</button>
  </div>
</div>
```

### CSS

```css
#circle-btn { 
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

.btn-container {
  position: relative;  
}

button {
  border: 0;
  border-radius: 50px;
  color: white;
  background: #5f55af;
  padding: 15px 20px 16px 60px;
  text-transform: uppercase;
  background: linear-gradient(to right, #f72585 50%, #5f55af 50%);
  background-size: 200% 100%;
  background-position: right bottom;
  transition:all 2s ease;
}

svg {
  background: #f72585;
  padding: 8px;
  border-radius: 50%;
  position: absolute;
  left: 0;
  top: 0%;
}

button:hover {
   background-position: left bottom;
}
```

### GIF效果

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/4719262123064ff683dfdff17555efde_tplv-k3u1fbpfcp-zoom-1.gif" alt="4719262123064ff683dfdff17555efde_tplv-k3u1fbpfcp-zoom-1" style="zoom:80%;" />



## 三、冰冻效果

### HTML

```html
<div id="frozen-btn">
  <button class="green">Hover me</button>
  <button class="purple">Hover me</button>
</div>
```

### CSS

```css
#frozen-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

button {
  border: 0;
  margin: 20px;
  text-transform: uppercase;
  font-size: 20px;
  font-weight: bold;
  padding: 15px 50px;
  border-radius: 50px;
  color: white;
  outline: none;
  position: relative;
}

button:before{
  content: '';
  display: block;
  background: linear-gradient(to left, rgba(255, 255, 255, 0) 50%, rgba(255, 255, 255, 0.4) 50%);
  background-size: 210% 100%;
  background-position: right bottom;
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  bottom:0;
  right:0;
  left: 0;
  border-radius: 50px;
  transition: all 1s;
  -webkit-transition: all 1s;
}

.green {
   background-image: linear-gradient(to right, #25aae1, #40e495);
   box-shadow: 0 4px 15px 0 rgba(49, 196, 190, 0.75);
}

.purple {
   background-image: linear-gradient(to right, #6253e1, #852D91);
   box-shadow: 0 4px 15px 0 rgba(236, 116, 149, 0.75);
}
  
.purple:hover:before {
  background-position: left bottom;
}

.green:hover:before {
  background-position: left bottom;
}
```

### GIF效果

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/b8572b77cd25496d85655d4d6fb12a7d_tplv-k3u1fbpfcp-zoom-1.gif" alt="b8572b77cd25496d85655d4d6fb12a7d_tplv-k3u1fbpfcp-zoom-1" style="zoom:80%;" />

## 四、闪亮效果

### HTML

```html
<div id="shiny-shadow">
  <button><span>Hover me</span></button>
</div>
```

### CSS

```css
#shiny-shadow {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: #1c2541;
}

button {
  border: 2px solid white;
  background: transparent;
  text-transform: uppercase;
  color: white;
  padding: 15px 50px;
  outline: none;
  overflow: hidden;
  position: relative;
}

span {
  z-index: 20;  
}

button:after {
  content: '';
    display: block;
    position: absolute;
    top: -36px;
    left: -100px;
    background: white;
    width: 50px;
    height: 125px;
    opacity: 20%;
    transform: rotate(-45deg);
}

button:hover:after {
  left: 120%;
  transition: all 600ms cubic-bezier(0.3, 1, 0.2, 1);
   -webkit-transition: all 600ms cubic-bezier(0.3, 1, 0.2, 1);
}
```

### GIF效果

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/50cbf9f8224c4349baaee20844006a67_tplv-k3u1fbpfcp-zoom-1.gif" alt="50cbf9f8224c4349baaee20844006a67_tplv-k3u1fbpfcp-zoom-1" style="zoom:80%;" />

## THANK

1. [8 amazing HTML button hover effects, that will make your website memorable](https://www.blog.duomly.com/html-button-hover-effects/)

