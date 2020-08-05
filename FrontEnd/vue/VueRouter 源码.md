# VueRouter 源码

在vue 生命周期beforeCreate 中，有一部分，判断传入的 options 是否有 router 这个选项，如果有的话就执行 router 的 init 方法

```javascript
beforeCreate() {
  if (isDef(this.$options.router)) {
    // ...
    this._router = this.$options.router
    this._router.init(this)
    // ...
  }
}  
```

而 init()的定义为:

```javascript
init (app: any) {
  process.env.NODE_ENV !== 'production' && assert(
    install.installed,
    `not installed. Make sure to call \`Vue.use(VueRouter)\` ` +
    `before creating root instance.`
  )

  this.apps.push(app)

  if (this.app) {
    return
  }

  this.app = app

  const history = this.history

  if (history instanceof HTML5History) {
    history.transitionTo(history.getCurrentLocation())
  } else if (history instanceof HashHistory) {
    const setupHashListener = () => {
      history.setupListeners()
    }
    history.transitionTo(
      history.getCurrentLocation(),
      setupHashListener,
      setupHashListener
    )
  }

  history.listen(route => {
    this.apps.forEach((app) => {
      app._route = route
    })
  })
}
```

这个函数传入一个 vue 实例，然后存储到 this.apps 中，并且会拿到当前的 history，并根据类型的不同(例如可能是 hash 的或者是 history 的)，执行不同的操作。

对于 hash 的则执行了`history.setupListeners()`, 然后

```javascript
history.transitionTo(
      history.getCurrentLocation(),
      setupHashListener,
      setupHashListener
)
```

而这个函数的定义为, 从上面可以看到这个函数传入的两个回调函数，不管成功失败都是 setupHashListener

```javascript
transitionTo (location: RawLocation, onComplete?: Function, onAbort?: Function) {
  const route = this.router.match(location, this.current)
  // ...
}
```

可以看到这个函数调用了 this.router.match, 而这个函数实际上调用了this.matcher.match去做匹配，

```javascript
match (
  raw: RawLocation,
  current?: Route,
  redirectedFrom?: Location
): Route {
  return this.matcher.match(raw, current, redirectedFrom)
}
```

总结以下，当 传入 vue 的options中有 router 的时候，就会在 beforeCreate 执行 router.init()函数，通过对于不同类型 router，执行不同的初始化操作，而对于 hash 类型的来说，实际上是执行matcher 这个类下的 Macher.matcha操作。下面来看一下 matcher 是怎么实现的。



## Matcher

先来看一下 matcher 的实现

```javascript
export type Matcher = {
  match: (raw: RawLocation, current?: Route, redirectedFrom?: Location) => Route;
  addRoutes: (routes: Array<RouteConfig>) => void;
};
```

可以看到他向外导出了两个函数

* 第一个是之前说到的 Matcher.match 
  * (这个表示对于用户传入的 path 或者 name 如何匹配对应路由)
* 第二个是 addRoutes
  * (这部分则表示如何把配置时候传入的 routes和对应的 component 建立联系)

可以看到match 接收的参数基本是两个类型，一个是 Location, 一个是 Route, 先来了解一下这两个类型

### Location

定义如下：

```javascript
declare type Location = {
  _normalized?: boolean;
  name?: string;
  path?: string;
  hash?: string;
  query?: Dictionary<string>;
  params?: Dictionary<string>;
  append?: boolean;
  replace?: boolean;
}
```

location 其实和window.location 很类似，都是对于一个 **url 的结构化定义**。

例如`url = /abc?name=Leo&id=1`, 其实对应的结构可能为

```javascript
{
  name: abc,
  path: '/abc'
  query:{
		name:leo,
    id : 1
	}
   
```

### Route

route 是路由中的一条线路，它除了描述了类似上面 Location 中的 path,name,query， 还有 **matched 表示匹配到的RouteRecord**

```javascript
declare type Route = {
  path: string;
  name: ?string;
  hash: string;
  query: Dictionary<string>;
  params: Dictionary<string>;
  fullPath: string;
  matched: Array<RouteRecord>;
  redirectedFrom?: string;
  meta?: any;
}
```

------



看完 Location 和 Route 之后来看一下 Matcher 是如何定义的

```javascript
export function createMatcher (
  routes: Array<RouteConfig>,
  router: VueRouter
): Matcher {
  const { pathList, pathMap, nameMap } = createRouteMap(routes)

  function addRoutes (routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
  }

  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    const location = normalizeLocation(raw, currentRoute, false, router)
    const { name } = location

    if (name) {
      const record = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        warn(record, `Route with name '${name}' does not exist`)
      }
      if (!record) return _createRoute(null, location)
      const paramNames = record.regex.keys
        .filter(key => !key.optional)
        .map(key => key.name)

      if (typeof location.params !== 'object') {
        location.params = {}
      }

      if (currentRoute && typeof currentRoute.params === 'object') {
        for (const key in currentRoute.params) {
          if (!(key in location.params) && paramNames.indexOf(key) > -1) {
            location.params[key] = currentRoute.params[key]
          }
        }
      }

      if (record) {
        location.path = fillParams(record.path, location.params, `named route "${name}"`)
        return _createRoute(record, location, redirectedFrom)
      }
    } else if (location.path) {
      location.params = {}
      for (let i = 0; i < pathList.length; i++) {
        const path = pathList[i]
        const record = pathMap[path]
        if (matchRoute(record.regex, location.path, location.params)) {
          return _createRoute(record, location, redirectedFrom)
        }
      }
    }
    return _createRoute(null, location)
  }

  // ...

  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    if (record && record.redirect) {
      return redirect(record, redirectedFrom || location)
    }
    if (record && record.matchAs) {
      return alias(record, location, record.matchAs)
    }
    return createRoute(record, location, redirectedFrom, router)
  }

  return {
    match,
    addRoutes
  }
}
```

可以看到 createMatcher 接收两个实例，一个是new Router 产生的 Router 实例，另一个是用户传入的 routes

在 addRoutes中，可以看到执行了一个函数， 而函数的参数就是之前执行 createRouteMap 的结果

```javascript
const { pathList, pathMap, nameMap } = createRouteMap(routes)
function addRoutes (routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
}


// createRouteMap的实现
export function createRouteMap (
  routes: Array<RouteConfig>,
  oldPathList?: Array<string>,
  oldPathMap?: Dictionary<RouteRecord>,
  oldNameMap?: Dictionary<RouteRecord>
): {
  pathList: Array<string>;
  pathMap: Dictionary<RouteRecord>;
  nameMap: Dictionary<RouteRecord>;
} {
  const pathList: Array<string> = oldPathList || []
  const pathMap: Dictionary<RouteRecord> = oldPathMap || Object.create(null)
  const nameMap: Dictionary<RouteRecord> = oldNameMap || Object.create(null)

  routes.forEach(route => {
    addRouteRecord(pathList, pathMap, nameMap, route)
  })

  for (let i = 0, l = pathList.length; i < l; i++) {
    if (pathList[i] === '*') {
      pathList.push(pathList.splice(i, 1)[0])
      l--
      i--
    }
  }

  return {
    pathList,
    pathMap,
    nameMap
  }
}
```

createRouteMap 的目的就是遍历用户传入的 route，将 route 转化成3个部分

1. pathList: 包含所有 path
2. pathMap：包含所有 path 到 routeRecord的映射关系
3. nameMap: 包含所有 name 到 routeRecord 的映射关系

### routeRecord 是什么

先来看一下定义

```javascript
declare type RouteRecord = {
  path: string;
  regex: RouteRegExp;
  components: Dictionary<any>;
  instances: Dictionary<any>;
  name: ?string;
  parent: ?RouteRecord;
  redirect: ?RedirectOption;
  matchAs: ?string;
  beforeEnter: ?NavigationGuard;
  meta: any;
  props: boolean | Object | Function | Dictionary<boolean | Object | Function>;
}
```

它的创建是遍历所有 routes，为每一条 route 执行`addRouteRecord`方法生成一条记录。

```javascript
function addRouteRecord (
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>,
  route: RouteConfig,
  parent?: RouteRecord,
  matchAs?: string
) {
  const { path, name } = route
  if (process.env.NODE_ENV !== 'production') {
    assert(path != null, `"path" is required in a route configuration.`)
    assert(
      typeof route.component !== 'string',
      `route config "component" for path: ${String(path || name)} cannot be a ` +
      `string id. Use an actual component instead.`
    )
  }

  const pathToRegexpOptions: PathToRegexpOptions = route.pathToRegexpOptions || {}
  const normalizedPath = normalizePath(
    path,
    parent,
    pathToRegexpOptions.strict
  )

  if (typeof route.caseSensitive === 'boolean') {
    pathToRegexpOptions.sensitive = route.caseSensitive
  }

  const record: RouteRecord = {
    path: normalizedPath,
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
    components: route.components || { default: route.component },
    instances: {},
    name,
    parent,
    matchAs,
    redirect: route.redirect,
    beforeEnter: route.beforeEnter,
    meta: route.meta || {},
    props: route.props == null
      ? {}
      : route.components
        ? route.props
        : { default: route.props }
  }

  if (route.children) {
    if (process.env.NODE_ENV !== 'production') {
      if (route.name && !route.redirect && route.children.some(child => /^\/?$/.test(child.path))) {
        warn(
          false,
          `Named Route '${route.name}' has a default child route. ` +
          `When navigating to this named route (:to="{name: '${route.name}'"), ` +
          `the default child route will not be rendered. Remove the name from ` +
          `this route and use the name of the default child route for named ` +
          `links instead.`
        )
      }
    }
    route.children.forEach(child => {
      const childMatchAs = matchAs
        ? cleanPath(`${matchAs}/${child.path}`)
        : undefined
      addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
    })
  }

  if (route.alias !== undefined) {
    const aliases = Array.isArray(route.alias)
      ? route.alias
      : [route.alias]

    aliases.forEach(alias => {
      const aliasRoute = {
        path: alias,
        children: route.children
      }
      addRouteRecord(
        pathList,
        pathMap,
        nameMap,
        aliasRoute,
        parent,
        record.path || '/'
      )
    })
  }

  if (!pathMap[record.path]) {
    pathList.push(record.path)
    pathMap[record.path] = record
  }

  if (name) {
    if (!nameMap[name]) {
      nameMap[name] = record
    } else if (process.env.NODE_ENV !== 'production' && !matchAs) {
      warn(
        false,
        `Duplicate named routes definition: ` +
        `{ name: "${name}", path: "${record.path}" }`
      )
    }
  }
}
```

首先看，这里创建 routeRecord代码如下

```javascript
const record: RouteRecord = {
  path: normalizedPath,
  regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
  components: route.components || { default: route.component },
  instances: {},
  name,
  parent,
  matchAs,
  redirect: route.redirect,
  beforeEnter: route.beforeEnter,
  meta: route.meta || {},
  props: route.props == null
    ? {}
    : route.components
      ? route.props
      : { default: route.props }
}
```

这里 path 是规范过后的 path， regex是一个正则的扩展，它利用了`path-to-regexp` 这个工具库，把 path 解析成了正则表达式扩展。components 是一个对象， instance 表示组件实例，parent 表示父级的 routeRecord, 如果配置了 children，那么可以递归进行addRouteRecord，并且把当前record当做parent 传入，这样深度遍历就可以拿到一个route 下的完整记录。

因为我们有的时候会使用 children 属性，所以整个RouteRecord也是一个树形的结构



整个函数的目的就是为了配置 pathList，nameMap 和 pathMap。

* 其中 pathList 是为了保留所有 path
* nameMap 是为了快速通过 name 找到对应的 routeRecord
* pathMap 是为了快速通过 path 找到对应的 routeRecord

## Match 方法

```javascript
function match (
  raw: RawLocation,
  currentRoute?: Route,
  redirectedFrom?: Location
): Route {
  const location = normalizeLocation(raw, currentRoute, false, router)
  const { name } = location

  if (name) {
    const record = nameMap[name]
    if (process.env.NODE_ENV !== 'production') {
      warn(record, `Route with name '${name}' does not exist`)
    }
    if (!record) return _createRoute(null, location)
    const paramNames = record.regex.keys
      .filter(key => !key.optional)
      .map(key => key.name)

    if (typeof location.params !== 'object') {
      location.params = {}
    }

    if (currentRoute && typeof currentRoute.params === 'object') {
      for (const key in currentRoute.params) {
        if (!(key in location.params) && paramNames.indexOf(key) > -1) {
          location.params[key] = currentRoute.params[key]
        }
      }
    }

    if (record) {
      location.path = fillParams(record.path, location.params, `named route "${name}"`)
      return _createRoute(record, location, redirectedFrom)
    }
  } else if (location.path) {
    location.params = {}
    for (let i = 0; i < pathList.length; i++) {
      const path = pathList[i]
      const record = pathMap[path]
      if (matchRoute(record.regex, location.path, location.params)) {
        return _createRoute(record, location, redirectedFrom)
      }
    }
  }
  
  return _createRoute(null, location)
}
```

match 方法接收3个参数

* Raw: 是 rawlocation 类型，可以是 url 字符串，也可以是上面说的 Location 类型
* currentRoute： curretRoute 是Route 类型，表示当前的位置
* redirectedFrom：表示和重定向有关，这里先忽略

match 方法返回一个 Route 类型的数据，表示从当前位置(currentRoute)到 rawLocation 位置的路径

上面有一个函数

```javascript
const location = normalizeLocation(raw, currentRoute, false, router)
```

主要是通过 raw 和 currentRoute 来计算出新的location， 它主要处理了 raw 的一些情况，例如传入的 raw 是有 path 的，或者传入的 raw 是有 name 的(或者是纯字符串的类似'/abc?id=1')

1. `name`

有 `name` 的情况下就根据 `nameMap` 匹配到 `record`，它就是一个 `RouterRecord` 对象，如果 `record` 不存在，则匹配失败，返回一个空路径；然后拿到 `record` 对应的 `paramNames`，再对比 `currentRoute` 中的 `params`，把交集部分的 `params` 添加到 `location` 中，然后在通过 `fillParams` 方法根据 `record.path` 和 `location.path` 计算出 `location.path`，最后调用 `_createRoute(record, location, redirectedFrom)` 去生成一条新路径，该方法我们之后会介绍。

2. `path`

通过 `name` 我们可以很快的找到 `record`，但是通过 `path` 并不能，因为我们计算后的 `location.path` 是一个真实路径，而 `record` 中的 `path` 可能会有 `param`，因此需要对所有的 `pathList` 做顺序遍历， 然后通过 `matchRoute` 方法根据 `record.regex`、`location.path`、`location.params` 匹配，如果匹配到则也通过 `_createRoute(record, location, redirectedFrom)` 去生成一条新路径。因为是顺序遍历，所以我们书写路由配置要注意路径的顺序，因为写在前面的会优先尝试匹配。



## 总结

总结一下，view-router 就是当 new vue(options)的时候如果有 router，就在 beforeCreate 生命周期里调用 router.init()进行初始化，router 种类有基于 hash 的也有基于 history 的，常用的是基于 hash 的，使用的是实例化的一个对象，叫做 matcher.match 的方法。

而 Matcher 这个 class 的意义，就是根据 user 传入的 route，调用createRouteMap， 构建起各个 router 的 path 和name 对应的 map，这个函数返回三个对象，一个是 pathList，是所有的 path 集合，还有 pathMap 和 nameMap, 即给定一个path 或者 name 要给定一个 route 出来，而构建的 RouteMap，因为route 中有时候有 children 的关系，本身构建出来的就是树的结构。

当传入一个 path 或者 name 的时候，Matcher.match 就是相当于用这个 url 去 PathMap 或者 nameMap 中找对应 route, 根据匹配返回一条路径，如果没有就为空。

