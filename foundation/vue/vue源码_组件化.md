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

对于 options 的合并有 2 种方式。无论哪种场景，都会执行实例的 _init(options) 方法，它首先会执行一个 merge options 的逻辑。
```
Vue.prototype._init = function (options?: Object) {
  // merge options
  if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
  } else {
    vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    )
  }
  // ...
}
```
## 外部调用场景
例子：
```
import Vue from 'vue'

let childComp = {
  template: '<div>{{msg}}</div>',
  created() {
    console.log('child created')
  },
  mounted() {
    console.log('child mounted')
  },
  data() {
    return {
      msg: 'Hello Vue'
    }
  }
}

Vue.mixin({
  created() {
    console.log('parent created')
  }
})

let app = new Vue({
  el: '#app',
  render: h => h(childComp)
})
```
当执行 new Vue 的时候，在执行 this._init(options) 的时候，就会执行如下逻辑去合并 options：
```
vm.$options = mergeOptions(
  resolveConstructorOptions(vm.constructor),
  options || {},
  vm
)
```
resolveConstructorOptions简单返回 vm.constructor.options，相当于 Vue.options，它在 initGlobalAPI(Vue) 的时候定义了这个值。主要初始化了：
```
Vue.options.components = {}
Vue.options.directives = {}
Vue.options.filters = {}

Vue.options._base = Vue

extend(Vue.options.components, builtInComponents)
```
把一些内置组件扩展到 Vue.options.components 上，Vue 的内置组件目前有 `<keep-alive>`、`<transition>` 和 `<transition-group>` 组件，这也就是为什么我们在其它组件中使用 `<keep-alive>`组件不需要注册的原因

mergeOptions 主要功能就是把 parent 和 child 这两个对象根据一些合并策略，合并成一个新对象并返回。比较核心的几步，先递归把 extends 和 mixins 合并到 parent 上，然后遍历 parent，调用 mergeField，然后再遍历 child，如果 key 不在 parent 的自身属性上，则调用 mergeField。

mergeField 函数，它对不同的 key 有着不同的合并策略。合并策略的定义都可以在 `src/core/util/options.js` 文件中看到

vm.$options 的值差不多是如下这样：
```
vm.$options = {
  components: { },
  created: [
    function created() {
      console.log('parent created')
    }
  ],
  directives: { },
  filters: { },
  _base: function Vue(options) {
    // ...
  },
  el: "#app",
  render: function (h) {
    //...
  }
}
```

## 组件场景
initInternalComponent(vm, options)
```
export function initInternalComponent (vm: Component, options: InternalComponentOptions) {
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
}
```
initInternalComponent 方法首先执行 const opts = vm.$options = Object.create(vm.constructor.options)，这里的 vm.constructor 就是子组件的构造函数 Sub，相当于 vm.$options = Object.create(Sub.options)。

接着又把实例化子组件传入的子组件父 VNode 实例 parentVnode、子组件的父 Vue 实例 parent 保存到 vm.$options 中，另外还保留了 parentVnode 配置中的如 propsData 等其它的属性。

这么看来，initInternalComponent 只是做了简单一层对象赋值，并不涉及到递归、合并策略等复杂逻辑。

执行完如下合并后，vm.$options 的值差不多是如下这样：
```
vm.$options = {
  parent: Vue /*父Vue实例*/,
  propsData: undefined,
  _componentTag: undefined,
  _parentVnode: VNode /*父VNode实例*/,
  _renderChildren:undefined,
  __proto__: {
    components: { },
    directives: { },
    filters: { },
    _base: function Vue(options) {
        //...
    },
    _Ctor: {},
    created: [
      function created() {
        console.log('parent created')
      }, function created() {
        console.log('child created')
      }
    ],
    mounted: [
      function mounted() {
        console.log('child mounted')
      }
    ],
    data() {
       return {
         msg: 'Hello Vue'
       }
    },
    template: '<div>{{msg}}</div>'
  }
}
```

## 总结

Vue 初始化阶段对于 options 的合并过程就介绍完了，我们需要知道对于 options 的合并有 2 种方式，子组件初始化过程通过 initInternalComponent 方式要比外部初始化 Vue 通过 mergeOptions 的过程要快，合并完的结果保留在 vm.$options 中。

纵观一些库、框架的设计几乎都是类似的，自身定义了一些默认配置，同时又可以在初始化阶段传入一些定义配置，然后去 merge 默认配置，来达到定制化不同需求的目的。

# 生命周期

每个 Vue 实例在被创建之前都要经过一系列的初始化过程。例如需要设置数据监听、编译模板、挂载实例到 DOM、在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做生命周期钩子的函数，给予用户机会在一些特定的场景下添加他们自己的代码。

在我们实际项目开发过程中，会非常频繁地和 Vue 组件的生命周期打交道，接下来我们就从源码的角度来看一下这些生命周期的钩子函数是如何被执行的。

源码中最终执行生命周期的函数都是调用 callHook 方法，它的定义在 src/core/instance/lifecycle 中：
```
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```
callHook 函数的逻辑很简单，根据传入的字符串 hook，去拿到 `vm.$options[hook]` 对应的回调函数数组，然后遍历执行，执行的时候把 vm 作为函数执行的上下文。

在上一节中，我们详细地介绍了 Vue.js 合并 options 的过程，各个阶段的生命周期的函数也被合并到 vm.$options 里，并且是一个数组。因此 callhook 函数的功能就是调用某个生命周期钩子注册的所有回调函数。

## beforeCreate & created

beforeCreate 和 created 函数都是在实例化 Vue 的阶段，在 _init 方法中执行的。

beforeCreate 和 created 的钩子调用是在 initState 的前后，initState 的作用是初始化 props、data、methods、watch、computed 等属性。那么显然 beforeCreate 的钩子函数中就不能获取到 props、data 中定义的值，也不能调用 methods 中定义的函数。

在这俩个钩子函数执行的时候，并没有渲染 DOM，所以我们也不能够访问 DOM，一般来说，如果组件在加载的时候需要和后端有交互，放在这俩个钩子函数执行都可以，如果是需要访问 props、data 等数据的话，就需要使用 created 钩子函数。

## beforeMount & mounted

顾名思义，beforeMount 钩子函数发生在 mount，也就是 DOM 挂载之前

在执行 vm._render() 函数渲染 VNode 之前，执行了 beforeMount 钩子函数，在执行完 vm._update() 把 VNode patch 到真实 DOM 后，执行 mounted 钩子。注意，这里对 mounted 钩子函数执行有一个判断逻辑，vm.$vnode 如果为 null，则表明这不是一次组件的初始化过程，而是我们通过外部 new Vue 初始化过程。那么对于组件，它的 mounted 时机在哪儿呢？

组件的 VNode patch 到 DOM 后，会执行 invokeInsertHook 函数，把 insertedVnodeQueue 里保存的钩子函数依次执行一遍，该函数会执行 insert 这个钩子函数。

每个子组件都是在这个钩子函数中执行 mounted 钩子函数，并且我们之前分析过，insertedVnodeQueue 的添加顺序是先子后父，所以对于同步渲染的子组件而言，mounted 钩子函数的执行顺序也是**先子后父**。

## beforeUpdate & updated
顾名思义，beforeUpdate 和 updated 的钩子函数执行时机都应该是在数据更新的时候。

beforeUpdate 的执行时机是在渲染 Watcher 的 before 函数中，注意这里有个判断，也就是在组件已经 mounted 之后，才会去调用这个钩子函数。update 的执行时机是在flushSchedulerQueue 函数调用的时候。

flushSchedulerQueue 函数我们之后会详细介绍，可以先大概了解一下，updatedQueue 是更新了的 wathcer 数组，那么在 callUpdatedHooks 函数中，它对这些数组做遍历，只有满足当前 watcher 为 vm._watcher 以及组件已经 mounted 这两个条件，才会执行 updated 钩子函数。

## beforeDestroy & destroyed

beforeDestroy 和 destroyed 钩子函数的执行时机在组件销毁的阶段。最终会调用 $destroy 方法，它的定义在 src/core/instance/lifecycle.js 中。

beforeDestroy 钩子函数的执行时机是在 $destroy 函数执行最开始的地方，接着执行了一系列的销毁动作，包括从 parent 的 $children 中删掉自身，删除 watcher，当前渲染的 VNode 执行销毁钩子函数等，执行完毕后再调用 destroy 钩子函数。

在 $destroy 的执行过程中，它又会执行 vm.__patch__(vm._vnode, null) 触发它子组件的销毁钩子函数，这样一层层的递归调用，**所以 destroy 钩子函数执行顺序是先子后父，和 mounted 过程一样**。

## activated & deactivated

activated 和 deactivated 钩子函数是专门为 keep-alive 组件定制的钩子

## 总结

这一节主要介绍了 Vue 生命周期中各个钩子函数的执行时机以及顺序，通过分析，我们知道了如在 created 钩子函数中可以访问到数据，在 mounted 钩子函数中可以访问到 DOM，在 destroy 钩子函数中可以做一些定时器销毁工作，了解它们有利于我们在合适的生命周期去做不同的事情。

# 组件注册
在 Vue.js 中，除了它内置的组件如 keep-alive、component、transition、transition-group 等，其它用户自定义组件在使用前必须注册。很多同学在开发过程中可能会遇到如下报错信息：
```
'Unknown custom element: <xxx> - did you register the component correctly?
 For recursive components, make sure to provide the "name" option.'
```
一般报这个错的原因都是我们使用了未注册的组件。Vue.js 提供了 2 种组件的注册方式，全局注册和局部注册。接下来我们从源码分析的角度来分析这两种注册方式。

## 全局注册
要注册一个全局组件，可以使用 Vue.component(tagName, options)。例如：
```
Vue.component('my-component', {
  // 选项
})
```
Vue.component 函数，它的定义过程发生在最开始初始化 Vue 的全局函数的时候。实际上 Vue 是初始化了 3 个全局函数，并且如果 type 是 component 且 definition 是一个对象的话，通过 `this.opitons._base.extend`， 相当于 Vue.extend 把这个对象转换成一个继承于 Vue 的构造函数，最后通过 `this.options[type + 's'][id] = definition` 把它挂载到 Vue.options.components 上。

## 局部注册
Vue.js 也同样支持局部注册，我们可以在一个组件内部使用 components 选项做组件的局部注册，例如：
```
import HelloWorld from './components/HelloWorld'

export default {
  components: {
    HelloWorld
  }
}
```
其实理解了全局注册的过程，局部注册是非常简单的。在组件的 Vue 的实例化阶段有一个合并 option 的逻辑，之前我们也分析过，所以就把 components 合并到 vm.$options.components 上，这样我们就可以在 resolveAsset 的时候拿到这个组件的构造函数，并作为 createComponent 的钩子的参数。

注意，局部注册和全局注册不同的是，只有该类型的组件才可以访问局部注册的子组件，而全局注册是扩展到 Vue.options 下，所以在所有组件创建的过程中，都会从全局的 Vue.options.components 扩展到当前组件的 vm.$options.components 下，这就是全局注册的组件能被任意使用的原因。

## 总结
通过这一小节的分析，我们对组件的注册过程有了认识，并理解了全局注册和局部注册的差异。其实在平时的工作中，当我们使用到组件库的时候，往往更通用基础组件都是全局注册的，而编写的特例场景的业务组件都是局部注册的。了解了它们的原理，对我们在工作中到底使用全局注册组件还是局部注册组件是有这非常好的指导意义的。

# 异步组件
在我们平时的开发工作中，为了减少首屏代码体积，往往会把一些非首屏的组件设计成异步组件，按需加载。Vue 也原生支持了异步组件的能力，如下：
```
Vue.component('async-example', function (resolve, reject) {
   // 这个特殊的 require 语法告诉 webpack
   // 自动将编译后的代码分割成不同的块，
   // 这些块将通过 Ajax 请求自动下载。
   require(['./my-async-component'], resolve)
})
```
示例中可以看到，Vue 注册的组件不再是一个对象，而是一个工厂函数，函数有两个参数 resolve 和 reject，函数内部用 setTimout 模拟了异步，实际使用可能是通过动态请求异步组件的 JS 地址，最终通过执行 resolve 方法，它的参数就是我们的异步组件对象。

除了刚才示例的组件注册方式，还支持 2 种，一种是支持 Promise 创建组件的方式，如下：
```
Vue.component(
  'async-webpack-example',
  // 该 `import` 函数返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```
另一种是高级异步组件，如下：
```
const AsyncComp = () => ({
  // 需要加载的组件。应当是一个 Promise
  component: import('./MyComp.vue'),
  // 加载中应当渲染的组件
  loading: LoadingComp,
  // 出错时渲染的组件
  error: ErrorComp,
  // 渲染加载中组件前的等待时间。默认：200ms。
  delay: 200,
  // 最长等待时间。超出此时间则渲染错误组件。默认：Infinity
  timeout: 3000
})
Vue.component('async-example', AsyncComp)
```

## 总结
通过以上代码分析，我们对 Vue 的异步组件的实现有了深入的了解，知道了 3 种异步组件的实现方式，并且看到高级异步组件的实现是非常巧妙的，它实现了 loading、resolve、reject、timeout 4 种状态。异步组件实现的本质是 2 次渲染，除了 0 delay 的高级异步组件第一次直接渲染成 loading 组件外，其它都是第一次渲染生成一个注释节点，当异步获取组件成功后，再通过 forceRender 强制重新渲染，这样就能正确渲染出我们异步加载的组件了。