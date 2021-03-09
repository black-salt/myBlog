# 002：创建型模式

<motto></motto>

> 工厂模式（工厂方法模式，抽象工厂模式，建造者模式）
>
> 单例模式
>
> 原型模式

## 工厂模式（重点）

* 将new操作单独封装
* 遇到new时，就要考虑是否该使用工厂模式

### 常见场景

* jQuery - $('div')
* React.createElement
* vue 异步组件

``` js
class Product {
    constructor(name) {
        this.name = name
    }
    init() {
        console.log("--- init ---")
    }
    fn1() {
        console.log("--- fn1 ---")
    }
}

class ProductFactory {
    create(name) {
        return new Product(name)
    }
}

let Factory = new ProductFactory()
let product1 = Factory.create("product1")
product1.init()
product1.fn1()
```

## 单例模式（重点）

* 系统中被唯一使用
* 一个类只有一个实例

``` js
class SingleObject {
    login() {
        console.log('login...')
    }
}

SingleObject.getInstance = (function() {
    let instance
    return function() {
        if (!instance) {
            instance = new SingleObject();
        }
        return instance
    }
})()

// 测试
let obj1 = SingleObject.getInstance()
obj1.login()
let obj2 = SingleObject.getInstance()
obj2.login()
console.log(obj1 === obj2)
```

## 原型模式

* clone自己，生成一个新对象
* java默认有clone接口，不用自己实现

Object.create 用到了原型模式的思想（虽然不是java中的clone）

基于一个原型创建一个对象
