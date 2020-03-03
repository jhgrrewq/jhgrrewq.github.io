---
title: vue3 composition api 简易实现
tag: 
  - vue3
  - 源码
---

vue3 已经推出[Pre-Alpha 版本](https://github.com/jhgrrewq/vue-next), 除了 SSR、keep-alive、transition 和v-on/v-model/v-text/v-pre/v-once/v-html/v-show 等指令外，已完成大部分功能。现在我们对 [vue composition-api](https://vue-composition-api-rfc.netlify.com/) 尝鲜，并对其响应式做基本实现

<!-- more -->

## vue3 composition-api 尝鲜

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app"></div>
  <script src="./dist/vue.global.js"></script>
  <script>
    const { createApp, reactive, computed, effect } = Vue
    const App = {
      template: `
        <button @click="increment">
          Count is: {{ state.count }}, double is: {{ state.double }}
        </button>`,
      setup() {
        const state = reactive({
          count: 0,
          double: computed(() => state.count * 2)
        })
        function increment() { 
          state.count++ 
        }
        effect(() => { console.log('数据初次执行') })
        return {
          state,
          increment
        }
      }
    }
    createApp().mount(App, '#app')
  </script>
</body>
</html>
```

## 响应式简易实现

此处针对 reactive，computed, effect 做简易实现，涉及 es6 [Proxy](http://es6.ruanyifeng.com/#docs/proxy)、[Reflect](http://es6.ruanyifeng.com/#docs/reflect)、[WeakSet、WeakMap](http://es6.ruanyifeng.com/#docs/set-map) 等知识

### Proxy 存在的问题

- 数组的多次触发 ( 不是数组自身元素的属性修改也会触发 )

```js
var obj = [1]
var proxy = new Proxy(obj, {
  get: function(target, key) {
    console.log('获取值', key)
    return Reflect.get(target, key)
	},
  set: function(target, key, value) {
    console.log('设置值', key, value)
    return Reflect.set(target, key, value)
  }
})
proxy.push(2)
// 获取值 push
// 获取值 length
// 设置值 1 2
// 设置值 length 2
```

- 对象嵌套

```js
var obj = { location: { cityName: 'beijing' } }
var proxy = new Proxy(obj, {
  get: function(target, key) {
    console.log('获取值', key)
    return Reflect.get(target, key)
	},
  set: function(target, key, value) {
    console.log('设置值', key, value)
    return Reflect.set(target, key, value)
  }
})
proxy.location.cityName = 'shanghai'
// 获取值 location
// 对于 location 对象中 cityName 的获取和修改没有触发
```

### 目标功能

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app"></div>
  <button id="btn">点我</button>
  <script src="./vue3.js"></script>
  <script>
    const root = document.querySelector('#app')
    const btn = document.querySelector('#btn')
    
    const obj = reactive({
      name: 'jack',
      age: 10
    })
    const double = computed(() => obj.age * 2)
    // effect 执行时会将使用的 依赖存储到 effectStack 中
    effect(() => {
      root.innerHTML = `<div>${obj.name}今年${obj.age}岁, 乘以2是 ${double.value}岁</div>`
    })
    btn.addEventListener('click', () => {
      obj.age++
    }, false)
  </script>
</body>
</html>
```

### reactive 响应式

```js
// 双向缓存
const toProxy = new WeakMap() // 键值对 原始对象 => 响应对象
const toRaw = new WeakMap() // 键值对 响应对象 => 原始对象

/**
 * 判断是否是对象
 * @param val
 */
function isObject(val) {
  return typeof val === 'object'
}

/**
 * 判断属性是否是对象自身具有
 * @param val
 * @param key
 */
function hasOwn(val, key) {
  const hasOwnProperty = Object.prototype.hasOwnProperty
  return hasOwnProperty.call(val, key)
}

const baseHandler = {
  get(target, key, receiver) {
    const res = Reflect.get(target, key, receiver)
    // 收集依赖
    track(target, key)
    // 递归
    return isObject(res) ? reactive(res) : res // 暂时不考虑 null
  },
  set(target, key, val, receiver) {
    const hasKey = hasOwn(target, key)
    const oldValue = target[key] // 获取老数据
    val = toRaw.get(val) || val // 获取响应式新数据
    const res = Reflect.set(target, key, val, receiver)
    
    if (!hasKey) {
      // 没有设置过数据, 触发依赖更新
      trigger(target, key, val)
    } else if (val !== oldValue) {
      // 数据改变, 触发依赖更新
      trigger(target, key, val)
    }
    
    return res
  }
}

function reactive(target) {
  // 查询缓存
  let observed = toProxy.get(target)
  // toProxy 通过原始对象查找响应式对象，如果存在直接返回
  if (observed) {
    return observed
  }
  // toRaw 通过响应式对象查找原始对象，如果存在直接返回响应式对象
  if (toRaw.get(target)) {
    return target
  }
  // 不存在，生成 Proxy 对象
  observed = new Proxy(target, baseHandler)
  // 设置缓存
  toProxy.set(target, observed)
  toRaw.set(observed, target)

  return observed // 返回响应式对象
}
```

### track 收集依赖

```js
// 存储 effect
let effectStack = []
let targetMap = new WeakMap()
// tagetMap 结构
// {
//   target: {
//     age: [effect], (set)
//     name: [effect]
//   }
// }
function track(target, key) {
  let effect = effectStack[effectStack.length - 1]
  if (effect) {
    let depsMap = targetMap.get(target)
    if (depsMap === undefined) {
      depsMap = new Map()
      targetMap.set(target, depsMap)
    }
    let dep = depsMap.get(key)
    if (dep === undefined) {
      dep = new Set()
      depsMap.set(key, dep)
    }
    // 以上做初始化空值处理
    if (!dep.has(effect)) {
      dep.add(effect)
      effect.deps.push(dep)
    }
  }
}
```

### trigger 触发更新

将 computed 作为一种特殊 effect

```js
function trigger(target, key, info) {
  // 获取依赖
  const depsMap = targetMap.get(target)
  if (depsMap === undefined) return

  const effects = new Set()
  const computedRunners = new Set()
  if (key) {
    let deps = depsMap.get(key)
    deps.forEach(effect => {
      // 分别过滤出 computed 和 非 computed effect
      if (effect.computed) {
        computedRunners.add(effect)
      } else {
        effects.add(effect)
      }
    })
  }
  // 执行依赖
  const run = effect => effect()
  effects.forEach(run)
  computedRunners.forEach(run)
}
```

### effect

```js
function effect(fn, options = {}) {
  // 非 computed 会先执行一次
  let e = createReactiveEffect(fn, options)
  if (!options.lazy) {
    e()
  }
  return e
}

function createReactiveEffect(fn, options) {
  const effect = function(...args) {
    return run(effect, fn, args)
  }
  effect.deps = []
  effect.computed = options.computed
  effect.lazy = options.lazy
  return effect
}

function run(effect, fn, args) {
  if (effectStack.indexOf(effect) === -1) {
    try {
      effectStack.push(effect)
      return fn(...args)
    } 
    finally{
      effectStack.pop()
    }
  }
}

function computed(fn) {
  const runner = effect(fn, {
    computed: true,
    lazy: true
  })
  return {
    effect: runner,
    get value() {
      return runner()
    }
  }
}
```



