# 行为型模式

<motto></motto>

* 策略模式
* 模板方法模式
* 观察者模式
* 迭代器模式
* 职责链模式
* 命令模式
* 备忘录模式
* 状态模式
* 访问者模式
* 中介者模式
* 解释器模式

## 观察者模式（重点）

``` js
// 主题，接收状态变化，触发每个观察者
class Subject {
    constructor() {
        this.state = 0
        this.observers = []
    } 
    getState() {
        return this.state
    }
    setState(state) {
        this.state = state
        this.notifyAllObservers()
    }
    attach(observer) {
        this.observers.push(observer)
    }
    notifyAllObservers() {
        this.observers.forEach(observer => {
            observer.update()
        })
    }
}

// 观察者，等待被触发
class Observer {
    constructor(name, subject) {
        this.name = name
        this.subject = subject
        this.subject.attach(this)
    }
    update() {
        console.log( `${this.name} update, state: ${this.subject.getState()}` )
    }
}

// 测试代码
let s = new Subject()
let o1 = new Observer('o1', s)
let o2 = new Observer('o2', s)
let o3 = new Observer('o3', s)

s.setState(1)
s.setState(2)
s.setState(3)
```

### 常用场景

* 网页事件绑定(例如， `onclick` 、 `attachEvent('click')` 、 `addEventListener('click')` )
* Promise 的状态变化
* jQuery callbacks
* nodejs自定义事件 EventEmitter ( 一般不直接用，通常继承EventEmitter)

``` js
const EventEmitter = require('events').EventEmitter
const emitter1 = new EventEmitter()
emitter1.on('some', () => {
    //监听 some事件
    console.log('some event is occured 11')
})

emitter1.on('some', () => {
    //监听 some事件
    console.log('some event is occured 2')
})

//触发some事件
emitter1.emit('some')
```

### 其他场景

* Nodejs 中：处理 http 请求(on('data', (chunk)=>{}))；多进程通讯(child_process模块，on(' message', function(message){} / send())

* Vue 和 React 组件生命周期触发

* Vue watch

多进程通讯:

``` js
//parent. js var cp=require(' child_process')
var n = cp.fork('./sub. js')
n.on('message', function(m) {
    console.log('PARENT got message:' + m)
})
n.send({
    hello: 'workd'
})
//sub. js process. on(' message', function(m){
console.log('CHILD got message:' + m)
})
process.send({
    foo: 'bar'
})
```

## 迭代器模式（重点）

* 顺序访问一个集合
* 使用者无需知道集合的内部结构(封装)

### 常用场景

* jQuery each
* ES6 Iterator

### ES6 Iterator为何存在？

* ES6语法中，有序集合的数据类型已经有很多
* Array Map Set String TypedArray arguments NodeList
* 需要有一个统一的遍历接口来遍历所有数据类型
* (注意，object不是有序集合，可以用Map代替)

以上数据类型，都有 `[Symbol.iterator]` 属性，属性值是函数，执行函数返回一个迭代器。

* 这个迭代器就有next 方法可顺序迭代子元素
* 可运行 Array.prototype[Symbol.iterator] 来测试

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-07-05_16-38-56.png" alt="Snipaste_2020-07-05_16-38-56" style="zoom: 50%; " />

ES6 Iterator 示例

``` js
let arr = [1, 2, 3, 4]
let nodeList = document.getElementsByTagName('p')
let m = new Map()
m.set('a', 100)
m.set('b', 200)

// function each(data) {
//     // 生成遍历器
//     let iterator = data[Symbol.iterator]()

//     // console.log(iterator.next())  // 有数据时返回 {value: 1, done: false}
//     // console.log(iterator.next())
//     // console.log(iterator.next())
//     // console.log(iterator.next())
//     // console.log(iterator.next())  // 没有数据时返回 {value: undefined, done: true}

//     let item = {done: false}
//     while (!item.done) {
//         item = iterator.next()
//         if (!item.done) {
//             console.log(item.value)
//         }
//     }
// }

function each(data) {
    //Symbol.iterator并不是人人都知道
    //也不是每个人都需要封装一个each方法
    //因此有了for...of语法
    for (let item of data) {
        console.log(item)
    }
}

each(arr)
each(nodeList)
each(m)
```

### ES6 Iterator 与 Generator

* Iterator的价值不限于上述几个类型的遍历
* 还有Generator函数的使用
* 即只要返回的数据符合Iterator 接口的要求
* 即可使用Iterator 语法，这就是迭代器模式

> 所以 Generator 也可以用 for ... of ...

## 状态模式

* 一个对象有状态变化
* 每次状态变化都会触发一个逻辑
* 不能总是用 if...else 来控制

``` js
class State {
    constructor(color) {
        this.color = color
    }
    handle(context) {
        console.log( `turn to ${this.color} light` )
        context.setState(this)
    }
}

class Context {
    constructor() {
        this.state = null
    }
    setState(state) {
        this.state = state
    }
    getState() {
        return this.state
    }
}

// 测试代码
let context = new Context()

let greed = new State('greed')
let yellow = new State('yellow')
let red = new State('red')

// 绿灯亮了
greed.handle(context)
console.log(context.getState())
// 黄灯亮了
yellow.handle(context)
console.log(context.getState())
// 红灯亮了
red.handle(context)
console.log(context.getState())
```

## 策略模式

* 不同策略分开处理
* 避免出现大量if ... else 或者 switch ... case

## 模板方法模式

## 职责链模式

* 一步操作可能分位多个职责角色来完成
* 把这些角色都分开，然后用一个链串起来
* 将发起者和各个处理者进行隔离

``` js
//请假审批，需要组长审批、经理审批、最后总监审批class Action{
constructor(name) {
    this.name = name
    this.nextAction = null
    setNextAction(action) {
        this.nextAction = action
    }
}
handle() {
    console.log( `${this.name}审批` )
    if (this.nextAction！ = null) {
        this.nextAction.handle()
    }
}

let al = new Action('组长')
let a2 = new Action('经理')
let a3 = new Action('总监')
a1.setNextAction(a2)
a2.setNextAction(a3)
a1.handle()
```

## 命令模式

网页富文本编辑器操作，浏览器封装了一个命令对象

* document.execCommand("bold")
* document.execCommand(undo)

## 备忘录模式

* 随时记录一个对象的状态变化
* 随时可以恢复之前的某个状态(如撤销功能)
* 未找到JS中经典应用，除了一些工具(如编辑器)

``` js
//状态备忘
class Memento {
    constructor(content) {
        this.content = content
    }

    getContent() {
        return this.content
    }
}

//备忘列表
class CareTaker {
    constructor() {
        this.list = []
    }
    add(memento) {
        this.list.push(memento)
    }
    get(index) {
        return this.list[index]
    }
}

//编辑器
class Editor {
    constructor() {
        this.content = null
    }
    setContent(content) {
        this.content = content
    }
    getContent() {
        return this.content
    }
    saveContentToMemento() {
        return new Memento(this.content)
    }
    getContentFromMemento(memento) {
        this.content = memento.getContent()
    }
}

//测试代码
let editor = new Editor()
let careTaker = new CareTaker()
editor.setContent('111')
editor.setContent('222')
careTaker.add(editor.saveContentToMemento() //存储备忘录
editor.setContent('333')
careTaker.add(editor.saveContentToMemento() //存储备忘录
editor.setContent(‘444')
console.log(editor.getContent()
editor.getContentFromMemento(careTaker.get(1) //撤销
console.log(editor.getContent()
editor.getContentFromMemento(careTaker.get(0) //撤销
console.log(editor.getContent()
```

## 中介者模式

对象和对象之间的访问都通过中介者

## 访问者模式

## 解释器模式

描述语言语法如何定义，如何解释和编译用于专业场景