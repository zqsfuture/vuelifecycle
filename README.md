# Vue学习之路

## 安装
- Vue **不支持** IE8 及以下版本，因为 Vue 使用了 IE8 无法模拟的 ECMAScript 5 特性。但它支持所有兼容 ECMAScript 5 的浏览器。
- **Vue项目如何做兼容IE8以下的兼容处理？**

## 介绍
- Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以**自底向上逐层应用**。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

- **如何理解渐进式和自底向上逐层应用？**

- 所有东西都是响应式的。**如何理解响应式？响应式的原理？**

- 在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。

## Vue实例

- 当一个 Vue 实例被创建时，它向 Vue 的响应式系统中加入了其 data 对象中能找到的所有的属性。当这些属性的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。
- 当这些数据改变时，视图会进行重渲染。值得注意的是只有当实例被创建时 data 中存在的属性才是响应式的。也就是说如果你添加一个新的属性，比如：

``` javascript
vm.b = 'hi'
```

- 那么对 b 的改动将不会触发任何视图的更新。如果你知道你会在晚些时候需要一个属性，但是一开始它为空或不存在，那么你仅需要设置一些初始值。

- 每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做生命周期钩子的函数，这给了用户在不同阶段添加自己的代码的机会。

### 生命周期图示

<p align="center">
  <img width="320" src="https://cn.vuejs.org/images/lifecycle.png">
</p>

- 在初始化时，会调用以下代码，生命周期就是通过 callHook 调用的。

``` javascript
Vue.prototype._init = function(options) {
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm, 'beforeCreate') // 拿不到 props data
  initInjections(vm)
  initState(vm)
  initProvide(vm)
  callHook(vm, 'created')
}
```

- 可以发现在以上代码中，beforeCreate 调用的时候，是获取不到 props 或者 data 中的数据的，因为这些数据的初始化都在 initState 中。

## 计算属性和侦听器

### 计算属性

- 对于任何复杂逻辑，你都应当使用计算属性。

- 我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是计算属性是基于它们的依赖进行缓存的。只在相关依赖发生改变时它们才会重新求值。这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。

- 这也同样意味着下面的计算属性将不再更新，因为 Date.now() 不是响应式依赖：

``` javascript
computed: {
  now: function () {
    return Date.now()
  }
}
```

- 相比之下，每当触发重新渲染时，调用方法将总会再次执行函数。

- 我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 A，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 A 。如果没有缓存，我们将不可避免的多次执行 A 的 getter！如果你不希望有缓存，请用方法来替代。

### 侦听器

- 虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。这就是为什么 Vue 通过 watch 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

## 条件渲染

### v-if vs v-show

- v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

- v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

- 相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

- 一般来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。

##列表渲染

### 注意事项
- 由于 JavaScript 的限制，Vue 不能检测以下变动的**数组**：

- 当你利用索引直接设置一个项时，例如：vm.items[indexOfItem] = newValue

- 当你修改数组的长度时，例如：vm.items.length = newLength

- 举个例子：
``` javascript
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})

vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

- 为了解决第一类问题，以下两种方式都可以实现和 vm.items[indexOfItem] = newValue 相同的效果，同时也将触发状态更新：

``` javascript
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

- 你也可以使用 vm.$set 实例方法，该方法是全局方法 Vue.set 的一个别名：

``` javascript
vm.$set(vm.items, indexOfItem, newValue)
```

- 为了解决第二类问题，你可以使用 splice：
``` javascript
vm.items.splice(newLength)
```

### 对象更改检测注意事项

还是由于 JavaScript 的限制，Vue 不能检测对象属性的添加或删除：

``` javascript
var vm = new Vue({
  data: {
    a: 1
  }
})

// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```

- 对于已经创建的实例，Vue 不能动态添加根级别的响应式属性。但是，可以使用 Vue.set(object, key, value) 方法向嵌套对象添加响应式属性。例如，对于：

``` javascript
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
```
- 你可以添加一个新的 age 属性到嵌套的 userProfile 对象：

``` javascript
Vue.set(vm.userProfile, 'age', 27)
```
- 你还可以使用 vm.$set 实例方法，它只是全局 Vue.set 的别名：
``` javascript
vm.$set(vm.userProfile, 'age', 27)
```

- 有时你可能需要为已有对象赋予多个新属性，比如使用 Object.assign() 或 _.extend()。在这种情况下，你应该用两个对象的属性创建一个新的对象。所以，如果你想添加新的响应式属性，不要像这样：

``` javascript
Object.assign(vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

- 你应该这样做：

``` javascript
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

### v-for with v-if

- 当它们处于同一节点，v-for 的优先级比 v-if 更高，这意味着 v-if 将分别重复运行于每个 v-for 循环中。当你想为仅有的一些项渲染节点时，这种优先级的机制会十分有用，如下：

``` javascript
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```
- 上面的代码只传递了未完成的 todos。

- 而如果你的目的是有条件地跳过循环的执行，那么可以将 v-if 置于外层元素 (或 <template>)上。如：

``` javascript
<ul v-if="todos.length">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
<p v-else>No todos left!</p>
```

