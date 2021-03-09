# 001 - Vue 开发实践经验技巧

### 1. 始终在 `v-for` 中使用 `:key` 

在需要操纵数据时，将 `key` 属性与 `v-for` 指令一起使用可以让程序保持恒定且可预测。

这是很有必要的，这样Vue就可以跟踪组件状态，并对不同的元素有一个常量引用。在使用动画或Vue转换时，key 非常有用。

如果没有 `key` ，Vue只会尝试使DOM尽可能高效。这可能意味着 `v-for` 中的元素可能会出现乱序，或者它们的行为难以预测。如果我们对每个元素都有唯一的键引用，那么我们可以更好地预测 `Vue` 应用程序将如何精确地处理 `DOM` 操作。

```html
<!-- 不好的做法-->
<div v-for='product in products'>  </div>

<!-- 好的做法 -->
<div v-for='product in products' :key='product.id'>
```

### 在事件中使用短横线命名

在发出定制事件时，最好使用短横线命名，这是因为在父组件中，我们使用相同的语法来侦听该事件。

因此，为了确保我们各组件之间的一致性，并使您的代码更具可读性，请在两个地方都坚持使用短横线命名。

``` js
this.$emit('close-window')
// 在父组件中
<popup-window @close-window='handleEvent()' />
```

### 3. 使用驼峰式声明 props，并在模板中使用短横线命名来访问 props

最佳做法只是遵循每种语言的约定。在 JS 中，驼峰式声明是标准，在HTML中，是短横线命名。因此，我们相应地使用它们。

幸运的是，Vue 已经提供了驼峰式声明和短横线命名之间转换，因此除了实际声明它们之外，我们不必担心任何事情。

``` html
不好的做法
<PopupWindow titleText='hello world' /> 
props: { 'title-text': String }

好的做法
<PopupWindow title-text='hello world' /> 
props: { titleText: String }
```

### 4.data 应始终返回一个函数

声明组件 `data` 时， `data` 选项应始终返回一个函数。如果返回的是一个对象，那么该 `data` 将在组件的所有实例之间共享。

``` js
// 不好的做法
data: {
  name: 'My Window',
  articles: []
}
```

但是，大多数情况下，我们的目标是构建可重用的组件，因此我们希望每个组件返回一个惟一的对象。我们通过在函数中返回数据对象来实现这一点。

``` js
// 好的做法
data () {
  return {
    name: 'My Window',
    articles: []
  }
}
```

### 5. 不要在同个元素上同时使用 `v-if` 和 `v-for` 指令

为了过滤数组中的元素，我们很容易将 `v-if` 与 `v-for` 在同个元素同时使用。

``` js
// 不好的做法
<div v-for='product in products' v-if='product.price < 500'>
```

问题是在 Vue 优先使用 `v-for` 指令，而不是 `v-if` 指令。它循环遍历每个元素，然后检查 `v-if` 条件。

``` js
this.products.map(function (product) {
  if (product.price < 500) {
    return product
  }
})
```

这意味着，即使我们只想渲染列表中的几个元素，也必须遍历整个数组。

这对我们来当然没有任何好处。

一个更聪明的解决方案是遍历一个计算属性，可以把上面的例子重构成下面这样的：

``` js
<div v-for='product in cheapProducts'>

computed: {
  cheapProducts: () => {
    return this.products.filter(function (product) {
      return product.price < 100
    })
  }
}
```

这么做有几个好处：

* 渲染效率更高，因为我们不会遍历所有元素
* 仅当依赖项更改时，才会重使用过滤后的列表
* 这写法有助于将组件逻辑从模板中分离出来，使组件更具可读性

### 6. 用正确的定义验证我们的 props

可以说这条是很重要，为什么？

在设计大型项目时，很容易忘记用于props的确切格式、类型和其他约定。如果你在一个更大的开发团队中，你的同事不会读心术，所以你要清楚地告诉他们如何使用你的组件。

因此，我们只需编写props验证即可，不必费力地跟踪组件来确定props的格式

从Vue文档中查看此示例。

``` js
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```

### 7. 组件全名使用驼峰或或者短横线

组件的通用命名约定是使用驼峰或短横线。无论我们使用哪个，最重要的是始终保持一致。我认为驼峰方式 效果最好，因为大多数IDE自动完成功能都支持它。

``` js
# 不好的做法

mycomponent.vue
myComponent.vue
Mycomponent.vue

# 好的做法

MyComponent.vue
```

### 8. 基本组件应该相应地加上前缀

根据Vue样式指南，基本组件是仅包含以下内容的组件：

* HTML 元素
* 额外的基础组件
* 第三方的UI组件

为这些组件命名的最佳实践是为它们提供前缀 `Base` 、 `V` 或 `App` 。同样，只要我们在整个项目中保持一致，可以使用其中任何一种。

``` js
BaseButton.vue
BaseIcon.vue
BaseHeading.vue
```

该命名约定的目的是使基本组件按字母顺序分组在文件系统中。另外，通过使用webpack导入功能，我们可以搜索与命名约定模式匹配的组件，并将所有组件自动导入为Vue项目中的全局变量。

### 9. 单实例组件命名应该带有前缀 `The` 

与基本组件类似，单实例组件(每个页面使用一次，不接受任何prop)应该有自己的命名约定。这些组件特定于我们的应用，通常是 `footer` ， `header` 或 `sider` 。

该组件只能有一个激活实例。

``` js
TheHeader.vue
TheFooter.vue
TheSidebar.vue
ThePopup.vue
```

### 10. 保持指令简写的一致性

在Vue开发人员中，一种常见的技术是使用指令的简写。例如：

* `@` 是 `v-on` 的简写
* `:` 是 `v-bind` 的简写
* `#` 是 `v-slot` 的简写

在你的Vue项目中使用这些缩写是很好的。但是要在整个项目中创建某种约定，总是使用它们或从不使用它们, 会使我们的项目更具内聚性和可读性。

### 11. 不要在“created”和“watch”中调用方法

Vue开发人员经常犯的一个错误是他们不必要地在 `created` 和 `watch` 中调用方法。其背后的想法是，我们希望在组件初始化后立即运行 `watch` 。

``` js
不好的做法
created: () {
  this.handleChange()
},
methods: {
  handleChange() {
    // stuff happens
  }
},
watch () {
  property() {
    this.handleChange()
  }
}
```

但是，Vue为此提供了内置的解决方案，这是我们经常忘记的Vue `watch` 属性。

我们要做的就是稍微重组 `watch` 并声明两个属性：

1. `handler (newVal, oldVal)-` 这是我们的watch方法本身。

1. `immediate: true` - 代表如果在 wacth 里声明了之后，就会立即先去执行里面的 `handler` 方法，如果为 `false` 就跟我们以前的效果一样，不会在绑定的时候就执行

``` js
    好的做法
    methods: {
      handleChange() {
        // stuff happens
      }
    },
    watch () {
      property {
        immediate: true
        handler() {
          this.handleChange()
        }
      }
    }
```

### 12. 模板表达式应该只有基本的 JS 表达式

在模板中添加尽可能多的内联功能是很自然的。但是这使得我们的模板不那么具有声明性，而且更加复杂，也让模板会变得非常混乱。

为此，让我们看看Vue样式指南中另一个规范化字符串的示例，看看它有多混乱。

``` js
不好的做法
{{
  fullName.split(' ').map(function (word) {
    return word[0].toUpperCase() + word.slice(1)
  }).join(' ')
}}
```

基本上，我们希望模板中的所有内容都直观明了。为了保持这一点，我们应该将复杂的表达式重构为适当命名的组件选项。

分离复杂表达式的另一个好处是可以重用这些值。

``` js
/、 好的做法
{{ normalizedFullName }}

// The complex expression has been moved to a computed property
computed: {
  normalizedFullName: function () {
    return this.fullName.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ')
  }
}
```

### 总结

这是12个最常见的最佳实践，它们将使我们的Vue代码更易于维护、可读性更好、更专业。希望这些技巧对您有用（因为它们绝对是我一直想记住的东西）。
