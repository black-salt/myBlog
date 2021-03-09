# 006 - Reflect Metadata 简单入门

<motto></motto>

## 四大概念

1. `Metadata Key` {Any} 后文简写 `k`。元数据的 Key，对于一个对象来说，他可以有很多元数据，每一个元数据都对应有一个 Key。一个很简单的例子就是说，你可以在一个对象上面设置一个叫做 `'name'` 的 Key 用来设置他的名字，用一个 `'created time'` 的 Key 来表示他创建的时间。这个 Key 可以是任意类型。在后面会讲到内部本质就是一个 `Map` 对象。
2. `Metadata Value` {Any} 后文简写 `v`。元数据的类型，任意类型都行。
3. `Target` {Object} 后文简写 `o`。表示要在这个对象上面添加元数据
4. `Property` {String|Symbol} 后文简写 `p`。用于设置在那个属性上面添加元数据。大家可能会想，这个是干什么用的，不是可以在对象上面添加元数据了么？其实不仅仅可以在对象上面添加元数据，甚至还可以在对象的属性上面添加元数据。其实大家可以这样理解，当你给一个对象定义元数据的时候，相当于你是默认指定了 `undefined` 作为 Property。

## 常用API

```typescript
namespace Reflect {
  // 用于装饰器
  metadata(k, v): (target, property?) => void
  
  // 在对象上面定义元数据
  defineMetadata(k, v, o, p?): void
  
  // 是否存在元数据
  hasMetadata(k, o, p?): boolean
  hasOwnMetadata(k, o, p?): boolean
  
  // 获取元数据
  getMetadata(k, o, p?): any
  getOwnMetadata(k, o, p?): any
  
  // 获取所有元数据的 Key
  getMetadataKeys(o, p?): any[]
  getOwnMetadataKeys(o, p?): any[]
  
  // 删除元数据
  deleteMetadata(k, o, p?): boolean
}
```



## 深入 Reflect Metadata

### 实现原理

如果你去翻看官网的文档，他会和你说，所有的元数据都是存在于对象下面的 `[[Metadata]]` 属性下面。一开始我也是这样认为的，新建一个 `Symbol('Metadata')`，然后将元数据放到这个 Symbol 对应的 Property 当中。直到我看了源码才发现并不是这样。请看例子

```typescript
@Reflect.metadata('name', 'A')
class A {}

Object.getOwnPropertySymbols(A) // []
```

哈哈，并没有所谓的 Symbol，那么这些元数据都存在在哪里呢？

其实是内部的一个 WeakMap 中。他正是利用了 WeakMap 不增加引用计数的特点，将对象作为 Key，元数据集合作为 Value，存到 WeakMap 中去。

如果你认真探寻的话，你会发现其内部的数据结构其实是这样的

```typescript
WeakMap<any, Map<any, Map<any, any>>>
```

是不是超级绕，但是我们从调用的角度来思考，这就一点都不绕了。

```typescript
weakMap.get(o).get(p).get(k)
```

先根据对象获取，然后在根据属性，最后根据元数据的 Key 获取最终要的数据。

## THANK

[JavaScript Reflect Metadata 详解](https://www.jianshu.com/p/653bce04db0b)