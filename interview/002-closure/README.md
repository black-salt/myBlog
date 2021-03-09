# 002: 闭包

<motto></motto>

## 什么是闭包？

MDN 对闭包的定义为：

> 闭包是指那些能够访问自由变量的函数。

自由变量:

> 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。

所以，闭包共有两部分组成：

> 闭包 = 函数 + 函数能够访问的自由变量

闭包（closure）是javascript的一大难点，也是它的特色。很多高级应用都要依靠闭包来实现。

要理解闭包，首先要理解javascript的全局变量和局部变量。

javascript语言的特别之处就在于：

函数内部可以直接读取全局变量，但是在函数外部无法读取函数内部的局部变量。

``` js
function f1() {
    var a = 10;

    function f2() {
        alert(a); //10
    }
```

## 如何从外部读取函数内部的局部变量？

我们有时候需要获取到函数内部的局部变量，正常情况下，这是办不到的！只有通过变通的方法才能实现。那就是在函数内部，再定义一个函数。

## 1、闭包的概念

上面代码中的f2函数，就是闭包。

各种专业文献的闭包定义都非常抽象，我的理解是: 

> 闭包就是能够读取其他函数内部变量的函数。

由于在javascript中，只有函数内部的子函数才能读取局部变量，所以说，闭包可以简单理解成“ `定义在一个函数内部的函数` “。

所以，在本质上，闭包是将函数内部和函数外部连接起来的桥梁。

## 2. 闭包的核心作用

闭包的两大核心作用：

1. 读取函数内部的变量；
2. 让变量始终保存在内存中。

## 3、闭包的用途

闭包可以用在许多地方。它的最大用处有两个，一个是前面提到的可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中，不会在f1调用后被自动清除。

为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。

在我们平时的代码中经常会用到闭包，

### 构造函数中使用闭包

第一种写法：

``` js
function a() {
    var n = e；
    this.add = function() {
        n++；
        console.log(n)；
    }；
}
var c = new a()；
c.add()； //控制台输出1
c.add()； //控制台输出2
```

第二种写法：

``` js
function a() {
    this.n = 0;
    this.add = function() {
        this.n++;
        console.log(this.n);
    };
}
var c = new a();
c.add(); //控制台输出1
c.add(); //控制台输出2
```

### 常见闭包写法

两种常见闭包：

``` js
function a() {
    var n = 0;

    function add() {
        n++;
        console.log(n);
    }
    return add;
}

var a1 = new a(); //注意，函数名只是一个标识（指向函数的指针），而（）才是执行函数；
a1(); //控制台输出1
a1(); //控制台输出2

// 另一种调用方法
function a() {
    var n = 0;

    function add() {
        n++;
        console.log(n);
    }
    return add;
}
a()() //控制台输出1，因为a()的返回值就是函数b，所以可以再进行一次调用，但是只能调用一次，我们永远也看不到2 
```

定义函数并立即调用（IIFE）

``` js
var a = (function() {
    var n = 0;
    return function() {
        n++;
        console.log(n);
    }
}())
a()
```

## 4. 闭包的实际应用

使用闭包可以：

* 模拟面向对象的代码风格；

* 更优雅，更简洁的表达出代码；

* 在某些方面提升代码的执行效率。

### 封装

``` js
var person = function() {
    var name = "白糖";
    return {
        getName: function() {
            return name;
        },
        setName: function(newName) {
            name = newName;
        }
    }
}()
// 通过person.name是无法获取到name的值，如果要获取到name的值可以通过
console.log(person.getName()); //直接获取到 白糖
person.setName("栗子"); //重新设置新的名字
print(person.getName()); //获取 栗子
```

### 继承

``` js
function Person() {
    var name = "白糖";
    return {
        getName: function() {
            return name;
        },
        setName: function(newName) {
            name = newName;
        }
    }
}
var ChinesePeople = function() {};
ChinesePeople.prototype = new Person();
var cnp = new ChinesePeople();
p.setName("栗子");
consoel.log(p.getName());
```

## 5. 面试实战

### 题目1：

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

答案：

覆盖+闭包

``` js
function count() {
    var n = 0;
    // 名称必须是count，不能是其他
    count = function() {
        console.log(n++);
    }
    count()
}

count(); // 0
count(); // 1
count(); // 2
count(); // 3
count(); // 4
```

### 题目2：

``` js
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(new Date, i);
    }, 1000);
}

console.log(new Date, i);
```

答案：

``` js
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
Fri Mar 20 2020 13: 50: 25 GMT + 0800(中国标准时间) 5
```

> 如果我们约定，用箭头表示其前后的两次输出之间有 1 秒的时间间隔，而逗号表示其前后的两次输出之间的时间间隔可以忽略，代码实际运行的结果该如何描述？

``` 
5 -> 5,5,5,5,5
```

#### 追问 1：闭包

> 如果这道题仅仅是考察候选人对 JS 异步代码、变量作用域的理解，局限性未免太大，接下来我会追问，如果期望代码的输出变成：5 -> 0, 1, 2, 3, 4，该怎么改造代码？熟悉闭包的同学很快能给出下面的解决办法：

**解法1：**

巧妙的利用 IIFE（Immediately Invoked Function Expression：声明即执行的函数表达式）来解决闭包造成的问题

``` js
for (var i = 0; i < 5; i++) {
    (function(j) { // j = i
        setTimeout(function() {
            console.log(new Date, j);
        }, 1000);
    })(i);
}

console.log(new Date, i);
```

**解法2：**

对循环体稍做手脚，让负责输出的那段代码能拿到每次循环的 i 值即可。该怎么做呢？

利用 JS 中基本类型（Primitive Type）的参数传递是 `按值传递（Pass by Value）` 的特征，不难改造出下面的代码：

``` js
var output = function(i) {
    setTimeout(function() {
        console.log(new Date, i);
    }, 1000);
};

for (var i = 0; i < 5; i++) {
    output(i); // 这里传过去的 i 值被复制了
}

console.log(new Date, i);
```

#### 追问 2：ES6

> 如果期望代码的输出变成 0 -> 1 -> 2 -> 3 -> 4 -> 5，并且要求原有的代码块中的循环和两处 console.log 不变，该怎么改造代码？

新的需求可以精确的描述为：

> 代码执行时，立即输出 0，之后每隔 1 秒依次输出 1, 2, 3, 4，循环结束后在大概第 5 秒的时候输出 5（这里使用大概是因为 JS 中的定时器触发时机有可能是不确定的)

**解法1：**

``` js
for (var i = 0; i < 5; i++) {
    (function(j) {
            setTimeout(function() {
                console.log(new Date, j);
            }, 1000 * j)); // 这里修改 0~4 的定时器时间
    })(i);
}

setTimeout(function() { // 这里增加定时器，超时设置为 5 秒
    console.log(new Date, i);
}, 1000 * i);
```

做法虽粗暴有效，但是不算是能额外加分的方案。

> 可以把这次的需求抽象为：
>
> 在系列异步操作完成（每次循环都产生了 1 个异步操作）之后，再做其他的事情，代码该怎么组织？
>
> 聪明的你是不是想起了什么？对，就是 Promise。

``` js
const tasks = []; // 这里存放异步操作的 Promise
const output = (i) => new Promise((resolve) => {
    setTimeout(() => {
        console.log(new Date, i);
        resolve();
    }, 1000 * i);
});

// 生成全部的异步操作
for (var i = 0; i < 5; i++) {
    tasks.push(output(i));
}

// 异步操作完成之后，输出最后的 i
Promise.all(tasks).then(() => {
    setTimeout(() => {
        console.log(new Date, i);
    }, 1000);
});
```

我们都知道使用 Promise 处理异步代码比回调机制让代码可读性更高，但是使用 Promise 的问题也很明显，即如果没有处理 Promise 的 reject，会导致错误被丢进黑洞，好在新版的 Chrome 和 Node 7.x 能对未处理的异常给出 Unhandled Rejection Warning，而排查这些错误还需要一些特别的技巧（[浏览器](https://link.zhihu.com/?target=https%3A//www.bennadel.com/blog/3239-logging-and-debugging-unhandled-promise-rejections-in-the-browser.htm)、[Node.js](https://link.zhihu.com/?target=https%3A//www.bennadel.com/blog/3238-logging-and-debugging-unhandled-promise-rejections-in-node-js-v1-4-1-and-later.htm)）。

#### 追问 3：ES7

``` js
// 模拟其他语言中的 sleep，实际上可以是任何异步操作
const sleep = (timeountMS) => new Promise((resolve) => {
    setTimeout(resolve, timeountMS);
});

(async () => { // 声明即执行的 async 函数表达式
    for (var i = 0; i < 5; i++) {
        await sleep(1000);
        console.log(new Date, i);
    }

    await sleep(1000);
    console.log(new Date, i);
})();
```

## 6. 总结

闭包就是一个函数引用另外一个函数的变量，因为变量被引用着所以不会被回收，因此可以用来封装一个私有变量。这是优点也是缺点，不必要的闭包只会徒增内存消耗！

