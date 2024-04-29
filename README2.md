---
theme: cyanosis
highlight: vs2015
---

# 前言

随着手机性能的提升，如今移动端APP的用户体验越来越好了，页面流畅不卡顿，转场动画优雅又丝滑

而大部分的 `web` 页面，尽管功能上比起 `10` 年前 `jQuery`
时代丰富了不少，但似乎在使用体验上还是一尽难尽的地步，应用最广泛的动画可能是弹框组件了，毕竟 `UI` 库直接提供了动画

---

这是我的 [**模仿抖音**](https://juejin.cn/column/7357362143396118528) 系列文章的第四篇：这篇我们将实现页面缓存，就像我们访问传统新闻网站一样
> 第一篇：[200行代码实现类似Swiper.js的轮播组件](https://juejin.cn/post/7360512664317018146)  
> 第二篇：[实现抖音 “视频无限滑动“效果](https://juejin.cn/post/7361614921519054883)  
> 第三篇：[Vue 路由使用介绍以及添加转场动画](https://juejin.cn/post/7362528152777130025)

# 最终效果

在线预览：[dy.ttentau.top/](https://dy.ttentau.top/)

Github地址：[https://github.com/zyronon/douyin](https://github.com/zyronon/douyin)

路由定义源码：[routes.ts](https://github.com/zyronon/douyin/blob/master/src/router/routes.ts)  
转场动画源码：[App.vue](https://github.com/zyronon/douyin/blob/master/src/App.vue)

## 思考

- 如何判断是“前进”还是“后退”而使用不同的动画方式？

# 一、路由使用

## 1. 初始化路由

```js
import Vue from 'vue'
import { createRouter, createWebHashHistory, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHashHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { top: 0 }
    }
  }
})

...

const app = createApp(App)
app.use(router)
```

## 2. 定义路由

接下来就可以进行路由组件的映射：  
为了践行模块化组件化思想，组件是从其他文件import进来

---
可以 `静态` 导入(在文件顶部用 `import` 导入组件) 使用

```js
import Home from '../pages/home/index.vue'
```

静态导入的组件会 `全部` 默认打包到一个 `js` 文件中

--- 

也可以 `动态` 导入使用

```js
{
  path: '/home', component
:
  () => import('@/pages/home/index.vue')
}
,
```

动态的导入的 `每一个` 组件，会被单独打包成一个 `js` 文件，一对一的，定义了 `50` 个路由，就会打包出 `50` 个 `js`
文件（当然我们可以在 `vite.config.js` 或 `vue.config.js` 里面配置打包分割，将组件进行分组统一打包到一个 `js` 文件中）

---

```js
import Home from '../pages/home/index.vue'
import Test from '../pages/test/Test.vue'

//创建routes数组，然后传入 createRouter 的 ‘routes’ 配置中
const routes = [
  //每个路由映射一个组件
  { path: '/', redirect: '/home' },
  { path: '/test', component: Test },
  { path: '/home', component: Home },
```

## 3. 路由出口

```js 
App.vue

< router - view > < /router-view>
```

我们定义好了路由映射之后，还需要定义一个路由`出口`，即 `<router-view></router-view>`
，这个标签放在哪个位置，哪个位置就会显示当前所匹配路由映射的 `Vue` 组件

默认我们是放在 `App.vue` 里面，这样就可以访问所有定义的 `一级` 路由了

## 4. 嵌套路由

上面只是路由的普通使用，只能访问一级页面，但我们做后台、管理系统等网站，基本上都有嵌套路由的要求  
嵌套路由只需在 `children` 字段里放同样的路由映射即可

```js
{
  path: '/layout',
    component
:
  Layout,
    children
:
  [
    { path: '/layout-child1', component: LayoutChild1 },
    { path: '/layout-child2', component: LayoutChild1 },
    { path: '/layout-child3', component: LayoutChild1 },
  ]
}
,
```

再定义子路由 `出口`，同样还是 `<router-view></router-view>` 标签，我们知道这个标签放在哪里，就会显示当前匹配的组件  
我们在 `App.vue` 里面放了一个，那么 `App.vue` 就会显示当前所匹配路由映射的 `Vue` 组件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85d99428370e4547b9e053fc7a322a9c~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1211&h=600&s=46014&e=png&b=2c2c2c)

子路由的出口也是同样的，你想在在哪显示就放在哪里，我们的 `Layout.vue` 组件需要有子路由，那么就放在 `Layout.vue` 里面就行了

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18d7110fa2a043869bd5bbf4607a1101~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1150&h=565&s=42350&e=png&b=2b2b2b)

# 二、转场动画

我们需要用到 `Vue` 的 `Transition` 这个组件  
具体使用请看官网，这里不详细介绍  
https://cn.vuejs.org/guide/built-ins/transition.html

还用到了 `css` 的 `transform` 属性，它允许你旋转，缩放，倾斜或平移给定元素，我们用它来平移两个路由页面  
https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform

还用到了 `css` 的 `transition` 属性，它可以使在不同 `css` 属性状态切换的时候产生过渡效果(可以理解为简单动画)
，我们用它来给 `transform` 加过渡效果  
https://developer.mozilla.org/zh-CN/docs/Web/CSS/transition

## 1. Transition组件

```js
Vue3
< router - view
v - slot = "{ Component }" >
  < transition
:
name = "transitionName" >
  < component
:
is = "Component" / >
  < /transition>
</router-view>

Vue2
< transition
:
name = "transitionName" >
  < router - view > < /router-view>
</transition>
```

**需要注意的是，`Vue3` 和 `Vue2` 的写法不同**

- `transition` 中 `name` 属性用于定义动画， `Vue` 会根据提供的 `name` 值去寻找对应的 `css`
  类名，并在合适的时机添加、移除 `css` 类名
- `transition` 的子元素只能有一个，并且当子元素符合以下条件时，会自动触发
    - 由`v-if`所触发的切换
    - 由`v-show`所触发的切换
    - 由特殊元素`<component>`切换的动态组件
    - 改变特殊的`key`属性

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5da8717d561a4e6b978c12a30653ac80~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1126&h=780&s=79926&e=png&b=141516)

比如说，我们将 `name` 定义为 `test`，那么每次动画执行时，`Vue` 会自动给我们添加上图的 6 个 `css`
类名：`test-enter-from` `test-enter-active` `test-enter-to` `test-leave-from` `test-leave-active` `test-leave-to`

---

当我们前进时：  
**当前页面需要从 0（屏幕中间） 平移至 -100%（屏幕左侧） ，下个页面 需要从 100%（屏幕右侧） 平移至 0（屏幕中间）**

当我们后退时：  
**当前页面需要从 0（屏幕中间） 平移至 100%（屏幕右侧） ，下个页面 需要从 -100%（屏幕左侧） 平移至 0（屏幕中间）**

所以我们需要两种动画，一种是 `前进` ，一种是 `返回`，不同状态展示不同动画

具体使用什么动画，我们用一个变量 `transitionName` 来定义

## 2.动画

我们在前进时，当前页面和下个页面的需要执行的动画是不同的，后退时也一样

这里直接放最终的 `css` 代码，
下面解释为什么这样写

### 前进动画

```css
/
/页面最终/ 最初的状态：屏幕正中间
.go-enter-to,
.go-leave-from {
    transform: translate3d(0, 0, 0);
}

/
/
动画持续时间
.go-enter-active,
.go-leave-active {
    transition: all 0.3s;
}

/
/
离去动画，表示离去页面最终的css：屏幕左侧
.go-leave-to {
    transform: translate3d(-100%, 0, 0);
}

/
/
进入动画，表示进入页面，初始的状态：屏幕右侧
.go-enter-from {
    transform: translate3d(100%, 0, 0);
}
```

### 后退动画

```css
/
/
和上面同理，只时 css 是反着的而已
.back-enter-to,
.back-enter-from {
    transform: translate3d(0, 0, 0);
}

.back-enter-active,
.back-leave-active {
    transition: all 0.3s;
}

.back-enter-from {
    transform: translate3d(-100%, 0, 0);
}

.back-leave-to {
    transform: translate3d(100%, 0, 0);
}
```

## 3. 动画原理

**两个页面平移动画的原理：**

当我们前进时：  
**当前页面需要从 0（屏幕中间） 平移至 -100%（屏幕左侧） ，下个页面 需要从 100%（屏幕右侧） 平移至 0（屏幕中间）**

当我们后退时：  
**当前页面需要从 0（屏幕中间） 平移至 100%（屏幕右侧） ，下个页面 需要从 -100%（屏幕左侧） 平移至 0（屏幕中间）**

---

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e0c495db95024e2cbcc8e120785de9e0~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1903&h=1101&s=534472&e=png&b=292f31)

通过上图可以看到  
一、当我们导航到下一页时，两个路由页面最外层的 `div` 是平级的，所以当动画执行时，*
*会看到当前页面从屏幕中间向左平移出去了，但下个页面并没有从屏幕右侧向屏幕中间平移，而是在动画结束后，突然显示在屏幕中间**
。这是因为按照正常的文档流，两个页面最外层的 `div` 都是 `block` ，所以下个页面单起一行放在当前页面的正下方

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4eca4719d7a46cbbae1a5460590ae9e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=387&h=671&s=22122&e=webp&f=30&b=ffff01)

解决办法：给每个页面加个统一的 `class` ，将页面统一设置为 `position:absolute;`，此时页面脱离文档流，不占空间，这样就不会把下一页挤下去，完成平滑过渡

二、由图可知，离去的页面 `Vue` 会自动给我们加上 `xxx-leave-active, xxx-leave-to` 这两个 `class`  
而进入的页面，`Vue` 会自动给我们加上 `xxx-enter-active, xxx-enter-to` 这两个 `class`  
还有这 `xxx-leave-from, xxx-enter-from`两个 `class` 是动画一开始就加上，然后立刻又删除了，截图里面展示不出来😅

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ded4aeb71d8d49be8fb82d9ffd1982ee~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=962&h=1187&s=134873&e=png&b=ffffff)

所以，当我们知道 `Vue` 执行动画时有哪些 `class`，就可以对应的编写 `css` 过渡了

就拿 **前进** 时 **离去的页面** 举例：

- 动画开始：`go-leave-from`，然后又立马就删除了。所以我们可以在这个 `class` 编写用于启动过渡的 `css` ，用于表明这个页面的初始状态

```css
/
/
页面的初始状态
.go-leave-from {
    transform: translate3d(0, 0, 0);
}
```

- 动画行程中：`go-leave-active, go-leave-to`，我们在这里定义整个过渡的时间，以及我们页面最终的 `css` 的样式

```css
/
/
整个过渡过程需要300毫秒
.go-leave-active {
    transition: all 0.3s;
}

/
/
最终状态，当动画结束时，页面需要 平移到

-
100
%
的地方
.go-leave-to {
    transform: translate3d(-100%, 0, 0);
}
```

其他的动画，基本同理。需要注意的是，同一个动画中，离去的页面和进入的页面的 `translate3d` 是相反的

## 4. 区分是“前进”还是“后退”

`$route` 作为 `vue` 实例的一个响应式属性，和在 `data` 中写的属性本质上是一样的，都可以通过 `this`
的方式拿到。既然你可以监听 `data` 中的属性变化，同样也可以监听 `$route` 的变化。`watch`
中监听的对象默认回调函数中的参数值就是 `newVal, oldVal`。作为 `$route` 属性来说当然也就是 `to` 和 `from` 的概念了

`to、from` 是最基本的路由对象，分别表示从(`from`)某个页面跳转到(`to`)另一个页面，`to.path`
（表示要跳转到的路由地址），`from.path` 同理

**该功能的难点在于怎样分辨是“前进”还是“后退”？**

还记得我们前面定义的 `routes` 数组吗？我们现在需要将每个路由**按层级依次排序**(暂不考虑嵌套路由的情况)
，依次排列每个路由的下一页、下下页...即保证每个“上一页”在“下一页”前面

总结下来就是：根据 `routes` 数组里定义的路由目录的顺序，在 `watch` 到路由变更的时候，用 `to.path` 和 `from.path`
去 `routes` 数组里去查找下标，**下标靠前的是上一页，靠后的是下一页，可由此判断是“前进”还是“后退”**

### Vue2写法

```js
// watch $route 决定使用哪种过渡
watch: {
  '$route'(to, from)
  {
    //底部tab的按钮，跳转是不需要用动画的
    let noAnimation = ['/', '/home', '/slide', '/me', '/attention', '/message', '/publish']
    if (noAnimation.indexOf(from.path) !== -1 && noAnimation.indexOf(to.path) !== -1) {
      return this.transitionName = ''
    }

    const toDepth = routes.findIndex(v => v.path === to.path)
    const fromDepth = routes.findIndex(v => v.path === from.path)
    this.transitionName = toDepth > fromDepth ? 'go' : 'back'
  }
,
}
,
```

### Vue3写法

```js
const route = useRoute()

// watch $route 决定使用哪种过渡
watch(
  () => route.path,
  (to, from) => {
    //底部tab的按钮，跳转是不需要用动画的
    let noAnimation = ['/', '/home', '/slide', '/me', '/attention', '/message', '/publish']
    if (noAnimation.indexOf(from) !== -1 && noAnimation.indexOf(to) !== -1) {
      return transitionName.value = ''
    }
    const toDepth = routes.findIndex((v: RouteRecordRaw) => v.path === to)
    const fromDepth = routes.findIndex((v: RouteRecordRaw) => v.path === from)
    transitionName.value = toDepth > fromDepth ? 'go' : 'back'
  }
)
``` 

## 总结

至此我们已经学会了路由使用，以及如何在 `Vue`
中怎么给路由添加转场动画了，我文章中所介绍的只是普通的平移动画，大家如果想要其他样式的动画，只需要对照着那几个 `Vue`
定义的那几个 `css` 修改就可以了

本来想把 `页面缓存` 也一起写了，不过由于篇幅所致，很多人都是 `太长不看`，所以还是单独写一篇吧~

# 结束

>
以上就是文章的全部内容，感谢看到这里，希望对你有所帮助或启发！创作不易，如果觉得文章写得不错，可以点赞收藏支持一下，我会更新更多实用的前端知识与技巧，期待与你共同成长~



