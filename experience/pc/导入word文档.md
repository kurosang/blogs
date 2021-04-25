# 导入 word 文档

### 一、需求调研

为了解决编辑器图文混合黏贴时图片丢失的问题，现在尝试一种新的方式，就是前端导入 word 文档，然后识别到 Word 文档中的图片，自动上传到文件资源服务器，并生成 html 插入到编辑器。

这篇文章目的是看看纯前端能做到什么程度。

### 二、关于 Mammoth.js

#### 简介

Mammoth 旨在转换 .docx 文档（例如由 Microsoft Word 创建的文档），并将其转换为 HTML。 Mammoth 的目标是通过使用文档中的语义信息并忽略其他细节来生成简单干净的 HTML。比如，Mammoth 会将应用标题 1 样式的任何段落转换为 h1 元素，而不是尝试完全复制标题的样式（字体，文本大小，颜色等）。

由于 .docx 使用的结构与 HTML 的结构之间存在很大的不匹配，这意味着对于较复杂的文档而言，这种转换不太可能是完美的。但如果你仅使用样式在语义上标记文档，则 Mammoth 能实现较好的转换效果。

想要了解更多，阅读[Mammoth.js 仓库](https://github.com/mwilliamson/mammoth.js)。

### 三、实现过程

- 1.我们先在页面上放一个上传按钮

```
<input type="file" @change="uploadFile" />
```

- 2.uploadFile 函数：主要读取文件转化为 arrayBuffer，再将 arrayBuffer 传递给生成 html 的函数，最后返回 html

```
async uploadFile(event) {
    const arrayBuffer = await this.readFileInputEventAsArrayBuffer(event)
    const data = await this.convertToHtml(arrayBuffer)
    this.vhtml = data.value
    this.$emit("updateIHTML", data.value)
}
```

- 3.读取文件

```
readFileInputEventAsArrayBuffer(event) {
    return new Promise((resolve, reject) => {
      const file = event.target.files[0]

      const reader = new FileReader()

      reader.onload = function(loadEvent: Event) {
        const arrayBuffer = loadEvent.target["result"]
        resolve(arrayBuffer)
      }

      reader.onerror = function(event) {
        alert("Failed to read file!\n\n" + reader.error)
        reader.abort()
        reject(event)
      }

      reader.readAsArrayBuffer(file)
    })
  }
```

- 4.生成 html

```
convertToHtml(arrayBuffer) {
    return mammoth.convertToHtml({ arrayBuffer })
}
```

通过上面的代码，可以成功地读取 word 文档并转化为 html。但是，会发现，图片是显示不出来。接下来：

**图片的处理**

在`mammoth.convertToHtml({ arrayBuffer })`这里，加上我们一个传入的配置，变成`mammoth.convertToHtml({ arrayBuffer }, this.mammothOptions)`。

具体配置：

```
 mammothOptions = {
    convertImage: mammoth.images.imgElement(image => {
      return image.read("base64").then(async imageBuffer => {
        const result = await this.uploadBase64Image(
          imageBuffer,
          image.contentType
        )
        return {
          src: result.data.url // 获取图片线上的URL地址
        }
      })
    })
  }

  /** 上传图片 */
  async uploadBase64Image(base64Image, mime) {
    // base64转Blob
    function _base64ToBlob(base64, mimeType) {
      let bytes = window.atob(base64)
      let ab = new ArrayBuffer(bytes.length)
      let ia = new Uint8Array(ab)
      for (let i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i)
      }
      return new Blob([ia], { type: mimeType })
    }

    const formData = new FormData()
    formData.append("files", _base64ToBlob(base64Image, mime))

    return await this.$api.upload_image("article", "", formData)
  }

```

在配置中，通过`mammoth.images`拦截文档里的图片，然后用后端给的上传图片的接口返回 src 再赋过去。

**关于样式**

可以在`mammothOptions.styleMap`中配置

### 四、demo 组件代码

```
<template>
  <div class="Convert">
    <input type="file" @change="uploadFile" />
    <div v-html="vhtml"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator"
import mammoth from "mammoth"

@Component({
  name: "Convert",
  components: {}
})
export default class Convert extends Vue {
  vhtml = ""

  /** 文件上传入口 */
  async uploadFile(event) {
    const arrayBuffer = await this.readFileInputEventAsArrayBuffer(event)
    const data = await this.convertToHtml(arrayBuffer)
    this.vhtml = data.value
    this.$emit("updateIHTML", data.value)
  }

  mammothOptions = {
    styleMap: [
      "p[style-name='Title'] => h1.title",
      "p[style-name='Subtitle'] => h4.subtitle",
      "p[style-name='heading 5'] => h5",
      "p[style-name='Heading 5'] => h5",
      "p[style-name='heading 6'] => h6",
      "p[style-name='Heading 6'] => h6",
      "p[style-name='Quote'] => blockquote.quote",
      "p[style-name='Intense Quote'] => blockquote.intense-quote > strong",
      "i => em",
      "u => ins",
      "strike => del"
    ],
    convertImage: mammoth.images.imgElement(image => {
      return image.read("base64").then(async imageBuffer => {
        const result = await this.uploadBase64Image(
          imageBuffer,
          image.contentType
        )
        return {
          src: result.data.url // 获取图片线上的URL地址
        }
      })
    })
  }

  /** 上传图片 */
  async uploadBase64Image(base64Image, mime) {
    // base64转Blob
    function _base64ToBlob(base64, mimeType) {
      let bytes = window.atob(base64)
      let ab = new ArrayBuffer(bytes.length)
      let ia = new Uint8Array(ab)
      for (let i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i)
      }
      return new Blob([ia], { type: mimeType })
    }

    const formData = new FormData()
    formData.append("files", _base64ToBlob(base64Image, mime))

    return await this.$api.upload_image("article", "", formData)
  }

  /** 读取文件 */
  readFileInputEventAsArrayBuffer(event) {
    return new Promise((resolve, reject) => {
      const file = event.target.files[0]

      const reader = new FileReader()

      reader.onload = function(loadEvent: Event) {
        const arrayBuffer = loadEvent.target["result"]
        resolve(arrayBuffer)
      }

      reader.onerror = function(event) {
        alert("Failed to read file!\n\n" + reader.error)
        reader.abort() //
        reject(event)
      }

      reader.readAsArrayBuffer(file)
    })
  }

  /** 生成HTML */
  convertToHtml(arrayBuffer) {
    return mammoth.convertToHtml({ arrayBuffer }, this.mammothOptions)
  }
}
</script>

<style scoped></style>

```

### 五、结论

- Mammoth.js 除了支持转化为 html 文档，还支持 Markdown 文档；
- 支持多图文混合导入并转化为 html；
- 缺点：无法提取到原有 word 文档的复杂样式；
- 支持统一拦截 html 标签并加上样式；
