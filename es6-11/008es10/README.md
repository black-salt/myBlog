# 七、ES10新特性（2019）

<motto></motto>

* 行分隔符（U + 2028）和段分隔符（U + 2029）符号现在允许在字符串文字中，与JSON匹配
* 更加友好的 JSON.stringify
* 新增了Array的 `flat()` 方法和 `flatMap()` 方法
* 新增了String的 `trimStart()` 方法和 `trimEnd()` 方法
* `Object.fromEntries()` 
* `Symbol.prototype.description` 
* `String.prototype.matchAll` 
* `Function.prototype.toString()` 现在返回精确字符，包括空格和注释
* 简化 `try {} catch {}` , 修改 `catch` 绑定
* 新的基本数据类型 `BigInt` 
* globalThis
* import()
* Legacy RegEx
* 私有的实例方法和访问器

## 1. 行分隔符（U + 2028）和段分隔符（U + 2029）符号现在允许在字符串文字中，与JSON匹配

以前，这些符号在字符串文字中被视为行终止符，因此使用它们会导致SyntaxError异常。

## 2. 更加友好的 JSON.stringify

如果输入 Unicode 格式但是超出范围的字符，在原先JSON.stringify返回格式错误的Unicode字符串。现在实现了一个改变JSON.stringify的第3阶段提案，因此它为其输出转义序列，使其成为有效Unicode（并以UTF-8表示）

## 3. 新增了Array的 `flat()` 方法和 `flatMap()` 方法

`flat()` 和 `flatMap()` 本质上就是是归纳（reduce） 与 合并（concat）的操作。

### Array.prototype.flat()

`flat()` 方法会按照一个可指定的深度递归遍历数组，并将所有元素与遍历到的子数组中的元素合并为一个新数组返回。

``` js
var arr1 = [1, 2, [3, 4]];
arr1.flat();

var arr2 = [1, 2, [3, 4, [5, 6]]];
arr2.flat();

var arr3 = [1, 2, [3, 4, [5, 6]]];
arr3.flat(2);

//使用 Infinity 作为深度，展开任意深度的嵌套数组arr3.flat(Infinity); 
```

* 其次，还可以利用 `flat()` 方法的特性来去除数组的空项

``` js
var arr4 = [1, 2, , 4, 5];
arr4.flat();
```

### Array.prototype.flatMap()

`flatMap()` 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组。它与 map 和 深度值1的 flat 几乎相同，但 flatMap 通常在合并成一种方法的效率稍微高一些。 这里我们拿map方法与flatMap方法做一个比较。

``` js
var arr1 = [1, 2, 3, 4];
arr1.map(x => [x * 2]);

arr1.flatMap(x => [x * 2]);

// 只会将 flatMap 中的函数返回的数组 “压平” 一层arr1.flatMap(x => [[x * 2]]);
```

## 4. 新增了String的 `trimStart()` 方法和 `trimEnd()` 方法

新增的这两个方法很好理解，分别去除字符串首尾空白字符，这里就不用例子说声明了。

## 5. `Object.fromEntries()` 

`Object.entries()` 方法的作用是返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for... in 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环也枚举原型链中的属性）。

**而 `Object.fromEntries()` 则是 `Object.entries()` 的反转。**

`Object.fromEntries()` 函数传入一个键值对的列表，并返回一个带有这些键值对的新对象。这个迭代参数应该是一个能够实现@iterator方法的的对象，返回一个迭代器对象。它生成一个具有两个元素的类似数组的对象，第一个元素是将用作属性键的值，第二个元素是与该属性键关联的值。

* 通过 Object.fromEntries， 可以将 Map 转化为 Object:

``` js
const map = new Map([
    ['foo', 'bar'],
    ['baz', 42]
]);
const obj = Object.fromEntries(map);
console.log(obj);
```

* 通过 Object.fromEntries， 可以将 Array 转化为 Object:

``` js
const arr = [
    ['0', 'a'],
    ['1', 'b'],
    ['2', 'c']
];
const obj = Object.fromEntries(arr);
console.log(obj);
```

## 6. `Symbol.prototype.description` 

通过工厂函数Symbol（）创建符号时，您可以选择通过参数提供字符串作为描述：

``` js
const sym = Symbol('The description');
```

以前，访问描述的唯一方法是将符号转换为字符串：

``` js
assert.equal(String(sym), 'Symbol(The description)');
```

现在引入了getter Symbol.prototype.description以直接访问描述：

``` js
assert.equal(sym.description, 'The description');
```

## 7. `String.prototype.matchAll` 

`matchAll()` 方法返回一个包含所有匹配正则表达式及分组捕获结果的迭代器。 在 matchAll 出现之前，通过在循环中调用regexp.exec来获取所有匹配项信息（regexp需使用/g标志：

``` js
const regexp = RegExp('foo*', 'g');
const str = 'table football, foosball';
while ((matches = regexp.exec(str)) !== null) {
    console.log( `Found ${matches[0]}. Next starts at ${regexp.lastIndex}.` );
    // expected output: "Found foo. Next starts at 19."}
```

如果使用matchAll ，就可以不必使用while循环加exec方式（且正则表达式需使用／g标志）。使用matchAll 会得到一个迭代器的返回值，配合 for... of, array spread, or Array.from() 可以更方便实现功能：

``` js
const regexp = RegExp('foo*', 'g');
const str = 'table football, foosball';
let matches = str.matchAll(regexp);
for (const match of matches) {
    console.log(match);
}
// Array [ "foo" ]

// Call matchAll again to create a new iteratormatches = str.matchAll(regexp);
Array.from(matches, m => m[0]);
```

### matchAll可以更好的用于分组

``` js
var regexp = /t(e)(st(\d?))/g;
var str = 'test1test2';
str.match(regexp);
let array = [...str.matchAll(regexp)];
array[0];
array[1];
```

## 8. `Function.prototype.toString()` 现在返回精确字符，包括空格和注释

``` js
function /* comment */ foo /* another comment */() {}

console.log(foo.toString());
// ES2019 会把注释一同打印console.log(foo.toString()); 
// 箭头函数const bar  = /* another comment */ () => {};
console.log(bar.toString());
```

## 9. 修改 `catch` 绑定

在 ES10 之前，我们必须通过语法为 catch 子句绑定异常变量，无论是否有必要。很多时候 catch 块是多余的。ES10 提案使我们能够简单的把变量省略掉。

不算大的改动。

之前是

``` js
try {} catch (e) {}
```

现在是

``` js
try {} catch {}
```

## 10. 新的基本数据类型 `BigInt` 

现在的基本数据类型（值类型）不止5种（ES6之后是六种）了哦！加上BigInt一共有七种基本数据类型，分别是：String、Number、Boolean、Null、Undefined、Symbol、BigInt

ES6、ES7、ES8学习指南

