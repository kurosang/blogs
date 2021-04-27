# js 实现黏贴

### 一、需求

好好研究一下黏贴相关的东西，从零实现 **纯文本黏贴**，**图片黏贴自动上传服务器**，**图文混合黏贴**等

本文涉及知识点：js 如何获取剪切板中的图片， 剪贴板 ClipboardData ，DataTransferItem，解析 XML 文本，各种复制来源分析等

### 二、实现所需要的基础知识介绍

#### 获取剪切板中的图片 - paste 事件

```
document.addEventListener('paste', function (event) {
    var items = event.clipboardData && event.clipboardData.items;
    ...
});
```

#### ClipboardData

clipboardData 属性是 ClipboardEvent 事件对象的一个属性，其本质上就是一个 DataTransfer 对象。

剪贴板中的数据存储在 clipboardData 对象中。这个对象有三个方法：getData()、setData()和 clearData ()

- getData()
  getData()方法用于从剪贴板中取得数据，它接受一个参数，即要取得的数据的格式。在 IE 中，有两种数据格式："text" 和 "URL"。在其他浏览器中，这个参数是一种 MIME 类型；不过，可以用"text"代表

- setData()
  setData()方法的第一个参数也是数据类型，第二个参数是要放在剪贴板中的文本。

- clearData()
  clearData()方法用于从剪贴板中删除数据，它接受一个参数，即要取得的数据的格式。

本文主要使用 getData 这个 api

[更多](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent/clipboardData)

#### DataTransferItemList 和 DataTransferItem

DataTransfer.items 属性，items 属性是一个 DataTransferItemList 对象（一个伪数组对象），每个元素都是 DataTransferItem 对象。

**DataTransferItem**
该对象的属性有:

- type：拖拽项（或者剪切板中保存的数据项）的类型，为 MIME 值；
- kind：拖拽项的性质，可以是 string 或者 file；

该对象的常用方法有：

- getAsFile()：尝试将拖拽项数据包装成一个 File 对象返回，不是文件就返回 null（kind 对应 file）。
- getAsString(callback)：接受一个回调函数，将拖拽项数据解析为字符串后将该字符串传递给回调函数（kind 对应 string）；

#### tinymce 黏贴插入内容 api

通过阅读 tinymce 源码，目录`modules/tinymce/src/core/main/ts/api/Editor.ts`文件下

```
  /**
   * Inserts content at caret position.
   *
   * @method insertContent
   * @param {String} content Content to insert.
   * @param {Object} args Optional args to pass to insert call.
   */
  public insertContent (content: string, args?: any) {
    if (args) {
      content = extend({ content }, args);
    }

    this.execCommand('mceInsertContent', false, content);
  }
```

得知插入内容应使用 execCommand('mceInsertContent'）Api

### 实现思路

整体

```
// 监听编辑器黏贴 钩子
editor.on("paste", e => {
    // 获取剪贴板内容
    let cbd = e.clipboardData

    //...根据各种情况分析剪贴板内容并生成html，下面展开

    //插入html，完成黏贴
    editor.execCommand('mceInsertContent', false, html)
})
```

中间的处理逻辑是最复杂的，因为需要从黏贴场景出发（纯文本黏贴，图片黏贴自动上传服务器，图文混合黏贴）结合我们常使用的复制来源（word 文档，wps，记事本 txt，微信，浏览器网页）进行分析，得出以下情况。

1. 纯文本类型处理

```
    // 获取文本数据
    let text = cbd.getData("text/plain")
    // 创建p标签，并处理换行和空格
    let textNode = document.createElement("p")
    text = text.replace(/\n/g, "</br>")
    text = text.replace(/\s+/g, "&nbsp;")
    textNode.innerHTML = text
    // 插入到编辑器
    editor.execCommand("mceInsertContent", false, textNode.innerHTML)
```

主要是使用 getData Api 拿到剪切板中的文本内容，再替换换行和空格，然后执行插入。

2. 图片类型处理

```
for (let i = 0; i < cbd.items.length; i++) {
      // 图片处理
    if (item.kind === "file") {
        let blob = item.getAsFile()
        if (blob.size === 0) {
          return
        }
        // blob 就是从剪切板获得的文件 可以进行上传或其他操作
        // 插入占位图
        let id = new Date().getTime() + ((Math.random() * 1000).toFixed(0) + "")
        editor.execCommand(
          "mceInsertContent",
          false,
          `<img id="upimg-${id}" class="upload-image">`
        )
        htmlContent += `<img id="upimg-${id}" class="upload-image">`
        // 请求后端接口上传
        let fd = new FormData()
        fd.append("files", blob)
        api
          .upload_image(fd)
          .then(result => {
            editor.dom.doc.querySelector(`#upimg-${id}`).src = result.data.url
          })
          .catch(err => {
            console.log(err)
          })
      }
    }
```

我们拿到剪切板的 items，循环遍历，如果是 file 类型的，则先插入一个空的 img 标签，然后拿图片资源上传到服务器，当服务器成功返回 url 时，再替换 url。

3. html 类型处理

```
    // 获取
    let html = cbd.getData("text/html")
    // 清理多余标签
    html = cleanPastedHTML(html)
    // 解析
    const $doc = new DOMParser().parseFromString(html, "text/html")

    editor.execCommand(
      "mceInsertContent",
      false,
      $doc.querySelector("body").innerHTML
    )
```

这种类型主要是处理 word 文档相关的来源，第二步骤`清理多余标签`是因为在黏贴 word 内容的时候，发现其中有许多无用的代码，比如：

```
<p><!--[if !mso]>
<style>
v\:* {behavior:url(#default#VML);}
o\:* {behavior:url(#default#VML);}
w\:* {behavior:url(#default#VML);}
.shape {behavior:url(#default#VML);}
</style>
<![endif]--><!--[if gte mso 9]><xml>
 <o:OfficeDocumentSettings>
  <o:AllowPNG></o:AllowPNG>
 </o:OfficeDocumentSettings>
</xml><![endif]--><!--[if gte mso 9]><xml>
 <w:WordDocument>
```

因此需要用 js 进行过滤，然后第 3 步的解析 html 字符串生成 dom 也是比较少见的，可以留意下

### 仍存在的问题

图文混合黏贴：仍然有可能会出现某些图文空白的情况（单张不会），具体是图文混合（类型为 html）的时候，我们无法拿到 file 资源文件，所以只能解析 html，而这些 html。空白的原因是 html 上某些 img 标签上的 src 地址是`file://`的地址，因此谷歌浏览器是限制访问的，就会产生空白。

- 在 word 文档中
  如果我们从网页拷贝了图文黏贴到 word 文档，再从 word 文档里复制黏贴到编辑器，那么图片是不会丢失的，但图片的 src 是原来别人网站的 src。
  如果我们是用截图工具截了一张图黏贴到 word 文档，再从 word 文档复制一段图文混合黏贴到编辑器，那么就会出现空白图片的情况，因为截图在 word 文档里是本地资源，file 协议的。

- 在 wps 中
  wps 就比较简单了，只要是图片黏贴进 wps，都会转化为电脑的本地资源，因此，wps 复制图文混合的内容，图片必然显示空白。

**感谢阅读**

```
import api from "@/api/api"

export default function doPaste(e, editor, cb) {
  console.log("doPaste", e)
  let htmlContent = ""
  let cbd = e.clipboardData
  // 如果有复制到file图片资源，并且不是在word文档里（rtf）
  if (
    cbd.types.join("").includes("Files") &&
    !cbd.types.join("").includes("text/rtf")
  ) {
    // 循环clipboardData items
    for (let i = 0; i < cbd.items.length; i++) {
      let item = cbd.items[i]
      // 字符串
      // if (item.kind === "string") {
      //   // 纯文本
      //   if (item.type === "text/plain") {
      //     item.getAsString(function(text) {
      //       // text 是获取到的字符串
      //       let textNode = document.createElement("p")
      //       text = text.replace(/\n/g, "</br>")
      //       text = text.replace(/\s+/g, "&nbsp;")
      //       textNode.innerHTML = text
      //       editor.execCommand("mceInsertContent", false, textNode.innerHTML)
      //       htmlContent += textNode.innerHTML
      //     })
      //   }
      //   // html
      //   else if (item.type === "text/html") {
      //     item.getAsString(function(text) {
      //       console.log(text)
      //       editor.execCommand("mceInsertContent", false, text)
      //       htmlContent += text
      //     })
      //   }
      // }
      // 图片处理
      if (item.kind === "file") {
        let blob = item.getAsFile()
        if (blob.size === 0) {
          return
        }
        // blob 就是从剪切板获得的文件 可以进行上传或其他操作
        // 插入占位图
        let id = new Date().getTime() + ((Math.random() * 1000).toFixed(0) + "")
        editor.execCommand(
          "mceInsertContent",
          false,
          `<img id="upimg-${id}" class="upload-image">`
        )
        htmlContent += `<img id="upimg-${id}" class="upload-image">`
        // 请求后端接口上传
        let fd = new FormData()
        fd.append("files", blob)
        api
          .upload_image("article", "", fd)
          .then(result => {
            editor.dom.doc.querySelector(`#upimg-${id}`).src = result.data.url
          })
          .catch(err => {
            console.log(err)
          })
      }
    }
  }
  // 如果只有纯文本
  else if (cbd.types.join("") === "text/plain") {
    let text = cbd.getData("text/plain")
    let textNode = document.createElement("p")
    text = text.replace(/\n/g, "</br>")
    text = text.replace(/\s+/g, "&nbsp;")
    textNode.innerHTML = text
    editor.execCommand("mceInsertContent", false, textNode.innerHTML)
    htmlContent += textNode.innerHTML
  }
  // 剩下的情况就是各种文档黏贴，采用html解析的方式
  else {
    // 获取
    let html = cbd.getData("text/html")
    // 清理多余标签
    html = cleanPastedHTML(html)
    // 解析
    const $doc = new DOMParser().parseFromString(html, "text/html")

    let imgNodes = $doc.querySelectorAll("img")
    imgNodes.forEach(item => {
      if (item.alt.includes("http")) {
        item.src = item.alt
      }
    })

    editor.execCommand(
      "mceInsertContent",
      false,
      $doc.querySelector("body").innerHTML
    )
    htmlContent += $doc.querySelector("body").innerHTML
  }

  setTimeout(() => {
    cb(htmlContent)
  }, 10)
}

function pasteHtml() {}
function pasteText() {}
function pasteImage() {}

function cleanPastedHTML(input) {
  // 1. remove line breaks / Mso classes
  let stringStripper = /(\n|\r| class=(")?Mso[a-zA-Z]+(")?)/g
  let output = input.replace(stringStripper, " ")
  // 2. strip Word generated HTML comments
  let commentSripper = new RegExp("<!--(.*?)-->", "g")
  output = output.replace(commentSripper, "")
  let tagStripper = new RegExp(
    "<(/)*(meta|link|span|\\?xml:|st1:|o:|font)(.*?)>",
    "gi"
  )
  // 3. remove tags leave content if any
  output = output.replace(tagStripper, "")
  // 4. Remove everything in between and including tags '<style(.)style(.)>'
  let badTags = ["style", "script", "applet", "embed", "noframes", "noscript"]

  for (let i = 0; i < badTags.length; i++) {
    tagStripper = new RegExp(
      "<" + badTags[i] + ".*?" + badTags[i] + "(.*?)>",
      "gi"
    )
    output = output.replace(tagStripper, "")
  }
  // 5. remove attributes ' style="..."'
  let badAttributes = ["style", "start"]
  for (let i = 0; i < badAttributes.length; i++) {
    let attributeStripper = new RegExp(
      " " + badAttributes[i] + '="(.*?)"',
      "gi"
    )
    output = output.replace(attributeStripper, "")
  }
  return output
}


```
