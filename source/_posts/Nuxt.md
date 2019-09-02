---
(title: Nuxt
tag:
  - Nuxt
  - vue ssr
---

> 参考：[Nuxt 官网](https://zh.nuxtjs.org/guide)

## Nuxt

### nuxt渲染流程

plugin(执行1次) => nuxtServerInit(执行1次) => middleware => validate =>  page(middleware/asyncData) => render



1. 当用户访问应用程序, 如果 store 中定义了 nuxtServerInit action，Nuxt.js 将调用它更新 store。

2. 将加载即将访问页面所依赖的任何中间件。Nuxt 首先从 nuxt.config.js 这个文件中，加载全局依赖的中间件，之后检测每个相应页面对应的布局文件 ，最后，检测布局文件下子组件依赖的中间件。以上是中间件的加载顺序。

3. 如果要访问的路由是一个动态路由, 且有一个相应的 validate() 方法路由的validate 方法，讲进行路由校验。

4. Nuxt.js 调用 asyncData() 和 fetch() 方法，在渲染页面之前加载异步数据。asyncData() 方法用于异步获取数据，并将 fetch 回来的数据，在服务端渲染到页面。 用 fetch() 方法取回的将数据在渲染页面之前填入store。

5. 将所有数据渲染到页面。

![nuxt-schema](https://zh.nuxtjs.org/nuxt-schema.svg)

plugin(执行1次) => nuxtServerInit(执行1次) => middleware => validate =>  page(middleware/asyncData) => render

### nuxt安装

运行 create-nuxt-app

```bash
 npx create-nuxt-app <项目名>
 # 或者 yarn
 yarn create nuxt-app <项目名>
```

>**assets**:  资源目录用于组织未编译的静态资源文件，会被打包
>
>**components**:  组件目录用于组织应用的 vue 组件，Nuxt 不会扩展增强该目录下的组件，即这些组件不会像页面组件具有 `asyncData` 方法等特性
>
>**layouts**: 布局目录用于组织页面的布局组件
>
>**middleware**：中间件目录用于存放应用的中间件
>
>**plugins**:  插件目录用于存放在 `根vue.js应用 `  实例化之前需要运行的插件
>
>**static**：静态文件目录的文件不会被 Nuxt 调用 webpack 进行构建编译处理。服务器启动时该目录的文件会映射到应用的根路径 /
>
>**store**: `store` 目录用于组织应用的 vuex 文件。Nuxt 集成了 vuex 相关功能配置，只需要在目录下创建一个 index.js 文件即可激活这些配置
>
>**nuxt.config.js**: 用于组织 Nuxt 的个性化配置，以便覆盖配置

### 路由

pages目录中所有 `*.vue` 文件自动生成应用的路由配置

- pages/detail.vue 详情页
- pages/admin.vue 商品管理页
- pages/login.vue 登录页

### 动态路由

**下划线作为前缀**的 vue 文件或者目录会被定义为动态路由，如下的文件结构

```bash
pages/
--| detail/
----| _id.vue
```

> 如果 detail/ 不存在 index.vue, :id 将作为可选参数

### 嵌套路由

创建子嵌套路由，需要创建一个 .vue 文件，同时创建一个和该文件同名的目录用于存放子视图组件

```bash
pages/
--| index/
----| _id.vue
--| index.vue
```

生成路由配置

```bash
  path: "/",
  name: "index",
  ...
  children: [{
    path: ":id?",
    name: "index-id",
    ...
  }]
```

### 导航

添加路由导航，layouts/default.vue

```vue
<nav>
<nuxt-link to="/">首页</nuxt-link>
<!--别名：n-link，NLink，NuxtLink-->
<NLink to="/admin">管理</NLink>
</nav>
```

> 禁用预加载 `<n-link no-prefetch>page not pre-fetched</n-link>`

### 视图

#### 默认布局

`layouts/default.vue`

```vue
  <template>
    <nuxt/>
  </template>
```

#### 自定义布局

创建空白布局页面 `layouts/blank.vue` ，用于login.vue

```vue
  <template>
    <div>
      <nuxt />
    </div>
  </template>
```

页面 `pages/login.vue` 使用自定义布局

```vue
  export default {
    layout: 'blank'
  }
```

#### 自定义错误页

创建`layouts/error.vue`

```vue
  <template>
    <div class="container">
      <h1 v-if="error.statusCode === 404">页面不存在</h1>
      <h1 v-else>应用发生错误异常</h1>
      <nuxt-link to="/">首 页</nuxt-link>
    </div>
  </template>
  <script>
  export default {
    props: ['error']
  }
</script>
```

### 页面

页面组件就是 vue 组件，Nuxt 只不过为这些组件添加了一些特殊配置项

如给页面添加标题和 meta

```vue
export default {
  head() {
    return {
      title: "课程列表",
      meta: [{ name: "description", hid: "description", content: "set page meta" }],
      link: [{ rel: "favicon", href: "favicon.ico" }],
    }
  }
}
```

### 异步数据获取

`asyncData` 方法可以在设置组件数据之前异步获取或处理数据，组件初始化之前，请求接口返回的数据会融合在 data 中，一并返回模版显示。服务端渲染，请求是在服务端发送的，由其他页面回退到该页面时，请求是在客户端发送

> 该方法在组件创建之前，因此不能访问 this

例子： 获取商品列表

#### 接口准备

- 安装依赖 `npm i koa-router koa-bodyparser -S`
- 创建接口文件 `server/api.js`

```js
const Koa = require('koa')
const app = new Koa()
const bodyparser = require("koa-bodyparser")
const router = require("koa-router")({ prefix: "/api" })
// 设置cookie加密秘钥
app.keys = ["some secret", "another secret"]
const goods = [
  { id: 1, text: "Web全栈架构师", price: 1000 },
  { id: 2, text: "Python架构师", price: 1000 }
]
router.get("/goods", ctx => {
  ctx.body = {
    ok: 1,
    goods
  }
})
router.get("/detail", ctx => {
  ctx.body = {
    ok: 1,
    data: goods.find(good => good.id == ctx.query.id)
  }
})
router.post("/login", ctx => {
  const user = ctx.request.body
  if (user.username === "jack" && user.password === "123") {
    // 将token存入cookie
    const token = 'a mock token'
    ctx.cookies.set('token', token)
    ctx.body = { ok: 1, token }
  } else {
    ctx.body = { ok: 0 };
  }
})
// 解析post数据并注册路由
app.use(bodyparser())
app.use(router.routes())
app.listen(8080, () => console.log('api服务已启动'))
```

#### 整合 axios

安装 @nuxtjs/axios 模块  `npm install @nuxtjs/axios -S`

配置 nuxt.config.js

```vue
  modules: [
    '@nuxtjs/axios',
  ],
  axios: {
    proxy: true
  },
  proxy: {
    "/api": "http://localhost:8080"
  }
```

测试代码，获取商品列表， index.vue

```vue
<script>
export default {
  async asyncData({ $axios, error }) {
  	const {ok, goods} = await $axios.$get("/api/goods");
    if (ok) {
      return { goods };
    }
    // 错误处理
    error({ statusCode: 400, message: "数据查询失败" });
  }
}
</script>
```

测试代码，获取商品详情，/index/_id.vue

```vue
<template>
  <div>
    <pre v-if="goodInfo">{{goodInfo}}</pre>
  </div>
</template>
<script>
export default {
  async asyncData({ $axios, params, error }) {
    if (params.id) {
      // asyncData中不能使用 this 获取组件实例
      // 但是可以通过上下文获取相关数据
      const { data: goodInfo } = await $axios.$get("/api/detail", { params });
      if (goodInfo) {
        return { goodInfo }
      }
      error({ statusCode: 400, message: "商品详情查询失败" })
    } else {
      return { goodInfo: null }
    }
  }
}
</script>
```

### 中间件

中间件会在一个页面或一个组件运行前执行定义好的函数，常用于权限控制、校验等任务

例子：管理员页面保护，创建 `middleware/auth.js`

```vue
export default function({ route, redirect, store }) {
  // 上下文中通过store访问vuex中的全局状态
  // 通过vuex中令牌存在与否判断是否登录
  if (!store.state.user.token) {
  	redirect("/login?redirect="+route.path);
  }
}
```

注册中间件，admin.vue

```vue
<script>
export default {
	middleware: ['auth']
}
</script>
```

> 全局 路由守卫可在 nuxt.config.js 的 router 配置
>
> ```vue
> router: {
> 	middleware: [auth]
> }
> ```

### 状态管理 vuex

应用根目录下如果存在 `store` 目录，Nuxt 会启动 vuex

Nuxt 有两种使用 store 方法：

- 普通：store/index.js 会返回一个 Vuex.Store 实例，`nuxtServerInit`  方法必须定义在该文件的 actions 中

- 模块：store 目录下的每个 .js 文件会被转换成为状态树指定命名的子模块（index 是根模块，相当设置了namespaced: true)）

Nuxt 提供了模块方式的简便写法：使用状态树模块化的方式，store/index.js 不需要返回 Vuex.Store 实例，和其他模块一样直接将 state、mutations 和 actions 导出暴露



例子： 用户登录和用户登录状态保存。创建 store/user.js

```js
export const state = () => ({
  token: ''
})

export const mutations = {
  SET_TOKEN(state, token) {
    state.token = token;
  }
}
export const getters = {
  isLogin(state) {
    return !!state.token
  }
}
export const actions = {
  login({ commit, getters }, u) {
    return this.$login(u).then(({ token }) => {
      if (token) {
        commit("SET_TOKEN", token)
      }
      return getters.isLogin
    })
  }
}
```

### 插件

Nuxt 在运行应用之前执行插件函数，只执行一次，对引入或设置 vue 插件、自定义模块、第三方模块时特别有用

例子： 接口注入，利用插件机制将服务接口注入组件实例、store 实例中，创建 plugins/api-inject.js

```js
  export default ({ $axios }, inject) => {
    inject("login", user => {
      return $axios.$post("/api/login", user);
    })
  }
```

注册插件，nuxt.config.js

```vue
plugins: [
	"@/plugins/api-inject"
]
```

登录页面逻辑， login.vue

```vue
<template>
  <div>
    <h2>用户登录</h2>
    <el-input v-model="user.username"></el-input>
    <el-input type="password" v-model="user.password"></el-input>
    <el-button @click="onLogin">登录</el-button>
  </div>
</template>

<script>
export default {
  layout: 'blank',
  data() {
    return {
      user: {
        username: "",
        password: ""
      }
    }
  },
  methods: {
    onLogin() { 
      this.$store.dispatch("user/login", this.user).then(ok => {
        if (ok) {
          const redirect = this.$route.query.redirect || "/";
          this.$router.push(redirect);
        }
      })
    }
  }
}
</script>
```

例子：添加请求拦截器附加 token，创建 plugins/interceptor.js

```js
export default function({ $axios, store }) {
  $axios.onRequest(config => {
    if (store.state.user.token) {
      config.headers.Authorization = "Bearer " + store.state.user.token
    }
    return config;
  })

  // 响应拦截，可做状态码处理
  $axios.onResponse(data => {
    console.log('data:::::::::', data)
  })
}
```

注册插件，nuxt.config.js

```vue
export default function({ $axios, store }) {
  $axios.onRequest(config => {
    if (store.state.user.token) {
      config.headers.Authorization = "Bearer " + store.state.user.token
    }
    return config;
  })

  // 响应拦截，可做状态码处理
  $axios.onResponse(data => {
    console.log('data:::::::::', data)
  })
}
```

### nuxtServerInit

在 store 的根模块 actions 定义 `nuxtServerInit` 方法，Nuxt 调用它时会将页面上下文对象作为第二个参数传给它。可以利用这个方法将一些服务端的数据传给客户端

```vue
export const actions = {
  // nuxtServerInit 只能定义在跟模块 actions 中
  // 参数1 actions 上下文， 参数2 组件 context 上下文
  // cookie-universal-nuxt 用于服务器和客户端无差别处理 cookie
  nuxtServerInit({ commit }, { app }) {
    const token = app.$cookies.get("token")
    if (token) {
      console.log("nuxtServerInit: token:"+token)
      commit("user/SET_TOKEN", token)
    }
  }
}
```

>安装依赖模块：cookie-universal-nuxt `npm i -S cookie-universal-nuxt`
>
>注册 nuxt.config.js `modules: ["cookie-universal-nuxt"]`

### Nuxt 常用配置项

#### 配置端口 ip

开发常会遇到服务地址端口被占用或者指定IP的情况。可以在根目录下的 package.json 里对 config 项进行配置

```bash
"config": {
	"nuxt": {
		"host": "127.0.0.1",
		"port": "1000"
	}
}
```

#### 全局 css

`nuxt.config.js`  中配置 css 字段，数组，每项是字符串

对于 `sass` 这类文件要预先安装  node-sass sass-loader 包

```js
css: [
  // 直接加载一个 Node.js 模块。（在这里它是一个 Sass 文件）
  'bulma',
  // 项目里要用的 CSS 文件
  '@/assets/css/main.css',
  // 项目里要使用的 SCSS 文件
  '@/assets/css/main.scss'
]
```

### 发布部署

#### 服务端渲染部署

先编译构建再启动 Nuxt 服务

```bash
npm run build
npm start
```

> 生成内容在 .nuxt/dist

#### 静态应用部署

Nuxt.js 可依据路由配置将应用静态化，使得我们可以将应用部署至任何一个静态站点主机服务商

```bash
npm run generate
```

> 注意渲染和接口服务器需要处于启动当中
>
> 生成内容在 .nuxt/dist



## 部分源码学习

### 开发环境服务器启动脚本

```js
const Koa = require('koa')
const consola = require('consola')
const { Nuxt, Builder } = require('nuxt')

const app = new Koa()

// Import and Set Nuxt.js options
const config = require('../nuxt.config.js')
config.dev = app.env !== 'production'

async function start () {
  // Instantiate nuxt.js
  // 实例化一个 Nuxt 类
  const nuxt = new Nuxt(config)

  const {
    host = process.env.HOST || '127.0.0.1',
    port = process.env.PORT || 3000
  } = nuxt.options.server

  // Build in development
  if (config.dev) {
    // 开发环境调用 Buildre 实例化对象的 build 方法进行文件打包构建
    const builder = new Builder(nuxt)
    await builder.build()
  } else {
    await nuxt.ready()
  }
	
  // 添加请求中间件
  app.use((ctx) => {
    ctx.status = 200
    ctx.respond = false // Bypass Koa's built-in response handling
    ctx.req.ctx = ctx // This might be useful later on, e.g. in nuxtServerInit or with nuxt-stash
    // 调用 nuxt render 方法
    nuxt.render(ctx.req, ctx.res)
  })

  app.listen(port, host)
  consola.ready({
    message: `Server listening on http://${host}:${port}`,
    badge: true
  })
}

start()

```

### 自动生成 routes

 `@nuxt/builder/dist/build.js` 中进行开发环境文件打包的 build 方法中用到一个 `generateRoutesAndFiles`方法，引入 glob 库对 pages 下的文件进行遍历，并进行了字符串的处理，最后将所有的 vue 文件地址，整个的项目地址和 pages 作为参数传给`createRoutes` 函数

```js
async generateRoutesAndFiles() {
    consola.debug('Generating nuxt files');

    // Plugins
    this.plugins = Array.from(this.normalizePlugins());

    const templateContext = new TemplateContext(this, this.options);

    await Promise.all([
      this.resolveLayouts(templateContext), // 处理 layouts
      this.resolveRoutes(templateContext), // 处理 routes
      this.resolveStore(templateContext), // 处理 store
      this.resolveMiddleware(templateContext) // 处理 middleware
    ]);

    await this.resolveCustomTemplates(templateContext);

    await this.resolveLoadingIndicator(templateContext);

    // Add vue-app template dir to watchers
    this.options.build.watch.push(this.globPathWithExtensions(this.template.dir));

    await this.compileTemplates(templateContext);

    consola.success('Nuxt files generated');
 }

// resolveRoutes 方法中对不同页面进行处理，最终调用 createRoutes 函数
async resolveRoutes({ templateVars }) {
    consola.debug('Generating routes...');
    ...
  	if (this._nuxtPages) {
      // 用户 pages 目录下页面文件
      // Use nuxt.js createRoutes bases on pages/
      const files = {};
      const ext = new RegExp(`\\.(${this.supportedExtensions.join('|')})$`);
      for (const page of await this.resolveFiles(this.options.dir.pages)) {
        const key = page.replace(ext, '');
        // .vue file takes precedence over other extensions
        if (/\.vue$/.test(page) || !files[key]) {
          files[key] = page.replace(/(['"])/g, '\\$1');
        }
      }
      templateVars.router.routes = utils.createRoutes({
        files: Object.values(files),
        srcDir: this.options.srcDir,
        pagesDir: this.options.dir.pages,
        routeNameSplitter: this.options.router.routeNameSplitter,
        supportedExtensions: this.supportedExtensions
      });
    } 
  	...
  }
```

`createRoutes`  函数中对传过来的所有文件地址进行遍历，再对每一个文件地址字符串处理，以中划线进行拼接。以此作为 route.name

```js
const r = function r(...args) {
  const lastArg = args[args.length - 1];
  if (startsWithSrcAlias(lastArg)) {
    return lastArg
  }
  return path.resolve(...args.map(normalize))
}
...
const createRoutes = function createRoutes({
  files,
  srcDir,
  pagesDir = '',
  routeNameSplitter = '-',
  supportedExtensions = ['vue', 'js', 'ts', 'tsx']
}) {
  const routes = [];
  files.forEach((file) => {
    const keys = file
      .replace(new RegExp(`^${pagesDir}`), '')
      .replace(new RegExp(`\\.(${supportedExtensions.join('|')})$`), '')
      .replace(/\/{2,}/g, '/')
      .split('/')
      .slice(1);
    
    // 如 ../index.vue => keys ['index']
    // 如 ../index/_id.vue => keys ['index', '_id']
    const route = { name: '', path: '', component: r(srcDir, file) };
    // route = { name: '', path: '', component: '/xxx/xxx.vue' }
    
    let parent = routes;
    keys.forEach((key, i) => {
      // remove underscore only, if its the prefix
      const sanitizedKey = key.startsWith('_') ? key.substr(1) : key;

      route.name = route.name
        ? route.name + routeNameSplitter + sanitizedKey
        : sanitizedKey;
      route.name += key === '_' ? 'all' : '';
      route.chunkName = file.replace(/\.(`${supportedExtensions.join('|')}`)$/, 	'');
      	
      // 对routes进行查找，这里就可以看出为什么使用嵌套路由要在同路径下再加一个同名的vue文件，它的判断条件就是在 routes 中找到 name：route.name 的集合

      // 如果有嵌套路由，暂时 route.path 为空，没有嵌套路由就直接以'/'拼接 route.path，这里就可以看到动态路由的合成原理，如果是动态路由，route.path将会以 : 替换 _ ,末尾加上 ?
      const child = parent.find(parentRoute => parentRoute.name === route.name);

      if (child) {
        child.children = child.children || [];
        parent = child.children;
        route.path = '';
      } else if (key === 'index' && i + 1 === keys.length) {
        route.path += i > 0 ? '' : '/';
      } else {
        route.path += '/' + getRoutePathExtension(key); // route.path 以 : 替换 _ ,末尾加上 ?

        if (key.startsWith('_') && key.length > 1) {
          route.path += '?';
        }
      }
    });
    parent.push(route);
  });
	
  sortRoutes(routes);
  return cleanChildrenRoutes(routes, false, routeNameSplitter)
}
```

将 route.name 和 route.path 都放入 routes 中，进行排序，路径短的先放入，最后调用 `cleanChildrenRoutes`函数，对嵌套路由进行处理。

至此对 routes 的 path 和 name 的命名的处理已经结束

### .nuxt

Nuxt 自动 watch 文件动态编译生成 .nuxt 目录文件， 看一下服务端渲染入口文件 `.nuxt/server.js`

```js
export default async (ssrContext) => {
  ...
  // 1. 服务端每次请求都会重新实例化一个 route store app
  // Create the app definition and the instance (created for each request)
  const { app, router, store } = await createApp(ssrContext)
  const _app = new Vue(app)

  // Add meta infos (used in renderer.js)
  ssrContext.meta = _app.$meta()
  // Keep asyncData for each matched component in ssrContext (used in app/utils.js via this.$ssrContext)
  ssrContext.asyncData = {}
	...
  // 获取匹配访问路由的组件，得到组件数组
  // Components are already resolved by setContext -> getRouteData (app/utils.js)
  const Components = getMatchedComponents(router.match(ssrContext.url))
	
  // 2. 执行 store 入口文件 nuxtServerInit 方法
  /*
  ** Dispatch store nuxtServerInit
  */
  if (store._actions && store._actions.nuxtServerInit) {
    try {
      await store.dispatch('nuxtServerInit', app.context)
    } catch (err) {
      console.debug('Error occurred when calling nuxtServerInit: ', err.message)
      throw err
    }
  }
  ...
 	// 3. 执行全局 middleware
  /*
  ** Call global middleware (nuxt.config.js)
  */
  let midd = []
  midd = midd.map((name) => {
    if (typeof name === 'function') return name
    if (typeof middleware[name] !== 'function') {
      app.context.error({ statusCode: 500, message: 'Unknown middleware ' + name })
    }
    return middleware[name]
  })
  await middlewareSeries(midd, app.context)
  // ...If there is a redirect or an error, stop the process
  if (ssrContext.redirected) return noopApp()
  if (ssrContext.nuxt.error) return renderErrorPage()

  // 4. 设置 layout
  /*
  ** Set layout
  */
  // 优先组件 layout 选项读取
  let layout = Components.length ? Components[0].options.layout : NuxtError.layout
  if (typeof layout === 'function') layout = layout(app.context)
  // 加载 layout
  await _app.loadLayout(layout)
  if (ssrContext.nuxt.error) return renderErrorPage()
  // 设置 layout
  layout = _app.setLayout(layout)
  ssrContext.nuxt.layout = _app.layoutName

  // 5. 执行页面级别中间件 middleware
  /*
  ** Call middleware (layout + pages)
  */
  midd = []
  layout = sanitizeComponent(layout)
  if (layout.options.middleware) midd = midd.concat(layout.options.middleware)
  Components.forEach((Component) => {
    if (Component.options.middleware) {
      midd = midd.concat(Component.options.middleware)
    }
  })
  midd = midd.map((name) => {
    if (typeof name === 'function') return name
    if (typeof middleware[name] !== 'function') {
      app.context.error({ statusCode: 500, message: 'Unknown middleware ' + name })
    }
    return middleware[name]
  })
  await middlewareSeries(midd, app.context)
  // ...If there is a redirect or an error, stop the process
  if (ssrContext.redirected) return noopApp()
  if (ssrContext.nuxt.error) return renderErrorPage()

  // 6. 执行页面路由校验
  /*
  ** Call .validate()
  */
  let isValid = true
  try {
    for (const Component of Components) {
      if (typeof Component.options.validate !== 'function') {
        continue
      }

      isValid = await Component.options.validate(app.context)

      if (!isValid) {
        break
      }
    }
  } catch (validationError) {
    // ...If .validate() threw an error
    app.context.error({
      statusCode: validationError.statusCode || '500',
      message: validationError.message
    })
    return renderErrorPage()
  }

  // ...If .validate() returned false
  if (!isValid) {
    // Don't server-render the page in generate mode
    if (ssrContext._generate) ssrContext.nuxt.serverRendered = false
    // Render a 404 error page
    return render404Page()
  }

  // If no Components found, returns 404
  if (!Components.length) return render404Page()

  // 7. 执行 asyncData 获取异步数据
  // Call asyncData & fetch hooks on components matched by the route.
  const asyncDatas = await Promise.all(Components.map((Component) => {
    const promises = []

    // Call asyncData(context)
    if (Component.options.asyncData && typeof Component.options.asyncData === 'function') {
      const promise = promisify(Component.options.asyncData, app.context)
      promise.then((asyncDataResult) => {
        ssrContext.asyncData[Component.cid] = asyncDataResult
        applyAsyncData(Component)
        return asyncDataResult
      })
      promises.push(promise)
    } else {
      promises.push(null)
    }

    // Call fetch(context)
    if (Component.options.fetch) {
      promises.push(Component.options.fetch(app.context))
    } else {
      promises.push(null)
    }

    return Promise.all(promises)
  }))

  if (process.env.DEBUG && asyncDatas.length)console.debug('Data fetching ' + ssrContext.url + ': ' + (Date.now() - s) + 'ms')

  // datas are the first row of each
  ssrContext.nuxt.data = asyncDatas.map(r => r[0] || {})

  // ...If there is a redirect or an error, stop the process
  if (ssrContext.redirected) return noopApp()
  if (ssrContext.nuxt.error) return renderErrorPage()

  // Call beforeNuxtRender methods & add store state
  await beforeRender()

  return _app
}

```

#### store

`.nuxt/store.js`

```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)

const VUEX_PROPERTIES = ['state', 'getters', 'actions', 'mutations']
let store = {}

void (function updateModules() {
 	// store 根文件
  store = normalizeRoot(require('../store/index.js'), 'store/index.js')

  // Enforce store modules
  store.modules = store.modules || {}

  resolveStoreModules(require('../store/user.js'), 'user.js')
  // If the environment supports hot reloading...
  
  // 模块热更新会重新执行
  if (process.client && module.hot) {
    // Whenever any Vuex module is updated...
    module.hot.accept([
      '../store/index.js',
      '../store/user.js',
    ], () => {
      // Update `root.modules` with the latest definitions.
      updateModules()
      // Trigger a hot update in the store.
      window.$nuxt.$store.hotUpdate(store)
    })
  }
})()

// createStore
// 生成 store 函数
export const createStore = store instanceof Function ? store : () => {
  return new Vuex.Store(Object.assign({
    strict: (process.env.NODE_ENV !== 'production')
  }, store))
}
```

`resolveStoreModules`  处理 modules,  `store/user.js`  中 state getters mutations actions 具名导出，这里可以看出将 `store/user.js`  作为一个模块，`mergeProperty(storeModule, moduleData[property], property)` 会将 state getters mutations actions 整合到对应模块中

```js
function resolveStoreModules(moduleData, filename) {
  moduleData = moduleData.default || moduleData
  // Remove store src + extension (./foo/index.js -> foo/index)
  const namespace = filename.replace(/\.(js|mjs|ts)$/, '')
  const namespaces = namespace.split('/')
  let moduleName = namespaces[namespaces.length - 1]
  const filePath = `store/${filename}`

  moduleData = moduleName === 'state'
    ? normalizeState(moduleData, filePath)
    : normalizeModule(moduleData, filePath)

  // If src is a known Vuex property
  if (VUEX_PROPERTIES.includes(moduleName)) {
    const property = moduleName
    const storeModule = getStoreModule(store, namespaces, { isProperty: true })

    // Replace state since it's a function
    mergeProperty(storeModule, moduleData, property)
    return
  }

  // If file is foo/index.js, it should be saved as foo
  const isIndexModule = (moduleName === 'index')
  if (isIndexModule) {
    namespaces.pop()
    moduleName = namespaces[namespaces.length - 1]
  }

  const storeModule = getStoreModule(store, namespaces)

  for (const property of VUEX_PROPERTIES) {
    mergeProperty(storeModule, moduleData[property], property)
  }

  if (moduleData.namespaced === false) {
    delete storeModule.namespaced
  }
}

function getStoreModule(storeModule, namespaces, { isProperty = false } = {}) {
  // If ./mutations.js
  if (!namespaces.length || (isProperty && namespaces.length === 1)) {
    return storeModule
  }

  const namespace = namespaces.shift()

  storeModule.modules[namespace] = storeModule.modules[namespace] || {}
  storeModule.modules[namespace].namespaced = true
  storeModule.modules[namespace].modules = storeModule.modules[namespace].modules || {}

  return getStoreModule(storeModule.modules[namespace], namespaces, { isProperty })
}
```

#### layout

从 `.nuxt/server.js` 中 看到设置 layout 其实是执行了 `.nuxt/App.js` 的 `setLayout ` 和 `loadLayout`  方法

```js
import _77180f1e from '../layouts/blank.vue'
import _6f6c098b from '../layouts/default.vue'

const layouts = { "_blank": _77180f1e,"_default": _6f6c098b }

export default {
 	...
	// 本质渲染 layoutEl
  render(h, props) {
    ...
    const layoutEl = h(this.layout || 'nuxt')
    const templateEl = h('div', {
      domProps: {
        id: '__layout'
      },
      key: this.layoutName
    }, [ layoutEl ])

    const transitionEl = h('transition', {
			...
		}, [ templateEl ])

    return h('div', {
      domProps: {
        id: '__nuxt'
      }
    }, [loadingEl, transitionEl])
  },
  data: () => ({
		...
    layout: null,
    layoutName: ''
  }),
  methods: {
    ...
    setLayout(layout) {
      if(layout && typeof layout !== 'string') throw new Error('[nuxt] Avoid using non-string value as layout property.')

      if (!layout || !layouts['_' + layout]) {
        layout = 'default'
      }
      this.layoutName = layout
      this.layout = layouts['_' + layout]
      return this.layout
    },
    // 加载异步 layout 组件
    loadLayout(layout) {
      if (!layout || !layouts['_' + layout]) {
        layout = 'default'
      }
      return Promise.resolve(layouts['_' + layout])
    }
  }
}

```

#### 组件

layout 组件一般都是会使用页面组件 `<nuxt/> `， 这个组件本质使用了嵌套路由组件 `<nuxt-child/>`

```vue
# layouts/default.vue
<template>
  <div>
    <nav>
      <!-- 推荐使用 nuxt-link -->
      <nuxt-link to="/">首页</nuxt-link>
      <!--别名：n-link，NLink，NuxtLink-->
      <NLink to="/admin"  no-prefetch>管理</NLink>
      <!-- <n-link to="/cart">购物车</n-link> -->
    </nav>
    <nuxt />
  </div>
</template>
```

#####  nuxt-child

整体功能其实是对 router-view 的封装

```vue	
<template>
  <div>
    <nuxt-child keep-alive :keep-alive-props="{ exclude: ['modal'] }" />
  </div>
</template>

<!-- 将被转换成以下形式 -->
<div>
  <keep-alive :exclude="['modal']">
    <router-view />
  </keep-alive>
</div>
```

`.nuxt/components/nuxt-child.js`

```js
export default {
  name: 'NuxtChild',
  functional: true,
  props: {
    nuxtChildKey: {
      type: String,
      default: ''
    },
    keepAlive: Boolean,
    keepAliveProps: {
      type: Object,
      default: undefined
    }
  },
  render(h, { parent, data, props }) {
    data.nuxtChild = true
    const _parent = parent
    const transitions = parent.$nuxt.nuxt.transitions
    const defaultTransition = parent.$nuxt.nuxt.defaultTransition
		// 嵌套层次处理
    let depth = 0
    while (parent) {
      if (parent.$vnode && parent.$vnode.data.nuxtChild) {
        depth++
      }
      parent = parent.$parent
    }
    data.nuxtChildDepth = depth
    
    const transition = transitions[depth] || defaultTransition
    const transitionProps = {}
    transitionsKeys.forEach((key) => {
      if (typeof transition[key] !== 'undefined') {
        transitionProps[key] = transition[key]
      }
    })

    const listeners = {}
    listenersKeys.forEach((key) => {
      if (typeof transition[key] === 'function') {
        listeners[key] = transition[key].bind(_parent)
      }
    })
    // Add triggerScroll event on beforeEnter (fix #1376)
    const beforeEnter = listeners.beforeEnter
    listeners.beforeEnter = (el) => {
      // Ensure to trigger scroll event after calling scrollBehavior
      window.$nuxt.$nextTick(() => {
        window.$nuxt.$emit('triggerScroll')
      })
      if (beforeEnter) return beforeEnter.call(_parent, el)
    }

    let routerView = [
      h('router-view', data)
    ]
    if (props.keepAlive) {
      routerView = [
        h('keep-alive', { props: props.keepAliveProps }, routerView)
      ]
    }
    return h('transition', {
      props: transitionProps,
      on: listeners
    }, routerView)
  }
}

const transitionsKeys = [
  'name',
  'mode',
  'appear',
  'css',
  'type',
  'duration',
  'enterClass',
  'leaveClass',
  'appearClass',
  'enterActiveClass',
  'enterActiveClass',
  'leaveActiveClass',
  'appearActiveClass',
  'enterToClass',
  'leaveToClass',
  'appearToClass'
]

const listenersKeys = [
  'beforeEnter',
  'enter',
  'afterEnter',
  'enterCancelled',
  'beforeLeave',
  'leave',
  'afterLeave',
  'leaveCancelled',
  'beforeAppear',
  'appear',
  'afterAppear',
  'appearCancelled'
]

```

##### nuxt-link

整体功能其实是对 router-link 的封装，增加了链接页面的预获取

```js
import Vue from 'vue'

export default {
  name: 'NuxtLink',
  extends: Vue.component('RouterLink'), // 继承全局注册的 router-link 组件
  props: {
    noPrefetch: {
      type: Boolean,
      default: false
    }
  }
}
```
