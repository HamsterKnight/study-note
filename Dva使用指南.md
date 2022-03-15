## react框架 Dva使用指南

1. dva默认导出一个函数，执行函数用来创建应用

   ```js
   import dva from 'dva'
   import {createBrowserHistory}
   
   
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

   

