# 003-JS继承

<motto></motto>

<!-- ## 组合式继承（原型链+构造函数继承）

```js
// 父类
function Animal(name) {
    this.name = name
}
Animal.prototype.showName = function () {
    console.log('名字是:' + this.name)
}

// 子类
function Dog(name, color) {
    Animal.call(this, name) // 继承属性
    this.color = color
}
Dog.prototype = new Animal()
Dog.prototype.constuctor = Dog

let d1 = new Dog('wangcai', 'white')
console.log(d1)
d1.showName()
```

## 寄生组合式继承

```js
function inheritPrototype(child, parent) {
  const prototype = Object.create(parent.prototype); // 创建父类原型的副本
  prototype.constructor = child; // 将副本的构造函数指向子类
  child.prototype = prototype; // 将该副本赋值给子类的原型
}
```

```js
function Vehicle(powerSource) {
  this.powerSource = powerSource;
  this.components = ['座椅', '轮子'];
}

Vehicle.prototype.run = function() {
  console.log('running~');
};

function Car(wheelNumber) {
  this.wheelNumber = wheelNumber;
  Vehicle.call(this, '汽油');
}

# 继承
inheritPrototype(Car, Vehicle);

Car.prototype.playMusic = function() {
  console.log('sing~');
};
```



## ES6的继承

```js
class People {
    constructor(name, age) {
        this.name = name
        this.age = age
        this._sex = -1
    }
    // static count = 0 //暂不支持
    get sex() { // 属性
        if (this._sex === 1) {
            return 'male'
        } else if (this._sex === 0) {
            return 'female'
        } else {
            return 'error'
        }
    }
    set sex(val) { // 1:male 0:female
        if (val === 0 || val === 1) {
            this._sex = val
        }
    }
    showName() {
        console.log(this.name)
    }
    // 静态方法
    static getCount() {
        return 5
    }
}
// 静态属性
People.count = 9
console.log(typeof People) // function
console.log(People.count)

let p1 = new People('xiecheng', 34)
console.log(p1)
p1.sex = 5
console.log(p1.sex)
console.log(People.getCount())

class Coder extends People {
    constructor(name, age, company) {
        super(name, age)
        this.company = company
    }
    showCompany() {
        console.log(this.company)
    }
}

let c1 = new Coder('zhangsan', 25, 'imooc')
console.log(c1)
c1.showName()
c1.showCompany()
c1.sex = 1
console.log(c1.sex)
console.log(Coder.getCount())
```

 -->



