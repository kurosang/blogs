# vue源码-响应式原理

>前面 2 章介绍的都是 Vue 怎么实现数据渲染和组件化的，主要讲的是初始化的过程，把原始的数据最终映射到 DOM 中，但并没有涉及到数据变化到 DOM 变化的部分。而 Vue 的数据驱动除了数据渲染 DOM 之外，还有一个很重要的体现就是数据的变更会触发 DOM 的变化。

# 响应式对象

## Object.defineProperty
Object.defineProperty 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象，先来看一下它的语法：
```
Object.defineProperty(obj, prop, descriptor)
```
obj 是要在其上定义属性的对象；prop 是要定义或修改的属性的名称；descriptor 是将被定义或修改的属性描述符。

比较核心的是 descriptor，它有很多可选键值，具体的可以去参阅它的文档。这里我们最关心的是 get 和 set，get 是一个给属性提供的 getter 方法，当我们访问了该属性的时候会触发 getter 方法；set 是一个给属性提供的 setter 方法，当我们对该属性做修改的时候会触发 setter 方法。

一旦对象拥有了 getter 和 setter，我们可以简单地把这个对象称为响应式对象。

## initState

在 Vue 的初始化阶段，_init 方法执行的时候，会执行 initState(vm) 方法。initState 方法主要是对 props、methods、data、computed 和 wathcer 等属性做了初始化操作。这里我们重点分析 props 和 data。
* initProps：
props 的初始化主要过程，就是遍历定义的 props 配置。遍历的过程主要做两件事情：一个是调用 `defineReactive` 方法把每个 prop 对应的值变成响应式，可以通过 vm._props.xxx 访问到定义 props 中对应的属性。另一个是通过 `proxy` 把 vm._props.xxx 的访问代理到 `vm.xxx` 上。
* initData：
data 的初始化主要过程也是做两件事，一个是对定义 data 函数返回对象的遍历，通过 `proxy` 把每一个值 vm._data.xxx 都代理到 `vm.xxx` 上；另一个是调用 `observe` 方法观测整个 data 的变化，把 data 也变成响应式，可以通过 vm._data.xxx 访问到定义 data 返回函数中对应的属性。

可以看到，无论是 props 或是 data 的初始化都是把它们变成响应式对象，这个过程我们接触到几个函数，接下来我们来详细分析它们。

## proxy
代理，代理的作用是把 props 和 data 上的属性代理到 vm 实例上，这也就是为什么比如我们定义了如下 props，却可以通过 vm 实例访问到它。

proxy 方法的实现很简单，通过 `Object.defineProperty` 把 `target[sourceKey][key]` 的读写变成了对 `target[key]` 的读写。所以对于 props 而言，对 vm._props.xxx 的读写变成了 `vm.xxx` 的读写，而对于 vm._props.xxx 我们可以访问到定义在 props 中的属性，所以我们就可以通过 `vm.xxx` 访问到定义在 props 中的 xxx 属性了。同理，对于 data 而言，对 vm._data.xxxx 的读写变成了对 vm.xxxx 的读写，而对于 vm._data.xxxx 我们可以访问到定义在 data 函数返回对象中的属性，所以我们就可以通过 vm.xxxx 访问到定义在 data 函数返回对象中的 xxxx 属性了。

## observe
observe 的功能就是用来监测数据的变化。
observe 方法的作用就是给非 VNode 的对象类型数据添加一个 Observer，如果已经添加过则直接返回，否则在满足一定条件下去实例化一个 Observer 对象实例。

## Observer
Observer 是一个类，它的作用是给对象的属性添加 getter 和 setter，用于依赖收集和派发更新.

Observer 的构造函数逻辑很简单，首先实例化 Dep 对象，这块稍后会介绍，接着通过执行 def 函数把自身实例添加到数据对象 value 的 `__ob__` 属性上

def 函数是一个非常简单的Object.defineProperty 的封装，这就是为什么我在开发中输出 data 上对象类型的数据，会发现该对象多了一个 `__ob__` 的属性。

回到 Observer 的构造函数，接下来会对 value 做判断，对于数组会调用 `observeArray` 方法，否则对纯对象调用 `walk`方法。可以看到 observeArray 是遍历数组再次调用 observe 方法，而 walk 方法是遍历对象的 key 调用 `defineReactive` 方法。

## defineReactive
defineReactive 的功能就是定义一个响应式对象，给对象动态添加 getter 和 setter.

defineReactive 函数最开始初始化 Dep 对象的实例，接着拿到 obj 的属性描述符，然后对子对象递归调用 observe 方法，这样就保证了无论 obj 的结构多复杂，它的所有子属性也能变成响应式的对象，这样我们访问或修改 obj 中一个嵌套较深的属性，也能触发 getter 和 setter。最后利用 Object.defineProperty 去给 obj 的属性 key 添加 getter 和 setter。

## 总结
响应式对象，核心就是利用 Object.defineProperty 给数据添加了 getter 和 setter，目的就是为了在我们访问数据以及写数据的时候能自动执行一些逻辑：getter 做的事情是依赖收集，setter 做的事情是派发更新，

# getter 依赖收集
Vue 会把普通对象变成响应式对象，响应式对象 getter 相关的逻辑就是做依赖收集.

defineReactive中 getter 部分的逻辑：只需要关注 2 个地方，一个是 `const dep = new Dep()` 实例化一个 Dep 的实例，另一个是在 get 函数中通过 `dep.depend` 做依赖收集，这里还有个对 childOb 判断的逻辑

## Dep

Dep是整个 getter 依赖收集的核心。Dep 是一个 Class，它定义了一些属性和方法，这里需要特别注意的是它有一个静态属性 target，这是一个全局唯一 Watcher，这是一个非常巧妙的设计，因为在同一时间只能有一个全局的 Watcher 被计算，另外它的自身属性 subs 也是 Watcher 的数组。Dep 实际上就是对 Watcher 的一种管理，Dep 脱离 Watcher 单独存在是没有意义的。

## Watcher
Watcher 是一个 Class，在它的构造函数中，定义了一些和 Dep 相关的属性：
```
this.deps = []
this.newDeps = []
this.depIds = new Set()
this.newDepIds = new Set()
```
其中，this.deps 和 this.newDeps 表示 Watcher 实例持有的 Dep 实例的数组；而 this.depIds 和 this.newDepIds 分别代表 this.deps 和 this.newDeps 的 id Set（这个 Set 是 ES6 的数据结构，它的实现在 src/core/util/env.js 中）。那么这里为何需要有 2 个 Dep 实例数组呢

Watcher 还定义了一些原型的方法，和依赖收集相关的有 get、addDep 和 cleanupDeps 方法

## 过程分析
当对数据对象的访问会触发他们的 getter 方法，那么这些对象什么时候被访问呢？

当我们去实例化一个渲染 watcher 的时候，首先进入 watcher 的构造函数逻辑，然后会执行它的 this.get() 方法，进入 get 函数，首先会执行：
```
pushTarget(this)
```
```
export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}
```
实际上就是把 Dep.target 赋值为当前的渲染 watcher 并压栈（为了恢复用）。接着又执行了：
```
value = this.getter.call(vm, vm)
```
this.getter 对应就是 updateComponent 函数，这实际上就是在执行：
```
vm._update(vm._render(), hydrating)
```
它会先执行 vm._render() 方法，因为之前分析过这个方法会生成 渲染 VNode，并且在这个过程中会对 vm 上的数据访问，这个时候就触发了数据对象的 getter。

那么每个对象值的 getter 都持有一个 dep，在触发 getter 的时候会调用 dep.depend() 方法，也就会执行 Dep.target.addDep(this)。

刚才我们提到这个时候 Dep.target 已经被赋值为渲染 watcher，那么就执行到 addDep 方法：
```
addDep (dep: Dep) {
  const id = dep.id
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id)
    this.newDeps.push(dep)
    if (!this.depIds.has(id)) {
      dep.addSub(this)
    }
  }
}
```
这时候会做一些逻辑判断（保证同一数据不会被添加多次）后执行 dep.addSub(this)，那么就会执行 this.subs.push(sub)，也就是说把当前的 watcher 订阅到这个数据持有的 dep 的 subs 中，这个目的是为后续数据变化时候能通知到哪些 subs 做准备。

所以在 vm._render() 过程中，会触发所有数据的 getter，这样实际上已经完成了一个依赖收集的过程。那么到这里就结束了么，其实并没有，在完成依赖收集后，还有几个逻辑要执行:
* 1.deep开启，`traverse(value)`。这个是要递归去访问 value，触发它所有子项的 getter
* 2.把 Dep.target 恢复成上一个状态，因为当前 vm 的数据依赖收集已经完成，那么对应的渲染Dep.target 也需要改变
* 3. 执行`this.cleanupDeps()`,依赖清空

考虑到 Vue 是数据驱动的，所以每次数据变化都会重新 render，那么 vm._render() 方法又会再次执行，并再次触发数据的 getters，所以 Watcher 在构造函数中会初始化 2 个 Dep 实例数组，newDeps 表示新添加的 Dep 实例数组，而 deps 表示上一次添加的 Dep 实例数组。

在执行 cleanupDeps 函数的时候，会首先遍历 deps，移除对 dep.subs 数组中 Wathcer 的订阅，然后把 newDepIds 和 depIds 交换，newDeps 和 deps 交换，并把 newDepIds 和 newDeps 清空。

那么为什么需要做 deps 订阅的移除呢，在添加 deps 的订阅过程，已经能通过 id 去重避免重复订阅了。

考虑到一种场景，我们的模板会根据 v-if 去渲染不同子模板 a 和 b，当我们满足某种条件的时候渲染 a 的时候，会访问到 a 中的数据，这时候我们对 a 使用的数据添加了 getter，做了依赖收集，那么当我们去修改 a 的数据的时候，理应通知到这些订阅者。那么如果我们一旦改变了条件渲染了 b 模板，又会对 b 使用的数据添加了 getter，如果我们没有依赖移除的过程，那么这时候我去修改 a 模板的数据，会通知 a 数据的订阅的回调，这显然是有浪费的。

因此 Vue 设计了在每次添加完新的订阅，会移除掉旧的订阅，这样就保证了在我们刚才的场景中，如果渲染 b 模板的时候去修改 a 模板的数据，a 数据订阅回调已经被移除了，所以不会有任何浪费，真的是非常赞叹 Vue 对一些细节上的处理。

## 总结
收集依赖的目的是为了当这些响应式数据发生变化，触发它们的 setter 的时候，能知道应该通知哪些订阅者去做相应的逻辑处理，我们把这个过程叫派发更新，其实 Watcher 和 Dep 就是一个非常经典的观察者设计模式的实现.

# setter 派发更新
我们了解了响应式数据依赖收集过程，收集的目的就是为了当我们修改数据的时候，可以对相关的依赖派发更新.

setter 的逻辑有 2 个关键的点，一个是 childOb = !shallow && observe(newVal)，如果 shallow 为 false 的情况，会对新设置的值变成一个响应式对象；另一个是 dep.notify()，通知所有的订阅者

## 过程分析

当我们在组件中对响应的数据做了修改，就会触发 setter 的逻辑，最后调用 dep.notify() 方法， 它是 Dep 的一个实例方法。

dep.notify() 的逻辑非常简单，遍历所有的 subs，也就是 Watcher 的实例数组，然后调用每一个 watcher 的 update 方法.

这里对于 Watcher 的不同状态，会执行不同的逻辑,在一般组件数据更新的场景，会走到最后一个 queueWatcher(this) 的逻辑

这里引入了一个队列的概念，这也是 Vue 在做派发更新的时候的一个优化的点，它并不会每次数据改变都触发 watcher 的回调，而是把这些 watcher 先添加到一个队列里，然后在 nextTick 后执行 flushSchedulerQueue。

这里有几个细节要注意一下
* 首先用 has 对象保证同一个 Watcher 只添加一次；
* 接着对 flushing 的判断，else 部分的逻辑稍后我会讲；
* 最后通过 waiting 保证对 nextTick(flushSchedulerQueue) 的调用逻辑只有一次

另外 nextTick 的实现我之后会抽一小节专门去讲，目前就可以理解它是在下一个 tick，也就是异步的去执行 flushSchedulerQueue。

flushSchedulerQueued的逻辑，这里有几个重要的逻辑要梳理一下：
* 队列排序

    `queue.sort((a, b) => a.id - b.id)` 对队列做了从小到大的排序，这么做主要有以下要确保以下几点：

    1.组件的更新由父到子；因为父组件的创建过程是先于子的，所以 watcher 的创建也是先父后子，执行顺序也应该保持先父后子。

    2.用户的自定义 watcher 要优先于渲染 watcher 执行；因为用户自定义 watcher 是在渲染 watcher 之前创建的。

    3.如果一个组件在父组件的 watcher 执行期间被销毁，那么它对应的 watcher 执行都可以被跳过，所以父组件的 watcher 应该先执行。

* 队列遍历

    在对 queue 排序后，接着就是要对它做遍历，拿到对应的 watcher，执行 watcher.run()。这里需要注意一个细节，在遍历的时候每次都会对 queue.length 求值，因为在 watcher.run() 的时候，很可能用户会再次添加新的 watcher，这样会再次执行到 queueWatcher

* 状态恢复

    逻辑非常简单，就是把这些控制流程状态的一些变量恢复到初始值，把 watcher 队列清空。

## watcher.run()
run 函数实际上就是执行 this.getAndInvoke 方法，并传入 watcher 的回调函数。getAndInvoke 函数逻辑也很简单，先通过 this.get() 得到它当前的值，然后做判断，如果满足新旧值不等、新值是对象类型、deep 模式任何一个条件，则执行 watcher 的回调，注意回调函数执行的时候会把第一个和第二个参数传入新值 value 和旧值 oldValue，这就是当我们添加自定义 watcher 的时候能在回调函数的参数中拿到新旧值的原因。

那么对于渲染 watcher 而言，它在执行 this.get() 方法求值的时候，会执行 getter 方法：
```
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```
所以这就是当我们去修改组件相关的响应式数据的时候，会触发组件重新渲染的原因，接着就会重新执行 patch 的过程，但它和首次渲染有所不同.

## 总结
当数据发生变化的时候，触发 setter 逻辑，把在依赖过程中订阅的的所有观察者，也就是 watcher，都触发它们的 update 过程，这个过程又利用了队列做了进一步优化，在 nextTick 后执行所有 watcher 的 run，最后执行它们的回调函数。

# nextTick

nextTick 的实现在 src/core/util/next-tick.js 中。

next-tick.js 申明了 microTimerFunc 和 macroTimerFunc 2 个变量，它们分别对应的是 micro task 的函数和 macro task 的函数。对于 macro task 的实现，优先检测是否支持原生 setImmediate，这是一个高版本 IE 和 Edge 才支持的特性，不支持的话再去检测是否支持原生的 MessageChannel，如果也不支持的话就会降级为 setTimeout 0；而对于 micro task 的实现，则检测浏览器是否原生支持 Promise，不支持的话直接指向 macro task 的实现。

这里使用 callbacks 而不是直接在 nextTick 中执行回调函数的原因是保证在同一个 tick 内多次执行 nextTick，不会开启多个异步任务，而把这些异步任务都压成一个同步任务，在下一个 tick 执行完毕。

nextTick 函数最后还有一段逻辑：
```
 if (!cb && typeof Promise !== 'undefined') {
  return new Promise(resolve => {
    _resolve = resolve
  })
}
```
这是当 nextTick 不传 cb 参数的时候，提供一个 Promise 化的调用，比如：
```
nextTick().then(() => {})
```
当 _resolve 函数执行，就会跳到 then 的逻辑中。

## 总结
数据的变化到 DOM 的重新渲染是一个异步过程，发生在下一个 tick。这就是我们平时在开发的过程中，比如从服务端接口去获取数据的时候，数据做了修改，如果我们的某些方法去依赖了数据修改后的 DOM 变化，我们就必须在 nextTick 后执行。


# 检测变化的注意事项

## 对象添加属性

对于使用 Object.defineProperty 实现响应式的对象，当我们去给这个对象添加一个新的属性的时候，是不能够触发它的 setter 的，比如：
```
var vm = new Vue({
  data:{
    a:1
  }
})
// vm.b 是非响应的
vm.b = 2
```
但是添加新属性的场景我们在平时开发中会经常遇到，那么 Vue 为了解决这个问题，定义了一个全局 API Vue.set 方法.

set 方法接收 3个参数，target 可能是数组或者是普通对象，key 代表的是数组的下标或者是对象的键值，val 代表添加的值。首先判断如果 target 是数组且 key 是一个合法的下标，则之前通过 splice 去添加进数组然后返回，这里的 splice 其实已经不仅仅是原生数组的 splice 了，稍后我会详细介绍数组的逻辑。接着又判断 key 已经存在于 target 中，则直接赋值返回，因为这样的变化是可以观测到了。接着再获取到 `target.__ob__` 并赋值给 ob，之前分析过它是在 Observer 的构造函数执行的时候初始化的，表示 Observer 的一个实例，如果它不存在，则说明 target 不是一个响应式的对象，则直接赋值并返回。最后通过 defineReactive(ob.value, key, val) 把新添加的属性变成响应式对象，然后再通过 ob.dep.notify() 手动的触发依赖通知.

## 数组
数组的情况，Vue 也是不能检测到以下变动的数组：
* 1.当你利用索引直接设置一个项时，例如：`vm.items[indexOfItem] = newValue`

* 2.当你修改数组的长度时，例如：vm.items.length = newLength

对于第一种情况，可以使用：Vue.set(example1.items, indexOfItem, newValue)；而对于第二种情况，可以使用 vm.items.splice(newLength)。

我们刚才也分析到，对于 Vue.set 的实现，当 target 是数组的时候，也是通过 target.splice(key, 1, val) 来添加的，那么这里的 splice 到底有什么黑魔法，能让添加的对象变成响应式的呢。
```
'push',
'pop',
'shift',
'unshift',
'splice',
'sort',
'reverse'
```

Vue，arrayMethods 首先继承了 Array，然后对数组中所有能改变数组自身的方法，如 push、pop 等这些方法进行重写。重写后的方法会先执行它们本身原有的逻辑，并对能增加数组长度的 3 个方法 push、unshift、splice 方法做了判断，获取到插入的值，然后把新添加的值变成一个响应式对象，并且再调用 ob.dep.notify() 手动触发依赖通知，这就很好地解释了之前的示例中调用 vm.items.splice(newLength) 方法可以检测到变化。

### 总结
在实际工作中遇到了这些特殊情况，我们就可以知道如何把它们也变成响应式的对象。
其实对于对象属性的删除也会用同样的问题，Vue 同样提供了 Vue.del 的全局 API，它的实现和 Vue.set 大同小异，甚至还要更简单一些.

# 计算属性 computed VS 侦听属性 watch

## computed
计算属性本质上就是一个 computed watcher，也了解了它的创建过程和被访问触发 getter 以及依赖更新的过程，其实这是最新的计算属性的实现，之所以这么设计是因为 Vue 想确保不仅仅是计算属性依赖的值发生变化，而是当计算属性最终计算的值发生变化才会触发渲染 watcher 重新渲染，本质上是一种优化。

## watch 
本质上侦听属性也是基于 Watcher 实现的，它是一个 user watcher。其实 Watcher 支持了不同的类型，下面我们梳理一下它有哪些类型以及它们的作用

## Watcher options
watcher 总共有 4 种类型:
* 1.deep watcher

    通常，如果我们想对一下对象做深度观测的时候，需要设置这个属性为 true.在对 watch 的表达式或者函数求值后，会调用 traverse 函数

    traverse 的逻辑也很简单，它实际上就是**对一个对象做深层递归遍历**，因为遍历过程中就是对一个子对象的访问，会触发它们的 getter 过程，这样就可以收集到依赖，也就是订阅它们变化的 watcher，这个函数实现还有一个小的优化，遍历过程中会把子响应式对象通过它们的 dep id 记录到 seenObjects，避免以后重复访问。

    那么在执行了 traverse 后，我们再对 watch 的对象内部任何一个值做修改，也会调用 watcher 的回调函数了。

    对 deep watcher 的理解非常重要，今后工作中如果大家观测了一个复杂对象，并且会改变对象内部深层某个值的时候也希望触发回调，一定要设置 deep 为 true，但是因为设置了 deep 后会执行 traverse 函数，**会有一定的性能开销，所以一定要根据应用场景权衡是否要开启这个配置**

* 2.user watcher

    通过 vm.$watch 创建的 watcher 是一个 user watcher，其实它的功能很简单，在对 watcher 求值以及在执行回调函数的时候，会处理一下错误

* 3.computed watcher

    computed watcher 几乎就是为计算属性量身定制的.

* 4.sync watcher

    在我们之前对 setter 的分析过程知道，当响应式数据发送变化后，触发了 watcher.update()，只是把这个 watcher 推送到一个队列中，在 nextTick 后才会真正执行 watcher 的回调函数。而一旦我们设置了 sync，就可以在当前 Tick 中同步执行 watcher 的回调函数。

    只有当我们需要 watch 的值的变化到执行 watcher 的回调函数是一个同步过程的时候才会去设置该属性为 true。

## 总结
通过这一小节的分析我们对计算属性和侦听属性的实现有了深入的了解，计算属性本质上是 computed watcher，而侦听属性本质上是 user watcher。就应用场景而言，计算属性适合用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测某个值的变化去完成一段复杂的业务逻辑。

同时我们又了解了 watcher 的 4 个 options，通常我们会在创建 user watcher 的时候配置 deep 和 sync，可以根据不同的场景做相应的配置。


# 组件更新

当数据发生变化的时候，会触发渲染 watcher 的回调函数，进而执行组件的更新过程。

组件的更新还是调用了 vm._update 方法

组件更新的过程，会执行 `vm.$el = vm.__patch__(prevVnode, vnode)`，它仍然会调用 patch 函数

这里执行 patch 的逻辑和首次渲染是不一样的，因为 oldVnode 不为空，并且它和 vnode 都是 VNode 类型，接下来会通过 sameVNode(oldVnode, vnode) 判断它们是否是相同的 VNode 来决定走不同的更新逻辑：

sameVnode 的逻辑非常简单，如果两个 vnode 的 key 不相等，则是不同的；否则继续判断对于同步组件，则判断 isComment、data、input 类型等是否相同，对于异步组件，则判断 asyncFactory 是否相同。

所以根据新旧 vnode 是否为 sameVnode，会走到不同的更新逻辑，我们先来说一下不同的情况。

## 新旧节点不同
如果新旧 vnode 不同，那么更新的逻辑非常简单，它本质上是要替换已存在的节点，大致分为 3 步:(创建新节点 -> 更新占位符节点 -> 删除旧节点)
* 1.创建新节点

    以当前旧节点为参考节点，创建新的节点，并插入到 DOM 中

* 2.更新父的占位符节点

    找到当前 vnode 的父的占位符节点，先执行各个 module 的 destroy 的钩子函数，如果当前占位符是一个可挂载的节点，则执行 module 的 create 钩子函数。

* 3.删除旧节点

    把 oldVnode 从当前 DOM 树中删除，如果父节点存在，则执行 removeVnodes 方法.

    删除节点逻辑很简单，就是遍历待删除的 vnodes 做删除，其中 removeAndInvokeRemoveHook 的作用是从 DOM 中移除节点并执行 module 的 remove 钩子函数，并对它的子节点递归调用 removeAndInvokeRemoveHook 函数；invokeDestroyHook 是执行 module 的 destory 钩子函数以及 vnode 的 destory 钩子函数，并对它的子 vnode 递归调用 invokeDestroyHook 函数；removeNode 就是调用平台的 DOM API 去把真正的 DOM 节点移除。

    当组件并不是 keepAlive 的时候，会执行 componentInstance.$destroy() 方法，然后就会执行 beforeDestroy & destroyed 两个钩子函数，两个钩子函数就是在执行 invokeDestroyHook 过程中，执行了 vnode 的 destory 钩子函数

## 新旧节点相同
组件 vnode 的更新情况是新旧节点相同，它会调用 patchVNode 方法.

patchVnode 的作用就是把新的 vnode patch 到旧的 vnode 上，这里我们只关注关键的核心逻辑，我把它拆成四步骤：

* 1.执行 prepatch 钩子函数

    当更新的 vnode 是一个组件 vnode 的时候，会执行 prepatch 的方法

    prepatch 方法就是拿到新的 vnode 的组件配置以及组件实例，去执行 updateChildComponent 方法

    updateChildComponent 的逻辑也非常简单，由于更新了 vnode，那么 vnode 对应的实例 vm 的一系列属性也会发生变化，包括占位符 vm.$vnode 的更新、slot 的更新，listeners 的更新，props 的更新等等。

* 2.执行 update 钩子函数

    回到 patchVNode 函数，在执行完新的 vnode 的 prepatch 钩子函数，会执行所有 module 的 update 钩子函数以及用户自定义 update 钩子函数.

* 3.完成 patch 过程

    如果 vnode 是个文本节点且新旧文本不相同，则直接替换文本内容。如果不是文本节点，则判断它们的子节点，并分了几种情况处理：

    * 1.oldCh 与 ch 都存在且不相同时，使用 updateChildren 函数来更新子节点，这个后面重点讲。

    * 2.如果只有 ch 存在，表示旧节点不需要了。如果旧的节点是文本节点则先将节点的文本清除，然后通过 addVnodes 将 ch 批量插入到新节点 elm 下。

    * 3.如果只有 oldCh 存在，表示更新的是空节点，则需要将旧的节点通过 removeVnodes 全部清除。

    * 4.当只有旧节点是文本节点的时候，则清除其节点文本内容。

* 4.执行 postpatch 钩子函数

    在执行完 patch 过程后，会执行 postpatch 钩子函数，它是组件自定义的钩子函数，有则执行。

    那么在整个 pathVnode 过程中，最复杂的就是 updateChildren 方法了，下面我们来单独介绍它。

## updateChildren

updateChildren 的逻辑比较复杂，直接读源码比较晦涩，我们可以通过一个具体的示例来分析它。
```
<template>
  <div id="app">
    <div>
      <ul>
        <li v-for="item in items" :key="item.id">{{ item.val }}</li>
      </ul>
    </div>
    <button @click="change">change</button>
  </div>
</template>

<script>
  export default {
    name: 'App',
    data() {
      return {
        items: [
          {id: 0, val: 'A'},
          {id: 1, val: 'B'},
          {id: 2, val: 'C'},
          {id: 3, val: 'D'}
        ]
      }
    },
    methods: {
      change() {
        this.items.reverse().push({id: 4, val: 'E'})
      }
    }
  }
</script>
```
当我们点击 change 按钮去改变数据，最终会执行到 updateChildren 去更新 li 部分的列表数据，我们通过图的方式来描述一下它的更新过程：

第一步：

![alt](../../imgs/diff1.png)

第二步：

![alt](../../imgs/diff2.png)

第三步：

![alt](../../imgs/diff3.png)

第四步：

![alt](../../imgs/diff4.png)

第五步：

![alt](../../imgs/diff5.png)

第六步：

![alt](../../imgs/diff6.png)

## 总结
组件更新的过程核心就是新旧 vnode diff，对新旧节点相同以及不同的情况分别做不同的处理。新旧节点不同的更新流程是创建新节点->更新父占位符节点->删除旧节点；而新旧节点相同的更新流程是去获取它们的 children，根据不同情况做不同的更新逻辑。最复杂的情况是新旧节点相同且它们都存在子节点，那么会执行 updateChildren 逻辑，这块儿可以借助画图的方式配合理解。

# Props (v2.6.11)
Props 作为组件的核心特性之一，也是我们平时开发 Vue 项目中接触最多的特性之一，它可以让组件的功能变得丰富，也是父子组件通讯的一个渠道。那么它的实现原理是怎样的，我们来一探究竟。

## 规范化
实际上 props 除了对象格式，还允许写成数组格式。

当 props 是一个数组，每一个数组元素 prop 只能是一个 string，表示 prop 的 key，转成驼峰格式，prop 的类型为空。

当 props 是一个对象，对于 props 中每个 prop 的 key，我们会转驼峰格式，而它的 value，如果不是一个对象，我们就把它规范成一个对象。

如果 props 既不是数组也不是对象，就抛出一个警告。

```
export default {
  props: ['name', 'nick-name']
}
// 经过 normalizeProps 后，会被规范成：
options.props = {
  name: { type: null },
  nickName: { type: null }
}
```
```
export default {
  props: {
    name: String,
    nickName: {
      type: Boolean
    }
  }
}
// 经过 normalizeProps 后，会被规范成：
options.props = {
  name: { type: String },
  nickName: { type: Boolean }
}
```
由于对象形式的 props 可以指定每个 prop 的类型和定义其它的一些属性，推荐用对象形式定义 props。

## 初始化
Props 的初始化主要发生在 new Vue 中的 initState 阶段,initProps 主要做 3 件事情：校验、响应式和代理。

* 1.校验

    所谓校验的目的就是检查一下我们传递的数据是否满足 prop的定义规范.校验的逻辑很简单，遍历 propsOptions，执行 validateProp.

    validateProp 主要就做 3 件事情：处理 Boolean 类型的数据，处理默认数据，prop 断言，并最终返回 prop 的值.

    * validateProp 函数的 Boolean 类型数据的处理逻辑和默认数据处理逻辑。除了 Boolean 类型的数据，其余没有设置 default 属性的 prop 默认值都是 undefined.
    * assertProp 函数的目的是断言这个 prop 是否合法。

* 2.响应式

    对 prop 做验证并且获取到 prop 的值后，接下来需要通过 defineReactive 把 prop 变成响应式。

    在开发环境中我们会校验 prop 的 key 是否是 HTML 的保留属性，并且在 defineReactive 的时候会添加一个自定义 setter，当我们直接对 prop 赋值的时候会输出警告

    关于 prop 的响应式有一点不同的是当 vm 是非根实例的时候，会先执行 toggleObserving(false)，它的目的是为了响应式的优化

* 3.代理

    在经过响应式处理后，我们会把 prop 的值添加到 vm._props 中，比如 key 为 name 的 prop，它的值保存在 vm._props.name 中，但是我们在组件中可以通过 this.name 访问到这个 prop，这就是代理做的事情。

    其实对于非根实例的子组件而言，prop 的代理发生在 Vue.extend 阶段。这么做的好处是不用为每个组件实例都做一层 proxy，是一种优化手段。

## Props 更新
我们知道，当父组件传递给子组件的 props 值变化，子组件对应的值也会改变，同时会触发子组件的重新渲染.

## 子组件 props 更新
首先，prop 数据的值变化在父组件，我们知道在父组件的 render 过程中会访问到这个 prop 数据，所以当 prop 数据变化一定会触发父组件的重新渲染，那么重新渲染是如何更新子组件对应的 prop 的值呢？

在父组件重新渲染的最后，会执行 patch 过程，进而执行 patchVnode 函数，patchVnode 通常是一个递归过程，当它遇到组件 vnode 的时候，会执行组件更新过程的 prepatch 钩子函数。

prepatch 函数内部会调用 updateChildComponent 方法来更新 props，注意第二个参数就是父组件的 propData，那么为什么 vnode.componentOptions.propsData 就是父组件传递给子组件的 prop 数据呢（这个也同样解释了第一次渲染的 propsData 来源）？原来在组件的 render 过程中，对于组件节点会通过 createComponent 方法来创建组件 vnode。

在创建组件 vnode 的过程中，首先从 data 中提取出 propData，然后在 new VNode 的时候，作为第七个参数 VNodeComponentOptions 中的一个属性传入，所以我们可以通过 vnode.componentOptions.propsData 拿到 prop 数据。

接着看 updateChildComponent，我们重点来看更新 props 的相关逻辑，这里的 propsData 是父组件传递的 props 数据，vm 是子组件的实例。vm._props 指向的就是子组件的 props 值，propKeys 就是在之前 initProps 过程中，缓存的子组件中定义的所有 prop 的 key。主要逻辑就是遍历 propKeys，然后执行 `props[key] = validateProp(key, propOptions, propsData, vm)` 重新验证和计算新的 prop 数据，更新 vm._props，也就是子组件的 props，这个就是子组件 props 的更新过程。

## 子组件重新渲染
其实子组件的重新渲染有 2 种情况，一个是 prop 值被修改，另一个是对象类型的 prop 内部属性的变化。

先来看一下 prop 值被修改的情况，当执行 props[key] = validateProp(key, propOptions, propsData, vm) 更新子组件 prop 的时候，会触发 prop 的 setter 过程，只要在渲染子组件的时候访问过这个 prop 值，那么根据响应式原理，就会触发子组件的重新渲染。

再来看一下当对象类型的 prop 的内部属性发生变化的时候，这个时候其实并没有触发子组件 prop 的更新。但是在子组件的渲染过程中，访问过这个对象 prop，所以这个对象 prop 在触发 getter 的时候会把子组件的 render watcher 收集到依赖中，然后当我们在父组件更新这个对象 prop 的某个属性的时候，会触发 setter 过程，也就会通知子组件 render watcher 的 update，进而触发子组件的重新渲染。

以上就是当父组件 props 更新，触发子组件重新渲染的 2 种情况。

## toggleObserving
最后我们在来聊一下 toggleObserving，它的定义在 src/core/observer/index.js 中
```
export let shouldObserve: boolean = true

export function toggleObserving (value: boolean) {
  shouldObserve = value
}
```
它在当前模块中定义了 shouldObserve 变量，用来控制在 observe 的过程中是否需要把当前值变成一个 Observer 对象。

在 updateChildComponent 过程中：
```
// update props
if (propsData && vm.$options.props) {
  toggleObserving(false)
  const props = vm._props
  const propKeys = vm.$options._propKeys || []
  for (let i = 0; i < propKeys.length; i++) {
    const key = propKeys[i]
    const propOptions: any = vm.$options.props // wtf flow?
    props[key] = validateProp(key, propOptions, propsData, vm)
  }
  toggleObserving(true)
  // keep a copy of raw propsData
  vm.$options.propsData = propsData
}
```
其实和 initProps 的逻辑一样，不需要对引用类型 props 递归做响应式处理，所以也需要 toggleObserving(false)

# 响应式原理

![alt](../../imgs/reactive.png)