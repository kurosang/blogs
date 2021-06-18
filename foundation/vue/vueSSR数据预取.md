### vue 服务端渲染中数据预取和一些注意事项

### 背景

分享最近做 SSR 项目其中遇到的一些关于数据预取，生命周期，代码编写的一些注意事项。

### 编写通用代码

> 服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在服务器和客户端上运行。

我们在写 SPA 应用时，大部分场景都是 CSR（Client Sider Render）客户端渲染，所以很少会考虑到这个问题，因为 CSR 只会在浏览器进行；而 SSR（Server Sider Render）服务端渲染是需要 node 服务器和浏览器配合完成的。

大概有以下几个方面需要注意：

- 1.访问特定平台(Platform-Specific) API

  > 通用代码不可接受特定平台的 API，因此如果你的代码中，直接使用了像 window 或 document，这种仅浏览器可用的全局变量，则会在 Node.js 中执行时抛出错误，反之也是如此。

- 2.自定义指令

  > 大多数自定义指令直接操作 DOM，因此会在服务器端渲染 (SSR) 过程中导致错误。

- 3.组件生命周期钩子函数（下部分讲）

### 组件生命周期钩子函数

除了我们平常所熟悉的那几个钩子函数

```
beforeCreate
created
beforeMount
mounted
beforeUpdate
updated
beforeDestroy
destroyed
```

在 vue2.6+中，增加了一个`serverPrefetch`的钩子函数。

专门用于 在服务器端渲染期间获取异步数据。它将获取的数据存储到全局存储中并返回一个 Promise。服务器渲染器将等待这个钩子，直到 Promise 被解析。这个钩子可以通过 访问组件实例 this。

它的执行时间是在 created 钩子函数和 beforeMount 之间。

这里我们回到上一部分`编写通用代码`，为什么会扯到生命周期，因为`客户端`和`服务端`两边所执行的生命周期并不完全一样。

- 服务端

```
beforeCreate => created => serverPrefetch
```

- 客户端

```
beforeCreate => created => beforeMount => mounted =>
beforeDestroy => destroyed
```

所以编写代码的时候，应该避免在 beforeCreate 和 created 生命周期时产生全局副作用的代码，例如在其中使用 setInterval 设置 timer。在纯客户端 (client-side only) 的代码中，我们可以设置一个 timer，然后在 beforeDestroy 或 destroyed 生命周期时将其销毁。但是，由于在 SSR 期间并不会调用销毁钩子函数，所以 timer 将永远保留下来。为了避免这种情况，请将副作用代码移动到 beforeMount 或 mounted 生命周期中。

### 数据预取

> 在 Vue 的服务端渲染中做数据预取的方式大概可以总结为两种，一种是以 nuxt/ream 为代表的 asyncData 方案，另一种是 Vue 原生提供的 serverPrefetch 组件选项。

（这里吐槽一下，在《vue ssr 指南》里，中文文档是用 asyncData 方案的例子，而英语日语等外语文档是使用 serverPrefetch 方案的）

其中，两种方案都各有优缺点：

- asyncData 方案：

  - 不能访问 this
  - 只能用于路由组件 (或 page 组件)

- serverPrefetch 方案：
  - 只运行于服务端，客户端需要另外编写数据获取逻辑，并避免数据的重复获取
  - 只能预取 store 数据，无法将数据暴露到组件级别的渲染环境并发送到客户端

经过对项目的考量，我最终选取了 serverPrefetch 方案，因为相对复杂的项目而言，如果只能在路由组件预取数据，那么大量的`数据`和`业务处理逻辑`将会混在路由组件里，不利于代码的分割和后期的维护。

#### serverPrefetch 用法

贴一个例子

```
<template>
  <div>
     <div v-for="item in gameList" :key="item._id">{{ item.name }}</div>
  </div>
</template>

<script>
export default {
  computed: {
    gameList () {
      return this.$store.state.gameList
    }
  },

  // Server-side only
  serverPrefetch () {
    return this.fetchList()
  },

  // Client-side only
  mounted () {
    if (!this.$store.state.gameList) {
      this.fetchList()
    }
  },

  methods: {
    fetchList () {
      return this.$store.dispatch('GET_LIST')
    }
  }
}
</script>
```

简单说明一下，就是 serverPrefetch 里请求数据，拿到`游戏列表`存到`store`里面，然后`模板template`根据`游戏列表`循环输出`游戏名称`，然后在 mounted 里判断是否已经存在 游戏列表，如果已存在，则不去获取游戏列表（这里主要是为了兼容纯客户端渲染的情况）

#### 开发中遇到的一个大坑 - 客户端 和 服务端 数据不一致（渲染报错）

发生这种情况的，很大可能就是 template 依赖的变量不是 store 全局变量里的，而是组件级别 data，就会出现这种情况。

比如：

```
<template>
  <div>
     <div v-for="item in list" :key="item._id">{{ item.name }}</div>
  </div>
</template>

<script>
export default {
  data:{
    list:[]
  },
  computed: {
    gameList () {
      return this.$store.state.gameList
    }
  },

  // Server-side only
  serverPrefetch () {
    return this.fetchList()
  },

  // Client-side only
  mounted () {
    if (!this.$store.state.gameList) {
      this.fetchList()
    }
  },

  methods: {
    fetchList () {
      return this.$store.dispatch('GET_LIST').then(()=>{
        this.list = this.$store.state.gameList
      })
    }
  }
}
</script>
```

以上例子运行实际情况就会是 客户端没渲染出列表，而 服务端渲染出列表。

原因是，回顾上文，serverPrefetch 的缺点：无法将数据暴露到组件级别的渲染环境并发送到客户端。

也就是说，服务端渲染的时候 fetchList 里请求的 then 语句（赋值操作），服务端是知道的，所以它渲染出来了，但由于无法发送到客户端，客户端渲染的时候 list 字段依然为空，因此，就会导致两端数据不一致。

#### 解决方案

- 首先，将 then 语句的操作抽出来，写成一个函数
- 然后，在 beforeMount 钩子里执行一次，再挂载前，保持两端数据一致

```
<template>
  <div>
     <div v-for="item in list" :key="item._id">{{ item.name }}</div>
  </div>
</template>

<script>
export default {
  data:{
    list:[]
  },
  computed: {
    gameList () {
      return this.$store.state.gameList
    }
  },

  // Server-side only
  serverPrefetch () {
    return this.fetchList().then(this.getList)
  },

  // Client-side only
  beforeMount () {
    if (this.$store.state.gameList) {
      this.getList()
    }
  },

  // Client-side only
  mounted () {
    if (!this.$store.state.gameList) {
      this.fetchList().then(this.getList)
    }
  },

  methods: {
    fetchList () {
      return this.$store.dispatch('GET_LIST')
    },
    getList () {
      this.list = this.$store.state.gameList
    }
  }
}
</script>

```

#### 最后

通用数据预取可能是服务器渲染应用程序中最复杂的问题了，感谢阅读。
