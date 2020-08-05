# Vue Router 功能

## 基本使用

Vue-Router 的能力十分强大，它支持 `hash`、`history`、`abstract` 3 种路由方式，提供了 `<router-link>` 和 `<router-view>` 2 种组件，还提供了简单的路由配置和一系列好用的 API。

* <router-link>会被渲染成 a 标签，点击会跳转到 router-link 下 to 指向的 route
* <router-view>会在 router-view 原地更换渲染这个当前 route组件的内容

例如

```javascript
div id="app">
  <h1>Hello App!</h1>
  <p>
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
	</p>
	<router-view></router-view>
</div>

//下面是 script 中
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
const router = new VueRouter({
  routes // (缩写) 相当于 routes: routes
})

const app = new Vue({
  router
}).$mount('#app')
```

这样一点击对应的 router-link，对应的 component 就会展现在 router-view 中

## 动态路由匹配

当跳转到一个''页面"的时候，经常会需要携带一些参数，例如跳转到用户信息页面，通常要携带 id 作为参数。可以使用动态路由参数解决这个问题

```javascript
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
    
  ]
})

//展示的时候
const User = {
  template: '<div>User {{ $route.params.id }}</div>' // 这里可以通过 $route.params.id 获取到这个
}
```

仅使用路径上匹配的方法

| 模式                          | 匹配路径            | $route.params                          |
| ----------------------------- | ------------------- | -------------------------------------- |
| /user/:username               | /user/evan          | `{ username: 'evan' }`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

如果是从 user/foo 跳转到 user/bar， 因为是同一个组件，所以这里组件会被**复用**，而不是重新创建。**不过，这也意味着组件的生命周期钩子不会再被调用**。



## 捕获所有路由或者404not found

```javascript
{
  // 会匹配所有路径
  path: '*'
}
{
  // 会匹配以 `/user-` 开头的任意路径
  path: '/user-*'
}
```

当使用通配符'*'的时候，route.param 上会添加一个名为 `pathMatch` 参数， 包含了被通配符匹配到的字符串

```javascript
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'
// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```



## 嵌套路由

一般情况下在根页面的 router-view 显示匹配到的组件，但是组件中还可能有 router-view，这时候就需要在vueRouter 中添加children 属性

```javascript
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

## 命名路由

在开始编程式导航之前，router 的定义方法中除了 `path`还可以定义其他几个属性, 例如`name`,在跳转的时候(router.push)就可以用已经定义的相应的命名来跳转到对应的路由

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})

<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
router.push({ name: 'user', params: { userId: 123 }})
```



## 编程式导航

| 声明式                    | 编程式             |
| ------------------------- | ------------------ |
| `<router-link :to="...">` | `router.push(...)` |

### Router.push()

通过 js 中控制跳转

```javascript
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

* 当参数为 path 的情况下，param会被忽略，要使用 query，同样适用于<router-link to = ''>
* router.push 接收第二个和第三个参数，分别是导航成功和失败时候的回调。

### Router.replace()

和 router.push 类似的还有 router.replace(), 参数用法相同，区别是 replace 不会向 history 添加新记录，而是替换掉当前的history

### Router.go()

这个方法接收一个整数，可以是负数，表示从当前的 router 前进或者后退多少步。



## 命名视图

```javascript
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>

//对应的注册
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

按照上面这么注册，那么下面两个对应的 view 就是 Bar 和 Baz

## 重定向和别名

routes中的对象还接收另一个关键字作为参数，redirect，表示从'/a'重定向到'/b'

```javascript
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})

//或者
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})

//或者
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
// 下面部分是别名 alias

const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})

```

别名显示则是当访问'/b'的时候，路由显示为'/b', 但是匹配的组件则是'/a'



## 路由懒加载

随着项目模块增加，引入文件数量剧增，如果不做任何处理，那么**首屏加载会特别缓慢** ，这个时候路由懒加载就出现了

在定义的时候，如果 webpack 版本< 2.4, 则

```javascript
const router = new Router({
		routes:[
      {
        path:'/',
        name:'home',
        components: resolve=>require(['@/components/home'], resolve)
			}
    ]
})
```

如果 webpack 版本 > 2.4 则

```javascript
const router = new Router({
		routes:[
      {
        path:'/',
        name:'home',
        components: ()=>import('@/components/home')
			}
    ]
})
```



