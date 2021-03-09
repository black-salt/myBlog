# 三、ES6 新特性（2015）

<motto></motto>

ES6的特性比较多，在 ES5 发布近 6 年（2009-11 至 2015-6）之后才将其标准化。两个发布版本之间时间跨度很大，所以ES6中的特性比较多。 在这里列举几个常用的：

* 类
* 模块化
* 箭头函数
* 函数参数默认值
* 模板字符串
* 解构赋值
* 延展操作符
* 对象属性简写
* Promise
* let与const

## 1. 类（class）

对熟悉Java，object-c，c#等纯面向对象语言的开发者来说，都会对class有一种特殊的情怀。ES6 引入了class（类），让JavaScript的面向对象编程变得更加简单和易于理解。

``` js
  class Animal {
      constructor(name, color) {
          this.name = name;
          this.color = color;
      }
      toString() {
          console.log('name:' + this.name + ',color:' + this.color);
      }
  }
  var animal = new Animal('dog', 'white');
  animal.toString();
  console.log(animal.hasOwnProperty('name'));
  console.log(animal.hasOwnProperty('toString'));
  console.log(animal.__proto__.hasOwnProperty('toString'));

  class Cat extends Animal {
      constructor(action) {
          // 如果没有置顶consructor,默认带super函数的constructor将会被添加  
          super('cat', 'white');
          this.action = action;
      }
      toString() {
          console.log(super.toString());
      }
  }
  var cat = new Cat('catch') cat.toString();

  console.log(cat instanceof Cat);
  console.log(cat instanceof Animal);
```

## 2. 模块化(Module)

ES5不支持原生的模块化，在ES6中模块作为重要的组成部分被添加进来。

模块的功能主要由 export 和 import 组成。每一个模块都有自己单独的作用域，模块之间的相互调用关系是通过 export 来规定模块对外暴露的接口，通过import来引用其它模块提供的接口。

同时还为模块创造了命名空间，防止函数的命名冲突。

### 导出(export)

ES6允许在一个模块中使用export来导出多个变量或函数。

**导出变量**

``` js
export var name = 'Rainbow'
```

> 心得：ES6不仅支持变量的导出，也支持常量的导出。
>
> `export const sqrt = Math.sqrt;//导出常量` 

ES6将一个文件视为一个模块，上面的模块通过 export 向外输出了一个变量。一个模块也可以同时往外面输出多个变量。

``` js
 var name = 'Rainbow';
 var age = '24';
 export {
     name,
     age
 };
```

**导出函数**

``` js
export function myModule(someArg) {
    return someArg;
}
```

### 导入(import)

定义好模块的输出以后就可以在另外一个模块通过import引用。

``` js
import {
    myModule
} from 'myModule';
import {
    name,
    age
} from 'test';
```

> 心得: 一条import 语句可以同时导入默认函数和其它变量。 
>
> `import default Method, { otherMethod } from 'xxx.js';` 

## 3. 箭头（Arrow）函数

这是ES6中最令人激动的特性之一。 `=>` 不只是关键字function的简写，它还带来了其它好处。

箭头函数与包围它的代码共享同一个 `this` , 能帮你很好的解决this的指向问题。

有经验的JavaScript开发者都熟悉诸如 `var self = this;` 或 `var that =this` 这种引用外围this的模式。但借助 `=>` ，就不需要这种模式了。

### 箭头函数的结构

箭头函数的箭头=>之前是一个空括号、单个的参数名、或用括号括起的多个参数名，而箭头之后可以是一个表达式（作为函数的返回值），或者是用花括号括起的函数体（需要自行通过return来返回值，否则返回的是undefined）。

``` js
() => 1

v => v + 1

(a, b) => a + b

() => {
    alert("foo");
}

e => {
    if (e == 0) {
        return 0;
    }
    return 1000 / e;
}
```

> 心得：不论是箭头函数还是bind，每次被执行都返回的是一个新的函数引用。
>
> 因此如果你还需要函数的引用去做一些别的事情（譬如卸载监听器），那么你必须自己保存这个引用。

### 卸载监听器时的陷阱

> **错误的做法**

``` js
class PauseMenu extends React.Component {
    componentWillMount() {
        AppStateIOS.addEventListener('change', this.onAppPaused.bind(this));
    }
    componentWillUnmount() {
        AppStateIOS.removeEventListener('change', this.onAppPaused.bind(this));
    }
    onAppPaused(event) {}
}
```

> **正确的做法**

``` js
class PauseMenu extends React.Component {
    constructor(props) {
        super(props);
        this._onAppPaused = this.onAppPaused.bind(this);
    }
    componentWillMount() {
        AppStateIOS.addEventListener('change', this._onAppPaused);
    }
    componentWillUnmount() {
        AppStateIOS.removeEventListener('change', this._onAppPaused);
    }
    onAppPaused(event) {}
}
```

除上述的做法外，我们还可以这样做：

``` js
class PauseMenu extends React.Component {
    componentWillMount() {
        AppStateIOS.addEventListener('change', this.onAppPaused);
    }
    componentWillUnmount() {
        AppStateIOS.removeEventListener('change', this.onAppPaused);
    }
    onAppPaused = (event) => {}
}
```

> 需要注意的是：不论是bind还是箭头函数，每次被执行都返回的是一个新的函数引用，因此如果你还需要函数的引用去做一些别的事情（譬如卸载监听器），那么你必须自己保存这个引用。

## 4. 函数参数默认值

ES6支持在定义函数的时候为其设置默认值：

``` js
function foo(height = 50, color = 'red') {}
```

> 不使用默认值：

``` js
function foo(height, color) {
    var height = height || 50;
    var color = color || 'red';
}
```

这样写一般没问题，但当 `参数的布尔值为false` 时，就会有问题了。比如，我们这样调用foo函数：

``` js
foo(0, "")
```

因为 `0的布尔值为false` ，这样height的取值将是50。同理color的取值为‘red’。

所以说， `函数参数默认值` 不仅能是代码变得更加简洁而且能规避一些问题。

## 5. 模板字符串

ES6支持 `模板字符串` ，使得字符串的拼接更加的简洁、直观。

> 不使用模板字符串：

``` js
var name = 'Your name is ' + first + ' ' + last + '.'
```

> 使用模板字符串：

``` js
var name = `Your name is ${first} ${last}.` 
```

在ES6中通过 `${}` 就可以完成字符串的拼接，只需要将变量放在大括号之中。

## 6. 解构赋值

解构赋值语法是JavaScript的一种表达式，可以方便的从数组或者对象中快速提取值赋给定义的变量。

### 获取数组中的值

从数组中获取值并赋值到变量中，变量的顺序与数组中对象顺序对应。

``` js
var foo = ["one", "two", "three", "four"];
var [one, two, three] = foo;
console.log(one);
console.log(two);
console.log(three);
//如果你要忽略某些值，你可以按照下面的写法获取你想要的值var [first, , , last] = foo;console.log(first); console.log(last); 
//你也可以这样写var a, b; 
[a, b] = [1, 2];
console.log(a);
console.log(b);
```

如果没有从数组中的获取到值，你可以为变量设置一个默认值。

``` js
var a, b;
[a = 5, b = 7] = [1];
console.log(a);
console.log(b);
```

通过解构赋值可以方便的交换两个变量的值。

``` js
var a = 1;
var b = 3;
[a, b] = [b, a];
console.log(a);
console.log(b);
```

### 获取对象中的值

``` js
const student = {
    name: 'Ming',
    age: '18',
    city: 'Shanghai'
};
const {
    name,
    age,
    city
} = student;
console.log(name);
console.log(age);
console.log(city);
```

## 7. 延展操作符(Spread operator)

`延展操作符...` 

可以在 `函数调用/数组构造` 时, 

* 将 `数组表达式` 或者 `string` 在语法层面展开；

还可以在 `构造对象` 时, 

* 将对象表达式按key-value的方式展开。

### 语法

> 函数调用：

``` js
myFunction(...iterableObj);
```

> 数组构造或字符串：

``` js
[...iterableObj, '4', ...'hello', 6];
```

> 构造对象时, 进行克隆或者属性拷贝（ECMAScript 2018规范新增特性）：

``` js
let objClone = {
    ...obj
};
```

### 应用场景

> 在函数调用时使用延展操作符

``` js
function sum(x, y, z) {
    return x + y + z;
}
const numbers = [1, 2, 3];

console.log(sum.apply(null, numbers));

console.log(sum(...numbers));
```

> 构造数组

没有展开语法的时候，只能组合使用 push，splice，concat 等方法，来将已有数组元素变成新数组的一部分。有了展开语法, 构造新数组会变得更简单、更优雅：

``` js
const stuendts = ['Jine', 'Tom'];
const persons = ['Tony', ...stuendts, 'Aaron', 'Anna'];
conslog.log(persions)
```

和参数列表的展开类似, `...` 在构造字数组时, 可以在任意位置多次使用。

> 数组拷贝

``` js
var arr = [1, 2, 3];
var arr2 = [...arr];
arr2.push(4);
console.log(arr2)
```

展开语法和 Object.assign() 行为一致, 执行的都是浅拷贝(只遍历一层)。

> 连接多个数组

``` js
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
var arr3 = [...arr1, ...arr2]; //等同于var arr4 = arr1.concat(arr2);
```

### 在ECMAScript 2018中延展操作符增加了对对象的支持

``` js
var obj1 = {
    foo: 'bar',
    x: 42
};
var obj2 = {
    foo: 'baz',
    y: 13
};
var clonedObj = {
    ...obj1
};

var mergedObj = {
    ...obj1,
    ...obj2
};
```

### 在React中的应用

通常我们在封装一个组件时，会对外公开一些 props 用于实现功能。大部分情况下在外部使用都应显示的传递 props 。但是当传递大量的props时，会非常繁琐，这时我们可以使用 `...(延展操作符,用于取出参数对象的所有可遍历属性)` 来进行传递。

#### 一般情况下我们应该这样写

``` react
<CustomComponent name ='Jine' age ={21} />
```

> 使用 ... ，等同于上面的写法

``` react
const params = {    name: 'Jine',    age: 21}
<CustomComponent {...params} />
```

> 配合解构赋值避免传入一些不需要的参数

``` react
var params = {    name: '123',    title: '456',    type: 'aaa'}
var { type, ...other } = params;
<CustomComponent type='normal' number={2} {...other} />//等同于<CustomComponent type='normal' number={2} name='123' title='456' />
```

## 8. 对象属性简写

在ES6中允许我们在设置一个对象的属性的时候不指定属性名。

> 不使用ES6

``` js
const name = 'Ming',
    age = '18',
    city = 'Shanghai';
const student = {
    name: name,
    age: age,
    city: city
};
console.log(student);
```

对象中必须包含属性和值，显得非常冗余。

> 使用ES6

``` js
const name = 'Ming',
    age = '18',
    city = 'Shanghai';
const student = {
    name,
    age,
    city
};
console.log(student);
```

对象中直接写变量，非常简洁。

## 9. Promise

Promise 是异步编程的一种解决方案，比传统的解决方案callback更加的优雅。它最早由社区提出和实现的，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。

> 不使用ES6

嵌套两个setTimeout回调函数：

``` js
setTimeout(function() {
    console.log('Hello');
    setTimeout(function() {
        console.log('Hi');
    }, 1000);
}, 1000);
```

> 使用ES6

``` js
var waitSecond = new Promise(function(resolve, reject) {
    setTimeout(resolve, 1000);
});
waitSecond.then(function() {
    console.log("Hello");
    return waitSecond;
}).then(function() {
    console.log("Hi");
});
```

上面的代码使用两个then来进行异步编程串行化，避免了回调地狱：

## 10. 支持let与const

在之前JS是没有块级作用域的，const与let填补了这方面的空白，const与let都是块级作用域。

> 使用var定义的变量为函数级作用域：

``` js
{
    var a = 10;
}
console.log(a);
```

> 使用let与const定义的变量为块级作用域：

``` js
{
    let a = 10;
}
console.log(a);
```

