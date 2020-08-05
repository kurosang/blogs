# 移动端overflow hidden不起作用

情景描述：弹窗里面有滚动条，html和body设置了overflow hidden，发现底层还是可以滑动。

## 解放方案：

```
scrollTopVal: number = 0
  @Watch(`dialogShow`)
  onChangeDialogShow(change: any) {
    console.log(`dialogShow change:${change}`)
    if (change) {
      this.scrollTopVal =
        window.pageYOffset ||
        document.documentElement.scrollTop ||
        document.body.scrollTop ||
        0
      document.documentElement.style.overflow = "hidden"
      document.body.style.overflow = "hidden"
      document.documentElement.style.position = "fixed"
      document.body.style.position = "fixed"
      document.documentElement.style.top = -1 * this.scrollTopVal + "px"
    } else {
      document.documentElement.style.overflow = ""
      document.body.style.overflow = ""
      document.documentElement.style.position = ""
      document.body.style.position = ""
      window.scrollTo(0, this.scrollTopVal)
    }
  }
```