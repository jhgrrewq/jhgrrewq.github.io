---
title: vue-router 知识点梳理
tag: 
	- vue
	- vue-router
---
<!-- markdownlint-disable MD010 -->

> 参考: [vue-router 官方文档](https://router.vuejs.org/zh-cn/)

<!-- more -->
## 基本用法

html

```html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
	<!-- 使用 router-link 组件来导航 -->
	<!-- <router-link/> 默认被渲染成 a 标签， 使用 tag 属性指定渲染的 html 标签， to 属性指定跳转链接-->
	<!-- <router-link/> 对应的路由匹配成功会自动添加 class 属性值 .router-link-active -->
	<router-link to="/user"></router-link>

	<!-- 路由匹配到的组件都会渲染到这里 -->
	<router-view></router-view>
</div>
```

js

```javascript
// 0 使用模块化编程，需要引入 Vue 和 VueRouter, 调用 Vue.use(VueRouter)
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)

// 1 定义（路由）组件，可以从其他文件 import
const User = {template: '<div>user</div>'}
// import User from 'components/user'

// 2 定义路由 routes 每个路由映射一个路由组件
const routes = [{
	path: '/user',
	component: User
}]

// 3 创建 router 实例，传入 routes 配置，当然可以传入其他配置
const router = new VueRouter({
	routes
})

// 4 创建和挂载根实例
// router 作为配置参数传入，使整个应用都具有路由功能
const app = new Vue({
	el: '#app',
	router
})
```

VueRouter 将整个 router 实例和 route 实例(**当前激活的路由信息对象，只读，但可以 watch 变化**)挂载在 Vue 的构造函数原型上， 因此可通过 this.$router 和 this.$route 全局访问

接下来都以在 vue 中使用举例

## 动态路由匹配

路由配置中动态路径参数以冒号 : 开头
同一个路径匹配多个路由时，匹配的优先级按照定义的先后顺序，最先定义的优先级最高

```javascript
const router = new VueRouter({
	routes: [{
		path: '/user/:id',
		...
	}]
})
```

**以 vue 举例， 匹配到一个路由时， 参数会被设置到 $route.params 中；url有查询参数，参数会被设置到 $route.query 中**

### 响应路由参数

使用路由参数，如 /user/a 导航到 /user/b，**原来的组件实例会被复用，因此组件的生命周期钩子不会被调用**

- watch $route 对象

```javascript
const User = {
	template: '...',
	watch: {
		$route(to, from) {
			// 对路由变化做出响应
		}
	}
}
```

- 使用 2.2 版本的 beforeRouteUpdate 钩子

```javascript
const User = {
	...
	beforeRouteUpdate(to, from, next) {
		// 对路由变化做出响应...
		// 最后调用 next()
	}
}
```

## 嵌套路由

组件可以包含嵌套的 router-view 组件，用于挂载匹配的子路由组件

子路由使用 children 属性配置，可以无限嵌套

**以 / 开头的嵌套路径会被当为根路径，因此子路由不用 / 开头， 当然子路由可以设 "" 来默认匹配子路由**

```javascript
const User = {
	template: '<div>user</div><router></router>'
}

const router = new VueRouter({
	routes: [{
		path: '/user/:id',
		component: User,
		children: [{
			// 当 /user/:id 匹配成功， UserHome 组件会被渲染在 User 组件中的 <router-view> 中
			// 可以设置默认匹配子路由
			path: '',
			component: UserHome
		}, {
			// 当 /user/:id/profile 匹配成功， UserHome 组件会被渲染在 User 组件中的 <router-view> 中
			path: 'profile',
			component: UserProfile
		}]
	}]
})
```

## 编程式导航

除了使用 router-link 组件创建 a 标签定义导航连接，还可以使用 router 的实例方法，通过编程代码实现

- **router.push(location, onComplete?, onAbort?)**

**注意: 在 Vue 实例内部， 通过 $router 访问路由实例，可以调用 this.$router.push**

点击 router-link 组件时，该方法会在内部调用， 点击 __<router-link :to="...">__ 等同于调用 __router.push(...)__

```javascript
// 传入字符串
this.$router.push('/home')

// 传入对象
this.$router.push({ path: '/home' })

// 命名路由
this.$router.push({ name: 'user', params: { userId: 123 } })

// 带查询参数 变成 /register?plan=private
this.$router.push({ path: 'register', query: { plan: 'private' } })

```

该方法会向 history 栈添加一个新的记录。该方法的参数可以是一个字符串路径，或者一个描述地址的对象。**如果提供了 path，params 会被忽略，但是 query 不会被忽略。同样的规则也使用 router-link 组件的 to 属性**。看下面的例子

```javascript
const userId = 123
this.$router.push({ name: 'user', params: { userId } }) // /user/123
this.$router.push({ path: '/user/${userId}' }) // /user/123
// 下面 params 不生效
this.$router.push({ path: '/user/${userId}', params: { userId } }) // /user
```

在 2.2.0+，在 router.push 和 router.replace 中提供了 onComplete 和 onAbort 回调作为第二个和第三个参数。这些回调会在导航成功完成（在左右异步钩子被解析之后）或终止（导航到相同路由、或在当前导航完成之前导航到另一个不同的路由）时进行相应调用。**如果目的路由和当前路由相同，只是参数发生改变（如 /user/2 -> /user/1），需要使用 beforeRouteUpdate 钩子来相应变化 或者 watch $route**

- **router.replace(location, onComplete?, onAbort?)**

该方法和 router.push 很像，唯一不同的是它不会向 history 栈添加新纪录，只是替换当前的 history 记录

- **router.go(n)**

该方法的参数是一个整数，即在 history 记录中前进或后退几步，类似 window.history.go(n)

```javascript
// 在浏览器记录中前进一步，等同 history.forward() history.go(1)
router.go(1)

// 在浏览器记录中后退一步，等同 history.back() history.go(-1)
router.go(-1)

// 前进3步
router.go(3)
```

router.push、 router.replace、router.go 其实仿照 window.history.pushState、window.history.replaceState 和 window.history.go

vue-router 的导航方法（push、replace、go）在不同路由模式下表现一致

## 命名路由

可以通过命名一个路由，方便进行链接或者跳转。可以再创建 router 实例时在 routes 配置中给某个路由命名

```javascript
const router = new VueRouter({
	path: '/user/:id',
	name: 'user',
	component: User
})
```

通过 router-link 组件 或者 router.push 方法都可以跳转

```vue
// 路由都导航到 /user/123
// 给 router-link 组件的 to 属性传递一个对象
<router-link :to="{ name: 'user', params: { userId: 123 }}"></router-link>
```

```javascript
// 路由都导航到 /user/123
router.push({ name: 'user', params: { userId: 123 } })
```

## 命名视图

想要在同级展示多个视图，而不是嵌套展示，需要使用命名视图。**如果 router-view 没有设置名字，默认是 default**

```vue
<router-view class="view1"></router-view>
<router-view class="view2" name="view2"></router-view>
<router-view class="view3" name="view3"></router-view>
```

一个 router-view 组件挂载渲染一个匹配的组件， 对于同一个路由，多个视图就需要多个组件，**在 router 实例的 routes 配置上 需要使用 components 而不是 component**

```javascript
const router = new VueRouter({
	routes: [{
		path: '/',
		components: {
			default: Foo,
			view1: Bar,
			view2: Baz
		}
	}]
})
```

## 重定向和别名

### 重定向

重定向也通过配置 routes 完成

```javascript
// 从 /a 重定向到 /b
const router = new VueRouter({
	path: '/a',
	redirect: '/b'
})

// 也可以重定向到一个命名路由
const router = new VueRouter({
	path: '/a',
	redirect: { name: 'foo' }
})

// 还可以是一个方法，动态返回重定向目标
const router = new VueRouter({
	path: '/a',
	redirect: to => {
		// 方法接受 目标路由 作为参数
		// return 重定向的 字符串路径/路径对象
	}
})
```

常见应用是添加默认跳转

```javascript
const router = new VueRouter({
	path: '/',
	redirect: '/b'
})
```

### 别名

重定向是指 访问 /a 时，URL 会被替换成 /b， 然后匹配路由 /b

**/a 的别名是 /b， 意味着访问 /b 时，URL 保持 /b， 但是匹配路由 /a, 就像访问 /a**

```javascript
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

看下面的案例

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<div><h1>Home</h1><router-view></router-view></div>' }
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const Baz = { template: '<div>baz</div>' }

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  routes: [
    { path: '/home', component: Home,
      children: [
        // absolute alias
        { path: 'foo', component: Foo, alias: '/foo' },
        // relative alias (alias to /home/bar-alias)
        { path: 'bar', component: Bar, alias: 'bar-alias' },
        // multiple aliases
        { path: 'baz', component: Baz, alias: ['/baz', 'baz-alias'] }
      ]
    }
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Route Alias</h1>
      <ul>
        <li><router-link to="/foo">
          /foo (renders /home/foo)
        </router-link></li>
        <li><router-link to="/home/bar-alias">
          /home/bar-alias (renders /home/bar)
        </router-link></li>
        <li><router-link to="/baz">
          /baz (renders /home/baz)</router-link>
        </li>
        <li><router-link to="/home/baz-alias">
          /home/baz-alias (renders /home/baz)
        </router-link></li>
      </ul>
      <router-view class="view"></router-view>
    </div>
  `
}).$mount('#app')
```

## html5 history 模式

vue-router 默认使用 hash 模式--使用 URL 的 hash 模拟一个完整的 URL, 当 URL 改变时页面不会重新加载

也可以改用路由 history 模式，该模式是利用 history.pushState API 完成 URL 跳转而无须重新加载页面。访问是就像正常 URL，而不是 host#/

- **首先通过生成 router 实例传入 mode 属性**

```javascript
const router = new VueRouter({
	mode: 'history',
	routes: [...]
})
```

- **后端配置**

如果没有进行后端配置，在浏览器直接访问会报 404，可以再后端增加一个覆盖全部情况的候选资源: **如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面**

- 原生 Node.js

```javascript
const http = require('http')
const fs = require('fs')
const httpPort = 80

http.createServer((req, res) => {
  fs.readFile('index.htm', 'utf-8', (err, content) => {
    if (err) {
      console.log('We cannot open "index.htm" file.')
    }

    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    })

    res.end(content)
  })
}).listen(httpPort, () => {
  console.log('Server listening on: http://localhost:%s', httpPort)
})
```

- 基于 Node.js 的 express 考虑使用 connect-history-api-fallback 中间件

**更优化的情况应该是覆盖全部的路由情况，给出一个 404 页面; 或者使用 Node.js,通过后端匹配 URL，没有匹配到路由的时候返回 404**

```javascript
const router = new VueRouter({
	mode: 'history',
	routes: [{
		path: '*',
		component: NotFoundComponent
	}]
})
```

## 导航守卫 （导航 表示路由正在改变）

vue-router 提供导航守卫只要用来通过跳转或取消的方式守卫导航。可以有不同机会植入路由导航过程中: **全局，单路由独享， 组件级**

**注意: 参数或查询的改变不会触发 进入/离开的导航守卫，可以通过 watch $route 对象，或者使用 beforeRouteUpdate 组件内守卫**

### 全局守卫

- **router.beforeEach**

属于 router 的方法，用于注册全局前置守卫

```javascript
const router = new VueRouter({
	...
})

router.beforeEach((to, from, next) => {
	// ...
})
```

该方法接受三个参数：

**to: Route:** 即将进入目标的 **路由对象**

**from: Route:** 当前导航正要离开的路由

**next: function:** 一定要调用该方法来 resolve 这个钩子，执行效果依赖 next 方法的参数

**最后一定要调用 next() ，否则钩子不会被 resolve **

```javascript

// 进入管道的下一个钩子
next()

//中断当前导航。如果浏览器的 URL 改变了（可能使用户手动或者浏览器后退按钮），那 URL 地址会中重置到 from 路由对应的地址
next(false)

//跳转到一个不同的地址。当前的导航被终端，然后进行一个新的导航。可通过向 next 传递任意位置对象，且允许设置诸如 replace: true、 name: 'home' 之类的选项以及任何在 router-link 的 to 属性或 router.push 中的选项
next('/') 或 next({path: '/'})
```


- **router.beforeResolve （2.5.0 新增）**

注册全局解析守卫， 和 router.beforeEach 类似，区别在导航被确认之前，**同时在所有组件内守卫和异步路由组件被解析之后**，解析守卫就被调用

- **router.afterEach**

注册全局后置守卫，和守卫不同，**这些钩子不会接受 next 函数，也不会改变导航本身**

```javascript
router.afterEach((to, from) => {
	// ...
})
```

### 路由独享守卫

可在路由配置上直接定义 beforeEnter 守卫，这些守卫和全局前置守卫 beforeEach 的方法参数是一样的

```javascript
const router = new VueRouter({
	routes: [{
		path: '/foo',
		component: Foo,
		beforeEnter: (to, from, next) => {
			// ...
		}
	}]
})
```

### 组件内守卫

- **beforeRouterEnter**
- **beforeRouterUpdate (2.2 新增)**
- **beforeRouterLeave**

```javascript
const Foo = {
	template: `...`,
	beforeRouteEnter(to, from, next) {
		// 在渲染该组件的对应路由被 confirm 前调用
		// 不 ！能 ！获取组件实例 this
		// 因为当守卫执行前，组件实例还没被创建
	},
	beforeRouteUpdate(to, from, next) {
		// 在当前路由改变， 但是该组件被复用时候调用
		// 举例来说，对一个带有动态参数的路径 /foo/:id， 在 /foo/2 和 /foo/1 之间跳转
		// 由于会渲染同样的 Foo 组件，因此组件实例会被复用。这个钩子就是在这种情况下呗调用
		// 可以访问组件实例 this
	},
	beforeRouteLeave(to, from, next) {
		// 导航离开该组件的对应路由时候调用
		// 可以访问组件实例 this
	}
}
```

**beforeRouteEnter 守卫不能访问 this, 因为守卫在导航确认前调用， 因此即将登场的新组建还没创建**

可通过传一个回调给 next 来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调用法的参数

```javascript
beforeRouteEnter(to, from, next) {
	next(vm => {
		// 通过 vm 访问组件实例
	})
}
```

**beforeRouteEnter 是支持 next 传递回调的唯一守卫。对于 beforeRouteUpdata 和 beforeRouteLeave 可以访问组件实例 this， 没有必要支持传递回调**

```javascript
beforeRouteLeave(to, from, next) {
	// 只能使用 this
	this.name = to.params.name
	next()
}
```

beforeRouteLeave 守卫通常用来禁止用户在还未保存修改前突然离开。**导航可以通过 next(false) 取消**

```javascript
beforeRouteLeave(to, from, next) {
	const answer = window.confirm('确定离开吗？您还有未保存的修改！')
	if (answer) {
		next()
	} else {
		next(false)
	}
}
```

### 完整的导航解析流程

- **导航被触发**
- **在失活的组件里调用离开守卫 beforeRouteLeave**
- **调用全局 beforeEach 守卫**
- **在复用的组件里调用 beforeRouteUpdate 守卫 （2.2+）**
- **在路由配置里调用 beforeEnter**
- **解析异步路由组件**
- **在被激活的组件里调用 beforeRouteEnter**
- **调用全局的 beforeResolve (2.5+)**
- **导航被确认**
- **调用全局的 afterEach 钩子**
- **触发 DOM 更新**
- **在创建后的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数**

## 路由元信息

定义路由时候可以配置 meta 字段。routes 配置中的每个路由对象为路由记录， 路由记录可以嵌套。 一个路由匹配到的所有路由记录会暴露为 $route 对象（包括导航守卫中的路由对象）的 $route.matched 数组中。**通过遍历 $route.matched 检查路由记录中的 meta 字段**

```javascript
// routes 配置
const router = new VueRouter({
	routes: [{
		path: '/foo',
		component: Foo,
		children: [{
			path: 'bar',
			component: Bar,
			// meta 字段
			meta: {
				requiresAuth: true
			}
		}]
	}]
})

// 全局守卫中检查
router.beforeEach((to, from, next) => {
	if (to.matched.some(record => record.meta.requiresAuth)) {
		// 该 route 需要 require auth，判断是否登录，没有登录则重定向到登录页面
		if (!auth.loggedIn()) {
			next({
				path: '/login',
				query: {
					redirect: to.fullPath
				}
			})
		} else {
			next()
		}
	} else {
		next() // 确保一定调用 next()
	}
})
```

## 过渡动效

### 所有路由过渡

可以直接给 router-view 组件添加过渡

```html
<transition>
	<router-view></router-view>
</transition>
```

### 单个路由的过渡

每个路由组件有自己的过渡效果，可在各路由组件内使用 transition 组件并设置不同的 name

```javascript
const Foo= {
	template: `
		<transition name="slide">
			<div class="foo">...</div>
		</transition>
	`
}
```

### 基于路由的动态过渡

还可以基于当前路由和目标路由的变化关系，动态设置过渡

```vue
// 使用动态 transition name
<transition :name="transitionName">
	<router-view></router-view>
</transition>

// 在父组件中 watch $route 决定使用哪种过渡
watch: {
	'$route' (to, from) {
		const toDepth = to.path.split('/').length
		const fromDepth = from.path.split('/').length
		this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
	}
}
```

## 数据获取

通常有两种形式从服务器获取用户数据

- **导航完成之后获取:** 先完成导航，然后在接下来的组件生命周期钩子中获取数据，在数据获取期间显示“加载”中之类的提示 **常用这种形式**

- **导航完成之前获取:** 导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航

### 导航完成后获取数据

```vue
<template>
  <div class="post">
    <div class="loading" v-if="loading">
      Loading...
    </div>

    <div v-if="error" class="error">
      {{ error }}
    </div>

    <div v-if="post" class="content">
      <h2>{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
export default {
  data () {
    return {
      loading: false,
      post: null,
      error: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.error = this.post = null
      this.loading = true
      // replace getPost with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}
```

### 导航完成前获取数据

```vue
export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  beforeRouteUpdate (to, from, next) {
    this.post = null
    getPost(to.params.id, (err, post) => {
      this.setData(err, post)
      next()
    })
  },
  methods: {
    setData (err, post) {
      if (err) {
        this.error = err.toString()
      } else {
        this.post = post
      }
    }
  }
}
```

## 路由懒加载

通过 vue 异步组件 结合 webpack 代码分割实现

### 异步组件

异步组件被定义为返回一个 promise 工厂函数，该函数返回的 promise 是 resolve 本身

```javascript
const Foo = () => Promise.resolve({
    ...
    // 定义组件本身
})
```

### webpack 代码分割

**webpack 2 使用动态 import 语法定义代码分割点**
es6 定义 import() 为一个在运行时能动态加载 es6 模块的方法，
webpack把 import() 作为代码分割点， 将需求模块放到单独的chunk代码块中。import() 返回一个 Promise。 **注意，import() 时必须指向模块的 .default 值，因为它才是 promise 处理后返回的 module 对象** 

```javascript
import('./foo.vue) // 返回 Promise
```

综上，两者结合能定义一个被webpack代码自动分割的异步组件, 其他路由配置不需要改变

```javascript
const Foo = () => import('./foo.vue')

const router = new VueRouter({
    route: [{
        path: '/foo',
        component: Foo
    }]
})
```