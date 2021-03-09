# Typescript 基础总结

## 前言 — 特点

1. TS 是 JS 的超集
2. TS 给 JS 带来类型系统

这意味着 TS 的根基是 JS, 是在 JS 的 基础上添加了类型系统。

 ### 类型声明

类型系统的一个特点是使用类型声明。

在 JS 中, 使用「等号=」给变量赋值。

经过 TS 的扩展, 可以使用「冒号:」给变量赋类型。代码中声明变量 foo 的类型是 string。

### 类型校验

类型系统的另一个特点是进行类型校验。

在 TS 中, 需要校验「等号=」左右的类型是否匹配。

值存在明确的类型, TS 会校验左右两侧的类型是否匹配, 若不匹配则提示错误。

在了解这些前置知识后, 来看看具体的 TS 知识点。

## 一、基本类型

继承 JS 的基本类型如：string、number、boolean、undefined、null。扩展了其他基本类型如：any、never、void。

### 1. 1 string

表示类型为字符串

### 1. 2 number

表示类型为数字

### 1. 3 boolean

表示类型为布尔类型

### 1. 4 undefined

表示类型为 undefined

### 1. 5 null

表示类型为 null

### 1. 6 void

表示类型为 undefined 或 null。

#### 1. 6. 1 对于变量

一个声明为 void 类型的变量, 只能被赋值为 undefined 或 null。这种应用场景较少。

#### 1. 6. 2 对于函数

void 用于表示函数没有返回值, 只是执行某些操作。这是 void 的普遍应用场景。

一个没有显式返回的函数, 默认 return undefined。符合 void 类型只能被赋值为 undefined 或 null。

### 1. 7 never

表示永远不存在值。

典型例子：一个只会抛出异常的函数, 它的返回值并不存在。

执行函数则抛出错误, 连 undefined 都不会返回。

### 1. 8 any

表示可能为任何类型。

典型例子：不确定变量的类型或类型是动态的。

## 二、引用类型

TS 中的引用类型, 除了 JS 中的 `数组类型` 、 `对象类型` , 还扩展了 `枚举类型` , `元祖类型` 。

### 2. 1 数组类型

表示由某(些)类型组成的数组。有两种写法：方括号表示法和尖括号表示法。

#### 2. 1. 1 方括号表示法

表示 arr 的类型是由数字组成的数组 `元素类型[]` 。

#### 2. 1. 2 尖括号表示法

使用 `Array<元素类型>` , 这是 `数组泛型` 的使用形式, 关于泛型后续会展开。

### 2. 2 只读数组

使用 `ReadonlyArray<属性类型>` 定义只读数组。只读数组只能在数组初始化时定义其值, 创建后不能进行修改。

只读数组和常规数组类型 `Array<T>` 类似。区别在于：常规数组存在修改数组的方法, 只读数组不存在修改数组的方法。

### 2. 3 object

表示类型为对象。除了 string、number、bigint、boolean、 undefined、null、symbol 基本类型外的引用类型。

### 2. 4 元组

由**定长数组**构成, 数组中的元素是某(些)类型 `[string, number]` 。

## 三、类型断言

明确告诉 TS 某个值的类型。有两种写法：尖括号写法、as 写法。

### 1. 写法

#### 1. 1 尖括号写法

``` typescript
let str:any = '白糖炒栗子'
let strLen = (<string> str).length
```

#### 1. 2 as 写法

``` typescript
let str:any = '白糖炒栗子'
let strLen = (str as string).length
```

> 注意事项:
>
> `类型断言` 和 `类型转换` 是有明确区别, 不能将它理解成类型转换。
>
> 在阐述此注意事项之前, 先引入另一个概念： `联合类型` 。

### 2. 联合类型

``` js
let foo: string | number = '白糖炒栗子'
//正确
foo = 9527
```

`string | number ` 表示的就是一个联合类型, 意味着 foo 的类型可以为 string 或 number。

### 3. 类型断言和类型转换之间的区别

``` js
let foo: string | number = '白糖炒栗子'
//正确
let answer: string = foo as string
```

上述代码中, `string | number` 联合类型包含 string 类型, 所以这两种类型之间存在联系。使用 as 将联合类型断言成更加具体的 string 类型。

``` js
let num: number = 9527
//正确
let other = num as string | number
```

上述代码中, 将 number 类型断言为 `string | number` 类型。同样是因为两种类型之间存在联系, 所以也允许断言。相当于将一个具体的 number 类型断言成更加广泛的联合类型。

``` js
let str: string = '白糖炒栗子'
//提示错误
let bar: number = str as number
```

而将 string 类型断言为 number 类型, 这是两种不同的类型, 没有任何联系, 断言会提示错误。

观察上述三个例子, 断言发生在两种类型存在联系的情况, 它并不是将一种类型转换成另一种类型。

有些脑瓜子灵活的朋友会想到, 借助两次断言来进行类似的类型转换。

``` js
let num1: number = 9527
let num2: string = < string > < number | string > num1
```

NOTE: 这是欺骗了 TS 校验, 后果只能自己承担。

## 四、接口

上面提到使用 object 描述对象类型。但是, 对于具有复杂结构的对象、函数。上述的 object 类型力有不逮。

``` js
function getName(o: object) {
    //return o.name
    getName({})
    getName({
        name: '白糖炒栗子'
    })
```

上述代码中, `即使是空对象, 也能通过 object 类型校验, 而不会校验对象的结构是否能满足后续使用` 。

**此时, 就需要使用接口**。

在 TS 中, 接口是**描述值的结构**。

``` js
function getName(o: {
    name: string
}) {
    return o.name
    //提示错误
    getName({})
    //正确
    getName({
        name: '白糖炒栗子'
    })
```

上述代码中, 就定义了一个接口来描述参数 o, 要求它是一个对象, 含有 name 属性, 且属性值是字符串。

所以, 不符合此结构的空对象 {} 就提示错误。

一般来说, **使用关键字 `interface` 来定义接口**。

重写上述接口 : 

``` js
interface Animal {
    name: string;
}

function getName(o: Animal) {
    return o.name
}
```

### 4. 1 固定属性

在接口中, 使用 `属性名：属性类型` 的结构定义固定属性。

``` js
interface Animal {
    name: string;
    age: number;
}

function getName(o: Animal) {
    return o.name
}
let cat = {
    name: '白糖炒栗子'
}
let dog = {
    name: '',
    age： 3
}
//提示错误
getName(cat)
//正确
getName(dog)
```

如上述代码所示, 传入的对象需要具有 name 和 age 两个属性, 否则会报错。

### 4. 2 可选属性

在接口中, 使用 `属性名?：属性类型` 的结构定义可选属性。顾名思义, 可选属性可以存在, 也可以不存在。

``` js
interface Animal {
    name: string;
    age ? : number;
}

function getName(o: Animal) {
    return o.name
}
let cat = {
    name: '白糖炒栗子'
}
let dog = {
    name: '',
    age： 3
}
//提示错误
getName(cat)
//正确
getName(dog)
```

### 4. 3 只读属性

在接口中, 使用 `readonly 属性名：属性类型` 的结构定义只读属性。只读属性只能在属性初始化时定义其值, 定义后不能进行修改。

``` js
interface Point {
    readonly x: number;
    readonly y: number;
}
let p1: Point = {
    x: 10,
    y: 20
};
//提示错误
p1.x = 5;
```

### 4. 4 额外检查

TS 会对对象字面量进行 `额外的属性检查` 。

``` js
interface Point {
    × ? : number;
    y ? : number;
}

function point(p: Point) {}
//提示错误
let p1: Point = {
    x: 1,
    z: 2
}
//提示错误
point({
    x： 1,
    z： 2
})
//正确
let a = {
    x： 1,
    z： 2
}
point(a)
```

在上述代码中, 接口 Point 定义了两个可选属性, 对象字面量中属性 x 和接口兼容, 属性 z 是多余无意义的。

虽然实际的属性比接口定义的多, 按照常规理解, 这是可以通过类型校验, 但事实却相反。

**TS 对于对象字面量是会进行额外的属性检查**, 体现在：

* 当对象字面量赋值给变量或它直接作为参数传递给函数时, 如果对象字面量的属性没有在接口中定义, 则会报错。

  换句话说, 对于对象字面量, 当它直接赋值给变量和函数参数时, 它的属性不能比接口描述的多。

  这里重点是直接赋值, 如果像例子中, 先将对象字面量赋值给变量, 再通过变量传参, 这样间接的方法可以绕过额外的检查。

### 4. 5 可索引类型

可索引类型包括字符串索引类型与数字索引类型。

#### 4. 5. 1 字符串索引类型

字符串索引类型具有字符串索引签名。

``` json
interface Point {
	[prop:string]:number;
} 
let p:Point;
p = {x:1,y:2,z:3}
```

#### 4. 5. 2 数字索引类型

数字索引类型具有数字索引签名

``` js
interface StringArray {
    //数字索引签名
    [index: number]： string；
}
let myArray: StringArray;
myArray = ['白糖炒栗子',
    '前端'
]
```

#### 4. 5. 3 混合索引签名

属性和索引签名可以形成混合索引签名, 但是属性需要和索引签名类型匹配。

``` js
interface Dict {
    [index: string]： number；
    //正确
    length: number；
    //提示错误
    name: string；
}
```

另外, TS 允许同时使用上述两种签名, 但是数字索引返回值的类型, 它必须是字符串索引返回值类型的子类型。

因为对于 JS 来说, 当使用数字索引时, 会将它转换成字符串进行索引。所以它们需要保持一致。

``` js
interface TwoD {
    x: number；
    y: number；
}
interface ThreeD {
    x: number；
    y: number；
    z: number；
}
//正确
interface PointA {
    [index: number]： ThreeD；
        [prop: string]： TwoD；
}
//错误
interface PointB {
    [index: number]： TwoD；
        [prop: string]： ThreeD；
}
```

上述代码中, ThreeD 是 TwoD 的子类型, 所以接口 PointA 正确。

#### 4. 5. 4 只读索引签名

可以将索引签名设置为只读, 只能在数组初始化时定义其值, 创建后不能进行修改。

``` js
interface ReadonlyStringArray {
    readonly[index: number]： string；
}
let myArray: ReadonlyStringArray = ['白糖炒栗子']
//提示错误
myArray[1] = 'test'
```

### 4. 6 函数类型

函数类型具有调用签名。

``` typescript
interface Fun {
    //调用签名
    (arg1： string,  arg2： number)： boolean；
}
```

如上述代码所示, 调用签名包括参数列表和返回值类型。

对于函数类型来说, 它校验的值当然是函数 :

``` js
interface Sum {
    (a: number, b: number): number;
}
let sum: Sum = function(numl, num2) {
    return a + b
}
sum(1, 2)
```

如上述代码所示, 函数的参数名可以与签名的参数名不同。关键是对应位置的参数类型需要相同。

### 4. 7 类类型

这里的概念有些复杂, 如果有良好的 JS 基础, 会较易理解。

类的关键字是 Class, 由 ES6 开始引入, 并逐步完善。本质上 Class 属于语法糖, 是基于 prototype 原型链实现的。

``` js
//ES5
function Person(age) {
    this.age = age;
}
Person.prototype.getAge = function() {
    return this.age
}
//ES6
class Person {
    constructor(age) {
        this.age = age
    }
    getAge() {
        return this.age
    }
}
```

**而无论是 ES5 或 ES6, 属性 age 是在创建的实例上, 而方法 getAge() 是在实例的原型链上。**

``` js
let p = new Person(24)
p.age === 24
p.getAge() === 24
```

而类本身, 它是不存在 age 属性 和 getAge() 方法的。

``` js
Person.age === undefined
Person.getAge === undefined
```

当然, 我们也可以给类本身定义属性和方法。 但给类本身定义的属性和方法并不能通过实例直接访问。

``` js
Person.type = '超人'
Person.showPower = function() {
    console.log('superman')
}
p.type === undefined
p.showPower === undefined
```

所以, 类与实例的属性和方法是割裂的。

**TS 将描述 `类` 的属性和方法部分称为 `类的静态部分类型` , **

**将描述 `实例` 的属性和方法部分称为 `类的实例部分类型` 。**

``` js
interface PersonIFS {
    age: number;
    getAge(): number;
}
class Person implements PersonIFS {
    age: number;
    constructor(age: number) {
        this.age = age
    }
    getAge(): number {
        return this.age
    }
}
```

在上述代码中, 使用关键字 implements 描述类实现了接口。更准确的是：描述 类的实例部分 实现了接口。

实现该接口的类的实例, 它是具有 age 属性, getAge() 方法。而类本身(即类的静态部分)具有何种属性与方法, 上述代码没有进行定义。

所以, 这里所说的类类型, 实际上是类(实例的)类型。

关于如何定义类的静态部分的类型, 在后续会详细介绍。

### 4. 8 接口继承

接口的可以使用关键字 extends 定义继承。一个接口可以继承多个接口。

``` js
interface TwoD {
    x: number;
    y: number;
}
interface ThreeD extends TwoD {
    z: number;
}
let point: ThreeD = {
    x: 1,
    y: 2,
    z: 3
}
```

### 4. 9 混合类型

一个对象可能混合多种类型。

例如定义一个带版本号的函数：

``` js
interface Fn {
    (arg: string): void;
    version: number;
}

function fn(): Fn {
    let fn = < Fn > function(arg: string) {}
    fn.version = 1
    return fn
}
```

值得注意, 上述使用类型断言, 将函数断言为 Fn, 即使 fn 在创建时并不存在 version 属性。

### 4. 10 接口继承类

当接口继承类类型时候, 表现在继承类的成员和结构, 但不包括其实现。

接口继承类的一个场景是, 定义一个子类的类类型。

``` js
class Rect {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = X;
        this.y = y;
    }
    getRect() {
        return this.x * this.y;
    }
}

interface RectIFS extends Rect {
    z: number;
    getCube(): number;
}
class Cube extends Rect implements RectIFS {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    getCube() {
        return this.x * this.y * this.z;
    }
}
```

## 五、类

类在上文有所提及, 这里作进一步了解。

### 类的典型结构

包括属性、方法、构造函数。使用关键字 new 创建类的实例。

``` js
class Rect {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y
    }
    getRect() {
        return this.x * this.y
    }
    let rect = new Rect(1, 2)
```

### 5. 1 类的继承

使用关键字 extends 定义类的继承。

``` js
class Rect {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    getRect() {
        return this.x * this.y;
    }

    class Cube extends Rect {
        z: number;
        constructor(x: number, y: number, z: number) {
            super(x, y)
            this.z = z
        }
        getCube() {
            return this.x * this.y * this.z
        }
    }
```

### 5. 2 类的成员修饰符

#### 5. 2. 1 public

类的成员修饰符默认是 public, 也可以显式声明。

``` js
class A {
    public a: string
    b: string
}
```

上述代码中, 属性 a 和 b 均被声明为公共的。

#### 5. 2. 2 private

将类的成员声明为私有的, 即只有类内部访问。

``` js
class Rect {
    private x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}
let rect = new Rect(1, 2)
//提示错误rect.x
//正确
rect.y
```

TS 是 `结构性类型系统(鸭式辨形)` , 如果两种类型的结构相同, 一般情况下, TS 会认为它们是兼容的。

``` js
class Rect {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}
//Point 类型与Rect类型兼容
let rect: Point = new Rect(1, 2)
```

可是, 当类的成员带有 private 或 protected 修饰符时, 情况却不一样。

``` js
class Rect {
    private x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    class Point {
        private x: number;
        y: number;
        constructor(x: number, y: number) {
            this.x = x;
            this.y = y;
        }
    }
    //报错
    let rect: Point = new Rect(1, 2)
```

上述代码中, 虽然看上去它们的结构相同。但是, 当一个类的成员中存在 private 或 protected 时, 如果需要另一个类与它兼容, 则需要两个类的声明来源于同一份声明, 例如通过继承。

``` js
class Rect {
    private x: number;
    y: number
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    class Point extends Rect {}
    //Point 类型与Rect类型兼容
    let rect: Point = new Rect(1, 2)
```

#### 5. 2. 3 protected

修饰符 protected 和 private 类似, 区别在于使用 protected 修饰的成员, 在派生类可以访问。

``` js
class Rect {
    private x: number;
    protected y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}
class Cube extends Rect {
    public z: number;
    constructor(x: number, y: number, z： number) {
        super(x, y)
        this.z = z
    }
    getx() {
        //提示错误
        return this.x
    }
    getY() {
        //正确
        return this.y
    }
}
```

#### 5. 2. 4 readonly

修饰符 readonly 将属性设置为只读。

``` js
class Rect {
    readonly x: number;
    readonly y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    setx() {
        //提示错误
        this.x = 1
    }
}
```

将属性设置为只读, 只能在初始化时候定义其值, 定义后不能修改。

#### 5. 2. 5 static

在类类型中, 我们区分了类的静态部分和类的实例部分。修饰符 static 将类成员声明为静态成员。

``` js
class Rect {
    x: number;
    y: number;
    static version: number = 1;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    static getVersion() {
        return Rect.version
    }
}
//正确
Rect.verson
//正确
Rect.getVersion
```

#### 5. 2. 6 抽象类

我们创建基类时, **如果不希望基类能被实例化, 可以使用 protected 修饰符**。

``` js
class Rect {
    x: number;
    y: number;
    protected constructor(x: number, y： number) {
        this.x = x;
        this.y = y;
    }
}
class Cube extends Rect {
    z: number;
    constructor(x: number, y: number, z： number) {
        super(x, y)
        this.z = z
    }
}
//提示错误
let rect = new Rect(1, 2)
//正确
let cube = new Cube(1, 2, 3)
```

更普遍的做法是, `使用 abstract 修饰符` 。 `abstract 能定义抽象类和抽象类中的抽象方法` 。

``` js
abstract class Rect {
    x: number;
    y: number;
    constructor(x: number, y： number) {
        this.x = x;
        this.y = y;
    }
    //抽象方法
    abstract getRect()： number
}
class Cube extends Rect {
    z: number;
    constructor(x: number, y: number, z： number) {
        super(x, y)
        this.z = z
    }
    getRect() {
        return this.x * this.y;
    }
}
//提示错误
let rect = new Rect(1, 2)
//正确
let cube = new Cube(1, 2, 3)
```

#### 5. 2. 7 getter 和 setter

很多时候, 对某个属性读取时, 同时需要对其进行处理, 例如。

``` js
class Name {
    constructor(public xing: string, public ming: string) {}
    getFulLName() {
        return `${this.xing}${this.ming}` ;
    }
    setFullName(name) {
        this.xing = name.split('')[0];
        this.ming = name.split('')[1];
    }
}

let someone = new Name('张', '三')

someone.getFullName()
//"张三"

someone.setFullName('李四')

someone.getFullName()
//"李四"
```

更加简单的办法是, 通过 存储器 处理。

getter 和 setter 统称 `存取器` , 其实在 ES5 就已经存在, 很多朋友没有留意。如果对存取器不熟悉的, 也可以通过 TS 来理解。

``` js
class Name {
    constructor(public xing: string, public ming: string) {}
    get fullName() {
        return `${this.xing}${this.ming}` ;
    }
    set fullName(name) {
        this.xing = name.split('')[0];
        this.ming = name.split('')[1];
    }
}

let someone = new Name('张', '三')

someone.fullName
//"张三"

someone.fullName = '李四')

someone.fullName
//"李四"
```

## 六、函数

在 TS 中, 有几种方式定义函数类型。

### 6. 1 函数签名

利用上文提到的函数签名来定义函数类型。

``` js
interface Sum {
    (a: number, b: number): number;
    let sum: Sum = function(numl, num2) {
        return a + b;
    }
}
sum(1, 2)
```

### 6. 2 完整的函数类型

除了使用函数签名外, 还可以直接对函数进行类型描述。

``` js
let sum: (a: number, b: number) => number =
    function(a: number,
        b: number
    ): number {
        return a + b;
    }
```

但是, 这种方式毫无疑问是太繁琐了, TS 存在类型推断的功能, 它能根据等号一侧已存在的函数类型, 而推断出另一侧的函数类型, 从而简化代码。

### 6. 3 类型推断

函数推断简化代码

``` js
//省略左侧类型描述
let sum = function(a: number, b: number)： number {
    return a + b
}

//省略右侧类型描述
let sum: (a: number, b: number) => number =
    function(a, b) {
        return a + b;
    }
```

### 6. 4 可选参数

在 JS 中, 参数都是可选的, 如果函数声明了参数, 但是实际没有传参, 则参数值是 undefined。

``` js
function foo(a) {
    return a;
}
foo();
//undefined
```

但确实存在某个参数可能存在, 可能不存在的情况。这时候就需要用到可选参数了。

``` js
function foo(a: number, b ? ：number) {
    if (b) {
        return b
    } else {
        return a
    }
    //正确
    foo(1)
```

值得注意的是, **`可选参数必须跟在必须参数之后。`**

### 6. 5 默认参数

在 ES6 中, 可以给参数设定默认值。

``` js
function foo(a, b = 1) {
    return a + b
}

foo(1)
//2
```

而在 TS 中, 默认参数的特性也是和 ES6 一致, 而**默认参数的类型**是： `默认值和 undefined 的联合类型` 。

``` js
function foo(a: number, b = 1) {
    return a + b
}

//正确
foo(1)
//正确
foo(1, 2)
//正确
foo(1, undefined)
//提示错误
foo(1, '11')
```

### 6. 6 剩余参数

在 ES6 中, 引入了剩余参数, 代表由剩余所有参数组成的参数数组。

``` js
function foo(a, ...rest) {
    return rest.length;
}
foo(1, 2, 3)
//2
```

在 TS 中给剩余参数添加类型, 和普通参数接近, 区别在于剩余参数肯定是数组类型。

``` js
function foo(a: number, ...rest: number[]) {
    return rest.length
}

foo(1, 2, 3)
//2
```

### 6. 7 函数重载

函数根据传入不同的参数而返回不同的数据, 这是十分普遍的情况。

``` js
function foo(a) {
    if (typeof a === 'number') {
        return true
    }
    if (typeof a === 'boolean') {
        return 0
    }
    return undefined
}
```

如何对这种情况进行类型描述呢？有些朋友可能会想到使用联合类型进行处理。

``` js
function foo(a: number | boolean): number | boolean | undefined {
    if (typeof a === 'number') {
        return true;
    }
    if (typeof a === 'boolean') {
        return 0;
    }
    return undefined
}
```

但是, 这样不能很好地描述输入和输出类型的关系。

在上述例子中, 参数和返回存在对应关系：参数是 number 类型, 则返回 boolean, 参数是 boolean 类型, 则返回 number, 其他则返回 undefined。

此时, 我们可以运用函数重载描述这些对应关系：

``` js
function foo(a: number): boolean;

function foo(a: boolean): number;

function foo(a: any): undefined;

function foo(a: any): any {
    if (typeof a === 'number') {
        return true
    }
    if (typeof a === 'boolean') {
        return 0
    }
    return undefined
}

foo(1)
foo(true)
foo(undefined)
```

上述代码中, 定义了三个 foo 函数的重载, 需要注意两点。

1. 具体的函数声明不属于重载部分, 即 **function foo(a: any): any 并不是重载, 而是具体的函数声明**, 所以例子中只有三个函数重载；
2. 当**匹配重载列表时候, 是根据定义时候的顺序 `由上到下匹配` 的**。在例子中, 传入参数是 undefined 时, 首先尝试匹配 number, 然后尝试匹配 boolean, 最后尝试匹配 any, 匹配成功则使用 any 这个重载定义。

## 7、泛型

泛型其实并不高深, 借助几个已知概念可以帮助理解。

### 7. 1 泛型函数

上文提到, 当 `函数的参数` 和 `返回的类型` 存在联系时, 我们使用了函数重载进行类型描述。

假如希望函数的参数和返回的值, 它们之间的类型一致。如果使用函数重载, 则需要：

``` js
function foo(a: number): number;

function foo(a: boolean): boolean;

function foo(a: undefined): undefined;

function foo(a: string): string;

function foo(a: null): null;

function foo(a: object): object;

function foo(a: any): any {
    return a foo(1);
}

foo(true)
foo([1, 2, 3])
```

这显得十分繁琐。而且, 当传入的参数是数组或其他对象时, 匹配的是函数重载的 object 部分, 返回值的类型也是 object, 丢失了原有的结构。

我们期望即使传入复杂的结构对象, 也保留它们的类型描述。像上述代码中, 传入是 number[] 类型的参数, 期望返回的也是 number[] 类型描述, 而不是 object 类型描述。

而使用泛型就能解决这个问题。以下借助变量的概念来对泛型进行解释。

变量我们都清楚, 当变量被赋值成某值或对象时, 该变量就代表着某值或对象。

而在类型系统中, 通过引入类型变量, 达到和变量相同的效果：当类型变量被赋值成某类型, 则类型变量就代表该类型。

``` js
function foo < T > (a: T): T {
    return a;
}
```

上述代码中, 在函数名后添加 `<T>` , 代表 `在类型系统中声明了类型变量 T` 。所以, 在对参数 a 和返回值进行类型描述时, 我们可以使用 T 这个类型变量。

另外, 上述代码使用字母 T 代表类型变量, 并不是固定的, 也可以使用字母 A、a 等合法的变量名。只不过约定俗成, T 代表着 Type(类型), 这样其他人阅读代码也一目了然。

既然是变量, 我们怎么给这个类型变量传参呢？

``` js
function foo < T > (a: T): T {
    return a;
}
foo < number > (1);
foo < boolean > (true);
foo < number[] > ([1, 2, 3]);
```

可以清晰地看到, 在调用函数时, 使用尖括号传入具体的类型值。以 `foo<number[]>([1, 2, 3])` 为例, 相当于应用 `function<number[]>(a: number[]): number[]` 类型校验。这样, 在参数和返回值中都保留了明确的数据结构。

上文我们提到类型推断, 同样在泛型中也可以应用。

``` js
function foo < T > (a: T): T {
    return a;
}

foo(1)
foo(true)
foo([1,2,3])
```

上述代码中, 省略了明确的类型传参, 但 TS 会根据我们传入的参数的类型, 自动确定 T 的类型。

### 7. 2 泛型接口

我们知道, 接口是具有函数签名的, 形如：

``` js
interface foo {
    (arg: string): string;
}
```

接口中的函数签名也存在泛型的形式：

``` js
interface foo {
    <
    T > (arg: T): T;
}
```

另外, 还可以将泛型参数作为接口的参数：

``` js
interface foo < T > {
    (arg: T): T;
}
```

### 7. 3 泛型类

**泛型类**和**泛型接口**类似, 声明泛型参数后, 在类中就可以使用。

``` js
class Rect < T > {
    x: T
    y: T
    getRect: () => T
}
```

## 结语

以上就是关于 TS 的关键知识点, 熟悉上下两篇文章的内容, 即可阅读和使用大部分的 TS 代码。

TS 的学习要点不多, 一个周末时间基本能搞明白。

学习 TS 的资料主要是官网, 特别是 TS handbook, 本文大部分内容也是基于其整理的, 希望能给大家带来一些帮助。

## THANK

本文主要摘自：

[一文学会 TypeScript 的 82% 常用知识点(上)](https://www.cnblogs.com/sexintercourse/p/11961086.html)

[一文学会 TypeScript 的 82% 常用知识点(下)](https://www.cnblogs.com/sexintercourse/p/11961089.html)
