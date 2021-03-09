# 005-JS类型转换

<motto></motto>

## 1. 其他转字符串String

> 下面是 null、undefined、布尔型、数字、数组、普通对象转换为字符串的规则。

* null：转为"null"
* undefined：转为"undefined"
* 布尔类型：true和false分别被转为"true"和"false"
* 数字类型：转为数字的字符串形式，如10转为"10"， 1e21转为"1e+21"
* 数组：转为字符串是将所有元素按照", "连接起来，相当于调用数组的Array.prototype.join()方法，如[1, 2, 3]转为"1, 2, 3"，空数组[]转为空字符串，数组中的null或undefined，会被当做空字符串处理
* 普通对象：转为字符串相当于直接使用Object.prototype.toString()，返回"[object Object]"

``` js
String(null) // 'null'
String(undefined) // 'undefined'
String(true) // 'true'
String(10) // '10'
String(1e15) // '1e+15'
String([1, 2, 3]) // '1,2,3'
String([]) // ''
String([null]) // ''
String([1, undefined, 5]) // '1,,5'
String({}) // '[object Object]'
```

## 2. 其他转数字Number

> 其他类型转换为数字类型的操作。

* null： 转为0
* undefined：转为NaN
* 字符串：如果是纯数字形式，则转为对应的数字，空字符转为0, 否则一律按转换失败处理，转为NaN
* 布尔型：true和false被转为1和0
* 数组：数组首先会被转为原始类型，也就是ToPrimitive，然后在根据转换后的原始类型按照上面的规则处理，关于ToPrimitive，会在下文中讲到
* 对象：同数组的处理

``` js
Number(null) // 0
Number(undefined) // NaN
Number('10') // 10
Number('10a') // NaN
Number('') // 0
Number(true) // 1
Number(false) // 0
Number([]) // 0
Number(['1']) // 1
Number({}) // NaN
```

## 3. 其他转布尔值Boolean

> 其他类型转换为布尔类型的操作。

js中的假值只有false、null、undefined、空字符、0和NaN，其它值转为布尔型都为true。

``` js
Boolean(null) // false
Boolean(undefined) // false
Boolean('') // flase
Boolean(NaN) // flase
Boolean(0) // flase
Boolean([]) // true
Boolean({}) // true
Boolean(Infinity) // true
```

## 4. 其他转原始类型Primitive

> 对象类型类型（如：对象、数组）转换为原始类型的操作。

* 当对象类型需要被转为原始类型时，它会先查找对象的 valueOf方法，如果valueOf方法返回原始类型的值，则转换的结果就是这个值
* 如果valueOf不存在或者valueOf方法返回的不是原始类型的值，就会尝试调用对象的toString方法，也就是会遵循对象的ToString规则，然后使用toString的返回值作为转换为原始类型的结果。

如果valueOf和toString都没有返回原始类型的值，则会抛出异常。

``` js
  Number([]) // 0
  Number(['10']) //10

  const obj1 = {
      valueOf() {
          return 100
      },
      toString() {
          return 101
      }
  }
  Number(obj1) // 100

  const obj2 = {
      toString() {
          return 102
      }
  }
  Number(obj2) // 102

  const obj3 = {
      toString() {
          return {}
      }
  }
  Number(obj3) // TypeError
```

前面说过，对象类型在ToNumber时会先ToPrimitive，再根据转换后的原始类型ToNumber

Number([])， 空数组会先调用valueOf，但返回的是数组本身，不是原始类型，所以会继续调用toString，得到空字符串，相当于Number('')，所以转换后的结果为"0"

同理，Number(['10'])相当于Number('10')，得到结果10

obj1的valueOf方法返回原始类型100，所以ToPrimitive的结果为100

obj2没有valueOf，但存在toString，并且返回一个原始类型，所以Number(obj2)结果为102

obj3的toString方法返回的不是一个原始类型，无法ToPrimitive，所以会抛出错误

## 5. 隐式转换（==惹的祸）

### 5.1 Boolean 类型参与

> 凡是布尔类型参与了==，那么Boolean类型会先转换为Number。
>
> true 转为 1，false 转为 0

### 5.2 Number类型和String类型比较

> String 类型会被转换为 Number 类型
>
> 如果是纯数字形式的字符串，则转为对应的数字，空字符转为 `0` , 否则一律按转换失败处理，转为 `NaN`

### 5.3 对象类型和原始类型比较

> 当 `对象类型` 和 `原始类型` 做相等比较时， `对象类型` 会依照 `转换原始类型` 规则转换为 `原始类型`

``` js
'[object Object]' == {} // true
'1,2,3' == [1, 2, 3] // true
[2] == 2 // true
[null] == 0 // true
[undefined] == 0 // true
[] == 0 // true
```

``` js
const a = {
    valueOf() {
        return 10
    }
    toString() {
        return 20
    }
}
a == 10 // true
const a = {
    // 定义一个属性来做累加
    i: 1,
    valueOf() {
        return this.i++
    }
}
a == 1 && a == 2 && a == 3 // true
```

### 5.4 null、undefined 参与的比较

``` js
null == false // false
undefined == false // false
```

`ECMAScript规范` 中规定 `null` 和 `undefined` 之间互相 `宽松相等（==）` ，并且也与其自身相等，但和其他所有的值都不 `宽松相等（==）` 。

## 练一练

``` js
[] == ![] // true

[] == 0 // true

[2] == 2 // true

['0'] == false // true

'0' == false // true

[] == false // true

[null] == 0 // true

null == 0 // false

[null] == false // true

null == false // false

[undefined] == false // true

undefined == false // false
```
