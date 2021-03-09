# 004-JS中的变量和函数提升(预处理/预编译)

<motto></motto>

在一个前端备战群里看到一道题主要是用到了闭包，但是，各位大佬们却都未能做出，这是因为里面还用到了一个知识点——变量提升。

虽然在这道题里并没有很好地体现变量提升，但是却也是提醒了我需要复习下这部分了，本题的答案就是因为变量提升的规则，而导致了将函数覆盖，假如在count前加上var，那么答案就又不对了。

下面给来解释一下变量提升和函数提升(该题在[闭包](http://lskreno.top:4000/interview/002-closure/#题目1：)中也有提到)。

## 一、一道面试题目

``` js
function count() {
    var n = 0;
    // code here,不可使用全局变量
    ...
}
count() // 输出1
count() // 输出2
count() // 输出3
```

一看到这个题，我们就会想到，这用闭包啊！但是当真的用起来的时候又会觉得emmmm，好像不对劲，是不是这题出错了？？？

用我们常用的闭包知识来考虑下，闭包有两个用处：

* 前面提到的可以读取函数内部的变量
* 让这些函数内部的变量的值始终保持在内存中，不会被垃圾回收。

那我们最熟悉的就是在函数a中有一个变量n，同时函数b在函数a中定义并且函数b使用了变量n，函数a将函数b给return出来。

``` js
function a() {
    var n = 0;

    function b() {
        n++;
        console.log(n);
    }
    return b;
}
var a1 = new a();
a1(); //1， a1事实上就是函数b，所以可以不停地增加
a1(); //2
```

实际上就是将函数a当做了构造函数在用。

因为a1引用了那个new出来的内存地址，所以那个内存地址所保存的对象不会被垃圾回收。

a1事实上就是函数b，所以n可以不停地增加。

---

可是我们再来看那道题，并没有将count()当做构造函数因为没有new出一个引用类型的对象。所以我们就陷入了沉思的误区。

### 答案：

``` js
function count() {
    var n = 0;
    count = function() {
        console.log(n++);
    }
    count();
}
count(); //0
count(); //1
count(); //2
```

如果你不明白为什么这样写的话，别着急，先看完下面的知识，相信回过头来，你会明白的。

我们先来了解下JS预处理。

## 二、JS中的解析和执行过程

### 1. 什么是预处理

预处理就是，在预处理后创建了一个词法环境（Lexical Environment，在后面简写为LE），扫描JS中的用声明的方式声明的函数，用var定义的变量并将它们加到预处理阶段的词法环境中去。

### 2. 全局环境下的预处理

``` js
var a = 1; //用var定义的变量，且已赋值
var b; //用var定义的变量，未赋值
c = 3; //未定义，直接赋值
function d() { //用声明的方式声明的函数
    console.log('hello');
}
var e = function() { //函数表达式
    console.log('world');
}
```

在预处理后，根据这段代码所创建的词法作用域可以这样表示：

``` js
LE { //此时的LE相当于window
    a: undefined
    b: undefined
    // 没有c
    d: 对函数的一个引用
    // 没有e
}
```

也就是说，在未执行到某行代码时，

* var声明的变量不管赋没赋值都会在词法作用域中是undefined。
* 直接声明的函数会在词法作用域中有对应的函数引用。
* `未声明的变量赋值` 和 `函数表达式` 都不会在词法作用域中创建对应的变量，只有在执行到该行代码时，才会创建变量，所以会报错。

#### 需要注意的有两点：

1、预处理的函数必须是JS中用声明的方式声明的函数（不可以是函数表达式）

``` js
foo();
bar();

function foo() { //用声明的方式声明的函数
    console.log('foo');
}
var bar = function() { //函数表达式
    console.log('bar');
}
```

打印结果分别为：

``` js
// foo
// Uncaught TypeError: bar is not a function at <anonymous>:2:1
```

可以从结果看出，用函数表达式的方式并不会在预处理中进行。

2、与处理的函数必须是用var定义的变量

``` js
console.log(a); //undefined
console.log(b); //undefined
console.log(c); //报错：c is not defined
var a = 1;
var b;
c = 3;
```

> 下面我们来看最容易迷惑的地方。

### 3. 命名冲突的预处理

> 请先看完下面的小栗子，再对照着看命名冲突的总结

#### 栗子1

``` js
console.log(foo);
var foo = 1;
var foo = 2;
```

打印结果为：

``` js
// undefined
```

#### 栗子2

``` js
console.log(foo);

function foo() {
    console.log('foo');
}

function foo() {
    console.log('foo bar');
}
```

打印结果为：

``` js
// function foo(){
//     console.log('foo bar');
// }
```

#### 栗子3

``` js
console.log(foo);
var foo = 1;

function foo() {
    console.log('foo');
}
```

打印结果为：

``` js
// function foo(){
//    console.log('foo');
// }
```

#### 栗子4

``` js
console.log(foo);

function foo() {
    console.log('foo');
}
var foo = 1;
```

打印结果依然为：

``` js
// function foo(){
//    console.log('foo');
// }
```

#### (1)命名冲突的预处理总结

1. 两个变量名一样时会忽略掉一个，或者理解为合并成一个(具体是后面的覆盖前面的还是后面的就直接被忽略掉了，还需要我未来进行补坑，不好意思)。
2. 两个函数名一样时，后面的函数会覆盖掉前面的函数。
3. 函数名和变量名一样时，以函数为主，预处理的结果最后指向对函数声明的引用。(如果你见到过 `CSS权重` 的概念的话，那么可以理解为 `函数的权重` 远比 `变量的权重` 要大)

### 4. 函数预处理

函数和全局的解析和执行过程的区别不大。

> 请认真体会下面的小栗子

#### 栗子1

``` js
console.log(a) // undefined
console.log(c) // 报错 Uncaught ReferenceError: c is not defined
var a = 1;
b();

function b() {
    console.log('b') // b
    console.log(c) //undefined
    var c = 1;
}
```

#### 栗子2

``` js
console.log(a) // undefined
console.log(c) // 报错 Uncaught ReferenceError: c is not defined 
var a = 1;
b();

function b() {
    console.log('b') // b
    console.log(c) //报错 Uncaught ReferenceError: c is not defined 
    c = 1;
}
console.log(c) //1
```

上面两个栗子，能搞明白输出，就是懂函数的预处理了。

**栗子2中要注意，如果没有用var声明的变量，在执行到该代码时，该变量会变成最外部LE的成员，即全局变量**

> 但是，还需要注意的是**函数中的参数arguments**。

#### 栗子3

``` js
function foo(a, b) {
    alert(a);
    alert(b);

    var b = 50;

    function a() {}
}
foo(1, 2);
```

分析函数的预处理，它和全局的预处理类似，它的词法结构如下：

``` js
LE {
    b: 2
    a: 指向函数的引用
    arguments： 2
}
//arguments，调用函数时实际调用的参数个数
```

根据上面的[命名冲突的预处理总结](#命名冲突的预处理总结)第三条：函数名和变量名一样时，以函数为主，预处理的结果最后指向对函数声明的引用。

可以得出，结果分别为：function a(){} 和 2

#### 栗子4

``` js
function foo(a, b) {
    alert(a);
    alert(b);

    var b = 50;

    function a() {}
}
foo(1);
```

分析函数的预处理，它和全局的预处理类似，它的词法结构如下：

``` js
LE {
    b: undefined
    a: 指向函数的引用
    arguments： 1
}
//arguments，调用函数时实际调用的参数个数
```

结果分别为：function a(){} 和 undefined

## 三、开头的题目的分析

### 1. 题目

``` js
function count() {
    var n = 0;
    // code here,不可使用全局变量
    ...
}
count() // 输出1
count() // 输出2
count() // 输出3
```

### 2. 答案

``` js
function count() {
    var n = 0;
    count = function() {
        console.log(n++);
    }
    count();
}
count(); //0
count(); //1
count(); //2
```

### 3. 分析

首先全局预处理：

``` js
LE {
    count: 对函数的引用
}
```

函数预处理：

``` js
LE {
    n: undefined
}
```

执行第8行代码：进入count()函数内部

执行第2行代码：

``` js
LE {
    n: 0
}
```

执行第3-5行代码：

因为count的函数表达式没有使用var进行定义，所以count被声明为全局变量，由于命名冲突，再根据命名冲突的总结，可以知道count变量被忽略，但是由于该行代码直接执行，所以后面的函数将原先定义的全局函数覆盖掉了。

执行第6行代码：count()前面什么也没有，实际上相当于是在全局环境下直接调用，因为刚刚最初的count函数已经被覆盖掉，所以依次调用可以打印出0, 1, 2。

有同学如果对 `为什么最初的count函数被覆盖了，后面的count函数依然可以访问在最初的count函数中定义的n变量?` 有疑问的话, 可以去看一下面试系列里的[闭包](http://lskreno.top:4000/interview/002-closure/)。相信你一定可以搞明白。

为了使大家更加清楚，我将函数内部的 `count函数表达式` 改成 `count = 100` , 看是否能够将 `windows.count` 替换掉。

``` js
function count() {
    var n = 0;
    count = 100
    console.log(count)
}
count(); // 100
count(); // Uncaught TypeError: count is not a function
count(); // Uncaught TypeError: count is not a function
```

根据结果我们可以知道，count已经不再是 `函数` 了，而是变成了 `100` 。

可能你突然被搞蒙了，觉得count函数应该不会被100覆盖掉，因为100是变量，而count是函数，根据命名冲突会觉得不会被覆盖。但是，那只是在预处理阶段，并非是代码执行阶段，由于JS是弱类型，所以100完全可以替换掉count函数。

## 四、总结

### 1. 全局预处理：

1、预处理的函数必须是JS中用声明的方式声明的函数（不可以是函数表达式）。如果是函数表达式的话，就按照普通变量的规则来就好。(**有var声明的话，就是正常将其当做变量名提升为undefined ；没有var声明的话，就是执行到该行代码时，将该变量直接变为全局变量，并给该变量名赋值**)

2、预处理的函数必须是用var定义的变量

### 2. 函数预处理：

1、 `函数实参` 在传进来函数的时候，就会直接对该变量进行赋值。

2、函数里的函数名和形参命名冲突时，变量会指向函数而不是实参。

3、如果没有用var声明的变量，在执行到该代码时，该变量会变成最外部LE的成员，即全局变量。

### 3. 命名冲突的预处理总结

* 两个变量名一样时会忽略掉一个，或者理解为合并成一个(具体是后面的覆盖前面的还是后面的就直接被忽略掉了，还需要我未来进行补坑，不好意思)。
* 两个函数名一样时，后面的函数会覆盖掉前面的函数。
* 函数名和变量名一样时，以函数为主，预处理的结果最后指向对函数声明的引用。(如果你见到过 `CSS权重` 的概念的话，那么可以理解为 `函数的权重` 远比 `变量的权重` 要大)

> 小解惑：foo和bar变量名就像是我们中文常用的 `张三李四/某某某` 一样

