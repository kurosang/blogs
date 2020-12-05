# webpack 高级概念

### 一、Tree Shaking

> tree shaking 是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块系统中的静态结构特性，例如 import 和 export。

通俗一点说，Tree Shaking 翻译中文就是摇树，将没用到的资源摇掉（剔除掉，不打包到 js 里去）。常见的，比如一个工具类 js 导出很多工具函数，我们只 import { a } from 'util.js'，那么 util.js 里面的其他工具函数就不会被打包进去。

`注意： Tree Shaking 只支持 ES Module 模式的引入（因为这是静态引入），require 不支持（动态）。`

然而，在实际工作中，我们常常会引入一些作用全局的代码。这些代码会被 webpack 检测为未引用代码(dead-code)，从而进行 tree shaking，这显然是不行的，这样的情况我们称为`副作用`。

解决方法：

有一个特殊的属性 `sideEffects`，就是为此而存在的。它有三个可能的值：

```
// 所有文件都有副作用，全都不可 tree-shaking
{
 "sideEffects": true
}
// 没有文件有副作用，全都可以 tree-shaking
{
 "sideEffects": false
}
// 只有这些文件有副作用，所有其他文件都可以 tree-shaking，但会保留这些文件
{
 "sideEffects": [
  "./src/file1.js",
  "./src/file2.js"
 ]
}

```

dev 环境开了 tree shaking 也不会去掉多余的代码，因为会对 sourcemap 有影响，影响我们的调试开发。

prod 正式环境默认开启 tree shaking，不需要手动设置 optimization.usedExports。

### 二、代码分割（code splitting）

假设场景：

// index.js

```
import _ from 'lodash'

console.log(_.join(['a', 'b', 'c'], '**'))
// 此处省略10万行业务
console.log(_.join(['a', 'b', 'c'], '**'))
```

假设 lodash 大小为 1MB,index.js 业务代码大小为 1MB，打包之后就 2MB

出现问题：

- 打包文件很大，加载时间长
- 修改业务逻辑之后重新访问页面，又要加载 2MB 的内容

解决方案：`代码分割`

把 main.js 拆成 lodash.js(1MB),main.js(1MB)，当页面业务逻辑发生变化时，只要加载 main.js 即可（1MB）.

webpack 开启方法：

```
//webpack.config
...
optimization: {
    splitChunks: {
      chunks: "all",
    },
  },
...
```

总结：

- 代码分割其实和 webpack 无关，只是 webpack 做代码分割很容易
- 两种代码分割：
  - 同步代码：顶部 import（ES Module）进来的，去 webpack.config 配置。
  - 异步代码：通过 import 方法异步加载的资源或模块，无需配置，即会放到新文件中。

### splitChunksPlugin

webpack 内部实现代码分割其实就是用了 `splitChunksPlugin` 这个插件。

实际上，splitChunks 有一份默认配置,当我们 splitChunks: {}为空时，就是读取默认

```
// 默认
splitChunks: {
    chunks: "async",
    minSize: 30000, // 30kb
    minChunks: 1,
    maxAsyncRequests: 5,
    maxInitialRequests: 3,
    automaticNameDelimiter: '~',
    name: true,
    cacheGroups: {
      vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          filename: 'vendors.js' // 如果想组的所有东西都打包在一个js，就设置这个
      },
      default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
      }
    }
}
```

chunks 和 cacheGroups 一般配合使用

chunks：async 只对异步代码生效，设置为 all，就可以打包同步代码，但还要在 cacheGroups.vendors 设置，

cacheGroups（缓存组）: 就是一个分组，vendors 就是检测引入的模块是不是 node_modules 的，如果是，就放进这个组，文件名前加组名。

cacheGroups.priority 数值越大，优先级越高。当一个文件符合两个组时，按优先级来打包到对应的组

cacheGroups.reuseExistingChunk 如果符合代码分割的代码之前已经被打包到某个文件里，设置为 true 就不会重复打包，直接使用之前的

minChunks：至少被引入了多少次，才代码分割。（就是打包之后的 chunk 里面有引入到的，符合这个次数才代码分割）

maxAsyncRequests：同时加载的模块数，最多的数量，超过就不会代码分割。

maxInitialRequests：入口文件做代码分割，最多的数量，超过就不会代码分割。

automaticNameDelimiter：组和文件之间的连接符

### 魔法注释

`Magic Comment`（魔法注释）：设置异步加载模块的名字，webpack 就会帮我们做代码分割

在安卓 web 后台项目中，`src/routes/dynamic.ts`我们就有用到魔法注释进行代码分割。

```
  {
    path: "/gamesManage",
    name: "games_menu",
    meta: {
      hidden: true,
      title: "游戏管理",
      icon: "icon-iconfontyouxihudong",
      keepAlive: false
    },
    component: () =>
      import(
        /* webpackChunkName: "AndroidGames" */ "@/views/GameManage/index.vue"
      )
  },
```

通过魔法注释，我们就把游戏管理相关的代码分割成一个单独的 js 文件，当我们点击游戏管理菜单时，才会加载游戏管理相关的 js。

### 三、lazy loading 懒加载和 chunk

懒加载或者按需加载，是一种很好的优化网页或应用的方式。这种方式实际上是先把你的代码在一些逻辑断点处分离开，然后在一些代码块中完成某些操作后，立即引用或即将引用另外一些新的代码块。这样加快了应用的初始加载速度，减轻了它的总体体积，因为某些代码块可能永远不会被加载。

懒加载不是 webpack 有的概念，而是 ES 里面的概念。webpack 只是可以识别 import 这种语法，对它进行一个代码分割。

代码分割出的每个 js 就是一个 chunk。

### 四、css 代码分割

一般 css 会打包到 js 文件里面，即 css in js。

MiniCssExtractPlugin：该插件将 CSS 提取到单独的文件中。它为每个包含 CSS 的 JS 文件创建一个 CSS 文件。（一般用于生产，因为 dev 的热更新支持不太好）

使用方法：

- 修改 prod 的 webpack 配置
- 修改 package.json 的 sideEffects，treeskaing 关掉 css 文件

如果用于生产环境，还需要安装插件 css-minimizer-webpack-plugin 进行 css 合并和压缩。

### 五、webpack 与浏览器缓存（caching）

在生产环境中，假设我们打包之后的 js 叫 main.js，当用户第一次访问之后就会缓存。当我们改了业务逻辑，重新打包发布之后，用户第二次刷新，浏览器获取的还是之前的 js，因为名字相同，浏览器会有本地缓存。因此 output 文件名要加上哈希值。

`[contenthash] `这个占位符，只有打包的内容有改动，哈希才会改变。

老版本 webpack 可能每次打包都会变哈希值，即使没有改动内容。需要做额外配置。

```
optimization: {
    runtimeChunk: {
      name: 'runtime',
    },
}
```

manifest：业务逻辑和库之间的关联，关联的代码，内置的代码。它存在于 main.js，也存在于 vendor.js 里面。

runtimeChunk 就是抽离 manifest 部分的代码。

### 六、shimming 垫片（全局变量）

webpack 编译器(compiler)能够识别遵循 ES2015 模块语法、CommonJS 或 AMD 规范编写的模块。然而，一些第三方的库(library)可能会引用一些全局依赖（例如 jQuery 中的 \$）。这些库也可能创建一些需要被导出的全局变量。这些“不符合规范的模块”就是 shimming 发挥作用的地方。

例子：

webpack 打包，默认每个模块是独立的，即每个 js 文件是独立的，并且文件内的引入的资源或者变量只能服务于该模块（文件），其他模块使用就会报错，如果其他模块也想用，只能在其他模块也引入。

webpack 自带的插件`webpack.ProvidePlugin`，可以定义

```
const webpack = require('webpack')
...
plugins: [
    ...,
    new webpack.ProvidePlugin({
      $: 'jquery',
      _join: ['lodash', 'join']
    }),
  ],
```

上面代码是打包时发现如果有 js 文件里有\$符号，就会自动帮你引入 jquery.

一般可以理解为这是一个垫片，通过配置它，解决一些库的问题，比如里面用了\$，但没有引入 jq。

`注意：webpack官方不推荐使用全局的东西！因为在 webpack 背后的整个概念是让前端开发更加模块化。也就是说，需要编写具有良好的封闭性(well contained)、彼此隔离的模块，以及不要依赖于那些隐含的依赖模块（例如，全局变量）。请只在必要的时候才使用这特性。`
