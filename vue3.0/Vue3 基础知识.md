# Vue3.0 笔记



## 1、事件修饰符详解

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

## 2、插槽

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

### 2.1 后备内容

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

### 2.2 具名插槽

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

### 2.3 独占默认插槽的缩写语法

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

### 2.4 结构插槽Prop

```html
// 作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里
// 这意味着 v-slot 的值实际上可以是任何能够作为函数定义中的参数的JavaScript表达式
// 使用 ES2015 解构来传入具体的插槽 prop，如下：
<todo-list v-slot="{ item }">
  <i class="fas fa-check"></i>
  <span class="green">{{ item }}</span>
</todo-list>
```

### 2.5 具名插槽的缩写

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



## 3、Vue2.0和Vue3.0的区别

### 3.1 响应式数据原理

#### 3.1.1 Vue2.0

* Vue2使用Object.defineProperty方法实现响应式数据，在初始化数据时，使用Object.defineProperty重新定义data中所有属性，当页面使用对应属性时，首先会进行依赖收集(收集当前组件的watcher)，如果属性发生变化会通过通知相关依赖进行更新操作。

* Object.defineProperty的缺点：

  ​	a. 无法检测到对象属性动态添加和删除

  ​	b. 无法检测到数组的下标和length属性的变更

* 解决方法：

  vue2提供Vue.$set动态给对象添加属性

  Vue.$delete动态删除对象属性

  重写数组的方法，检测数组变更

#### 3.1.2 Vue3.0

* Vue3使用Proxy实现响应式数据，Proxy可以直接监听对象和数组的变化，并且作为新标准将受到浏览器厂商重点持续的性能优化。

* Proxy的缺点：

  a. ES6的Proxy不支持低版本浏览器

  b. 会针对IE11出一个特殊版本进行支持

* Proxy的优点：

  a. 可以检测到代理对象属性的动态新增和删除

  b. 可以检测到数组的下标和length属性的变化

## 4. 生命周期

**vue2.x的生命周期**

<img src="https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/vue2_lifecycle.png" alt="vue2_lifecycle" style="zoom: 33%;" />

**vue3.x的生命周期**

<img src="https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/vue3_lifecycle.png" alt="vue3_lifecycle" style="zoom:33%;" />

### 4.1 与2.x版本生命周期相对应的组合式API

- ~~beforeCreate~~ -> 使用 `setup()`
- ~~created~~ -> 使用 `setup()`
- `beforeMount` -> `onBeforeMount`
- `mounted` -> `onMounted`
- `beforeUpdate` -> `onBeforeUpdate`
- `updated` -> `onUpdated`
- `beforeDestroy` -> `onBeforeUnmount`
- `destroyed` -> `onUnmounted`
- `errorCaptured` -> `onErrorCaptured`

## 5. 依赖注入

**2.2 provide/inject 示例**

````javascript
// 父级组件提供 'foo'
var Provider = {
  provide: {
    foo: 'bar'
  },
  // ...
}
// 子组件注入 'foo'
var Child = {
  inject: ['foo'],
  created () {
    console.log(this.foo) // => "bar"
  }
  // ...
}
````

**3.0 provide/inject 示例**

`provide` 和 `inject` 都只能在当前活动组件实例的 `setup()` 中调用。

````javascript
import { provide, inject } from 'vue'

const ThemeSymbol = Symbol()
const Ancestor = {
  setup() {
    provide(ThemeSymbol, 'dark')
  },
}

const Descendent = {
  setup() {
    const theme = inject(ThemeSymbol, 'light' /* optional default value */)
    return {
      theme,
    }
  },
}
````

### 5.1 处理响应性

默认情况下，`provide/inject`绑定*不是*b被动绑定，所以值的修改不会反映在注入的对象中。可以通过将`ref`property或`reactive`对象传递给`provide`来实现响应式；还可以使用过组合式API`computed`property实现。

**ref 或 reactive 方式**

````javascript
// 提供者：
const themeRef = ref('dark')
provide(ThemeSymbol, themeRef)

// 使用者：
const theme = inject(ThemeSymbol, ref('light'))
watchEffect(() => {
  console.log(`theme set to: ${theme.value}`)
})
````

**computed方式**

````javascript
// 提供者：
const todos = ['Feed a cat', 'Buy tickets']
const todoLength = computed(() => todos.length)
````

## 6. 混入

混入(mixin)封装Vue组件中的可复用功能。当组件使用混入对象，所有混入对象的选项被“混合”到组件自身中。

### 6.1 基础

```js
// define a mixin object
const myMixin = {
  created() {
    this.hello()
  },
  methods: {
    hello() {
      console.log('hello from mixin!')
    }
  }
}

// define an app that uses this mixin
const app = Vue.createApp({
  mixins: [myMixin]
})

app.mount('#mixins-basic') // => "hello from mixin!"
```

### 6.2 选项合并

当组件和混入对象含有同名选项时，默认按照如下方式进行合并：

* 数据对象同名：组件数据优先。
* 钩子函数同名：钩子函数合并为一个数组，混入对象的钩子先调用，然后再调用组件自身的钩子。
* 值为对象：如`methods`、`components`和`directives`，将被合并为同一个对象。同名对象的键名冲突时，优先取组件对象的键值对。

````javascript
const myMixin = {
  methods: {
    foo() {
      console.log('foo')
    },
    conflicting() {
      console.log('from mixin')
    }
  }
}

const app = Vue.createApp({
  mixins: [myMixin],
  methods: {
    bar() {
      console.log('bar')
    },
    conflicting() {
      console.log('from self')
    }
  }
})

const vm = app.mount('#mixins-basic')

vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
````

### 6.3 自定义选项合并策略

自定义选项将使用默认策略，即简单的覆盖已有值。通过向`app.config.optionMergeStrategies`添加函数实现自定义合并逻辑：

```js
const app = Vue.createApp({})

app.config.optionMergeStrategies.customOption = (toVal, fromVal) => {
  // return mergedVal
}
```

## 7. 自定义指令(Vue3.x)

### 7.1 注册指令

#### 7.1.1 全局注册

````javascript
const app = Vue.createApp({})
// 注册一个全局的自定义指令'v-focus'
app.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  mounted(el) {
    // Focus the element
    el.focus()
  }
})
````

#### 7.1.2 局部注册

````javascript
export default defineComponent({
  directives: {
    focus: {
      // 指令的定义
      mounted(el) {
        el.focus()
      }
    }
  }
});
````

### 7.2 指令的钩子函数

```js
import { createApp } from 'vue'
const app = createApp({})

// 注册
app.directive('my-directive', {
  // 指令是具有一组生命周期的钩子：
  // 在绑定元素的父组件挂载之前调用，可以做一次性初始化设置
  beforeMount() {},
  // 绑定元素的父组件挂载时调用
  mounted() {},
  // 在包含组件的 VNode 更新之前调用
  beforeUpdate() {},
  // 在包含组件的 VNode 及其子组件的 VNode 更新之后调用
  updated() {},
  // 在绑定元素的父组件卸载之前调用
  beforeUnmount() {},
  // 卸载绑定元素的父组件时调用
  unmounted() {}
})

// 注册 (功能指令) => 函数简写
app.directive('my-directive', () => {
  // 这将被作为 `mounted` 和 `updated` 调用
})

// getter, 如果已注册，则返回指令定义
const myDirective = app.directive('my-directive')
```

指令钩子的参数说明：

**el**

指令绑定到的元素，可用于DOM操作。

**binding**

包含以下property的对象。

* `instance`：使用指令的组件实例

* `value`：传递给指令的值。 如，`v-my-directive="1 + 1"`中，value为2.

* `oldValue`：旧值， 仅在`beforeUpdate`和`updated`中可用。

* `arg`：传递给指令的参数(如果有)。如，`v-my-directive:foo`中，arg为"foo"。

* `modifiers`：包含修饰符的对象(如果有)。 如，`v-my-directive.foo.bar`中，修饰符对象为`{foo: true, bar: true}`

* `dir`：注册指令时作为参数传递的*对象*。如：

  ````javascript
  app.directive('focus', {
    mounted(el) {
      el.focus()
    }
  })
  
  // dir对象为：
  {
    mounted(el) {
      el.focus()
    }
  }
  ````

#### 7.2.1 动态指令参数

通过`v-mydirective:[argment]="value"`语法可以灵活更新自定义指令的`argment`参数。

````vue
<template>
	<p v-pin="200">Stick me 200px from the top of the page</p>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
export default defineComponent({
	directive:{
  	pin: {
      mounted(el, binding) {
    		el.style.position = 'fixed'
    		el.style.top = binding.value + 'px'
  		}
    }
	}
})
</script>
````

该例子会把元素固定在距离顶部200像素的位置。实际的使用场景很可能是左侧或者右侧，这样的话，就需要动态参数对组件实例进行更新。

````vue
<template>
	<p v-pin:[direction]="200">Stick me 200px from the top of the page</p>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
export default defineComponent({
	directive:{
  	pin: {
      mounted(el, binding) {
    		el.style.position = 'fixed'
        // 获取指令的参数
        const s = binding.arg || 'top'
    		el.style[s] = binding.value + 'px'
  		}
    }
	},
  setup() {
    const direction = 'right'
    return {
      direction
    }
  }
})
</script>
````

还可以通过修改绑定值让定制指令更灵活。

````vue
<template>
	<p v-pin:[direction]="pinPadding">Stick me 200px from the top of the page</p>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
export default defineComponent({
	directive:{
  	pin: {
      mounted(el, binding) {
    		el.style.position = 'fixed'
        // 获取指令的参数
        const s = binding.arg || 'top'
    		el.style[s] = binding.value + 'px'
  		},
      // 增加updated钩子，当值更新时重新计算元素的距离
      updated(el, binding) {
        const s = binding.arg || 'top'
    		el.style[s] = binding.value + 'px'
      }
    }
	},
  setup() {
    const direction = 'right'
    const pinPadding = 200
    return {
      direction,
      pinPadding
    }
  }
})
</script>
````

上面的代码可以简写如下：

````vue
<template>
	<p v-pin:[direction]="pinPadding">Stick me 200px from the top of the page</p>
</template>

<script lang='ts'>
import { defineComponent } from 'vue'
export default defineComponent({
	directive:{
    pin: (el, binding) => {
      // `mounted` 和 `updated` 调用
      el.style.position = 'fixed'
  		const s = binding.arg || 'top'
  		el.style[s] = binding.value + 'px'
    }
	},
  setup() {
    const direction = 'right'
    const pinPadding = 200
    return {
      direction,
      pinPadding
    }
  }
})
</script>
````

#### 7.2.2 传入多个值

````vue
<div v-demo="{color: 'white', text: 'hello'}"></div>

<script lang='ts'>
import { defineComponent } from 'vue'
export default defineComponent({
	directive:{
    demo: (el, binding) => {
      console.log(binding.value.color) // => "white"
  		console.log(binding.value.text) // => "hello!"
    }
	}
})
</script>
````

## 8. 自定义字体

* 在public/static目录下新建fonts目录，将字体放入该文件夹

* 在assets/style目录下新建font.css文件

  ````css
  @font-face {
    font-family: 'Bebas';
    font-display: auto;
    src: local('Bebas'), url('/static/fonts/BEBAS.woff2');
  }
  ````

* 在main.ts或者APP.vue文件中引入font.css

* 使用自定义字体即可。
