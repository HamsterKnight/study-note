## connected-react-router：将路由状态保存在redux仓库中

作用：

本质上，router中的某些数据可能会跟数据仓库中的数据进行联动

该组件会将下面的路由数据和仓库保持同步

1. action：它不是redux的action，它表示当前路由跳转的方式（PUSH、POP、REPLACE）
2. location：它记录了当前的地址信息

**步骤**

**一、安装相关第三方包**

1. 安装connected-react-router

```shell
yarn add connected-react-router
```

2. chrome浏览器安装chrome插件redux-devtools

3. 安装redux-devtools-extension,将仓库暴露给chrome

```shell
yarn add -D redux-devtools-extension
```

示例

```js
import { composeWithDevTools } from "redux-devtools-extension"

const store = createStore(reducer,
    composeWithDevTools(applyMiddleware(routerMid, sagaMid, logger))
)
```

**二、相关步骤**

1.因为需要保存路由状态在仓库中，所以需要一个reducer处理函数，reducer处理函数来自connected-react-router中的`connectRouter`,该函数接受一个参数，为history对象，返回一个函数

```js
import { connectRouter } from "connected-react-router"

combineReducers({
    //添加路由状态
    router: connectRouter(history)
})
```

2.因为要监听路由发生变化，且保存在仓库中，所以需要触发特殊的action；通过使用connected-react-router中的一个中间件函数创建函数`routerMiddleware`。来创建一个中间键函数，该创建函数需要接受一个history参数对象。

创建完中间件后，需要应用中间件

```js
//1.引用中间件创建函数
import { routerMiddleware } from "connected-react-router"
import { composeWithDevTools } from "redux-devtools-extension"
//2.引用同一个history对象
import history from "./history"
//3. 创建中间件函数
const routerMid = routerMiddleware(history)

const sagaMid = createSagaMiddleware(); //创建一个saga的中间件

//4. 使用中间件
const store = createStore(reducer,
    composeWithDevTools(applyMiddleware(routerMid, sagaMid, logger))
)

```

3.connected-react-router新制作一个组件`ConnectedRouter`，因为该库必须保证整个过程使用的是同一个history对象

`ConnectedRouter`用于向上下文提供一个history对象和其他的路由信息（与react-router提供的信息一致）



示例

```shell
import { ConnectedRouter } from 'connected-react-router'
import history from "./history"

 <ConnectedRouter history={history}>
    <Switch>
        <Route path="/login" component={Login} />
        <Route path="/" component={Admin} />
    </Switch>
</ConnectedRouter>
```

4.如果调用router的相关标签进行跳转的话，可以正常监听到状态和路径信息的变化；但是需要在方法调用跳转，不能直接调用history的push方法，这样无法正常记录数据。

调用方法跳转，并监听到数据，正常操作如下

​	1. 从connected-react-router中获取触发路由变化的action, 通过store中分发操作dispatch的形式

   2. 通过dipatch触发action,进行地址更改

      ```js
      // 导出的push是一个函数，函数接受的参数跟history.push一致，函数执行后会返回一个action
      import { push } from "connected-react-router"
      import { connect } from "react-redux"
      
      function ReactRouterTest(props) {
        return <Router>
          <Switch>
            <Route path='/page1' component={Page1}>234</Route>
            <Route path='/page2' component={Page2}>123</Route>
          </Switch>
          <NavLink to="/page1">to page1</NavLink>
          <NavLink to="/page2">to page2</NavLink>
          <button onClick={()=> {
            props.toPage && props.toPage()
          }}>to page1</button>
        </Router>
      }
      const mapDispatchToProps = dispatch => ({
        toPage: () => {
            dispatch(push("/page1"))
        }
      })
      
      export default connect(null, mapDispatchToProps)(ReactRouterTest)
      
      // 注：connect是react-redux中将redux和react链接起来的组件
      
      ```

      
