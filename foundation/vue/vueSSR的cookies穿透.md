### VueSSR 的 Cookies 穿透

### 背景

由于客户端的 http 请求首先达到 SSR 服务器，再由 SSR 服务器去后端服务器获取相应的接口数据。在客户端到 SSR 服务器的请求中，客户端是携带有 cookie 数据的。但是在 SSR 服务器请求后端接口的过程中，却是没有相应的 cookie 数据的。因此在 SSR 服务器进行接口请求的时候，我们需要手动拿到客户端的 cookie 传给后端服务器。

在 ssr 期间我们需要截取客户端的 cookie，保持用户会话唯一性。

### 网上的解决方案

网上大多数给的解决方案，主要是这三种：

#### 一、把 Cookies 注入到 state 中

思路：SSR 服务器获取 cookies 注入 render 上下文，然后，在`server-entry.js`中，将 cookies 注入到 store，这样我们就可以在 dispatch 调用 api 的时候接收根 store 的 cookies。

```
// actions.js
async ['getList']({commit, rootState: {cookies}}, config) {
    const { data: { data, code} } = await api.get('/api/article/list', config, cookies)
    // 其他代码省略
}
```

#### 二、把 Cookies 注入到 global 中

思路：将 cookies 注入到 ssr 的 context 里, 然后在请求 api 时读取, 再追加到 axios 的 header 里。

但是从Vue@2.3.0开始, 在 createBundleRenderer 方法的选项中, 添加了 runInNewContext 选项, 使用 runInNewContext: false，bundle 代码将与服务器进程在同一个 global 上下文中运行，所以我们不能再将 cookies 放在 global, 因为这会让所有用户共用同一个 cookies。

举个例子就是： 在多个用户同时请求时，如 A 先请求，先设置了 A 的 cookie，A 的请求还没处理完成，B 也来请求了，这样 B 的 cookie 会覆盖掉 A 的 cookie，并且后续 A 的请求就会被污染

#### 三、把 Cookies 注入到组件的 asyncData 方法

思路：将 Cookies 作为参数注入到组件的 asyncData 方法, 然后 dispatch 获取数据时，用传参数的方法把 cookies 传给 api。

在 entry-server.js 里, 将 cookies 作为参数传给 asyncData 方法

```
Promise.all(matchedComponents.map(({asyncData}) => asyncData && asyncData({
    store,
    route: router.currentRoute,
    cookies: context.cookies,
    isServer: true,
    isClient: false
}))).then(() => {
    context.state = store.state
    context.isProd = process.env.NODE_ENV === 'production'
    resolve(app)
}).catch(reject)
```

搜一圈下来发现没有适合自己这个项目的方案：

方案一这种写法虽然可以做到，但是意味着每个接口都需要手动设置 cookie，而我们通常习惯 cookie 是放在拦截器处统一添加的，这样写麻烦而且不太优雅。

方案二不适用，过于影响性能。

方案三这种 asyncData 的一个解决方案，其实也跟方案一差不多，手动传 cookies 给 api，并且我这个 ssr 项目不用 asyncData，所以也不适用。

### 最终解决方案

在 github 上看到一个想法 💡，感觉挺不错，最后自己动手改造了一下。

思路：像创建 app、router、store 一样去创建 axios。

在`http.ts`中，导出创建 axios 实例的方法

```
import axios from "axios"
export default function createAxios(cookies?) {
  const service = axios.create({
    timeout: 30000,
    headers: {
      "Content-Type": "application/json"
    }
  })

  //省略...

  // 添加请求拦截器
  service.interceptors.request.use(
    config => {

      // 在此处加上token
      config.headers["Token"] = cookies.access_token

      return config
    },
    error => {
      return Promise.reject(error)
    }
  )

   //省略...

  return service
}

```

在`api.ts`中，导出创建 api 实例的方法

```
import createAxios from "./http"

export default function createApi(cookies?) {
  const service = createAxios(cookies)
  service.defaults.baseURL = apiBaseUrl

  return {
    service,
    auth: {
      qqLogin(data: { code: string }) {
        return service.post(`/login/qq?view=web`, data)
      }
    },
  }
}
```

在`entry-server.ts`中

```
import { createApp } from "./main"

export default context => {

  return new Promise((resolve, reject) => {
    // 拦截context上下文cookies
    // 然后在创建vue实例的时候作为参数传过去
    const { app, router, store } = createApp({
      access_token: context.cookies.get("access_token")
    })

    router.push(context.url).catch(() => {})

    router.onReady(() => {
      context.rendered = () => {
        context.state = store.state
      }
      resolve(app)
    }, reject)
  })
}
```

### 结论

原本在 SSR 服务器上下文中只有一个 axios 实例，但我们使用工厂模式给每个用户创建与它自己绑定的 axios 实例，这样既保持了原有的 api 封装，又避免了 cookies 的全局污染，该思路是可行的。
