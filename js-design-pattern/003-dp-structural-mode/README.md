# 结构型模式

<motto></motto>

* 适配器模式
* 装饰器模式
* 代理模式
* 外观模式

* 桥接模式
* 组合模式
* 享元模式

## 适配器模式（重点）

* 旧接口格式和使用者不兼容
* 中间加一个适配转换接口

### 常见场景

* 封装旧接口
* Vue computed

``` js
class Adaptee {
    specificRequest() {
        return "英国标准插头")
}
}

class Target {
    constructor() {
        this.adaptee = new Adaptee();
    }
    request() {
        let info = this.adaptee.specificRequest()
        return `${info} - 转换器 - 中国标准插头`
    }
}

let target = new Target();
let res = target.request()
console.log(res)
```

## 装饰器模式（重点）

* 为对象添加新功能
* 不改变其原有的结构和功能

``` js
class Circle {
    draw() {
        console.log("--- 画了一个圆 ---")
    }
}

class Decorator {
    constructor(circle) {
        this.circle = circle;
    }

    draw() {
        this.circle.draw();
        this.setRedBorder(this.circle);
    }

    setRedBorder(circle) {
        console.log("--- 给圆画了一个边圈 ---")
    }
}

let circle = new Circle();
circle.draw();
let dec = new Decorator();
dec.draw();
```

> 使用 @decorator 的形式的话，调试需要用 babel 插件来进行转换
>
> `npm install babel-plugin-transform-decorator-legacy --save-dev`

mixin的例子

``` js
// @testable
// class MyTestableClass {
//   // ...
// }

// function testable(target) {
//   target.isTestable = true;
// }

// alert(MyTestableClass.isTestable) // true

function mixins(...list) {
    return function(target) {
        Object.assign(target.prototype, ...list)
    }
}

const Foo = {
    foo() {
        alert('foo')
    }
}

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

readonly的例子

``` js
function readonly(target, name, descriptor) {
    // descriptor对象原来的值如下
    // {
    //   value: specifiedFunction,
    //   enumerable: false,
    //   configurable: true,
    //   writable: true
    // };
    descriptor.writable = false;
    return descriptor;
}

class Person {
    constructor() {
        this.first = 'A'
        this.last = 'B'
    }

    @readonly
    name() {
        return `${this.first} ${this.last}`
    }
}

var p = new Person()
console.log(p.name())
p.name = function() {} // 这里会报错，因为 name 是只读属性
```

log的例子

``` js
function log(target, name, descriptor) {
    var oldValue = descriptor.value;

    descriptor.value = function() {
        console.log( `Calling ${name} with` , arguments);
        return oldValue.apply(this, arguments);
    };

    return descriptor;
}

class Math {
    @log
    add(a, b) {
        return a + b;
    }
}

const math = new Math();
const result = math.add(2, 4);
console.log('result', result);
```

deprecate的例子(使用 `core-decorators` 包)

``` js
// import { readonly } from 'core-decorators'

// class Person {
//     @readonly
//     name() {
//         return 'zhang'
//     }
// }

// let p = new Person()
// alert(p.name())
// // p.name = function () { /*...*/ }  // 此处会报错

import {
    deprecate
} from 'core-decorators';

class Person {
    @deprecate
    facepalm() {}

    @deprecate('We stopped facepalming')
    facepalmHard() {}

    @deprecate('We stopped facepalming', {
        url: 'http://knowyourmeme.com/memes/facepalm'
    })
    facepalmHarder() {}
}

let person = new Person();

person.facepalm();
// DEPRECATION Person#facepalm: This function will be removed in future versions.

person.facepalmHard();
// DEPRECATION Person#facepalmHard: We stopped facepalming

person.facepalmHarder();
// DEPRECATION Person#facepalmHarder: We stopped facepalming
//
//     See http://knowyourmeme.com/memes/facepalm for more details.
```

## 代理模式（重点）

### 常见场景

- 网页事件代理
- jQuery $.proxy
- ES6 Proxy

明星经纪人（Proxy）

``` js
// 明星
let star = {
    name: '张XX',
    age: 25,
    phone: '13910733521'
}

// 经纪人
let agent = new Proxy(star, {
    get: function(target, key) {
        if (key === 'phone') {
            // 返回经纪人自己的手机号
            return '18611112222'
        }
        if (key === 'price') {
            // 明星不报价，经纪人报价
            return 120000
        }
        return target[key]
    },
    set: function(target, key, val) {
        if (key === 'customPrice') {
            if (val < 100000) {
                // 最低 10w
                throw new Error('价格太低')
            } else {
                target[key] = val
                return true
            }
        }
    }
})

// 主办方
console.log(agent.name)
console.log(agent.age)
console.log(agent.phone)
console.log(agent.price)

// 想自己提供报价（砍价，或者高价争抢）
agent.customPrice = 150000
// agent.customPrice = 90000  // 报错：价格太低
console.log('customPrice', agent.customPrice)
```

## 外观模式

![](https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-07-05_13-53-09.png)





## 桥接模式

- 用于把抽象化与实现化解耦
- 使得二者可以独立变化

## 组合模式

- 生成树形结构，表示“整体-部分”关系
- 让整体和部分都具有一致的操作方式

虚拟 DOM 中的 vnode 是这种形式，但数据类型简单

- 整体和单个节点的操作是一致的
- 整体和单个节点的数据结构也保持一致

## 享元模式（共享、元数据）

- 共享内存（主要考虑内存，而非效率）
- 相同的数据，共享使用
- （JS中未找到经典应用场景）



