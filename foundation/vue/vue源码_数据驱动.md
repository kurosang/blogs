# VUE源码-数据驱动

>Vue.js 一个核心思想是数据驱动。所谓数据驱动，是指视图是由数据驱动生成的，我们对视图的修改，不会直接操作 DOM，而是通过修改数据。它相比我们传统的前端开发，如使用 jQuery 等前端库直接修改 DOM，大大简化了代码量。特别是当交互复杂的时候，只关心数据的修改会让代码的逻辑变的非常清晰，因为 DOM 变成了数据的映射，我们所有的逻辑都是对数据的修改，而不用碰触 DOM，这样的代码非常利于维护。

# 模板和数据如何渲染成最终的 DOM？

我们从主线上把模板和数据如何渲染成最终的 DOM 的过程，我们可以通过下图更直观地看到从初始化 Vue 到最终渲染的整个过程。

![](../../imgs/vue1.png)

***

## init初始化

初始化主要就干了几件事情，合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等等。

在初始化的最后，检测到如果有 el 属性，则调用 `vm.$mount` 方法挂载 vm，挂载的目标就是把模板渲染成最终的 DOM

Vue 的初始化逻辑写的非常清楚，把不同的功能逻辑拆成一些单独的函数执行，让主线逻辑一目了然，这样的编程思想是非常值得借鉴和学习的。

## $mount 挂载

$mount 方法实际上会去调用 `mountComponent` 方法，`mountComponent` 核心就是先实例化一个渲染`Watcher`，在它的回调函数中会调用 `updateComponent` 方法，在此方法中调用 `vm._render` 方法先生成虚拟 Node，最终调用 `vm._update` 更新 DOM。

`Watcher` 在这里起到两个作用，一个是初始化的时候会执行回调函数，另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数。

完成整个渲染工作，最核心的 2 个方法：`vm._render` 和 `vm._update`。

**额外工作：compile**

编译，调用compiler 版本的 $mount方法中的compileToFunctions把 el 或者 template 字符串转换成 render 方法。如果有定义render则没有此步骤。

## vm._render

通过执行 createElement 方法并返回的是 vnode，它是一个虚拟 Node。

**Virtual DOM**，其实 VNode 是对真实 DOM 的一种抽象描述，它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等。Virtual DOM 除了它的数据结构的定义，映射到真实的 DOM 实际上要经历 VNode 的 create、diff、patch 等过程。

`createElement`，Vue.js 利用 `createElement` 方法创建 VNode。`createElement` 方法实际上是对 `_createElement` 方法的封装，它允许传入的参数更加灵活，在处理这些参数后，调用真正创建 VNode 的函数 `_createElement`。

createElement 函数的流程略微有点多，我们接下来主要分析 2 个重点的流程 —— **children 的规范化**以及 **VNode 的创建**。

**children 的规范化**
* 由于 Virtual DOM 实际上是一个树状结构，每一个 VNode 可能会有若干个子节点，这些子节点应该也是 VNode 的类型。
* `_createElement` 接收的第 4 个参数 children 是任意类型的，因此我们需要把它们规范成 VNode 类型。

**VNode 的创建**
* 回到`createElement` 函数，规范化 children 后，接下来会去创建一个 VNode 的实例：
* 这里先对 `tag` 做判断，如果是 `string` 类型，则接着判断如果是内置的一些节点，则直接创建一个普通 VNode，如果是为已注册的组件名，则通过 `createComponent` 创建一个组件类型的 VNode，否则创建一个未知的标签的 VNode。 如果`tag`是一个 `Component` 类型，则直接调用 `createComponent` 创建一个组件类型的 VNode 节点。createComponent本质上它还是返回了一个 VNode。

那么至此，我们大致了解了 createElement 创建 VNode 的过程，每个 VNode 有 children，children 每个元素也是一个 VNode，这样就形成了一个 VNode Tree，它很好的描述了我们的 DOM Tree。

回到 mountComponent 函数的过程，我们已经知道 vm._render 是如何创建了一个 VNode，接下来就是要把这个 VNode 渲染成一个真实的 DOM 并渲染出来，这个过程是通过 vm._update 完成的，

## vm._update

Vue 的 _update 是实例的一个私有方法，它被调用的时机有 2 个，一个是首次渲染，一个是数据更新的时候；由于我们这只分析首次渲染部分，数据更新部分会在之后分析响应式原理的时候涉及。

`vm._update` 方法的作用是把 VNode 渲染成真实的 DOM。核心就是调用`vm.__patch__`方法（这个方法实际上在不同的平台，比如 web 和 weex 上的定义是不一样的），根据vnode递归创建了一个完整的 DOM 树并插入到 Body 上。因为是递归调用，子元素会优先调用 insert，所以整个 vnode 树节点的插入顺序是先子后父。
