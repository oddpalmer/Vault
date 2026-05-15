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


我们之后都基于组合式去开发项目

## 基本

### 导入导出
#question 属于javascript的语法？ 是的

命名导出：一个文件可导出多个，名称必须一致
```javascript
// 导出
export const useTokenStore = defineStore('token',()=>{})

// 导入（必须用花括号 + 原名）
import { useTokenStore } from './stores/token'
```
默认导出：一个文件只能有一个，名称可自定义
```javascript
// 分开写
const useUserInfoStore = defineStore('userInfo',()=>{})
export default useUserInfoStore

// 匿名导出（导入时自己起名）
export default defineStore('userInfo',()=>{})

// 导入（不用花括号，名称可自定义）
import useUserInfoStore from './stores/userInfo'
import xxx from './stores/userInfo'  // 名字随便取
```
核心规则
- `export default` 后面跟**表达式/值**，不跟声明语句（`const/let/var`）
- 项目中推荐**命名导出**，IDE 支持更好，重构更安全

---
### Setup语法糖
#question 
>[!tips]+ 注意
>加上setup后，组件是默认导出！！！！

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
  history: createWebHistory(),  // 路由模式
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
  <!-- <RouterView /> 是占位出口，匹配到当前页面组件就会渲染在这里 -->
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
```js
// router/index.js
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




好的，我帮你整理补全这份笔记：

---

# 组件通信

## props
组件对外暴露的**输入接口**，用于父组件向子组件传递数据。
- 父组件：数据的**拥有者**
- 子组件：数据的**使用者**

这就是 Vue 的**单向数据流**（One-Way Data Flow）。
理解：单向数据流 = 数据只能由"拥有者"修改，其他组件只能通过约定方式请求修改。

父组件传值
```vue
<template>
  <!-- 不加冒号 → 传字符串常量 -->
  <ChildComponent message="你好" />

  <!-- 加冒号 → 传变量、表达式、数字、布尔等 -->
  <ChildComponent :count="10" :user="userObj" :active="true" />
</template>
```

子组件接收
```js
// 方式1：字符串数组（最简单，无类型约束）
defineProps(['message', 'count'])

// 方式2：对象（限定类型）
defineProps({
  message: String,
  count: Number
})

// 方式3：完整配置（推荐，有校验）
defineProps({
  message: {
    type: String,
    required: true,      // 必填
    default: '默认消息'  // 默认值
  },
  count: {
    type: Number,
    validator: (val) => val > 0  // 自定义校验函数
  }
})

// 方式4：TypeScript 类型声明（最简洁，纯类型约束）
const props = defineProps<{
  message?: string      // ? → 可选，父组件可以不传
  count: number         // 必传
  data?: object         // 可接收 undefined
}>()
```

子组件使用
```vue
<template>
  <h1>父组件传入的 message = {{ message }}</h1>
  <h2>父组件传入的 count = {{ count }}</h2>
</template>
```

虽然默认是单向通信，但props可以通过以下方式让父组件的数据发生变化：
```vue
<!-- 父组件 -->
<template>
  <Child :on-update="handleUpdate" />
</template>
<script setup>
const count = ref(0)
const handleUpdate = (val: number) => { count.value = val }
</script>

<!-- 子组件 -->
<script setup>
const props = defineProps<{ onUpdate: (val: number) => void }>()
props.onUpdate(10)  // 调用父组件方法，间接修改
</script>
```



## emit
emit 是子组件向父组件**发出事件**的机制。
- 子组件：**触发**事件（携带数据）
- 父组件：**监听**事件（接收数据并处理）
- 一定要用变量接收defineEmits方法才能使用
方向：**子 → 父**，与 props 相反。

emit 的三种声明方式
```vue
<script setup>
// 1. 纯运行时（无类型检查，最简单）
const emit = defineEmits(['change', 'update'])

// 2. 运行时校验（有验证逻辑）
const emit = defineEmits({
  change: (id: number) => id > 0,  // 返回 true/false 做校验
  update: null  // 不校验
})

// 3. 纯类型声明（仅 TypeScript 类型检查，Vue 3.3+）
const emit = defineEmits<{
  close: []                                         // 无参数
  submit: [data: { name: string; age: number }]     // 一个参数（对象）
  drag: [x: number, y: number]                      // 多个参数
  click: [MouseEvent]                               // 不具名参数
}>()
</script>
```

emit 语法详解（TS 类型声明）
- emit('事件名', 参数1, 参数2, 参数3, ...)
```typescript
defineEmits<{
  change: [id: number]
  //       ↑具名元组，参数名叫 id，类型是 number
  // ↑事件名
}>()

// 使用
emit('change', 123)  // ✅ 有参数名提示
emit('change')       // ❌ 缺少参数
```

- 键 = 事件名称
- 值 = 元组类型，代表参数列表
- 具名元组给参数起名，IDE 会有更好的提示

---

父组件监听 emit
```vue
<!-- 父组件 -->
<template>
  <Child
    @close="handleClose"
    @submit="handleSubmit"
    @drag="handleDrag"
  />
</template>

<script setup>
const handleClose = () => { /* ... */ }
const handleSubmit = (data: { name: string; age: number }) => { /* ... */ }
const handleDrag = (x: number, y: number) => { /* ... */ }
</script>
```

## props vs emit

|           | props               | emit                   |
| --------- | ------------------- | ---------------------- |
| **方向**    | 父 → 子，数据流入          | 子 → 父，事件冒泡             |
| **本质**    | 接收数据                | 触发事件                   |
| **父组件写法** | `:propName="value"` | `@eventName="handler"` |
| **子组件声明** | `defineProps`       | `defineEmits`          |
| **子组件角色** | 被动接收                | 主动通知                   |
| **数据修改权** | 只有父组件能改             | 子组件通知，父组件改             |

>[!note] 区别
> 记住一句话：props 拿进来，emit 发出去。

## v-model（语法糖）

两种用法
>[!tips]- [[Vue3#指令|html上的v-mode]]
>在普通 HTML 元素上 → 真正的双向绑定:
> - 页面改变，数据改变;
> - 数据改变，页面改变;

组件`v-model` 本质是 props + emit 的封装：
```vue
<!-- 父组件 -->
<Child v-model="count" />
<!-- 等价于 -->
<Child :modelValue="count" @update:modelValue="count = $event" />

<!-- 子组件 -->
<script setup>
defineProps<{ modelValue: number }>()
const emit = defineEmits<{ 'update:modelValue': [value: number] }>()
emit('update:modelValue', 10)
</script>
```




# md-editor-v3
[Vue3 中集成 md-editor-v3](https://cloud.tencent.com.cn/developer/article/2558580?from=15425&frompage=seopage)

