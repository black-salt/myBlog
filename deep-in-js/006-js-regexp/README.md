# 006-总是忘记的正则表达式

## 一、创建正则表达式

JS中有**字面量**与**对象**这两种方式创建正则表达式

### 1. 字面量创建

使用`//`包裹是字面量创建正则的方式之一，但不能在`//`中使用变量

```js
let foo = "https://foobar.com";
console.log(/^https?\/\/$/.test(foo)); //true
```

使用 `pattern` 变量时将不可以查询

```js
let foo = "https://foobar.com";
let pattern = "http";
console.log(/pattern/.test(foo)); //false
```

虽然也可以使用 `eval` 转换为JS语法来实现将变量解析到正则中，但是显然不太优雅.

```js
let foo = "https://foobar.com";
let pattern = "http";
console.log(eval(`/${pattern}/`).test(foo)); //true
```

所以，当需要动态的创建正则表达式时，可以下面使用`对象创建`的方式。

### 2. 对象创建

当需要动态创建正则表达式时，可以使用对象创建

```js
let foo = "https://foobar.com";
let host = "foobar";
let reg = new RegExp(host);
console.log(reg.test(foo)); //true
```

## 二、基础知识

### 边界符

使用字符边界符用来控制匹配内容的起始与结束。

| 边界符 | 说明                         |
| ------ | ---------------------------- |
| ^      | 匹配字符串的开始             |
| $      | 匹配字符串的结束，忽略换行符 |

### 转义符

由于正则表达式中会有一些特殊的字母或者符号来表达一些正则表达式的规则，所以当我们就是需要匹配那些特殊的字母或符号时，就不能够随意地使用这些字母或符号，这时我们就需要转义字符`\`来将这个要匹配的但在正则中有特殊的含义的字符转移成普通的字符就好啦。

其实在最开始我就已经暗戳戳地使用过转义字符啦。

下面假如我们想通过正则查找`/`符号，但是 `/`在正则中有特殊的意义。如果写成`///`这会造成解析错误，所以要使用转义语法 `/\//`来匹配`/`。

```js
const url = "https://foobar.com";
console.log(/^https:\/\//.test(url)); //true
```

## 三、元字符

元字符是正则表达式中的最小元素，只代表单一（一个）字符

### 字符列表

| 元字符 | 说明                                                 | 示例          |
| ------ | ---------------------------------------------------- | ------------- |
| \d     | 匹配任意一个数字                                     | [0-9]         |
| \D     | 匹配除了数字以外的任何一个字符                       | [^0-9]        |
| \w     | 匹配任意一个英文字母,数字或下划线                    | [a-zA-Z_]     |
| \W     | 匹配除了字母,数字或下划线外与任何字符                | [^a-zA-Z_]    |
| \s     | 匹配任意一个空白字符，如空格，制表符`\t`，换行符`\n` | [\n\f\r\t\v]  |
| \S     | 匹配除了空白符外任意一个字符                         | [^\n\f\r\t\v] |
| .      | 匹配除换行符外的任意字符                             |               |

### 所有字符

可以使用 `[\s\S]` 或 `[\d\D]` 来匹配所有字符

## 四、匹配模式

正则表达式在执行时会按默认执行方式进行，但有时候默认的处理方式总不能满足我们的需求，所以可以使用模式修正符更改默认方式。

### 字符列表

| 修饰符 | 说明                                         |
| ------ | -------------------------------------------- |
| i      | 不区分大小写字母的匹配                       |
| g      | 全局搜索所有匹配内容                         |
| m      | 视为多行                                     |
| s      | 视为单行忽略换行符，使用`.` 可以匹配所有字符 |
| y      | 从 `regexp.lastIndex` 开始匹配               |
| u      | 正确处理四个字符的 UTF-16 编码               |

### i

将所有字母统一为小写。

### g

使用 `g` 修饰符可以全局操作内容。常与`replace`方法搭配。

### m

用于将内容视为多行匹配，主要是对 `^`和 `$` 的修饰。

### u

每个字符都有属性，如`L`属性表示是字母，`P` 表示标点符号，需要结合 `u` 模式才有效。其他属性简写可以访问 [属性的别名](https://www.unicode.org/Public/UCD/latest/ucd/PropertyValueAliases.txt) 网站查看。

```js
// 可使用\p{L}属性匹配字母
// 可使用\p{P}属性匹配标点
```

字符也有 unicode 文字系统属性 `Script=文字系统`，下面是使用 `\p{sc=Han}` 获取中文字符 `han`为中文系统，其他语言请查看 [文字语言表](http://www.unicode.org/standard/supported.html)

```js
let foo = `
张三:010-99999999,李四:020-88888888`;
let res = foo.match(/\p{sc=Han}+/gu);
console.log(res);
```

使用 `u` 模式可以正确处理四个字符的 UTF-16 字节编码

```js
let str = "𝒳𝒴";
console.table(str.match(/[𝒳𝒴]/)); //结果为乱字符"�"

console.table(str.match(/[𝒳𝒴]/u)); //结果正确 "𝒳"
```

### lastIndex

RegExp对象`lastIndex` 属性可以返回或者设置正则表达式开始匹配的位置

- 必须结合 `g` 修饰符使用
- 对 `exec` 方法有效
- 匹配完成时，`lastIndex` 会被重置为0

### y

使用 `y` 模式可以在匹配不到时停止匹配。

## 五、原子表

在一组字符中匹配某个元字符，在正则表达式中通过元字符表来完成，就是放到`[]` (方括号)中。

### 使用语法

| 原子表 | 说明                                                        |
| ------ | ----------------------------------------------------------- |
| []     | 只匹配其中的一个原子                                        |
| [^]    | 只匹配"除了"其中字符的任意一个原子，^在原子表中代表非的意思 |
| [0-9]  | 匹配0-9任何一个数字                                         |
| [a-z]  | 匹配小写a-z任何一个字母                                     |
| [A-Z]  | 匹配大写A-Z任何一个字母                                     |

## 六、原子组

- 如果一次要匹配多个元子，可以通过元子组完成
- 原子组与原子表的差别在于原子组一次匹配多个元子，而原子表则是匹配其中任意一个字符
- 元字符组用 `()` 包裹

下面使用原子组匹配 `h1` 标签，如果想匹配 `h2` 只需要把前面原子组改为 `h2` 即可。

```js
const foo = `<h1>foobar.com</h1>`;
console.log(/<(h1)>.+<\/\1>/.test(foo)); //true
```

### 基本使用

没有添加 `g` 模式修正符时只匹配到第一个，匹配到的信息包含以下数据

| 变量    | 说明             |
| ------- | ---------------- |
| 0       | 匹配到的完整内容 |
| 1,2.... | 匹配到的原子组   |
| index   | 原字符串中的位置 |
| input   | 原字符串         |
| groups  | 命名分组         |

在`match`中使用原子组匹配，会将每个组数据返回到结果中

- 0 为匹配到的完成内容
- 1/2 等 为原子组内容
- index 匹配的开始位置
- input 原始数据
- groups 组别名

原子组主要搭配`match`方法使用。

## 七、匹配方法傻傻搞不清

### test

调用者：**正则表达式的字面量** 或者 **RegExp对象**

参数：**字符串**

返回值：**Boolean类型**

栗子：

```js
let foo = "foo"
// 使用 字面量 的形式
console.log(/o/.test(foo)) // true,因为有o
console.log(/#/.test(foo)) // false,因为没有#

// 使用 字面量+变量 的形式
let pattern = "o"
console.log(eval(`/${a}/`).test(foo)) // true,因为有o

// 使用 对象创建 的形式
let reg = new RegExp("o","g") // 第二个参数g是全局模式的意思
console.log(eval(`/${a}/`).test(foo)) // true,因为有o

// 使用 对象创建+变量 的形式
let pattern = "o"
let reg = new RegExp(pattern,"g") // 第二个参数g是全局模式的意思
console.log(reg.test(foo)) // true,因为有o
```

### exec

调用者：**正则表达式的字面量** 或者 **RegExp对象**

参数：**字符串**

返回值：**数组(需注意其属性)**

栗子：

```js
let foo = "foo1998bar"
let reg = /\w/g

while((res = reg.exec(foo))) { // 利用了 lastIndex 属性
      console.log(res)
}
```

<img src="https://lskreno-typora.oss-cn-beijing.aliyuncs.com/img/Snipaste_2020-08-15_17-12-38.png" alt="Snipaste_2020-08-15_17-12-38" style="zoom:50%;" />



### match

调用者：**字符串** 

参数：**正则表达式的字面量** 或者 **RegExp对象**

返回值：**数组(需注意其属性，但在全局模式g下，没有多余属性，只是简单的数组)**

栗子：

```js
let foo = "foo1998bar"
let reg1 = /\w/
let reg2 = /\w/g

// 非全局模式下
console.log(foo.match(reg1)) // ["f", index: 0, input: "foo1998bar", groups: undefined]

// 全局模式g下
console.log(foo.match(reg2)) // ["f", "o", "o", "1", "9", "9", "8", "b", "a", "r"]
```



### replace

调用者：**字符串** 

参数：

1. **正则表达式的字面量** 或者 **RegExp对象**，或者，**字符串**
2. **要替换成的字符串（注意需要使用原子组的数据时可以使用 $ 进行获取）**

返回值：**替换完的新字符串**

栗子：

```js
let str=`
	<h1>foo</h1>
		<span>foo1998bar</span>
	<h2>bar</h2>
`
let reg=/<(h[1-6])>([\s\S]+)<\/\1>/gi; // 这里的\1是获取第一个元组的值
console.log(str.replace(reg,"<p>$2</p>"));
```

## 八、常用正则表达式

### 8.1 密码强度正则

```js
//密码强度正则，最少6位，包括至少1个大写字母，1个小写字母，1个数字，1个特殊字符
var pPattern = /^.*(?=.{6,})(?=.*\d)(?=.*[A-Z])(?=.*[a-z])(?=.*[!@#$%^&*? ]).*$/;
//输出 true
console.log("=="+pPattern.test("ShenMeGui*/0"));
```

### 8.2 整数正则

```js
//正整数正则
var posPattern = /^\d+$/;
//负整数正则
var negPattern = /^-\d+$/;
//整数正则
var intPattern = /^-?\d+$/;
//输出 true
console.log(posPattern.test("408"));
//输出 true
console.log(negPattern.test("-408"));
//输出 true
console.log(intPattern.test("-408"));
```

### 8.3 数字正则

可以是整数也可以是浮点数

```js
//正数正则
var posPattern = /^\d*\.?\d+$/;
//负数正则
var negPattern = /^-\d*\.?\d+$/;
//数字正则
var numPattern = /^-?\d*\.?\d+$/;
console.log(posPattern.test("408.8"));
console.log(negPattern.test("-408.8"));
console.log(numPattern.test("-408.8"));
```

### 8.4 Email正则

```js
//Email正则
var ePattern = /^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/;
//输出 true
console.log(ePattern.test("85785710@qq.com"));
```

### 8.5 手机号码正则

```js
//手机号正则
var mPattern = /^1[34578]\d{9}$/;
//输出 true
console.log(mPattern.test("15836002888"));
```