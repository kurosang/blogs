# 移动端 web 页面截图分享

### 一、需求

光环助手安卓 app 徽章系统，需要实现一个图片分享，根据【我的徽章列表】生成一个长截图。

### 二、实现思路

- 进入分享页，首先从接口获取数据，渲染页面。
- 通过`html2canvas.js`，将 Html DOM 节点转换为 canvas。
- 通过`CanvasAPI`的`toDataURL`，将`canvas`转换为`Base64`的格式，并将它设置为`img src`属性值。
- 最后将 img 显示出来，并隐藏列表 dom 节点。

### 三、关于 html2canvas.js

#### 简介

- html2canvas 能够实现在用户浏览器端直接对整个或部分页面进行截屏。
- 这个 html2canvas 脚本将当页面渲染成一个 Canvas 图片，通过读取 DOM 并将不同的样式应用到这些元素上实现。（换句话说，实际上这个库并不是真的对页面进行截图，而是基于从 DOM 读取的元素及属性来一点点的绘制 canvas。 因此，它只能正确地呈现它理解的元素和属性，这意味着有许多 CSS 属性不起作用。）
- 它不需要来自服务器任何渲染，整张图片都是在客户端浏览器创建。

想要了解更多，阅读[html2canvas 官方文档](http://html2canvas.hertzen.com/)。

#### 使用

- 安装：yarn add html2canvas

- 调用：

```
import Html2canvas from "html2canvas"
Html2canvas(dom, {}).then(canvas => {
    this.url = canvas.toDataURL("image/png")
})
```

### 四、开发过程中遇到的问题及解决办法

**1.截图模糊**

在手机上保存图片后看到的图片比较模糊，这个是因为移动端像素密度计算导致的。

> 设备像素比 dpr（devicePixelRatio）是设备的物理像素分辨率与 CSS 像素分辨率的比值，该值也可以被解释为像素大小的比例：即一个 CSS 像素的大小相对于一个物理像素的大小的比值。

解决方法：使用 window.devicePixelRatio 获取 dpr，然后在 html2canvas 配置中加入

```
{
    scale: window.devicePixelRatio && window.devicePixelRatio > 1 ? window.devicePixelRatio : 1
}
```

**2.图片空白/模糊**

图片空白

原因是图片没有跨域，导致 js 是读不到图片信息。图片是放在 cdn 上，cdn 需要设置 cors 相关设置，也就是图片请求的响应头里需要设置 Access-Control-Allow-Origin: \*。

图片模糊

web 页面中引入图片有两种方式：1.一种是 CSS 的 background-image 引入。2.img 标签设置 src 引入。

得出结论是模糊是因为用了第一种方法，所以要全部改成 img 标签设置 src 引入，包括背景图。

**3.图片超出一屏幕高度，截图不全**

这个解决方法可以在 html2canvas 配置 截图长宽

```
{
    height: dom.scrollHeight,
    width: dom.offsetWidth
}
```

### 五、完整代码

```
<template>
  <div class="page-container">
    <div v-if="dataUrl" class="share-img-box">
      <img width="100%" height="100%" :src="dataUrl" alt="" />
    </div>
    <div v-else class="main" id="main-share">
      //
    </div>
  </div>
</template>

<script>
import { Component, Prop, Vue, Watch } from "vue-property-decorator"
import Html2canvas from "html2canvas"

@Component({
  name: "Share"
})
export default class Share extends Vue {
  mounted() {
    this.getUserBadges()
  }

  async getUserBadges() {
    // 获取徽章列表...
    this.$nextTick().then(() => {
        this.createShareImage()
    })
  }

  createShareImage() {
    const dom = document.querySelector("#main-share")
    const height = dom.scrollHeight
    const width = dom.offsetWidth
    Html2canvas(dom, {
      useCORS: true,
      scale: window.devicePixelRatio && window.devicePixelRatio > 1 ? window.devicePixelRatio : 1,
      logging: false,
      height,
      width
    }).then(canvas => {
      this.dataUrl = canvas.toDataURL("image/png")
    })
  }
}
</script>

<style lang="scss" scoped></style>

```
