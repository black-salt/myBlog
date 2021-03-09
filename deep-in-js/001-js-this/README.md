# 001-JavaScript-JS中this的指向

<motto></motto>


> 本文将主要总结JS中this的指向

## 一、隐式绑定的场景:
- 全局上下文
- 直接调用函数
- 对象.方法的形式调用
- DOM事件绑定(特殊)
- new构造函数绑定
- 箭头函数

### 1. 全局上下文
全局上下文默认this指向window, 严格模式下指向undefined。

### 2. 直接调用函数
比如:
```js
let obj = {
  a: function() {
    console.log(this);
  }
}
let func = obj.a;
func();
```
这种情况是直接调用。this相当于全局上下文的情况(默认this指向window, 严格模式下指向undefined。)。

### 3. 对象.方法的形式调用
还是刚刚的例子，我如果这样写:
```js
obj.a();
```
这就是对象.方法的情况，this指向这个对象

### 4. DOM事件绑定
onclick和addEventerListener中 this 默认指向绑定事件的元素。

IE比较奇异，使用attachEvent，里面的this默认指向window。

### 5. new绑定
使用构造函数来`new`一个对象，会把构造函数中的`this`绑定到新new出来的对象上。

### 6. 箭头函数
箭头函数没有this, 因此也不能绑定。里面的this会指向当前最近的非箭头函数的this，**且指向函数定义时的this而非执行时**。找不到就是window(严格模式是undefined)。比如:
```js
let obj = {
  a: function() {
    let do = () => {
      console.log(this);
    }
    do();
  }
}
obj.a(); // 找到最近的非箭头函数a，a现在绑定着obj, 因此箭头函数中的this是obj
```

注意：

1. 字面量创建的对象，作用域是`window`，如果里面有箭头函数属性的话，`this`指向的是`window`
2. 箭头函数的`this`无法通过`bind、call、apply`来**直接**修改，但是可以通过改变作用域中`this`的指向来间接修改。

#### 避免使用箭头函数的场景

> 该部分总结来自[LinDaiDai_霖呆呆](https://juejin.im/post/5e6358256fb9a07cd80f2e70#heading-43)

根据箭头函数的特性，所以我们应该**避免**在以下四种场景中使用它：

1. **使用箭头函数定义对象的方法**

```js
let obj = {
    value: 'LinDaiDai',
    getValue: () => console.log(this.value)
}
obj.getValue() // undefined
```

2. **定义原型方法**

```js
function Foo (value) {
    this.value = value
}
Foo.prototype.getValue = () => console.log(this.value)

const foo1 = new Foo(1)
foo1.getValue() // undefined
```

3. **构造函数使用箭头函数**

```js
const Foo = (value) => {
    this.value = value;
}
const foo1 = new Foo(1)
// 事实上直接就报错了 Uncaught TypeError: Foo is not a constructor
console.log(foo1);
```

4. **作为事件的回调函数**

```js
const button = document.getElementById('myButton');
button.addEventListener('click', () => {
    console.log(this === window); // => true
    this.innerHTML = 'Clicked button';
});
```

## 二、显式绑定的场景:

注意点：

- 使用`call()`或者`apply()`的函数是会直接执行的
- `bind()`是创建一个新的函数，需要手动调用才会执行
- `call()`和`apply()`用法基本类似，不过`call`接收若干个参数，而`apply`接收的是一个数组

> call, apply, bind都是为了调用函数，然后改变函数中的this的指向。
> 它们允许你在调用函数时为函数指定上下文

### 1.call
- call用于显式地设置函数的上下文，call方法将对象绑定到this。
- call 需要分别传参
- call会立即执行函数
```js
function show(title) {
    alert(`${title+this.name}`);
}
let foo = {
    name: 'foo'
};
let bar = {
    name: 'bar'
};
show.call(foo, '傻憨憨'); //重点
```
### 2.apply
> apply与call用于显式地设置函数的上下文，两个方法作用一样都是将对象绑定到this，只是在传递参数上有所不同。
>
> 因为，我们可以看到，call方法必须一个一个传递参数，这样有些恼人。如果我们可以把整个数组作为第二个参数并让 JavaScript 为我们自动展开就好了。
>
> 所以，这个时候apply，就派上用场了
>
> apply 用数组传参
>
> call 需要分别传参
>
> 与 bind 不同 call/apply 会立即执行函数


- apply用于显式地设置函数的上下文，apply方法将对象绑定到this。
- apply 需要分别传参
- apply会立即执行函数

```js
function show(title) {
    alert(`${title+this.name}`);
}
let foo = {
    name: 'foo'
};
let bar = {
    name: 'bar'
};
show.apply(foo, ['傻憨憨']); //重点
```
> 也就是说，apply和call如果不传参数的话，只传想要改变的this的指向，那他们用法，效果都是一样的。
>
> func().call(self) //意思是将func()这个函数中的this指向self
>
> func().apply(self) //意思是将func()这个函数中的this指向self

**小栗子：**
找数组中的数值最大值
```js
let arr = [1, 3, 2, 8];
console.log(Math.max(arr)); //NaN
console.log(Math.max.call(Math, arr)); //NaN, 因为call除了要传的this以外，后面的参数必须用逗号隔开，现在传的是数组
console.log(Math.max.apply(Math, arr)); //8， apply后的参数传的是数组
console.log(Math.max(...arr)); //8， 当然我们用展开语法也是完全可以的
```

### 3.bind
bind()是将函数绑定到某个对象，比如 a.bind(hd) 可以理解为将a函数绑定到hd对象上即 hd.a()。
其实，`.bind` 和 `.call` 完全相同，除了不会立刻调用函数，而是返回一个能以后调用的新函数。

> 与 call/apply 不同bind不会立即执行
>
> bind 是复制函数行为,会返回新函数
>
> bind是复制函数行为
```js
let a = function() {};
let b = a;
console.log(a === b); //true
//bind是新复制函数
let c = a.bind();
console.log(a == c); //false
```
然后呢，因为调用bind方法并不会立即执行该函数，所以就给了我们两次传参的机会；
绑定参数注意事项
```js
function func(a, b) {
  return this.f + a + b;
}

//使用bind会生成新函数
let newFunc = func.bind({ f: 1 }, 3);

//1+3+2 参数2赋值给b即 a=3,b=2
console.log(newFunc(2)); //注意如果在前面bind的时候给某些参数赋值过了，在这里就不需要赋值了，现在赋了也没用
```

> 优先级: new  > call、apply、bind  > 对象.方法 > 直接调用。

### 4.数组的forEach、map、filter

`forEach、map、filter`函数的第二个参数可以显式绑定`this`。

```js
function foo (item) {
  console.log(item, this.a)
}
var obj = {
  a: 'obj'
}
var a = 'window'
var arr = [1, 2, 3]

// arr.forEach(foo, obj)
// arr.map(foo, obj)
arr.filter(function (i) {
  console.log(i, this.a)
  return i > 2
}, obj)
```

输出：

```js
1 "obj"
2 "obj"
3 "obj"
```

## 三、练习题

**LinDaiDai_霖呆呆**的整理，加深你的this理解，一定要做啊！！！尤其是显示绑定和箭头函数部分。

[【建议👍】再来40道this面试题酸爽继续(1.2w字用手整理)](https://juejin.im/post/5e6358256fb9a07cd80f2e70)

