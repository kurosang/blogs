# VUE源码-编译

## 编译入口-baseCompile（）
baseCompile 在执行 createCompilerCreator 方法时作为参数传入，如下：
```
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  optimize(ast, options)
  const code = generate(ast, options)
  return {
    ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
})
```
它主要就是执行了如下几个逻辑：
* 解析模板字符串生成 AST
```
const ast = parse(template.trim(), options)
```
* 优化语法树
```
optimize(ast, options)
```
* 生成代码
```
const code = generate(ast, options)
```

## parse

编译过程首先就是对模板做解析，生成 AST，它是一种抽象语法树，是对源代码的抽象语法结构的树状表现形式。在很多编译技术中，如 babel 编译 ES6 的代码都会先生成 AST。

这个过程是比较复杂的，它会用到大量正则表达式对字符串解析，如果对正则不是很了解，建议先去补习正则表达式的知识。为了直观地演示 parse 的过程，我们先来看一个例子：
```
<ul :class="bindCls" class="list" v-if="isShow">
    <li v-for="(item,index) in data" @click="clickItem(index)">{{item}}:{{index}}</li>
</ul>
```
经过 parse 过程后，生成的 AST 如下：
```
ast = {
  'type': 1,
  'tag': 'ul',
  'attrsList': [],
  'attrsMap': {
    ':class': 'bindCls',
    'class': 'list',
    'v-if': 'isShow'
  },
  'if': 'isShow',
  'ifConditions': [{
    'exp': 'isShow',
    'block': // ul ast element
  }],
  'parent': undefined,
  'plain': false,
  'staticClass': 'list',
  'classBinding': 'bindCls',
  'children': [{
    'type': 1,
    'tag': 'li',
    'attrsList': [{
      'name': '@click',
      'value': 'clickItem(index)'
    }],
    'attrsMap': {
      '@click': 'clickItem(index)',
      'v-for': '(item,index) in data'
     },
    'parent': // ul ast element
    'plain': false,
    'events': {
      'click': {
        'value': 'clickItem(index)'
      }
    },
    'hasBindings': true,
    'for': 'data',
    'alias': 'item',
    'iterator1': 'index',
    'children': [
      'type': 2,
      'expression': '_s(item)+":"+_s(index)'
      'text': '{{item}}:{{index}}',
      'tokens': [
        {'@binding':'item'},
        ':',
        {'@binding':'index'}
      ]
    ]
  }]
}
```
可以看到，生成的 AST 是一个树状结构，每一个节点都是一个 ast element，除了它自身的一些属性，还维护了它的父子关系，如 parent 指向它的父节点，children 指向它的所有子节点。先对 AST 有一些直观的印象，那么接下来我们来分析一下这个 AST 是如何得到的。

### 整体流程
```
export function parse (
  template: string,
  options: CompilerOptions
): ASTElement | void {
  getFnsAndConfigFromOptions(options)
  parseHTML(template, {
    // options ...
    start (tag, attrs, unary) {
      let element = createASTElement(tag, attrs)
      processElement(element)
      treeManagement()
    },
    end () {
      treeManagement()
      closeElement()
    },
    chars (text: string) {
      handleText()
      createChildrenASTOfText()
    },
    comment (text: string) {
      createChildrenASTOfComment()
    }
  })
  return astRootElement
}
```
* 从 options 中获取方法和配置
* 解析 HTML 模板
* 处理开始标签
* 处理闭合标签
* 处理文本内容

### 流程图

![](../../imgs/parse.png)

### 总结

那么至此，parse 的过程就分析完了，看似复杂，但我们可以抛开细节理清它的整体流程。parse 的目标是把 template 模板字符串转换成 AST 树，它是一种用 JavaScript 对象的形式来描述整个模板。那么整个 parse 的过程是利用正则表达式顺序解析模板，当解析到开始标签、闭合标签、文本的时候都会分别执行对应的回调函数，来达到构造 AST 树的目的。

AST 元素节点总共有 3 种类型，type 为 1 表示是普通元素，为 2 表示是表达式，为 3 表示是纯文本。

## optimize

当我们的模板 template 经过 parse 过程后，会输出生成 AST 树，那么接下来我们需要对这颗树做优化，optimize 的逻辑是远简单于 parse 的逻辑，所以理解起来会轻松很多。

为什么要有优化过程，因为我们知道 Vue 是数据驱动，是响应式的，但是我们的模板并不是所有数据都是响应式的，也有很多数据是首次渲染后就永远不会变化的，那么这部分数据生成的 DOM 也不会变化，我们可以在 patch 的过程跳过对他们的比对。

我们在编译阶段可以把一些 AST 节点优化成静态节点，所以整个 optimize 的过程实际上就干 2 件事情，`markStatic(root)` 标记静态节点 ，`markStaticRoots(root, false)` 标记静态根。

* 标记静态节点

首先执行 `node.static = isStatic(node)`

`isStatic` 是对一个 AST 元素节点是否是静态的判断，如果是表达式，就是非静态；如果是纯文本，就是静态；对于一个普通元素，如果有 pre 属性，那么它使用了 `v-pre` 指令，是静态，否则要同时满足以下条件：没有使用 `v-if、v-for`，没有使用其它指令（不包括 `v-once`），非内置组件，是平台保留的标签，非带有 `v-for` 的 `template` 标签的直接子节点，节点的所有属性的 `key` 都满足静态 key；这些都满足则这个 AST 节点是一个静态节点。

如果这个节点是一个普通元素，则遍历它的所有 `children`，递归执行 `markStatic`。因为所有的 `elseif` 和 `else` 节点都不在 children 中， 如果节点的 `ifConditions` 不为空，则遍历 `ifConditions` 拿到所有条件中的 `block`，也就是它们对应的 AST 节点，递归执行 `markStatic`。在这些递归过程中，一旦子节点有不是 `static` 的情况，则它的父节点的 `static` 均变成 false。

* 标记静态根

markStaticRoots 第二个参数是 isInFor，对于已经是 static 的节点或者是 v-once 指令的节点，node.staticInFor = isInFor。接着就是对于 staticRoot 的判断逻辑，从注释中我们可以看到，对于有资格成为 staticRoot 的节点，除了本身是一个静态节点外，必须满足拥有 children，并且 children 不能只是一个文本节点，不然的话把它标记成静态根节点的收益就很小了。

接下来和标记静态节点的逻辑一样，遍历 children 以及 ifConditions，递归执行 markStaticRoots。

### 总结

那么至此我们分析完了 optimize 的过程，就是深度遍历这个 AST 树，去检测它的每一颗子树是不是静态节点，如果是静态节点则它们生成 DOM 永远不需要改变，这对运行时对模板的更新起到极大的优化作用。

## codegen

编译的最后一步就是把优化后的 AST 树转换成可执行的代码。

为了方便理解，我们还是用之前的例子：
```
<ul :class="bindCls" class="list" v-if="isShow">
    <li v-for="(item,index) in data" @click="clickItem(index)">{{item}}:{{index}}</li>
</ul>
```
它经过编译，执行 const code = generate(ast, options)，生成的 render 代码串如下：
```
with(this){
  return (isShow) ?
    _c('ul', {
        staticClass: "list",
        class: bindCls
      },
      _l((data), function(item, index) {
        return _c('li', {
          on: {
            "click": function($event) {
              clickItem(index)
            }
          }
        },
        [_v(_s(item) + ":" + _s(index))])
      })
    ) : _e()
}
```
重点关注一下这个 render 代码串的生成过程:
* generate
```
const code = generate(ast, options)
```
generate 函数首先通过 genElement(ast, state) 生成 code，再把 code 用 with(this){return ${code}}} 包裹起来。

基本就是判断当前 AST 元素节点的属性执行不同的代码生成函数。

* genIf

genIf 主要是通过执行 genIfConditions，它是依次从 conditions 获取第一个 condition，然后通过对 condition.exp 去生成一段三元运算符的代码，: 后是递归调用 genIfConditions，这样如果有多个 conditions，就生成多层三元运算逻辑。这里我们暂时不考虑 v-once 的情况，所以genTernaryExp 最终是调用了 genElement。

* genFor

genFor 的逻辑很简单，首先 AST 元素节点中获取了和 for 相关的一些属性，然后返回了一个代码字符串。

* genData

enData 函数就是根据 AST 元素节点的属性构造出一个 data 对象字符串，这个在后面创建 VNode 的时候的时候会作为参数传入。

* genChildren

genChildren 的就是遍历 children，然后执行 genNode 方法，根据不同的 type 执行具体的方法。

### 总结

这一节通过例子配合解析，我们对从 `ast -> code` 这一步有了一些了解，编译后生成的代码就是在运行时执行的代码。