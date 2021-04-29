# webpack 打包拦截图片资源加上 CDN

### 现状

一般我们在开发的时候，是这样引入一个图片的

```
<img src="@/assets/images/logo.png">
```

打包之后：

```
<img src="/static/img/logo.ef959158.png">
```

`vue-cli`会将我们的图片打包成，以绝对路径的形式，放在`static/img`目录下。

### 为什么需要加上 CDN

我们部署 SSR 项目时，是需要我们启动一个服务器的，它不像 CSR 一样，只是一些静态的文件。将耗费资源的图片服务分离出来，可以提高服务器的性能和稳定性，不需要主服务器来处理这些请求，减小服务器压力。

还有一些 CDN 的好处，这里就不展开谈了。

### 做法

**1.原始做法**

直接在开发的时候就写上最终 cdn 的地址

```
<img src="https://cdn.com/xxx/images/logo.png">
```

这种方法有很多缺点，比如：

1）当我们在开发时，设计师改了图片，我们需要重新把图片拉到服务器上

2）整个开发页面会充斥着很多`https://cdn.com/xxx`，cdn 变了改起来也很麻烦;

**2.通过修改 webpack 的配置，拦截图片资源**

打开`vue.config.js`文件

```
module.exports = {
  // ....
  chainWebpack: config => {
    if (isProd) {
      // 拦截代码中的image，生产环境加上cdn域名
      config.module
        .rule("images")
        .test(/\.(jpe?g|png|gif)$/i)
        .use("url-loader")
        .loader("url-loader")
        .options({
          limit: 1000,
          publicPath: CDN_URL,
          outputPath: "static/img",
          name: "[name].[contenthash:5].[ext]"
        })
        .end()
    }
  }
}
```

- 首先通过`isProd`判断如果是生产环境，才进行拦截
- `options.publicPath`就是配置 cdn 的地方
- `options.limit`可以限制图片的大小，当超过这个数值时，我们才进行添加 cdn 并打包输出到`static/img`目录下，否则就转为 base64；
