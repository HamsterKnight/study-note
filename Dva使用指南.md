## react框架 Dva使用指南

1. dva默认导出一个函数，执行函数用来创建应用

   ```js
   import dva from 'dva'
   
   //通过函数来创建应用
   const app = dva()
   ```
   
   1.1 创建应用后，通过`router`方法来渲染组件,router方法接受一个函数，函数返回的react节点，会在dva启动后被渲染出来
   
   ```js
   app.router(()=><App />)
   ```
   
   ​	**将路由和仓库联系起来**，dva内部已经安装了connectd-react-router
   
   ​	(connected-react-router将react-router和react-redux联系起来需要三步
   
     1. 创建reducerMid中间件函数,用来拦截出发路由action，创建中间件函数需要传入history对象
   
     2. 创建reducer处理函数，用来处理路由状态，需要传入history对象
   
     3. 创建替换browserHistory对象的组件，connectReactRouter， 需要传入history对象
   
        这三个history对象都需要是同一个history对象
   
   
   
   ​	dva内部已经处理好了第一和第二步，第三步，需要我们自己处理
   
   ​	为了保证history对象统一，dva创建应用的时候，我们需要传入一个统一的history对象
   
   ```js
   import { createBrowserHisotry } from 'history'
   const app = dva({
       history: createBrowserHisotry()
   })
   ```
   
      统一的history对象传入完成后，我们还需要在根组件中传入history, dva已经给我们在跟组件的props中传入了history属性，我们直接传入ConnectRouter中即可，但是ConnectRouter不是从connected-react-router中获取，而是从dva/router中取，dva/router中用routerRedux接收了connected-react-router的所有导出内容，所以我们最终的用法是

   ```js
   import {routerRedux} from 'dva/router'
   
   function App(props) {
   return (
       <routerRedux.ConnectedRouter history={props.history}>
   
       
   	</routerRedux.ConnectedRouter>)
   }
   ```
   
   
   
   ​	
   
   ​	
   
   ​	)
   
   1.2 配置模型`model`，用于将react-redux, react,react-router连接起来
   
   接受一个对象参数，该参数有以下几个属性
   
   - namespace(string类型,必须要有，命名空间)
   
   - state（any类型，必須要有，初始值）
   
   - reducers(object类型，跟redux一样，是对state的处理，对象的每个属性是对state的处理函数)
   
     - 处理函数接受两个参数
       - 第一个参数是state，
       - 第二个参数是action
   
     ```js
      reducers: {
         increase(state, action) {
           return state + 1;
         },
         decrease(state, action) {
           return state - 1;
         },
       }
     ```
   
   - effects(对象类型，跟redux中副作用函数内容相关，对象的每个属性都是一个生成器函数)
   
     - 生成器函数有两个参数
       - 第一个参数是action
       - 第二个参数是saga对象
   
     ```js
      effects: {
         *asyncIncrease(action, { put }) {
           yield put({
               type: "increase"
           });
         },
       },
     
     ```
   
   - subscriptions(对象类型，对象的每个属性都是一个函数，定义的属性函数，会在模型加入到仓库后**只执行一次**)
   
     - 函数有一个参数，该参数是一个对象，对象中有两个属性，`dispatch`和`history`
     - 为什么要在subscriptions中定义多个函数呢？
       - 1.便于功能模块分离
       - 2.避免重复定义函数
   
   ```js
    subscriptions: {
         resizeIncrease({dispatch, history}) {
           window.onresize = function() {
               dispatch({
                   type: 'increase'
               })
           }
         }
     }
   ```
   
   
   
   1.3 当配置都完成后，需要通过`start`方法启动dva,该方法接受一个字符串（内部使用queryselector，来查找对应的元素）
   
   ```js
   app.start('#root')
   ```
   1.4 **dva的配置项**, 配置项是在dva创建应用的时候传入一个配置对象
   
   - **history**: 路由的形式
   
     - createBrowserHistory() h5路由
     - createHashHistory() 哈希路由
     - 注：相关创建history对象的函数来自history库
   
   - **initialState**: 创建redux仓库时，使用的默认状态
   
   - **onError**: 是一个函数，当`effect` 执行错误或 `subscription` 通过 `done` 主动抛错时触发函数，可用于管理全局出错状态
   
     - 有两个参数onError: (err, dispatch)=> {}
       - err 是抛出的错误
       - dispatch是store的dispatch方法
   
   - **onAction**: 可以是一个中间件函数或者是一个中间件函数数组，当有action被触发的时候执行。
   
   - **onStateChange**: 是一个函数，当状态发生改变的时候执行函数
   
     - 函数接收一个参数   (state) => {}
   
   - **onReducer**: 是一个函数，函数接收的一个参数(为旧的reducer); 函数返回一个函数，返回的函数会被当作新的reducer,新的reducer需要返回新的状态值
   
   - **onEffect**: 对模型中的effect进一步的封装
   
     - 函数接收四个参数，需要返回一个新的generator函数，（因为是符合saga执行的函数，所以只接收一个参数action）
   
       - oldEffect 原来的effect方法
   
       - sage   sage对象
   
       - model 原模型
   
       - action的类型
   
         
   
   - **extraReducers**: 接收一个对象，用于配置额外的reducer； 对象的每个属性都是一个方法
   
   - **extraEnhancers**: 额外的增强，用于封装createStore, 函数接收一个参数，为原仓库创建函数，返回一个新的创建仓库的函数,函数必须放在数组中
   ```js
   import dva from 'dva'
   import createLogger from 'redux-logger';
   //通过函数来创建应用
   const app = dva({
   	history: createBrowserHistory()
       initialState:{
       	counter: 1
   	},
       onError: function(err, dispatch) {
       	console.log(err)
   	},
       onAction: createLogger(),
       onStateChange: (state)=>{},
       onReducer: (oldReducer) => {
           return (state, action) {
               const newState = oldReducer(state, action)
               return newState
           }
       },
       onEffect: (oldEffect, sagaEffect, model, action)=> {
           return function*(action) {
               // 做一些自己的操作
               
               // 执行原来的effect
               yiled oldEffect(action)
           }
       },
       extraReducer: {
           abc(state, action) {
               return state
           }
       }，
       extraEnhancers: [(createStore) => {
           return function(...args){
               console.log('即将创建仓库')
               return createStore(...args)
           }
       }]
   })
   ```
   
   
   ```js
   import dva from 'dva'
   import {createBrowserHistory} from history
   
   
   // dva函数中可以传入一些配置
   const app = dva({
       history: createBrowserHistory() // 传入history对象是为了保持history对象统一
   })
   
   // router.model 类似数据仓库中的一个状态属性, 传入一个配置对象
   router.model({
       namespace: '命名空间', //定义一个命名空间
    	// reducers中传入一个对象，键值会被当做actionType吗，
       // 键值是一个函数，函数里是是状态的处理，该函数有两个参数，第一个参数是state，第二个参数是action   
       reducers: {
       	increase(state, action) {
       		return state + 1
   		}
   	},
       // effects是对redux-saga的封装处理，键名是字符串，键值是生成器函数
       // 键值的生成器函数，接受两个参数，第一个是action,第二个是sage对象
       effects： {
       	*asyncIncrease(action, saga) {
       
   		}
   	},
       subscriptions：配置为一个对象，该对象中可以写任意数量任意名称的属性，每个属性是一个函数，这些函数会在模型加入到仓库中后(app.start()后)立即运行。
   })
   
   
   // 启动前，接受路由配置，路由配置可以是一个函数，返回一个组件或者可渲染的标签
   // 可以用于将router和仓库连接起来（通过第三方库connected-react-redux）
   app.router((props)= <App/>)
   
   //启动一个任务
   app.start()
   
   ```
   
   

