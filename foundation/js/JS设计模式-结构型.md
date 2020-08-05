# JS设计模式-结构型

### 这里主要说说：装饰器模式，适配器模式，代理模式。

***

## 1.装饰器模式
>装饰器模式，又名装饰者模式。它的定义是“在不改变原对象的基础上，通过对其进行包装拓展，使原有对象可以满足用户的更复杂需求”。

使用场景：**不想去关心它现有的业务逻辑是啥样的。对它已有的功能做个拓展，只关心拓展出来的那部分新功能如何实现**

例如：已有点击事件触发弹窗，现在需要增加新功能：点击出现弹窗并且**按钮置灰，按钮文案改变**。

为了不被已有的业务逻辑干扰，当务之急就是将旧逻辑与新逻辑分离，把旧逻辑抽出去：
```
// 将展示Modal的逻辑单独封装
function openModal() {
    const modal = new Modal()
    modal.style.display = 'block'
}
```
编写新逻辑：
```
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
```
然后把三个操作逐个添加open按钮的监听函数里：
```
document.getElementById('open').addEventListener('click', function() {
    openModal()
    changeButtonStatus()
})
```
如此一来，我们就实现了“只添加，不修改”的装饰器模式，使用changeButtonStatus的逻辑装饰了旧的按钮点击逻辑。以上是ES5中的实现，ES6中，我们可以以一种更加面向对象化的方式去写：
```
// 定义打开按钮
class OpenButton {
    // 点击后展示弹框（旧逻辑）
    onClick() {
        const modal = new Modal()
    	modal.style.display = 'block'
    }
}

// 定义按钮对应的装饰器
class Decorator {
    // 将按钮实例传入
    constructor(open_button) {
        this.open_button = open_button
    }
    
    onClick() {
        this.open_button.onClick()
        // “包装”了一层新逻辑
        this.changeButtonStatus()
    }
    
    changeButtonStatus() {
        this.changeButtonText()
        this.disableButton()
    }
    
    disableButton() {
        const btn =  document.getElementById('open')
        btn.setAttribute("disabled", true)
    }
    
    changeButtonText() {
        const btn = document.getElementById('open')
        btn.innerText = '快去登录'
    }
}

const openButton = new OpenButton()
const decorator = new Decorator(openButton)

document.getElementById('open').addEventListener('click', function() {
    // openButton.onClick()
    // 此处可以分别尝试两个实例的onClick方法，验证装饰器是否生效
    decorator.onClick()
})
```
大家这里需要特别关注一下 ES6 这个版本的实现，这里我们把按钮实例传给了 Decorator，以便于后续 Decorator 可以对它为所欲为进行逻辑的拓展。在 ES7 中，Decorator 作为一种语法被直接支持了，它的书写会变得更加简单，但背后的原理其实与此大同小异。

### ES7 中的装饰器

**Part1：函数传参&调用**

上一节我们使用 ES6 实现装饰器模式时曾经将按钮实例传给了 Decorator，以便于后续 Decorator 可以对它进行逻辑的拓展。这也正是装饰器的最最基本操作——定义装饰器函数，将被装饰者“交给”装饰器。这也正是装饰器语法糖首先帮我们做掉的工作 —— 函数传参&调用。

**类装饰器的参数**
当我们给一个类添加装饰器时：
```
function classDecorator(target) {
    target.hasDecorator = true
  	return target
}

// 将装饰器“安装”到Button类上
@classDecorator
class Button {
    // Button类的相关逻辑
}
```
此处的 target 就是被装饰的类本身。
**方法装饰器的参数**
而当我们给一个方法添加装饰器时：
```
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
```
此处的 target 变成了Button.prototype，即类的原型对象。这是因为 onClick 方法总是要依附其实例存在的，修饰 onClik 其实是修饰它的实例。但我们的装饰器函数执行的时候，Button 实例还并不存在。为了确保实例生成后可以顺利调用被装饰好的方法，装饰器只能去修饰 Button 类的原型对象。
**装饰器函数调用的时机**
装饰器函数执行的时候，Button 实例还并不存在。这是因为实例是在我们的代码运行时动态生成的，而装饰器函数则是在编译阶段就执行了。所以说装饰器函数真正能触及到的，就只有类这个层面上的对象。

**Part2：将“属性描述对象”交到你手里**
在编写类装饰器时，我们一般获取一个target参数就足够了。但在编写方法装饰器时，我们往往需要至少三个参数：
```
function funcDecorator(target, name, descriptor) {
    let originalMethod = descriptor.value
    descriptor.value = function() {
    console.log('我是Func的装饰器逻辑')
    return originalMethod.apply(this, arguments)
  }
  return descriptor
}
```
第一个参数的意义，前文已经解释过。第二个参 数name，是我们修饰的目标属性属性名，也没啥好讲的。关键就在这个 descriptor 身上，它也是我们使用频率最高的一个参数，它的真面目就是“属性描述对象”（attributes object）。这个名字大家可能不熟悉，但Object.defineProperty方法我想大家多少都用过，它的调用方式是这样的：
```
Object.defineProperty(obj, prop, descriptor)
```
此处的descriptor和装饰器函数里的 descriptor 是一个东西，它是 JavaScript 提供的一个内部数据结构、一个对象，专门用来描述对象的属性。它由各种各样的属性描述符组成，这些描述符又分为数据描述符和存取描述符：

    数据描述符：包括 value（存放属性值，默认为默认为 undefined）、writable（表示属性值是否可改变，默认为true）、enumerable（表示属性是否可枚举，默认为 true）、configurable（属性是否可配置，默认为true）。

    存取描述符：包括 get 方法（访问属性时调用的方法，默认为 undefined），set（设置属性时调用的方法，默认为 undefined ）

很明显，拿到了 descriptor，就相当于拿到了目标方法的控制权。通过修改 descriptor，我们就可以对目标方法的逻辑进行拓展了~

在上示例中，我们通过 descriptor 获取到了原函数的函数体（originalMethod），把原函数推迟到了新逻辑（console）的后面去执行。这种做法和我们上一节在ES5中实现装饰器模式时做的事情一狗一样，所以说装饰器就是这么回事儿，换汤不换药~

**生产实践**

场景：Vue+elementUI,将confirm提示确认框提取成装饰器
```
//decorators.ts
import { MessageBox } from "element-ui"

function confirmAction(message = "是否继续该操作?") {
  return function(target, name, descriptor) {
    let originalMethod = descriptor.value
    descriptor.value = function() {
      MessageBox.confirm(message, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(originalMethod.apply(this, arguments))
        .catch(() => {})
    }

    return descriptor
  }
}

export { confirmAction }
```
使用方法：在用到的vue页面引入
```
import { confirmAction } from '@/utils/decorators.ts'

@confirmAction("tip")
handleXXX(){
    // ..操作
}
```

**优质的源码阅读材料——**[core-decorators](https://github.com/jayphelps/core-decorators)
***

## 2.适配器模式
>适配器模式通过**把一个类的接口变换成客户端所期待的另一种接口**，可以帮我们解决**不兼容**的问题。

场景：现在手上有一个基于fetch的http方法库：
```
export default class HttpUtils {
  // get方法
  static get(url) {
    return new Promise((resolve, reject) => {
      // 调用fetch
      fetch(url)
        .then(response => response.json())
        .then(result => {
          resolve(result)
        })
        .catch(error => {
          reject(error)
        })
    })
  }
  
  // post方法，data以object形式传入
  static post(url, data) {
    return new Promise((resolve, reject) => {
      // 调用fetch
      fetch(url, {
        method: 'POST',
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        // 将object类型的数据格式化为合法的body参数
        body: this.changeData(data)
      })
        .then(response => response.json())
        .then(result => {
          resolve(result)
        })
        .catch(error => {
          reject(error)
        })
    })
  }
  
  // body请求体的格式化方法
  static changeData(obj) {
    var prop,
      str = ''
    var i = 0
    for (prop in obj) {
      if (!prop) {
        return
      }
      if (i == 0) {
        str += prop + '=' + obj[prop]
      } else {
        str += '&' + prop + '=' + obj[prop]
      }
      i++
    }
    return str
  }
}
```
当我想使用 fetch 发起请求时，只需要这样轻松地调用，而不必再操心繁琐的数据配置和数据格式化：
```
// 定义目标url地址
const URL = "xxxxx"
// 定义post入参
const params = {
    ...
}

// 发起post请求
 const postResponse = await HttpUtils.post(URL,params) || {}
 
 // 发起get请求
 const getResponse = await HttpUtils.get(URL) || {}
```
现在需求是：希望你能把公司所有的业务的网络请求都迁移到你这个 HttpUtils 上来，而该公司第1代员工封装的网络请求库，是基于 XMLHttpRequest 的，差不多长这样：
```
function Ajax(type, url, data, success, failed){
    // 创建ajax对象
    var xhr = null;
    if(window.XMLHttpRequest){
        xhr = new XMLHttpRequest();
    } else {
        xhr = new ActiveXObject('Microsoft.XMLHTTP')
    }
 
   ...(此处省略一系列的业务逻辑细节)
   
   var type = type.toUpperCase();
    
    // 识别请求类型
    if(type == 'GET'){
        if(data){
          xhr.open('GET', url + '?' + data, true); //如果有数据就拼接
        } 
        // 发送get请求
        xhr.send();
 
    } else if(type == 'POST'){
        xhr.open('POST', url, true);
        // 如果需要像 html 表单那样 POST 数据，使用 setRequestHeader() 来添加 http 头。
        xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
        // 发送post请求
        xhr.send(data);
    }
 
    // 处理返回数据
    xhr.onreadystatechange = function(){
        if(xhr.readyState == 4){
            if(xhr.status == 200){
                success(xhr.responseText);
            } else {
                if(failed){
                    failed(xhr.status);
                }
            }
        }
    }
}
```
实现逻辑我们简单描述了一下，这个不是重点，重点是它是这样调用的：
```
// 发送get请求
Ajax('get', url地址, post入参, function(data){
    // 成功的回调逻辑
}, function(error){
    // 失败的回调逻辑
})
```
不仅接口名不同，入参方式也不一样，这手动改要改到何年何日呢？

这里就可以专门为我们**抹平差异**的适配器模式，，我们只需要在引入接口时进行一次适配，便可轻松地 cover 掉业务里可能会有的多次调用（具体的解析在注释里）：
```
// Ajax适配器函数，入参与旧接口保持一致
async function AjaxAdapter(type, url, data, success, failed) {
    const type = type.toUpperCase()
    let result
    try {
         // 实际的请求全部由新接口发起
         if(type === 'GET') {
            result = await HttpUtils.get(url) || {}
        } else if(type === 'POST') {
            result = await HttpUtils.post(url, data) || {}
        }
        // 假设请求成功对应的状态码是1
        result.statusCode === 1 && success ? success(result) : failed(result.statusCode)
    } catch(error) {
        // 捕捉网络错误
        if(failed){
            failed(error.statusCode);
        }
    }
}

// 用适配器适配旧的Ajax方法
async function Ajax(type, url, data, success, failed) {
    await AjaxAdapter(type, url, data, success, failed)
}
```
如此一来，我们只需要编写一个适配器函数AjaxAdapter，并用适配器去承接旧接口的参数，就可以实现新旧接口的无缝衔接了~

**生产实践：axios中的适配器**

axios本身就用到了我们的适配器模式，它的兼容方案值得我们学习和借鉴。

除了简明优雅的api之外，axios 强大的地方还在于，它不仅仅是一个局限于浏览器端的库。在Node环境下，我们尝试调用上面的 api，会发现它照样好使 —— axios 完美地**抹平了两种环境下api的调用差异**，靠的正是对适配器模式的灵活运用。

这么一来，通过 axios 发起跨平台的网络请求，不仅调用的接口名是同一个，连入参、出参的格式都只需要掌握同一套。这导致它的学习成本非常低，开发者看了文档就能上手；同时因为足够简单，在使用的过程中也不容易出错，带来了极佳的用户体验，axios 也因此越来越流行。

这正是一个好的适配器的自我修养——把变化留给自己，把统一留给用户。在此处，所有关于 http 模块、关于 xhr 的实现细节，全部被 Adapter 封装进了自己复杂的底层逻辑里，暴露给用户的都是十分简单的统一的东西——统一的接口，统一的入参，统一的出参，统一的规则。用起来就是一个字 —— 爽！

[axios源码](https://github.com/axios/axios)
***

## 3.代理模式

>代理模式，式如其名——在某些情况下，出于种种考虑/限制，一个对象不能直接访问另一个对象，需要一个第三者（代理）牵线搭桥从而间接达到访问目的，这样的模式就是代理模式。

本节选取业务开发中最常见的四种代理类型：事件代理、虚拟代理、缓存代理和保护代理来进行讲解。

**事件代理**
事件代理，可能是代理模式最常见的一种应用方式，也是一道实打实的高频面试题。它的场景是一个父元素下有多个子元素，像这样：
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>事件代理</title>
</head>
<body>
  <div id="father">
    <a href="#">链接1号</a>
    <a href="#">链接2号</a>
    <a href="#">链接3号</a>
    <a href="#">链接4号</a>
    <a href="#">链接5号</a>
    <a href="#">链接6号</a>
  </div>
</body>
</html>
```
我们现在的需求是，希望鼠标点击每个 a 标签，都可以弹出“我是xxx”这样的提示。比如点击第一个 a 标签，弹出“我是链接1号”这样的提示。这意味着我们至少要安装 6 个监听函数给 6 个不同的的元素(一般我们会用循环，代码如下所示），如果我们的 a 标签进一步增多，那么性能的开销会更大。
```
// 假如不用代理模式，我们将循环安装监听函数
const aNodes = document.getElementById('father').getElementsByTagName('a')
  
const aLength = aNodes.length

for(let i=0;i<aLength;i++) {
    aNodes[i].addEventListener('click', function(e) {
        e.preventDefault()
        alert(`我是${aNodes[i].innerText}`)                  
    })
}
```
考虑到事件本身具有“冒泡”的特性，当我们点击 a 元素时，点击事件会“冒泡”到父元素 div 上，从而被监听到。如此一来，点击事件的监听函数只需要在 div 元素上被绑定一次即可，而不需要在子元素上被绑定 N 次——这种做法就是事件代理，它可以很大程度上提高我们代码的性能。

**事件代理的实现**
```
// 获取父元素
const father = document.getElementById('father')

// 给父元素安装一次监听函数
father.addEventListener('click', function(e) {
    // 识别是否是目标子元素
    if(e.target.tagName === 'A') {
        // 以下是监听函数的函数体
        e.preventDefault()
        alert(`我是${e.target.innerText}`)
    }
} )
```
在这种做法下，我们的点击操作并不会直接触及目标子元素，而是由父元素对事件进行处理和分发、间接地将其作用于子元素，因此这种操作从模式上划分属于代理模式。

**虚拟代理**

懒加载,它是针对图片加载时机的优化：在一些图片量比较大的网站，比如电商网站首页，或者团购网站、小游戏首页等。如果我们尝试在用户打开页面的时候，就把所有的图片资源加载完毕，那么很可能会造成白屏、卡顿等现象。

此时我们会采取“先占位、后加载”的方式来展示图片 —— 在元素露出之前，我们给它一个 div 作占位，当它滚动到可视区域内时，再即时地去加载真实的图片资源，这样做既减轻了性能压力、又保住了用户体验。

除了图片懒加载，还有一种操作叫**图片预加载**。预加载主要是为了避免网络不好、或者图片太大时，页面长时间给用户留白的尴尬。常见的操作是先让这个 img 标签展示一个占位图，然后创建一个 Image 实例，让这个 Image 实例的 src 指向真实的目标图片地址、观察该 Image 实例的加载情况 —— 当其对应的真实图片加载完毕后，即已经有了该图片的缓存内容，再将 DOM 上的 img 元素的 src 指向真实的目标图片地址。此时我们直接去取了目标图片的缓存，所以展示速度会非常快，从占位图到目标图片的时间差会非常小、小到用户注意不到，这样体验就会非常好了。

上面的思路，我们可以不假思索地实现如下

```
class PreLoadImage {
    // 占位图的url地址
    static LOADING_URL = 'xxxxxx'
    
    constructor(imgNode) {
        // 获取该实例对应的DOM节点
        this.imgNode = imgNode
    }
    
    // 该方法用于设置真实的图片地址
    setSrc(targetUrl) {
        // img节点初始化时展示的是一个占位图
        this.imgNode.src = PreLoadImage.LOADING_URL
        // 创建一个帮我们加载图片的Image实例
        const image = new Image()
        // 监听目标图片加载的情况，完成时再将DOM上的img节点的src属性设置为目标图片的url
        image.onload = () => {
            this.imgNode.src = targetUrl
        }
        // 设置src属性，Image实例开始加载图片
        image.src = targetUrl
    }
}
```
这个 PreLoadImage 乍一看没问题，但其实违反了我们设计原则中的**单一职责原则**。PreLoadImage 不仅要负责图片的加载，还要负责 DOM 层面的操作（img 节点的初始化和后续的改变）。这样一来，就**出现了两个可能导致这个类发生变化的原因**。

好的做法是将两个逻辑分离，让 PreLoadImage 专心去做 DOM 层面的事情（真实 DOM 节点的获取、img 节点的链接设置），再找一个对象来专门来帮我们搞加载——这两个对象之间缺个媒婆，这媒婆非代理器不可：
```
class PreLoadImage {
    constructor(imgNode) {
        // 获取真实的DOM节点
        this.imgNode = imgNode
    }
     
    // 操作img节点的src属性
    setSrc(imgUrl) {
        this.imgNode.src = imgUrl
    }
}

class ProxyImage {
    // 占位图的url地址
    static LOADING_URL = 'xxxxxx'

    constructor(targetImage) {
        // 目标Image，即PreLoadImage实例
        this.targetImage = targetImage
    }
    
    // 该方法主要操作虚拟Image，完成加载
    setSrc(targetUrl) {
       // 真实img节点初始化时展示的是一个占位图
        this.targetImage.setSrc(ProxyImage.LOADING_URL)
        // 创建一个帮我们加载图片的虚拟Image实例
        const virtualImage = new Image()
        // 监听目标图片加载的情况，完成时再将DOM上的真实img节点的src属性设置为目标图片的url
        virtualImage.onload = () => {
            this.targetImage.setSrc(targetUrl)
        }
        // 设置src属性，虚拟Image实例开始加载图片
        virtualImage.src = targetUrl
    }
}
```
ProxyImage 帮我们调度了预加载相关的工作，我们可以通过 ProxyImage 这个代理，实现对真实 img 节点的间接访问，并得到我们想要的效果。

在这个实例中，virtualImage 这个对象是一个“幕后英雄”，它始终存在于 JavaScript 世界中、代替真实 DOM 发起了图片加载请求、完成了图片加载工作，却从未在渲染层面抛头露面。因此这种模式被称为“虚拟代理”模式。

**缓存代理**

缓存代理比较好理解，它应用于一些计算量较大的场景里。在这种场景下，我们需要“用空间换时间”——当我们需要用到某个已经计算过的值的时候，不想再耗时进行二次计算，而是希望能从内存里去取出现成的计算结果。这种场景下，就需要一个代理来帮我们在进行计算的同时，进行计算结果的缓存了。

一个比较典型的例子，是对传入的参数进行求和：
```
// addAll方法会对你传入的所有参数做求和操作
const addAll = function() {
    console.log('进行了一次新计算')
    let result = 0
    const len = arguments.length
    for(let i = 0; i < len; i++) {
        result += arguments[i]
    }
    return result
}

// 为求和方法创建代理
const proxyAddAll = (function(){
    // 求和结果的缓存池
    const resultCache = {}
    return function() {
        // 将入参转化为一个唯一的入参字符串
        const args = Array.prototype.join.call(arguments, ',')
        
        // 检查本次入参是否有对应的计算结果
        if(args in resultCache) {
            // 如果有，则返回缓存池里现成的结果
            return resultCache[args]
        }
        return resultCache[args] = addAll(...arguments)
    }
})()
```
我们发现 proxyAddAll 针对重复的入参只会计算一次，这将大大节省计算过程中的时间开销。现在我们有 6 个入参，可能还看不出来，当我们针对大量入参、做反复计算时，缓存代理的优势将得到更充分的凸显。

**保护代理**

所谓“保护代理”，就是在访问层面做文章，在 getter 和 setter 函数里去进行校验和拦截，确保一部分变量是安全的。

Proxy，它本身就是为拦截而生的，所以我们目前实现保护代理时，考虑的首要方案就是 ES6 中的 Proxy。

婚介所例子
```
// 未知妹子
const girl = {
  // 姓名
  name: '小美',
  // 自我介绍
  aboutMe: '...'（大家自行脑补吧）
  // 年龄
  age: 24,
  // 职业
  career: 'teacher',
  // 假头像
  fakeAvatar: 'xxxx'(新垣结衣的图片地址）
  // 真实头像
  avatar: 'xxxx'(自己的照片地址),
  // 手机号
  phone: 123456,
}
```
婚介所收到了小美的信息，开始营业。大家想，这个姓名、自我介绍、假头像，这些信息大差不差，曝光一下没问题。但是人家妹子的年龄、职业、真实头像、手机号码，是不是属于非常私密的信息了？要想 get 这些信息，平台要考验一下你的诚意了 —— 首先，你是不是已经通过了实名审核？如果通过实名审核，那么你可以查看一些相对私密的信息（年龄、职业）。然后，你是不是 VIP ？只有 VIP 可以查看真实照片和联系方式。

1.满足了这两个判定条件，你才可以顺利访问到别人的全部私人信息，不然，就劝退你提醒你去完成认证和VIP购买再来-getter 层面的拦截。

2.假设我们还允许会员间互送礼物，每个会员可以告知婚介所自己愿意接受的礼物的价格下限，我们还可以作 setter 层面的拦截。：
```
// 规定礼物的数据结构由type和value组成
const present = {
    type: '巧克力',
    value: 60,
}

// 为用户增开presents字段存储礼物
const girl = {
  // 姓名
  name: '小美',
  // 自我介绍
  aboutMe: '...'（大家自行脑补吧）
  // 年龄
  age: 24,
  // 职业
  career: 'teacher',
  // 假头像
  fakeAvatar: 'xxxx'(新垣结衣的图片地址）
  // 真实头像
  avatar: 'xxxx'(自己的照片地址),
  // 手机号
  phone: 123456,
  // 礼物数组
  presents: [],
  // 拒收50块以下的礼物
  bottomValue: 50,
  // 记录最近一次收到的礼物
  lastPresent: present,
}

// 掘金婚介所推出了小礼物功能
const JuejinLovers = new Proxy(girl, {
  get: function(girl, key) {
    if(baseInfo.indexOf(key)!==-1 && !user.isValidated) {
        alert('您还没有完成验证哦')
        return
    }
    
    //...(此处省略其它有的没的各种校验逻辑)
  
    // 此处我们认为只有验证过的用户才可以购买VIP
    if(user.isValidated && privateInfo.indexOf(key)!==-1 && !user.isVIP) {
        alert('只有VIP才可以查看该信息哦')
        return
    }
  }
  
  set: function(girl, key, val) {
 
    // 最近一次送来的礼物会尝试赋值给lastPresent字段
    if(key === 'lastPresent') {
      if(val.value < girl.bottomValue) {
          alert('sorry，您的礼物被拒收了')
          return
      }
    
      // 如果没有拒收，则赋值成功，同时并入presents数组
      girl[lastPresent] = val
      girl[presents] = [...presents, val]
    }
  }
 
})
```
**小结**

代理模式行文至此，相信大家都已经做到了心中有数。在本节，我们看到代理模式的目的是十分多样化的，既可以是为了加强控制、拓展功能、提高性能，也可以仅仅是为了优化我们的代码结构、实现功能的解耦。无论是出于什么目的，这种模式的套路就只有一个—— A 不能直接访问 B，A 需要借助一个帮手来访问 B，这个帮手就是代理器。需要代理器出面解决的问题，就是代理模式发光发热的应用场景。