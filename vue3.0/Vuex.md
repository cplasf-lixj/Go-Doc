# Vuex

## 1. Vuex原理

![vuex](https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/vuex.png)

<video src="https://raw.githubusercontent.com/cplasf-lixj/photo-album/main/vuex_flow.MOV"></video>

* 通过Actions触发行为，Actions调用Commit到Mutations中，Mutations实现State的变更。
* 通过Getter获取State状态。

### 1.1 Vuex的特点

1. Vuex的状态是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 不能直接改变store中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样方便跟踪每个状态的变化，从而可以借助一些工具了解应用。

## 2. 核心概念

### 2.1 State

Vuex使用**单一状态树**，每个应用仅包含一个`store`实例。调用`Vue.use(Vuex)`将状态根组件“注入”到每个子组件，通过在根实例中注册`store`选项，将`store`从根组件注入到所有子组件。

```js
// 根组件
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})

// 子组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

#### 2.1.1 `mapState`辅助函数

当组件需要获取多个状态时，为了避免声明计算属性的重复和冗余，可以使用`mapState`辅助函数生成计算属性：

```js
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,
    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

当映射的计算属性的名称与 state 的子节点名称相同时，也可以给 `mapState` 传一个字符串数组。

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

`mapState`辅助函数解决了声明计算属性重复和冗余的问题，但是无法与组件内局部计算属性混合使用，对此需要使用下面的对象展开运算符

#### 2.1.2 对象展开运算符

`mapState`函数返回的是一个对象，通过**对象展开运算符**将`mapState`对象的内容合并到`computed`中。

```js
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```

### 2.2 Getters

可以理解getters是基于state派生出的状态。

```js
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

上面的例子中，doneTodoCount计算属性是对列表进行过滤后的数量统计。如果有多个组件需要用到此属性，每处都实现该函数，就会造成代码冗余和重复。

Vuexti提供了“getter”模块(可以认为是store的计算属性)，getter的返回值会基于依赖进行缓存，只有当依赖的值发生了改变才会被重新计算。

Getter 接受 state 作为其第一个参数：

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

#### 2.2.1 访问方式

##### 2.2.1.1 属性访问

通过Getter暴露的`store.getters`对象可以直接访问属性

````js
store.getters.doneTodos		// [{ id: 1, text: '...', done: true }]
````

Getter 也可以接受其他 getter 作为第二个参数：

```js
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}

store.getters.doneTodosCount // -> 1
```

**注意，getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的。**

##### 2.2.1.2 方法访问

通过让 getter 返回一个函数，来实现给 getter 传参.

```js
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}

store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

**注意，getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。**

#### 2.2.2 `mapGetters`辅助函数

`mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

如果想将一个 getter 属性另取一个名字，使用对象形式：

```js
...mapGetters({
  // 把 `this.doneCount` 映射为 `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```

### 2.3 Mutations

在 Vuex 中，**mutation 都是同步事务**，一条重要的原则就是 **mutation 必须是同步函数**

#### 2.3.1 原理

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的**事件类型 (type)\**和一个\**回调函数 (handler)**

```js
const store = createStore({
  state: {
    count: 1
  },
  mutations: {
		// 回调函数接受 state 作为第一个参数
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
```

#### 2.3.2 变更方式

调用 **store.commit** 方法：

```js
store.commit('increment')
```

#### 2.3.3 提交载荷(Payload)

可以向 `store.commit` 传入额外的参数，即 mutation 的**载荷（payload）**

```js
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}

store.commit('increment', 10)
```

当载荷包含多个自动时，载荷应是一个对象，这样也更已读：

```js
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {
  amount: 10
})
```

#### 2.3.4 对象风格的提交方式

提交 mutation 的另一种方式是直接使用包含 `type` 属性的对象：

```js
// 使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，因此处理函数保持不变：
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

// 对象风格的提交方式
store.commit({
  type: 'increment',
  amount: 10
})
```

#### 2.3.5 `mapMutations`辅助函数

在组件中提交mutation，可以使用`store.commit('xxx')`提交，或者使用`mapMutations`辅助函数将组件中methods映射为`store.commit`调用

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

### 2.4 Actions

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
    // 参数结构的代码
    increment( { commit }) {
  		commit('increment')
		}
  }
})
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 `context.commit` 提交一个 mutation，或者通过 `context.state` 和 `context.getters` 来获取 state 和 getters。

#### 2.4.1 分发Action

Action 通过 `store.dispatch` 方法触发：

```js
store.dispatch('increment')
```

 action 内部执行**异步**操作：

```js
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

Actions 支持同样的载荷方式和对象方式进行分发:

```js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

#### 2.4.2 `mapActions`辅助函数

在组件中分发action，可以使用`store.dispatch('xxx')`，或者使用`mapActions`辅助函数将组件的methods映射为`store.dispatch`调用。

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`

      // `mapActions` 也支持载荷：
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

#### 2.4.3 组合Action

##### 2.4.3.1 Promise

`store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise：

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}

// 分发
store.dispatch('actionA').then(() => {
  // ...
})
```

在另外一个 action 中也可以：

```js
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

##### 2.4.3.2 async/await

利用**<font color="green"> async / await</font>**，也可以组合 action：

```js
// 假设 getData() 和 getOtherData() 返回的是 Promise

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

### 2.5 Modules

Vuex 允许将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

```js
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

#### 2.5.1 模块内部的状态

##### 2.5.1.1 局部状态

模块内部的`mutation`和`getter`，第一个参数为**`模块的局部状态对象`**，`action`的局部状态通过`context.state`暴露出来。

````js
const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) {
      // 这里的 `state` 对象是模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  },
  
  actions: {
    incrementIfOddOnRootSum ({ state, commit }) {
      commit('increment')
    }
  }
}
````

##### 2.5.1.2 根节点状态

模块内部的action的根节点状态为`context.rootState`暴露出来

````js
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
````

模块内部的getter的根节点状态作为第三个参数暴露出来(第二个参数为其他getter)

````js
const moduleA = {
  // ...
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
````

#### 2.5.2 命名空间

参考[官网文档](https://vuex.vuejs.org/zh/guide/modules.html#%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4)

## 3. 进阶

### 3.1 项目结构

Vuex 规定了一些需要遵守的规则：

1. 应用层级的状态应该集中到单个 store 对象中。
2. 提交 **mutation** 是更改状态的唯一方法，并且这个过程是同步的。
3. 异步逻辑都应该封装到 **action** 里面。

项目结构示例：

```sh
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```

参考[购物车示例](https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart)

### 3.2 插件



### 3.3 严格模式



### 3.4 表单处理



### 3.5 测试



### 3.6 热重载

