# VUE源码-组件化

>Vue.js 另一个核心思想是组件化。所谓组件化，就是把页面拆分成多个组件 (component)，每个组件依赖的 CSS、JavaScript、模板、图片等资源放在一起开发和维护。组件是资源独立的，组件在系统内部可复用，组件和组件之间可以嵌套。我们在用 Vue.js 开发实际项目的时候，就是像搭积木一样，编写一堆组件拼装生成页面。

# 组件VNode 是如何创建、初始化、渲染?

## render

```
import Vue from 'vue'
import App from './App.vue'

var app = new Vue({
  el: '#app',
  // 这里的 h 是 createElement 方法
  render: h => h(App)
})
```
这段代码相信很多同学都很熟悉，它和我们上一章相同的点也是通过 render 函数去渲染的，不同的这次通过 createElement 传的参数是一个组件而不是一个原生的标签，那么接下来我们就开始分析这一过程。

## createComponent
上一章我们在分析 `createElement` 的实现的时候，它最终会调用 `_createElement` 方法，其中有一段逻辑是对参数 `tag` 的判断，如果是一个普通的 html 标签，像上一章的例子那样是一个普通的 div，则会实例化一个普通 VNode 节点，否则通过 `createComponent` 方法创建一个组件 VNode。

createComponent渲染组件的3个关键步骤：
* 构造子类构造函数
* 安装组件钩子函数
* 实例化 vnode。

## 构造子类构造函数

createComponent 里的代码逻辑会执行到 baseCtor.extend(Ctor)，在这里 baseCtor 实际上就是 Vue，在了解了 baseCtor 指向了 Vue 之后，相当于执行了 Vue.extend 函数。

Vue.extend 的作用就是构造一个 Vue 的子类，它使用一种非常经典的原型继承的方式把一个纯对象转换一个继承于 Vue 的构造器 Sub 并返回，然后对 Sub 这个对象本身扩展了一些属性，如扩展 options、添加全局 API 等；并且对配置中的 props 和 computed 做了初始化工作；最后对于这个 Sub 构造函数做了缓存，避免多次执行 Vue.extend 的时候对同一个子组件重复构造。

这样当我们去实例化 Sub 的时候，就会执行 this._init 逻辑再次走到了 Vue 实例的初始化逻辑，实例化子组件的逻辑在之后的章节会介绍。

## 安装组件钩子函数
```
// install component management hooks onto the placeholder node
installComponentHooks(data)
```
Virtual DOM的一个特点是在 VNode 的 patch 流程中对外暴露了各种时机的钩子函数，方便我们做一些额外的事情，Vue.js 也是充分利用这一点，在初始化一个 Component 类型的 VNode 的过程中实现了几个钩子函数。

整个 `installComponentHooks` 的过程就是把 `componentVNodeHooks` 的钩子函数合并到 `data.hook` 中，在 VNode 执行 `patch` 的过程中执行相关的钩子函数，具体的执行我们稍后在介绍 patch 过程中会详细介绍。这里要注意的是合并策略，在合并过程中，如果某个时机的钩子已经存在 `data.hook` 中，那么通过执行 `mergeHook` 函数做合并，这个逻辑很简单，就是在最终执行的时候，依次执行这两个钩子函数即可。

## 实例化 VNode
```
const name = Ctor.options.name || tag
const vnode = new VNode(
  `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
  data, undefined, undefined, undefined, context,
  { Ctor, propsData, listeners, tag, children },
  asyncFactory
)
return vnode
```
最后一步非常简单，通过 new VNode 实例化一个 vnode 并返回。需要注意的是和普通元素节点的 vnode 不同，组件的 vnode 是没有 children 的，这点很关键，在之后的 patch 过程中我们会再提。

## render总结
这一节我们分析了 createComponent 的实现，了解到它在渲染一个组件的时候的 3 个关键逻辑：构造子类构造函数，安装组件钩子函数和实例化 vnode。createComponent 后返回的是组件 vnode，它也一样走到 vm._update 方法，进而执行了 patch 函数，

## update

当我们通过 `createComponent` 创建了组件 VNode，接下来会走到 `vm._update`，执行 ``vm.__patch__`` 去把 VNode 转换成真正的 DOM 节点。

# 合并配置

通过之前章节的源码分析我们知道，new Vue 的过程通常有 2 种场景，一种是外部我们的代码主动调用 new Vue(options) 的方式实例化一个 Vue 对象；另一种是我们上一节分析的组件过程中内部通过 new Vue(options) 实例化子组件。

无论哪种场景，都会执行实例的 _init(options) 方法，它首先会执行一个 merge options 的逻辑。