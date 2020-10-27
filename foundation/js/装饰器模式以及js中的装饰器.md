# 装饰器模式以及 js 中的装饰器

### 一、装饰者模式介绍

装饰器模式，又名装饰者模式。它的定义是“在不改变原对象的基础上，通过对其进行包装拓展，使原有对象可以满足用户的更复杂需求”。属于结构型模式的一种。通俗点说就是：在不改变原来代码的基础上，通过先保留原来函数，重新改写，在重写的代码中调用原来保留的函数。

使用装饰者模式的优点：**把对象核心职责和要装饰的功能分开了**。非侵入式的行为修改。

### 二、ES5 运用装饰器模式

使用场景：假设我们的初始需求是：每个业务中的按钮在点击后都弹出「您还未登录哦」的弹框。

伪代码实现：

```
//html
<button id='open'>点击打开</button>
<button id='close'>关闭弹框</button>

//js
// 点击打开按钮展示模态框
document.getElementById('open').addEventListener('click', function() {
    // 未点击则不创建modal实例，避免不必要的内存占用
  const modal = new Modal('您还未登录哦~')
  modal.style.display = 'block'
})

// 点击关闭按钮隐藏模态框
document.getElementById('close').addEventListener('click', function() {
  const modal = document.getElementById('modal')
  if(modal) {
      modal.style.display = 'none'
  }
})
```

发布上线后，过了几天。忽然有一天，产品经理找到你，说这个弹框提示还不够明显，我们应该在弹框被关闭后把按钮的文案改为“快去登录”，同时把按钮置灰。

正常听到这个消息，我们都会翻出之前的代码，找到了按钮的 click 监听函数，手动往里面添加了文案修改&按钮置灰逻辑。

但是这样做有很多坏处，比如：我需要了解这个 click 函数写了什么逻辑，我添加的新逻辑会不会对其造成影响。还有，当需要迭代多了之后，这个 click 函数是否变得无比庞大，维护成本巨高。

其实，我们绝大多数人都压根不想去关心它现有的业务逻辑是啥样的，只是想对它已有的功能做个拓展，只关心拓展出来的那部分新功能如何实现，对不对？于是便有了**装饰器模式**。

使用装饰器模式改造：

```
// 将展示Modal的逻辑单独封装
function openModal() {
    const modal = new Modal()
    modal.style.display = 'block'
}

// 按钮文案修改逻辑
function changeButtonText() {
    const btn = document.getElementById('open')
    btn.innerText = '快去登录'
}

// 按钮置灰逻辑
function disableButton() {
    const btn =  document.getElementById('open')
    btn.setAttribute("disabled", true)
}

// 新版本功能逻辑整合
function changeButtonStatus() {
    changeButtonText()
    disableButton()
}

document.getElementById('open').addEventListener('click', function() {
    openModal()
    changeButtonStatus()
})
```

上述代码：

- 1.将旧逻辑与新逻辑分离，把旧逻辑抽出去
- 2.编写新逻辑
- 3.然后把三个操作逐个添加 open 按钮的监听函数里

### 三、 ES7 语法 decorator 装饰器

@decorator 装饰器是 es7 更新的提案，使用的时候需要 babel 的支持。

在 ES7 中，我们可以像写 python 一样通过一个@语法糖轻松地给一个类装上装饰器：

```
// 装饰器函数，它的第一个参数是目标类
function classDecorator(target) {
    target.hasDecorator = true
  	return target
}

// 将装饰器“安装”到Button类上
@classDecorator
class Button {
    // Button类的相关逻辑
}

// 验证装饰器是否生效
console.log('Button 是否被装饰了：', Button.hasDecorator)
```

当我们给一个类添加装饰器时,target 就是被装饰的类本身。

也可以用同样的语法糖去装饰类里面的方法：

```
// 装饰器函数
function funcDecorator(target, name, descriptor) {
    let originalMethod = descriptor.value
    descriptor.value = function() {
    console.log('我是Func的装饰器逻辑')
    return originalMethod.apply(this, arguments)
  }
  return descriptor
}

class Button {
    @funcDecorator
    onClick() {
        console.log('我是Func的原有逻辑')
    }
}

// 验证装饰器是否生效
const button = new Button()
button.onClick()
```

当我们给一个类里面的方法添加装饰器时，

- 第一个参数 target 实际是`Button.prototype`，即类的原型对象。因为装饰器函数执行时，类的实例还未存在。
- 第二个参数 name，是我们修饰的目标属性属性名。
- 第三个参数 descriptor，它的真面目就是“属性描述对象”，也是我们使用频率最高的函数。`Object.defineProperty`方法应该都有接触过:

```
Object.defineProperty(obj, prop, descriptor)
```

此处的 descriptor 和装饰器函数里的 descriptor 是一个东西，点击了解[descriptor](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

### 四、 装饰器实践

**1.最熟悉的就是我们现在日常开发使用的 vue+typescript 就是用到了装饰器。**

```
<template>
  <div class="page-container"></div>
</template>

<script lang="ts" type="ts">
import { Component, Prop, Watch, Vue } from "vue-property-decorator"

@Component({
  name: "XXXX"
})
export default class XXXX extends Vue {

  @Prop
  ...

  @Watch("keyword")
  onKeyword(change: string) {
    //...
  }
}
</script>

<style scoped lang="scss">
</style>

```

**2.自己封装装饰器**

场景：我们在后台管理项目中，很多操作都是要弹出一个提示框，来提示操作者是否确认执行。代码如下：

```
submitConfirm() {
  this.$confirm("是否确认此操作？", "提示", {
      confirmButtonText: "确定",
      cancelButtonText: "取消",
      type: "warning"
    })
      .then(() => {
        // 业务逻辑
        api()
      })
      .catch(() => {})
}

submitConfirm2() {
  this.$confirm("是否确认此操作？", "提示", {
      confirmButtonText: "确定",
      cancelButtonText: "取消",
      type: "warning"
    })
      .then(() => {
        // 业务逻辑2
        api2()
      })
      .catch(() => {})
}
....
```

以上代码函数的意思就是先弹出提示框，当用户点击确认，则会走进 then 逻辑，如果点击取消，就会走进 catch 逻辑。

我们可以看到，其实我们真正的业务逻辑代码，可能就是发送一个请求然后刷新数据，但由于提示框的问题，使我们的代码写了很多重复的代码。

于是，我采用了装饰器对其进行改造，将提示框的逻辑抽出成一个装饰器，与函数本身处理的业务逻辑区分开。

代码如下：

```
// decorator.js 封装 装饰器工具库
// 装饰器函数
function confirmAction(message: string = "是否继续该操作?") {
  return function(target, name, descriptor) {
    let originalMethod = descriptor.value
    descriptor.value = function(...args) {
      MessageBox.confirm(message, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(originalMethod.bind(this, ...args))
        .catch(() => {})
    }

    return descriptor
  }
}
export { confirmAction }
```

```
// xxx.vue  业务

//引入
import { confirmAction } from "@/utils/decorator"

//调用
@confirmAction("是否继续该操作?")
submitConfirm() {
  // 业务逻辑
  api()
}

//调用
@confirmAction("是否继续该操作?")
submitConfirm2() {
  // 业务逻辑2
  api2()
}
```

我们将装饰器函数放在工具库里，当哪个.vue 页面需要用到提示确认框的时候再引入，整个项目的重复代码会减少许多，代码更多整洁。
