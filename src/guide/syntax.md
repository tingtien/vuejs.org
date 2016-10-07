---
title: 模板语法
type: guide
order: 4
---

Vue.js使用基于HTML的模板语法，允许你以声明的方式将渲染过DOM绑定到底层Vue实例的数据。所有的Vue.js模板是一个能被符合规范的浏览器及HTML解析器解析的标准的HTML。

在底层，Vue编辑模板到虚拟DOM渲染方法。与反应系统相结合，一旦应用程序状态被改变，Vue能智的计算出最小的组件重新渲染和应用最少的DOM操作。

如果你熟悉虚拟DOM概念并喜欢原生的JavaScript，你也可以[直接写渲染函数](/guide/render-function.html)替代模板，可选JSX语法支持。

## 插值

### 文本

最基本的数据绑定形式是使用“Mustache”语法（双大括号）的文本插值。

``` html
<span>Message: {{ msg }}</span>
```

mustache标签将替换为 `msg` 属性对应的数据对象。每当数据对象的 `msg` 变动，它也会被更新。

[`v-once` 指令](/api/#v-once) 。指令执行一次数据改动而不会更新，但会影响同一个节点的任何绑定。

``` html
<span v-once>This will never change: {{ msg }}</span>
```

### 原始HTML

用Mustache解析数据为纯文本，而不是HTML。为了输出真的HTML，你需要使用 `v-html` 指令。

``` html
<div v-html="rawHtml"></div>
```

内容被插入了原始HTML-数据绑定将被忽略。注意你不能使用 `v-html` 去修改局部模板，因为Vue不是基于字符串的模板引擎。相反，组件是首选作为UI重用和组成的基本单元。

内容里插入一个原始HTML-数据绑定将被忽略。 

<p class="tip">在网站里，动态渲染任意HTML是非常危险的。因为这样很容易导致 [XSS攻击](https://en.wikipedia.org/wiki/Cross-site_scripting). </p>

### 属性

Mustaches语法不能在HTML属性内部使用，改用 [`v-bind` 指令 ](/api/#v-bind):

``` html
<div v-bind:id="dynamicId"></div>
```

它还适用于布尔属性-如果条件判断得到false，这个属性将被移除。

``` html
<button v-bind:disabled="someDynamicCondition">Button</button>
```

### 使用JavaScript表达式

目前为止，模板中我们只绑定了简单的属性键，但实际上在数据绑定内部，Vue支持所有原生JavaScript表达式。

``` html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

在所有者Vue实例的数据作用域中，这些表达式将作为JavaScript。一个限制是每个绑定只能包含 **一个单一的表达式**，所以下面将**不起**作用：

``` html
<!-- 这是一个语句，不是表达式: -->
{{ var a = 1 }}

<!-- 任务流也不能被执行, 使用三元表达式 -->
{{ if (ok) { return message } }}
```

<p class="tip">模板表达式是沙箱并只访问全局，如 `math` 和 `Date` 。你不要在模板表达式中试图访问用户定义的全局</p>

### 自定义过滤器

Vue允许你自定义过滤器应用于常见的文本格式。过滤器应该被追加到Mustache插值的末尾，由“|”符号标出：

``` html
{{ message | capitalize }}
```

<p class="tip">Vue 2.X过滤器只能被用于mustache语法内。为了在指令内实现相同的行为，你可以用 [计算属性](/guide/computed.html) 。</p>

过滤函数总会收到表达式的值作为第一个参数。

``` js
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
```

多个表达式可以被链接:

``` html
{{ message | filterA | filterB }}
```

过滤器是个JavaScript函数，因此它们可以使用参数：

``` html
{{ message | filterA('arg1', arg2) }}
```

这里的普通字符串 `arg1` 将被传递到过滤器中作为第二个参数，并且表达式 `arg2` 的值作为第三个参数。

## 指令

指令是带有 `v-` 前缀特殊属性。指令属性值是一个单一的JavaScript表达式（ `v-for` 除外，这个在后面单独讨论 ）。当其表达式的值改变，指令的工作是被动地将副作用应用到DOM。我们回顾一下我们在文档里见过的例子。

``` html
<p v-if="seen">Now you see me</p>
```

这里 `v-if` 指令根据表达式 `seen` 真的决定移除/插入 `<p>` 元素。

### 参数

一些指令能带参数，由指令名称后面的冒号表示。例如，`v-bind` 指令被用于被动更新一个HTML属性。

``` html
<a v-bind:href="url"></a>
```

这里的 `href` 是一个参数，它告诉 `v-bind` 指令绑定元素的 `href` 属性的 `url` 值。

一些例子用 `v-on` ，它能监听DOM事件:

``` html
<a v-on:click="doSomething">
```

这里参数就是监听DOM事件的名称。我们将更详细地讨论事件处理。

### 修饰符

修饰符是用点表示特殊后缀，它表明该指令被绑定在一些特殊方式中。例子，`prevent` 修饰符让 `v-on` 指令调用 `event.preventDefault()` 触发事件。

``` html
<form v-on:submit.prevent="onSubmit"></form>
```

我们深入的了解 `v-on` 和 `v-model` 时，后面我们将看到很多地方使用修饰符。 

## 简写语法

`v-` 前缀是用作视觉提示，它是Vue特定属性。当你使用Vue把动态行为应用到一些现有标记，这是非常有用的，但是感觉有很多冗长常用的指令。与此同时，当你构建[单页面应用](https://en.wikipedia.org/wiki/Single-page_application) ，这里Vue管理每个模板，`v-` 前缀并不要么重要。因此Vue对 `v-bind` 和 `v-on` 提供特殊简写.

### `v-bind` 简写

``` html
<!-- 完整语法 -->
<a v-bind:href="url"></a>

<!-- 简写语法 -->
<a :href="url"></a>
```

### `v-on` 简写

``` html
<!-- 完整语法 -->
<a v-on:click="doSomething"></a>

<!-- 简写语法 -->
<a @click="doSomething"></a>
```

它们看起来与原始HTML有点不同，但是 `:` 和 `@` 是属性名称的有效字符，而且所有支持Vue浏览器都能正确地解析它。此外，在最终呈现的标记中不会出现。简写语法是可选的，当你了解到它的用法以后，你将会非常喜欢用它。

