# vue文档查漏补缺

1.**Object.freeze()**。这会阻止修改现有的 property，也意味着响应系统无法再追踪变化。这里可以作为一个性能优化，比如长展示数据列表。

2.computed对比method，watch的优势在哪？

**computed vs method**

计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 依赖的值 还没有发生改变，多次访问 定义的计算属性，计算属性会立即返回之前的计算结果，而不必再次执行函数。

相比之下，每当触发重新渲染时，调用方法将总会再次执行函数。

我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 A，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 A。如果没有缓存，我们将不可避免的多次执行 A 的 getter！如果你不希望有缓存，请用方法来替代。

**computed vs watch**

个人认为computed优势在于可以依赖几个监听属性并返回一个最终值。
比如 我们要得到Fullname需要Firstname和Lastname，使用computed我们只需要定义一次，如果是watch，则需要分别watch Firstname和Lastname。

watch优势：一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时。使用 watch 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

3.**v-if vs v-show**.

v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。

>注意，v-show 不支持 `<template>` 元素，也不支持 v-else。

4. **v-for**.

**用 v-for 把一个数组对应为一组元素**

可以用 of 替代 in 作为分隔符，因为它更接近 JavaScript 迭代器的语法：
```
<div v-for="item of items"></div>
```

**可以用 v-for 来遍历一个对象的 property**
```
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>

new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})

结果：
How to do lists in Vue
Jane Doe
2016-04-10
```
也可以这样使用：
```
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>
```
>在遍历对象时，会按 Object.keys() 的结果遍历，但是不能保证它的结果在不同的 JavaScript 引擎下都一致。

当 Vue 正在更新使用 v-for 渲染的元素列表时，它默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。

>不要使用对象或数组之类的非基本类型值作为 v-for 的 key。请用字符串或数值类型的值。

Vue 将被侦听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。这些被包裹过的方法包括：
```
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

新数组替换旧数组时，可能认为这将导致 Vue 丢弃现有 DOM 并重新渲染整个列表。幸运的是，事实并非如此。Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。
>由于 JavaScript 的限制，Vue 不能检测数组和对象的变化。