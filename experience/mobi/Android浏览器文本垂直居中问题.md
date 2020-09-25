# Android 浏览器文本垂直居中问题

情景描述：在开发中，我们常使用 line-height 属性来实现文本的垂直居中，但是在安卓浏览器渲染中有一个常见的问题，渲染出来的效果并不是文字垂直居中，而是会偏上一些。

## 解放方案：

```
<div class="text">我要垂直居中</div>

.text{
  height:40px;
  display:flex;
  align-items:center;
  line-height:normal;
}

```
