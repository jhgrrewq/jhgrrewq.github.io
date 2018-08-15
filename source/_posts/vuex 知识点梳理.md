---
title: vuex 知识点梳理
tag: 
	- vue
	- vuex
---

<!-- markdownlint-disable MD010 -->

> 参考: [vuex 官方文档](https://vuex.vuejs.org/zh-cn/intro.html)

## 概念

### 单向数据流

一个状态自管理应用包含以下几个部分：

- **state:** 驱动应用的数据源

- **view:** 以声明的方式将 state 映射到视图

- **action:** 响应在 view 上用户的输入导致的状态变化

![](http://ony85apla.bkt.clouddn.com/18-1-31/80051283.jpg)

<!-- more -->

但是多组件共享状态时候，单向数据流的简洁性很容易遭到破坏：

- 传参的方法对于多层嵌套组件非常繁琐，并且对于兄弟之间组件的状态传递无能为力

- 通过父子组件直接引用或者通过事件来变更同步状态的多份拷贝，这些都会使代码变得难以维护

### Vuex

**vuex 的背后思想就是，将组件的共享状态抽离，用一个全局的单例模式管理。**在该模式下，组件树构成一个巨大的“视图”，不管在哪个位置，任何组件都能获取状态或触发行为。

vuex 是单独为 vue 设计的状态管理库，以利用 vue.js 细粒度数据响应机制来进行高效的状态更新

![](http://ony85apla.bkt.clouddn.com/18-1-31/71982948.jpg)

### 什么时候使用 vuex

不打算开发大型单页应用/多页应用时候，使用 vuex 可能是繁琐冗余的。一般使用 global event bus 就足够了。也就是**在非父子组件通信使用一个空的 Vue 实例作为事件总栈**

```javascript
var bus = new Vue()

// 触发 A 组件的事件
bus.$emit('selected', item)

// 在 B 组件创建的钩子中监听
bus.$on('selected', (item) => {
    // ...
})
```

## state

vuex 使用**单一状态树**————**即一个对象就包含了全部共享状态。每个应用将只包含一个 store 实例**

**注意：不要把所有状态放入 vuex。对于只属于单个组件的状态，最好还是作为组件的局部状态**

### vue 组件中获取 vuex 状态

从 store 实例中获取状态最简单的方法是在 computed 中返回状态

**vuex 通过 store 选项，通过在 vue 根实例中注册 store 选项，该 store 实例会注入到根组件下的全部子组件，并且子组件能通过 this.$store 访问** （需要使用 Vue.use(Vuex)）

```javascript
// 入口文件 index.js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)

new Vue({
	el: '#index',
	// 把 store 对象提供给 ‘store’ 选项， 这样把 store 实例注入到所有子组件
	store,
	components: { Counter },
	template: `
		<div class="index">
      		<counter></counter>
    	</div>`
})

// 子组件 Counter
const Counter = {
	template: `<div>{{ count }}</div>`,
	computed: {
		count() {
			return this.$store.state.count
		}
	}
}
```

### mapState

vuex 提供了 mapState 语法糖方便生成计算属性，简化书写

**mapState 函数返回一个对象，为了和组件的局部计算属性混合使用，用 展开运算符  ... 能将多个对象合并到一个，使得将最后的对象传递给 computed 属性**

```javascript
import {mapState} from 'vuex'

export default {
	computed: {
		...mapState({
			count: state => state.count

			// 传递字符串参数 'count' 相当于 `state => state.count`
			countAlias: 'count',
			countPlusLocalState (state) {
				return state.count + this.localCount
			}
		})
	}
}
```

当映射的计算属性名称和 state 的子节点名称一致，可以给 mapState 传递一个数组

```javascript
computed: {
	...mapState([
		// 映射 this.count 为 this.$store.state.count
		'count'
	])
}
```

## getters

有时我们需要从 store 的 state 中派生出一些状态， vuex 允许在 store 中定义 getters （可以认为是 store 的计算属性）。**同计算属性一样， getters 的返回值会根据它的依赖缓存起来，并且只有它的依赖值发生了改变才会重新计算**

getters 会暴露为 store.getters 对象，接受 state 作为第一个参数，**也可以接受其他的 getters 作为第二个参数**

```javascript
const store = new Vuex.store({
	state: {
		todos: [
			{ id: 1, text: '...', done: true }
			{ id: 2, text: '...', done: false }
		]
	},
	getters: {
		// 简化 state 同名取值的书写
		todos: state => state.todos,
		// 从 state 中派发其他状态
		doneTodos: state => {
			return state.todos.filter(todo => todo.done)
		},
		// 接受其他 getters 作为参数
		doneTodosCount: (state, getters) => {
			return getters.doneTodos.length
		}
	}
})

// 在组件中使用
computed: {
	todos() {
		return this.$store.gettes.todos
	},
	doneTodos() {
		return this.$store.getters.doneTodos // [{ id: 1, text: '...', done: true }]
	},
	doneTodosCount() {
		return this.$store.getters.doneTodosCount
	}
}
```

可以让 getters 返回一个函数，实现 getters 传参。在对 store 中的数组查询时非常有用

```javascript
getters: {
	getTodoById: (state) => (id) => {
		return state.todos.filter(todo => todo.id === id)
	}
}

...
store.getters.getTodoById(2) // { id: 2, text: '...', done: true }
```

### mapGetters

同样类似 mapState， mapGetters 语法糖是将 store 中的 getters 映射到组件的局部计算属性

```javascript
import {mapGetters} from 'vuex'

export default {
	...
	computed: {
		// 使用展开运算符将 getters 混入到 computed 中
		...mapGetters([
			'doneTodosCount'
		])
	}
}
```

对 getters 属性另取一个名字， 使用对象形式

```javascript
computed: {
	...mapGetters({
		// 映射 this.doneCount 为 this.$store.getters.doneTodosCount
		'doneCount': 'doneTodosCount'
	})
}
```

## mutations

**更改 store 中的状态的唯一方法就是提交 mutations。而且 mutaions 必须是同步函数, 异步操作或者提交多个 mutations 放到 actions中**

vuex 的 mutations 类似事件：每个 mutation 都有一个字符串的事件类型（type）和一个回调函数（hanlder）。**在回调函数中进行状态修改，接受 state 作为第一个参数，当然也可以传入其他参数（playload 载荷）**

```javascript
const store = new Vuex.store({
	state: {
		count: 1
	},
	mutations: {
		increment(state) {
			// 变更状态
			state.count++
		},
		// 第二个参数可以传递其他参数
		decrement(state, n) {
			state.count -= n
		},
		// payload 可以是一个对象，这样可以包含多个字段使 mutations 更易读
		multiIncrement(state, payload) {
			state.count += playload.amount
		}
})
```

不能直接调用 mutations 的回调，更像是事件注册：当触发一个类型为 increment 的 mutation 时，调用该函数。**提交 mutations 需要以对应的 type 调用 store.commit 方法**

```javascript
store.commit('increment')

store.commit('decrement', 3)

store.commit('multiIncrement', {
	amount: 10
})
```

提交 mutations 的另一种方式是直接使用含有 type 属性的对象，当使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数， 回调（handler）保持不变

```javascrip
store.commit({
	type: 'multiIncrement',
	amount: 10
})
```

### mutations 需遵循 vue 的响应原则

- 最好提前在 store 中初始化好所有所需的属性

- 在对象上添加新属性时，可以用新对象替换老对象 或调用 Vue.set()

```javascript
// 调用 Vue.set()
Vue.set(obj, 'newProp', 123)

// 用新对象替换老对象 展开运算符
state.obj = { ...state.obj, newProp: 123 }
```

### 使用常量替代 mutations 事件类型

使用常量替代 mutations 事件方便 eslint 等代码检查工具进行语法检查，同时将这些常量放在单独的文件中方便了解整个应用的 mutations

```javascript
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'

// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.store({
	state: { ... },
	mutations: {
		// 使用 es6 的计算属性命名功能来使用一个常量作为函数名
		[SOME_MUTATION](state) {
			// mutation state
		}
	}
})
```

### 组件中提交 mutations

- **使用 this.$store.commit('xxx') 提交 mutations**

- **使用 mapMutations 语法糖将组件中的 methods 映射为 store.commit 调用（需要在根节点注入 store）**

```javascript
import { mapMutations } from 'vuex'

export default {
	// ...
	methods: {
		...mapMutations([
			'increment', // 调用 this.increment() 映射为 this.$store.commit('increment')

			// mapMutations 也支持 载荷
			'decrement' // 将调用 this.decrement(amount) 映射为 this.$store.commit('decrement', amount)
		]),
		// 也可以传入对象，如果想给 mutations 另外命名
		...mapMutations({
			add: 'increment' // 调用 this.add() 映射为 this.$store.commit('increment')
		})
	}
}
```

## actions

actions 类似 mutations，不同在于：

- actions 内部提交 mutations **(可提交多个)**， 不是直接更改状态
- actions 内部可以**包含异步操作**

actions 函数接受一个**和 store 实例具有相同方法和属性的 context 对象**， 因此可以通过调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 获取 state 和 getters

```javascript
const store = new Vuex.store({
	state: {
		count: 0
	},
	mutations: {
		increment(state) {
			state.count++
		},
		decrement(state, payload) {
			state.count -= payload.amount
		}
	},
	actions: {
		// context 对象是和 store 实例具有相同的方法和属性
		increnment(context) {
			context.commit('increment')
		},
		// 使用 es6 参数结构简化代码
		incrementAsync({ commit }) {
			// 可以包含异步操作
			setTimeout(() => {
				commit('increment')
			}, 1000)
		},
		decrement({ commit }, payload) {
			commit('decrement', payload)
		}
	}
})
```

### 分发 actions

**外部调用通过 store.dispatch 方法触发 actions**。也支持载荷方式和对象方式进行分发

```javascript
store.dispatch('increment')

// 载荷形式分发
store.dispatch('decrement', {
	amount: 10
})

// 对象形式分发
store.dispatch({
	type: 'decrement',
	amount: 10
})
```

### 组件中分发 actions

- **使用 this.$store.dispatch('xxx') 分发 actions**

- **使用 mapActions 语法糖将组件中的 methods 映射为 store.dispatch 调用（需要在根节点注入 store）**

```javascript
import { mapActions } from 'vuex'

export default {
	// ...
	methods: {
		...mapActions([
			'increment', // 调用 this.increment() 映射为 this.$store.dispatch('increment')

			// mapActions 也支持 载荷
			'decrement' // 将调用 this.decrement(amount) 映射为 this.$store.dispatch('decrement', amount)
		]),
		// 也可以传入对象，如果想给 actions 另外命名
		...mapMutations({
			add: 'increment' // 调用 this.add() 映射为 this.$store.dispatch('increment')
		})
	}
}
```

### 组合 actions

store.dispatch 可以处理被触发的 action 的处理函数返回的 Promise，并且 store.dispatch 仍然返回 Promise

```javascript
actions: {
	actionA({ commit }) {
		// 封装 Promise
		return new Promise(function(resolve, reject) {
			commit('someMutation')
			resolve()
		}, 1000)
	},
	actionB({ dispatch, commit }) {
		return dispatch('actionA').then(() => {
			commit('someOtherMutation')
		})
	}
}
```

如通过 async/await 形成组合 actions

```javascript
// 假设 getData() 和 getOtherData() 返回 Promise
actions: {
	async actionA({ commit }) {
		commit('someMutation', await getData())
	},
	async actionB({ dispatch, commit }) {
		await dispatch('actionA') // 等待 actionA 完成
		commit('actionB', await getOtherData())
	}
}
```

## modules

vuex 允许将 store 分割成 模块（module），每个模块拥有自己的 state、mutation、action、getter、甚至桥涛的子模块

```javascript
const moduleA = {
	state: {...},
	mutations: {...},
	actions: {...},
	getters: {...}
}

const moduleB = {
	state: {...},
	mutations: {...},
	actions: {...}
}

const store = new Vuex.store({
	// ...

	module: {
		a: moduleA,
		b: moduleB
	}
})

store.state.a // moduleA 的状态
store.state.b // moduleB 的状态
```

### 模块的局部状态

对于模块内部的 mutations 和 getters，接受的第一个参数是**模块的局部状态对象**

```javascript
const moduleA = {
	state: { count: 0 },
	mutations: {
		increment(state) {
			// 这里 state 是模块的局部状态
			state.count++
		}
	},
	getters: {
		doubleCount(state) {
			return state.count * 2
		}
	}
}
```

同样对于模块内部的 actions，局部状态通过 context.state 暴露出来， **根节点状态则为 context.rootState**

```javascript
const moduleA = {
	// ...
	actions: {
		incrementIfOddOnRootSum({ state, commit, rootState }) {
			if ((state.count + rootState.count) % 2 === 1) {
				commit('increment')
			}
		}
	}
}
```

对于**模块内部的 getters，根节点状态会最为第三个参数**暴露

```javascript
const moduleA = {
	// ...
	getters: {
		sumWithRootCount({ state, getters, rootState }) {
			return state.count + rootState.count
		}
	}
}
```

## 项目结构

vuex 并不限制代码结构，但必须遵循一些规则： 

- 应用层级的状态应该集中到单个 store对象中

- 提交 mutations 是改变状态的唯一方法，并且这个过程是同步的

- 异步逻辑都应该封装到 actions

```html
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

## 严格模式

在严格模式下，无论何时发生了状态变更而不是由 mutations 函数引起的，将会抛出错误。但是**不要在生产发布环境下启用严格模式，以免影响性能**

开启严格模式，只需要在创建 store 时候传入 strict: true

```javascript
const store = new Vuex.store({
	// ...
	// strict: true
	strict: process.env.NODE_ENV !== "production"
})
```

## 插件

vuex 的 store 接受 plugins 选项，**该选项暴露出每次 mutation 的钩子， vuex 插件就是一个函数，接受 store 作为唯一参数。而且在插件中也不允许直接修改状态，只能通过提交 mutations 触发变化**

```javascript
const myPlugin = store => {
	// 当 store 初始化后调用
	store.subscribe((mutation, state) => {
		// 每次 mutation 之后调用
		// mutation 的格式为 { type, payload }
		// 只能通过 提交 mutation 触发变化
	})
}

const store = new Vuex.store({
	//...
	plugins: [myPlugin]
})
```

### 生成 state 快照

有时候插件需要获得状态的“快照”，比较改变前后的状态。需要对状态对象进行深拷贝。**生成状态快照的插件应该只在开发阶段使用**

```javascript
const myPluginWithSnapshot = store => {
	let prevState = _.cloneDeep(store.state)
	store.subscribe((mutation, state) => {
		let nextState = _.cloneDeep(state)

		// 比较 prevState 和 nextState

		// 保存状态， 用于下一次 mutation
		prevState = nextState
	})
}

const store = new Vuex.store({
	// ...
	plugins: process.env.NODE_ENV !== 'production' ? [myPluginWithSnapshot]: []
})
```

vuex 自带的日志插件 createLogger 用于一般的调试，该插件会生成状态快照，只能在开发环境使用

```javascript
import createLogger from 'vuex/dist/logger'

const store = new Vuex.store({
	//...
	plugins: process.env.NODE_ENV !== 'production' ? [createLogger()]: []
})
```