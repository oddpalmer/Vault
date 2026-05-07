# Vite
Vite 是一个现代化的前端构建工具(脚手架)；加快、规范vue开发
## 工程目录
```
my-vue-app/
├── .vite/                  # Vite 缓存目录
├── dist/                   # 构建产物
├── node_modules/           # 依赖包
├── public/                 # 静态资源（不参与编译）
├── src/                    # 核心源代码
│   ├── assets/             # 需要编译的资源
│   ├── components/         # 公共组件 (可被根组件组合)
│   ├── composables/        # 组合式函数（hooks）
│   ├── router/             # 路由配置
│   ├── hooks/              # 模块化管理功能
│   ├── stores/             # 状态管理（Pinia）
│   ├── views/pages              # 页面级组件
│   ├── App.vue             # 根组件
│   └── main.js             # 入口文件
├── index.html              # HTML 模板（项目入口）
│
├── vite.config.js          # Vite 配置文件
│
├── package.json            # 项目依赖与脚本
└── jsconfig.json           # 路径别名等编辑器配置
```
## 启动
Vite脚手架自动构建并启动我们的项目，并打包我们的代码，类似Maven
```shell
npm create vue@latest
cd <your-project-name> 
npm install 
npm run dev // 启动开发服务器
npm run build // 构建生产版本
npm run preview// 预览生产构建结果
```

## Vite配置文件
配置vite项目
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],                 // 引入 @vitejs/plugin-vue 插件，让 Vite 能够识别和编译 .vue 单文件组件
  resolve: {
    alias: {
      '@': './src'                  // 设置 @ 别名指向 src 目录，可以用 `@/xxx/A.vue` 代替 `../../xxx/A.vue`
    }
  },
  server: {  // Vite 开发服务器会代理转发到后端服务器
    proxy: {
      '/api': {
        target: 'http://localhost:8080',              // 代理到后端服务器所在URL源
        changeOrigin: true,                           // 是否修改URL源，修改请求头中的 origin 字段
        rewrite: (path) => path.replace(/^\/api/, '') // 将路径中的 /api 前缀去掉
      }
    }
  }
})
```
server.proxy: **代理规则**
- 当前端请求 `/api/user/login` 时 
```txt
浏览器请求: http://localhost:5173/api/user/login
         ↓
Vite 代理转发: http://localhost:8080/user/login
```

## main文件
应用入口文件，负责：
- 导入全局样式
- 创建 Vue 应用实例
- 配置 Element Plus UI 库（中文本地化）
- 配置 Vue Router
- 配置 Pinia 状态管理（带持久化插件）
- 挂载应用到 DOM
- .....
```typescript
// main.ts
import { createApp } from 'vue'  // 导入vue库中的createApp函数
import App from './App.vue'      // 引入App.vue文件

createApp(App).mount('#app')     // 选择App文件作为项目的根组件，并挂载到一个id = app的标签(容器)上启动	
/* 等价于
	const app = createApp(App)
	app.mount('#app')
*/ 
```

## html文件
```vue
<!-- index.html -->
  <body>
    <div id="app"></div>          // main.js挂载app的地方
    <script type="module" src="/src/main.ts"></script>
  </body
```

## Vue文件
```vue
<template>
	// 定义结构(html)
</template>

<script>
	// 定义行为(js/ts)
</script>

<style scoped>
	// 定义样式(css)
</style
```

## App.Vue
#question
一般负责渲染当前路由组件？
```vue
<template>
  <!-- 导入的组件以该格式使用 -->
  <Component1/>
  <Component2/>
</template>

<script setup lang="ts">
  import Component1  from './components/Component1.vue'
  import Component2  from './components/Component2.vue'
</script>

<style scoped>

</style>
```

## 总结
- `vite.config.js`：配置vite脚手架，api，别名，代理转发等
- `index.html`：是**项目入口**，项目最外层，单html文件；vite解析index.html `<script type="module" src="xxx">`指向**main.ts**文件
- `main.js`：导入并开启路由、状态管理、第三方组件库等等组件，最主要的就是管理全局组件，使用全局组件
- `Vue3`：通过createApp 函数创建应用实例


# Vue
## 选项式API
Vue 2常用，default里面全是配置的选项，**默认就是响应式数据**，将不同功能放在一起，分散、灵活程度低，不好管理

#响应式
>[!tips]- 响应式数据
>js数据实际被更改，也会动态的响应在页面上; `{{ xxx }}`插值表达式就是使用export dafault中的响应式变量

```vue
<script lang="ts">          // 说明脚本语言为 TypeScript
  export default {          // 默认导出组件配置对象，其他文件 import 时直接使用
    name: 'Person',         // 组件名，用于 DevTools 调试和递归组件自引用
    
    // -------- 数据 --------
    data() {                // 定义响应式数据，必须是一个返回对象的函数
      return {              // 返回的数据会被 Vue 处理成响应式（getter/setter 代理）
        name: "oddpalmer",  // 响应式属性，template标签中可用 插值表达式{{ name }} 获取/更改变量的属性
        age: 18,            
      }
    },
    
    // -------- 方法 --------
    methods: {              // 定义组件方法，可在模板事件中绑定，也可通过 this 互相调用
      greet() {             // 方法名
        console.log(`Hello, I'm ${this.name}`)  // 用 this.xxx 访问 data 中的数据
      },
      incrementAge() {
        this.age += 1       // 修改响应式数据，视图自动更新
      }
    },
  }
</script>
```
## 组合式API
基于setup, **内部定义的数据默认不是响应式数据**，要使用ref声明(什么是ref后续会提到)
注意不能在setup函数内部用this，但是和选项式配置项的data、method一起用的时候，可以在data、method...内部使用this.xxx去调用setup函数里面的变量
```vue
<script lang="ts">                                    // 说明脚本语言为 TypeScript
  import { ref } from 'vue'                           // 按需引入响应式 API
  export default {                                    // 默认导出组件配置对象（与 Options API 相同）
    name: 'Person',                                   // 组件名，用于 DevTools 调试
    setup() {                                         // 组合式 API 的入口函数，所有逻辑写在这里
      // -------- 数据 --------
      const name = ref<string>('oddpalmer')           // ref() 包装成响应式数据，可存任意类型
      const age = ref<number>(18)                     // 模板中自动解包写 {{ age }}，脚本中写 age.value
      // -------- 方法 --------
      function greet() {                              // 声明普通函数即可，不需要 methods 选项
        console.log(`Hello, I'm ${name.value}`)       // 脚本中访问 ref 必须加 .value
      }
      function incrementAge() {
        age.value += 1                                // 修改 ref 值也必须加 .value
      }
      // -------- 返回给模板 --------
      return {                                        // setup() 必须返回，模板才能使用
        name,                                         // ES6 简写，等价于 name: name
        age,
        greet,
        incrementAge
      }
    }
  }
</script>
```

图例对比
![[opts.gif]] 
![[compos1.gif]]![[compos2.gif]]


我们之后都基于组合式去开发项目，

## Setup语法糖

原来讲解组合式的时候，我们规定setup必须要返回值，但是在script标签加上setup属性，可以将定义过的变量、函数..自动返回。

省略了export default{}、setup(){}、name(组件名)、return；如果还想改变名字可以再写一个script但不要setup属性，单独声明组件名。或者下载插件，用属性在标签内部解决(并不重要)
```vue
<script setup lang="ts">                       // 加上setup属性，简化代码量                        
  import { ref } from 'vue'
  const name = ref<string>('oddpalmer') 
  const age = ref<number>(18) 
  
  function greet() { 
    console.log(`Hello, I'm ${name.value}`)
  }
  function incrementAge() {
    age.value += 1
  }
</script>
```


## 基本
### 指令
相当于在 HTML 里直接写 JS 逻辑，不用手动操作 DOM。更灵活更简便

```vue
<!-- 属性单向绑定 -->
<img v-bind:src="imgUrl" />              <!-- 完整 -->
<img :src="imgUrl" />                    <!-- 简写 -->

<div v-bind:class="{ active: isActive }"></div>  <!-- 完整 -->
<div :class="{ active: isActive }"></div>        <!-- 简写 -->

<!-- 属性双向绑定 -->
<input v-model="username" />
<!-- v-model 没有简写 -->


<!-- 事件 -->
<button v-on:click="handleClick">点击</button>   <!-- 完整 -->
<button @click="handleClick">点击</button>       <!-- 简写 -->

<form v-on:submit.prevent="onSubmit"></form>     <!-- 完整 -->
<form @submit.prevent="onSubmit"></form>         <!-- 简写 -->

<!-- 条件 -->
<p v-if="score > 60">及格</p>
<p v-else>不及格</p>
<!-- v-if / v-else 没有简写 -->

<!-- 列表 -->
<li v-for="(item, index) in list" v-bind:key="item.id">{{ item.name }}</li>  <!-- 完整 -->
<li v-for="(item, index) in list" :key="item.id">{{ item.name }}</li>        <!-- 简写 -->
```

>[!tips]
>- 单向绑定：JS 数据变了，页面跟着变；页面变了，JS 数据不变。
>- 双向绑定：JS 数据变了页面跟着变，用户输入改了页面，JS 数据也自动更新。**只用于表单元素。**
>- `v-if` 和 `v-for` 别放同一标签，用 `<template>` 包一层
>- `v-for` 必须加 `:key`，用唯一 id
>- 频繁切换用 `v-show`(条件为假自动设置display:none，一直存在)，不频繁用 `v-if`(每次为真都是单独的渲染)



### ref / reactive
ref： 
  - ref使基本类型，对象类型成为响应式
  - script访问：必须 .value
  - template访问：自动解包，不用 .value
  - 整体替换：可以，直接 xxx.value = 新值
reactive：
  - reactive把对象变成响应式
  - script访问：直接 .属性名
  - template访问：直接 .属性名
  - 整体替换：不行，一换就丢失响应式

```vue
<script setup>
// 引入
import { ref, reactive } from 'vue'

// ref：包装基本类型，访问要 .value
const name = ref('oddpalmer')
name.value = '新名字'

// reactive：包装对象，直接访问属性
const person = reactive({
  name: 'oddpalmer',
  age: 18
})
person.name = '新名字'   // 不用 .value
person.age = 20          // 直接改
</script>
```

**注意**
```vue
<script setup>
// 不能整体去引用一个新的对象，原对象是响应式，更换响应式就没了
let state = reactive({ count: 0 })
state = { count: 1 }

// ref 可以整体换
let state = ref({ count: 0 })
state.value = { count: 1 }  // 正常
</script>
```

**在开发者工具里面观察**
- ref定义的响应式对象，显示为RefImpl{...}, 其中value属性：Proxy(Object)
- reactive定义的响应式对象，显示为Proxy(Object)



### toRefs / toRef
#todo 

### Watch监视
#todo


### Computed计算属性
根据表单的已有数据**自动计算**出一个新值，**有缓存**，依赖不变就不重新算。

下面的代码就是一个例子，根据input输入姓、名，在span里面自动显示计算后的全名。在script层，fullname是**只读的**，想修改要**同时**声明get(),set()方法，在set方法里才能更改，不常用
```vue
<template>
	姓：<input type="text" v-model="firstName"><br>
	名：<input type="text" v-model="lastName"><br>
    全名:<span>{{ fullName }}</span>
    <!-- <button @click="fullName">{{ fullName }}</button> -->
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const firstName = ref('odd')
const lastName = ref('palmer')

// computed：有缓存，依赖不变直接返回上次结果
const fullName = computed(() => {
    console.log('计算了')        // 只有 firstName 或 lastName 变了才触发
    return firstName.value + ' ' + lastName.value
})

// methods：每次访问都重新执行
function getFullName() {
    console.log('执行了')        // 每次模板渲染都触发
    return firstName.value + ' ' + lastName.value
}

// 只读的，直接更改会报错
function changeFullName(){
	fullname.value = "xxx"  
}
</script>
```


### 生命周期
每个 `.vue` 文件都可以独立定义自己在各个生命周期阶段的行为。

这是在选项式 API 中，组件不同阶段自动执行的函数。

| 阶段     | Vue 2 钩子        | Vue 3 钩子（选项式）   | Vue 3 组合式（`setup` 内）      |
| ------ | --------------- | --------------- | ------------------------- |
| **创建** | `beforeCreate`  | `beforeCreate`  |                           |
|        | `created`       | `created`       | 在内部使用，比如定义函数和变量是在组件创建后执行的 |
| **挂载** | `beforeMount`   | `beforeMount`   | `onBeforeMount`           |
|        | `mounted`       | `mounted`       | `onMounted`               |
| **更新** | `beforeUpdate`  | `beforeUpdate`  | `onBeforeUpdate`          |
|        | `updated`       | `updated`       | `onUpdated`               |
| **卸载** | `beforeDestroy` | `beforeUnmount` | `onBeforeUnmount`         |
|        | `destroyed`     | `unmounted`     | `onUnmounted`             |


Vue3组合式
```vue
<script setup>
	import { onMounted, onUnmounted, ref } from 'vue'
	const count = ref(0)
	// 直接在 setup 顶层写，等价于 created
	console.log('组件创建了')
	
	onMounted(() => {
	  console.log('DOM 挂载完成，可以发请求、初始化图表')
	})
	
	onUnmounted(() => {
	  console.log('组件要销毁了，清理定时器、取消订阅')
	})
</script>
```


### Hooks
Vue3引入组合式 API 后的核心思想：
- 将原先不同功能的数据和方法，分离的文件放在hooks目录下实现模块化管理。即分别用单个 **js或ts** 文件中，带有返回值的函数进行管理。
- 可在 `setup` 或 `<script setup>` 中导入，并引用。

定义数据和功能
```typescript
// hooks/useCount.ts 
import {ref, reactive, onMounted} from 'vue'

export default function(){    -- 将该方法默认暴露出去
	let count = ref(0)
	
	function addCount(){
		count.value++
	}
	
	return {count, addCount} -- 返回数据和方法
}
```

component(组件)去引用功能
```vue
// component/Test.vue
<template>
    <h1>count is {{ count }}</h1>
    <button @click="addCount">+1</button>
</template>

<script setup lang="ts">
    import {ref, reactive} from 'vue'
    import useCount from '@/hooks/useCount';    // 导入Hooks

    const {count, addCount} = useCount()        // 解构对象属性
</script>

<style scoped>
</style>
```


## 异步执行
Vue 中的 **`async/await`**，是 JavaScript 处理异步操作的重要语法

javaScript 中有很多**需要等待**的操作，比如：
- 从服务器获取数据
- 读取文件
- 定时器
这些操作不会立即完成，如果直接写代码会出问题：
```js
// 错误示例：数据还没回来就想用
let data = fetchDataFromServer() // 需要1秒
console.log(data) // undefined，因为数据还没到
```

声明异步方法
```js
async function getData() {
  // 这个函数现在是异步的
}
```
等待结果
```js
async function getData() {
  const result = await fetchDataFromServer() // 会等数据返回才继续
  console.log(result) //  能拿到数据了，----await 只能在 async 函数里用
}
```

优点
```js
async function riskyOperation() {
  try {
	// 按顺序执行，每步都等前一步完成
    await step1()
    await step2()
    await step3()
  } catch (error) {
    // 任何一步出错都会被捕获
    console.error('出错了', error)
  }
}
```



## 路由

### 介绍
简单来说，Vue 的路由就是**管理页面跳转的一套规则和关系(界面和URL的映射)**。它让单页面应用(一个html文件)（SPA）也能像传统多页面网站一样，根据URL显示不同内容，但整个过程**无需刷新浏览器**。

Vue Router 是 Vue.js 官方的路由管理器，它能将不同的 URL 路径，映射到不同的 Vue 组件上，实现无刷新的页面切换。
>[!tips]- 工程规范
>路由组件就是组合了多个一般组件，直接展示在浏览器上的页面，一般放在pages或views目录下管理
>路由模式，一般就选用History，美观(URL不带#)，但要去后端调配, 项目首选；Hash模式省略

### 实现路由跳转
0. 安装vue-router路由包
```shell
npm install vue-router
```

1. 定义路由规则
```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'   // 引入
import Home from '@/pages/Home.vue'
import About from '@/pages/About.vue'

const routes = [
  { path: '/', component: Home },       // 访问 / 显示 Home
  { path: '/about', component: About }, // 访问 /about 显示 About
]

const router = createRouter({
  history: createWebHistory(),  // 路由模式，笔记在后方
  routes,                       // 组件和URL映射规则
})

export default router
```

2. 在main文件中启动路由功能
```ts
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'      // 引入刚才创建的路由实例
const app = createApp(App)

app.use(router)   // ✅ 必须这一步，把路由功能注入整个应用
app.mount('#app')
```

3. 定义路由组件
>[!note]- 标签用法 
> `<RouterView />` 自闭合，没内容时用这个
>`<RouterView></RouterView>` 成对标签，效果一模一样

在模板中使用
```vue
// views/Test.vue
<template>
  <!-- RouterLink 相当于一个特殊的 <a> 标签，会生成可点击的链接 -->
  <RouterLink to="/">首页</RouterLink>
  <RounterLink to="/about">关于</RouterLink>
  <!-- <RouterView /> 是占位出口，匹配到的页面组件就会渲染在这里 -->
  <RouterView />
</template>
```

在script中使用
```vue
<script setup>
	import { useRouter, useRoute } from 'vue-router'
	const router = useRouter()  // 获取路由实例，用于执行跳转（push、replace、go 等）。
	const route = useRoute()    // 获取当前路由对象，用于读取参数（query、params等），是只读的响应式引用。
	                            // 跳转只用 `useRouter`，读参数才用 `useRoute`
	// 字符串路径
	function goHome() {
	  router.push('/home')
	}
	// 命名路由 + params
	function goDetail() {
	  router.push({ name: 'UserDetail' })
	}
	// 替换当前历史记录（无法返回）
	function replaceToHome() {
	  router.replace('/home')
	}
	// 前进/后退
	function goBack() {
	  router.go(-1)   // 后退一步
	}
	function goForward() {
	  router.go(1)    // 前进一步
	}
</script>
```

router.push() 的作用是**导航到一个新的 URL**，并在浏览器历史记录栈中添加一条新记录。
```txt
// 当前你在 /home 页面
	router.push('/about')
// → URL 变成 /about，页面切换到 About 组件
// → 浏览器历史记录：/home → /about
// → 用户可以点“后退”回到 /home
```

router.replace()跳转并替换当前记录当前记录被覆盖,无法后退
```txt
// 当前在 /login
	router.replace('/home')
// → 历史记录中的 /login 被替换成 /home
// → 用户无法后退回 /login
```

### 路由命名
给路由起个名字，跳转时可以用名字代替路径，避免到处硬编码 URL, 更灵活

定义时加 `name`：
```typescript
// router/index.js
const routes = [
  {
    path: '/user/id/profile',   // 写死的路径
    name: 'userProfile',   // 起个名字
    component: UserProfile,
  },
]
```

```vue
<!-- 声明式 -->
<RouterLink :to="{ name: 'userProfile'}">
  用户资料
</RouterLink>
```

### 嵌套路由
嵌套路由(Nested Routes)就是构建带有公共布局的多页面应用(但其实还是单个html)。

路由规则如下
```vue
// router/index.js
<script>
const routes = [
  { path: '/login', component: LoginVue },  // 独立页面，无布局
  
  {
    path: '/',                              // 父路由
    component: LayoutVue,                   // 使用布局组件
    redirect: '/article/manage',            // 路由重定向：指定当访问父路径时，自动跳转到哪个children子路由。
    children: [                             // 所有子路由都在 LayoutVue 中渲染
      { path: '/article/category', component: ArticleCategoryVue },
      { path: '/article/manage', component: ArticleManageVue },
      { path: '/user/info', component: UserInfoVue },
      { path: '/user/avatar', component: UserAvatarVue },
      { path: '/user/resetPassword', component: UserResetPasswordVue }
    ]
  }
]
</script>
```

结构示例
```vue
<template>
  <div class="layout">
    <!-- 公共头部【固定】 -->
    <header>网站头部</header>
    
    <!-- 公共侧边栏【固定】 -->
    <aside>侧边导航</aside>
    
    <!-- 子路由页面在这里渲染【固定】 -->
    <main>
      <routerView />  <!-- 重要！子组件显示的位置【只有这里会变】 -->     
    </main>
    
    <!-- 公共底部 【固定】-->
    <footer>网站底部</footer>
  </div>
</template>
```

相当于 当用户访问 "/" 时
```text
访问: http://localhost:5173/
  ↓ 自动重定向
实际显示: http://localhost:5173/article/manage  (URL 会改变)

// 场景1：访问首页
用户输入: /
实际跳转: /article/manage
显示效果: LayoutVue(布局) + ArticleManageVue(文章信息内容)

// 场景2：直接访问子路由
用户输入: /user/info
显示效果: LayoutVue(布局) + UserInfoVue(用户信息内容)

// 场景3：访问登录页
用户输入: /login
显示效果: 只有 LoginVue（无布局，全屏登录页）
```

#question
>[!note]- 问题
>这种设计模式常用于：
>- **后台管理系统**：左侧导航 + 顶部栏 + 内容区
>- **需要登录验证**：LayoutVue 中可以检查登录状态
>- **统一布局**：多个页面共享相同的界面结构
你的路由配置实现了一个典型的后台管理布局：访问根路径自动跳转到文章管理页，所有功能页面都在同一个布局框架内显示。


### query传参

>[!tips]- 需要导入
>const route = useRoute() 获取当前路由对象，用于读取参数（query、params等），是只读的响应式引用

跳转,URL：`/user?id=1&name=张三`
```javascript
router.push({ path: '/user', query: { id: 1, name: '张三' } })
```
接收：
```javascript
route.query.id    // 1
route.query.name  // '张三'
```
特点：参数在 `?` 后面，跳转用 path 或 name 都行，**路由配置不用改**。

### params传参

路由配置
```javascript
{ path: '/user/:id/:name', name: 'UserDetail' }
```

跳转(必须用 name),URL：`/user/2/张三`
```javascript
router.push({ name: 'UserDetail', params: { id: 2, name: '张三' } })
```

接收：
```javascript
route.params.id    // 2
route.params.name  // '张三'
```

特点：参数成为路径的一部分，必须用 name 跳转，路由必须配占位符 `:xxx`。参数可选加 `?`：`/:name?`。

>[!note]- 场景选择 
>搜索筛选用 query，唯一资源标识用 params



# Axios插件

**唯一作用就是发 HTTP 请求、收响应**。前端通过它向后端 API 要数据（GET）或提交数据（POST/PUT/DELETE），然后把返回的数据更新到页面上，整个过程**不刷新页面**，这就是 AJAX 思想的具体落地。是前端页面和后端服务器之间数据的传输者。值得注意的是使用Axios一定会遇见的[[零散碎片待整理#跨域问题|跨域问题介绍]]
```shell
npm install axios                 // 下载Axios包到项目的node_modules目录：
import axios from 'axios'         // 引入到script中
```

只展示别名语法(简单易用，行数少)
```javascript
<script setup lang="ts">
	axios.get("url", {
		// config设置,规定如何传参，请求头配置等等
		param: {
			// GET请求参数必须放在params里，axios会自动拼接到URL末尾
			// 例如 /users?page=1&pageSize=10&keyword=张三
			....
		}
	}).then(result=>{
		// 请求成功处理
		xxx = result.data
	}).catch(err=>{
		//错误处理
		console.log(err)
	})
</script>
```


# Pinia插件

Pinia 是 Vue 的全局状态管理库，解决跨组件共享状态的问题。你可以把它理解为整个应用都能访问的“数据中心”。

**核心场景：**
- 用户登录信息（头像、昵称）需要在导航栏、个人中心、评论框等多个不相关组件中显示和修改
- 购物车数据在商品列表、购物车图标、结算页之间同步
- 多个组件需要访问和修改同一份数据，避免通过层层 props 传递

## 安装
#question
持久化插件被整合了？
```
npm install pinia
npm install pinia-plugin-persistedstate // 持久化插件，防止刷新数据丢失
```

## 创建实例并挂载
```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()
const app = createApp(App)
pinia.use(piniaPluginPersistedstate) // 必须！不然 persist 不生效
app.use(pinia)
app.mount('#app')
```

## 定义Store

>[!note]- 什么是Store
>就是一个存储和管理共享状态的地方，像一个全局的响应式数据仓库。任何组件都能读、能改；当 Store 里的数据变化时，所有用到该数据的组件视图都会自动重新渲染。defineStore 的返回值 useUserStore 是一个函数，调用它就能拿到这个仓库的实例。

核心是用 `defineStore` 定义一个函数，返回你想暴露的数据和方法。以下是使用pinia实现前后端登陆（组合式写法）
```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { loginAPI } from '@/api/user'
/*  defineStore('name', function)
    第一个参数:名字,唯一性
    第二个参数:函数,函数的内部可以定义状态的所有内容,有返回值 */
export const useUserStore = defineStore('user', () => {
  // --- 状态（用 ref/reactive） ---
  const userInfo = ref({name: '', token: '', role: 'guest'})
  // --- actions（普通函数） ---
  // 异步登录，后端校验账号密码，校验通过才返回 token 并设置登录状态
  async function loginAction(username, password) {
    const res = await loginAPI({ username, password })
    
    // 后端校验通过，才更新用户信息和登录状态
    if (res.code === 200) {
      userInfo.value.name = res.data.username
      userInfo.value.token = res.data.token
      userInfo.value.role = res.data.role
      isLoggedIn.value = true
    } else {
      throw new Error(res.message || '登录失败')  // 后端校验失败，保持未登录
    }
    
  }
  
  function logout() {
    userInfo.value = { name: '', token: '', role: 'guest' }
  }
  // --- 暴露出去 ---
  return {
    userInfo,
    loginAction,
    logout,
  }
},{
    persist: true
});
```

## 使用
```vue
<template>
  <!-- v-model 用于输入框双向绑定 store 的数据 -->
  <input v-model="username" placeholder="用户名" />
  <input v-model="password" placeholder="密码" />
  
  <!-- username 变了，这里自动显示新值，不需要任何指令，store内不需要computed()，除非有复杂的计算逻辑 -->
  <p v-if="userStore.isLoggedIn">当前用户：{{ userStore.userName }}</p>
  <p v-if="errorMsg">{{ errorMsg }}</p>
  
  <button @click="handleLogin">登陆</button>
  <button @click="userStore.logout()">退出</button>
</template>
<script setup>
  import { ref } from 'vue'
  import { useUserStore } from '@/stores/user'
  const userStore = useUserStore()
  const username = ref('')
  const password = ref('')
  const errorMsg = ref('')
  async function handleLogin() {
	  async function handleLogin() {
	  errorMsg.value = ''
	  try {
	    await userStore.loginAction(username.value, password.value)
	    // 走到这说明登录成功，不需要再判断 isLoggedIn
	  } catch (error) {
	    errorMsg.value = error.message || '账号或密码错误'
	  }
}
  }
</script>
```

## 解构storeToRefs
直接解构 store 会丢失响应式，必须用 `storeToRefs`：
```javascript
import { storeToRefs } from 'pinia'
// 变量这样解构，响应式没了
const { userName, isLoggedIn } = userStore

// 正确做法
const { userName, isLoggedIn } = storeToRefs(userStore)
// 函数直接解构不受影响
const { loginAction, logout } = userStore
```

# ElementPlus组件库
#todo

