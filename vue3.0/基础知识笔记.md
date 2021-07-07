# Vue3.0 笔记



## 1、v-if和v-show的区别

`v-if`是“真正的”条件渲染，条件切换时，条件块内的事件监听器和子组件会被适当的销毁或重建。同时，`v-if`只会在条件第一次为真的时候渲染条件块，为假的时候什么都不做。

`v-show`的元素始终会被渲染在DOM中，通过切换CSS的`display` property控制元素的显示与否。

`v-if`切换开销更高，`v-show`渲染开销更高。因此，如果频繁切换时，使用`v-show`较好；条件变化很少时，使用`v-if`较好。

## 2、事件修饰符详解

• `.stop` 阻止事件冒泡，即不允许当前元素的事件继续往外触发

• `.prevent` 阻止默认事件的发生，如阻止超链接的点击挑战，form表单的点击提交

• `.self` 只响应当前元素触发的事件，不会响应冒泡触发的事件，且不会阻止冒泡继续向外触发

• `.capture` 改变js默认的事件机制，默认是冒泡，capture功能是将冒泡改为捕获模式

• `.once` 事件只响应一次

• `.passive` 滚动事件的默认行为(滚动行为)将会立即触发，而不会等待onScroll完成。

````html
<!-- 阻止单击事件继续传播 -->
<a @click.stop="doThis"></a>

<!-- 提交时间不再重载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a @click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form @submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div @click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div @click.self="doThat">...</div>

<!-- 点击事件将只会触发一次 -->
<a @click.once="doThis"></a>

<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发   -->
<!-- 而不会等待 `onScroll` 完成                   -->
<!-- 这其中包含 `event.preventDefault()` 的情况   -->
<div @scroll.passive="onScroll">...</div>
````

## 3、插槽

在封装的组件中放置`<slot>`元素，当组件渲染时，用组件嵌套的内容替换`<slot>`元素。

````html
// 调用todo-button组件
<todo-button>
	Add Todo
</todo-button>
````

````html
// todo-button 组件
<button class="btn-primary">
  <slot></slot>
</button>
````

组件渲染的时候，DOM结构如下：

````html
<!-- 渲染 HTML -->
<button class="btn-primary">
  Add Todo
</button>
````

### 3.1 后备内容

后备内容即组件插槽的默认内容。如在`<submit-button>`组件中设置后备内容:

````html
// slot元素直接的内容为后备内容
<button type="submit">
  <slot>Submit</slot>
</button>
````

当使用 `<submit-button` > 并且不提供任何插槽内容时：

```html
<submit-button></submit-button>
```

渲染后：

```html
<button type="submit">
  Submit
</button>
```

当使用 `<submit-button` > 并且提供内容时：

```html
<submit-button>
  Save
</submit-button>
```

渲染后:

```html
<button type="submit">
  Save
</button>
```

### 3.2 具名插槽

当有多个插槽时，可以通过`<slot>`元素的attribute：`name`命名插槽。如`<base-layout>` 组件：

```html
// 一个不带 name 的 <slot> 出口会带有隐含的名字“default”。
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称：

```html
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <template v-slot:default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

渲染的 HTML 结果是：

```html
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```

### 3.3 独占默认插槽的缩写语法

当组件只有默认插槽时，组件的标签可以被当做插槽的模板来使用，可以把`v-slot`直接用在组件上：

```html
todo-list v-slot:default="slotProps">
  <i class="fas fa-check"></i>
  <span class="green">{{ slotProps.item }}</span>
</todo-list>
// 或简写
// 不带参数的 v-slot 被假定对应默认插槽
// 注意默认插槽的缩写语法不能和具名插槽混用
todo-list v-slot="slotProps">
  <i class="fas fa-check"></i>
  <span class="green">{{ slotProps.item }}</span>
</todo-list>
```

### 3.4 结构插槽Prop

```html
// 作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里
// 这意味着 v-slot 的值实际上可以是任何能够作为函数定义中的参数的JavaScript表达式
// 使用 ES2015 解构来传入具体的插槽 prop，如下：
<todo-list v-slot="{ item }">
  <i class="fas fa-check"></i>
  <span class="green">{{ item }}</span>
</todo-list>
```

### 3.5 具名插槽的缩写

```html
// 跟 v-on 和 v-bind 一样，v-slot 也有缩写，即把参数之前的所有内容 (v-slot:) 替换为字符 #。
// 例如 v-slot:header 可以被重写为 #header
<base-layout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <template #default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```



## 4、Vue2.0和Vue3.0的区别

### 4.1 响应式数据原理

#### 4.1.1 Vue2.0

* Vue2使用Object.defineProperty方法实现响应式数据，在初始化数据时，使用Object.defineProperty重新定义data中所有属性，当页面使用对应属性时，首先会进行依赖收集(收集当前组件的watcher)，如果属性发生变化会通过通知相关依赖进行更新操作。

* Object.defineProperty的缺点：

  ​	a. 无法检测到对象属性动态添加和删除

  ​	b. 无法检测到数组的下标和length属性的变更

* 解决方法：

  vue2提供Vue.$set动态给对象添加属性

  Vue.$delete动态删除对象属性

  重写数组的方法，检测数组变更

#### 4.1.2 Vue3.0

* Vue3使用Proxy实现响应式数据，Proxy可以直接监听对象和数组的变化，并且作为新标准将受到浏览器厂商重点持续的性能优化。

* Proxy的缺点：

  a. ES6的Proxy不支持低版本浏览器

  b. 会针对IE11出一个特殊版本进行支持

* Proxy的优点：

  a. 可以检测到代理对象属性的动态新增和删除

  b. 可以检测到数组的下标和length属性的变化

