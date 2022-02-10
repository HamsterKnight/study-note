# React数据处理方案Redux

1. react、vue采用的都是单项数据流的形式，传递数据需要一层层往下传递，如果传递层级很深，兄弟组件间共享数据就很麻烦，需要将数据提升到他们的父组件中，显示有点繁琐。

2. 为什么react不能通过数据上下文的形式来实现组件间的的数据共享及数据处理呢？

   因为如果共享数据出问题了，排查起来很麻烦，所有子组件都可以操作改变数据，也不知道哪个方法改变了数据。项目的体积越大，排查起来越麻烦

​	3 .Redux是一个数据处理方案，跟react解耦，它包含**action, store, reducer**

​		5用了redux，组件只要分发一个动作，数据处理都不用管，交给reducer处理，出问题了，可以直接在reducer中找问题

        - **action**用于描述要做什么，分发操作
        - **store**将action，和对应的state传递给reducer
        - **reducer**再处理改变store中的数据,reducer充当了mvc中类似controller的角色





## action 

1. action是一个plain-object（平面对象）
   1. 它的__proto__指向Object.prototype
2. 通常，使用payload属性表示附加数据（没有强制要求）
3. action中必须有type属性，该属性用于描述操作的类型
   1. 但是，没有对type的类型做出要求
4. 在大型项目，由于操作类型非常多，为了避免硬编码（hard code），会将action的类型存放到一个或一些单独的文件中(样板代码)。
5. 为了方面传递action，通常会使用action创建函数(action creator)来创建action
   1. action创建函数应为无副作用的纯函数
      1. 不能以任何形式改动参数
      2. 不可以有异步
      3. 不可以对外部环境中的数据造成影响
6. 为了方便利用action创建函数来分发（触发）action，redux提供了一个函数`bindActionCreators`，该函数用于增强action创建函数的功能，使它不仅可以创建action，并且创建后会自动完成分发。

**提示**： 硬编码（hard code）指，各个地方用字符串写同一个值，如果值变化没法一起同步更改



## reducer

Reducer是用于改变数据的函数

1. 一个数据仓库，有且仅有一个reducer，并且通常情况下，一个工程只有一个仓库，因此，一个系统，只有一个reducer

2. 为了方便管理，通常会将reducer放到单独的文件中。

3. **reducer被调用的时机**

   1. 通过store.dispatch，分发了一个action，此时，会调用reducer
   2. 当创建一个store的时候，会调用一次reducer
      1. 可以利用这一点，用reducer初始化状态
      2. 创建仓库时，不传递任何默认状态
      3. 将reducer的参数state设置一个默认值

4. reducer内部通常使用switch来判断type值

5. **reducer必须是一个没有副作用的纯函数**

   1. 为什么需要纯函数
      1. 纯函数有利于测试和调试
      2. 有利于还原数据
      3. 有利于将来和react结合时的优化
   2. 具体要求
      1. 不能改变参数，因此若要让状态变化，必须得到一个新的状态
      2. 不能有异步
      3. 不能对外部环境造成影响

6. 由于在大中型项目中，操作比较复杂，数据结构也比较复杂，因此，需要对reducer进行细分。

   1. redux提供了方法，可以帮助我们更加方便的合并reducer

   2. combineReducers: 合并reducer，得到一个新的reducer，该新的reducer管理一个对象，该对象中的每一个属性交给对应的reducer管理。

   3. bindActionCreators: 增强分发操作action操作，有两个参数

      - 第一个是action创建函数的集成对象
      - 第二是store.dispatch

      

   
   
   # Store
   
   Store：用于保存数据
   
   通过createStore方法创建的对象。
   
   该对象的成员：
   
   - **dispatch**：分发一个action
   - **getState**：得到仓库中当前的状态
   - replaceReducer：替换掉当前的reducer
   - **subscribe**：注册一个监听器，监听器是一个无参函数，该分发一个action之后，会运行注册的监听器。该函数会返回一个函数，用于取消监听

**store内部细节**：

1. 创建仓库的时候，会分发一个特殊的action
2. subscribe添加的函数在action分发，完成状态更改后，执行
3. subscribe添加监听函数，会返回一个解除监听函数



​	源码实现createStore

```js

function isPlainObject(obj) {
  return Object.getPrototypeOf(obj) === Object.prototype
}

function actionIsCorrect(action) {
  if (typeof action !== 'object') {
    console.log('action', action)
    throw new Error('action must be a object')
  }
  if (action.type == undefined) {
    throw new Error('action must has a property type')
  }

  if (!isPlainObject(action)) {
    throw new Error('action must be a plain-objecy')
  }
  return true
}

/**
 * 产生一个长度为keyLength的随机字符串
 * @param {*} keyLength 
 * @returns 
 */

function createRandomStr(keyLength) {
  return Math.random().toString(32).slice(2, keyLength).split('').join(',')
}

/**
 * 用于创建一个store对象
 * @param {function} reducer 
 * @param {any} state 
 * @returns {object: {dispatch, getState, subscribe}} 返回一个store对象，包含以上属性
 */
function myCreateStore(reducer, state) {
  let currentState = state,
    currentReducer = reducer,
    listens = []

  /**
   * 传进来的action必须是一个plan-object, 且必须包含type属性
   * @param {object} action 
   */
  function dispatch(action) {
    if (actionIsCorrect(action)) {
      currentState = currentReducer(currentState, action)
      for(let listen of listens) {
        listen()
      }
    }
  }

  function getState() {
    return currentState
  }

  function subscribe(listen) {
    listens.push(listen)
    let isRemove = false
    return () => {
      if (isRemove) return
      isRemove = true
      const index = listens.indexOf(listen)
      listens.splice(index, 1)
    }
  }

  dispatch({
    type: `@@redux/INIT${createRandomStr()}`
  })

  return {
    dispatch,
    getState,
    subscribe
  }

}

export default myCreateStore
```



## combineReducers 作用和源碼

作用：用于整合多个reducer

源码实现细节：

- 传入的reducers必须是一个plain-object(平面对象)
- 整合的reducer必须要有默认值，通过传入两个action来进行判断
  1. type为 @@redux/INITx.x.x.x.x.x：判断初始化的值是否是undefined
  2. type为@@redux/PROBE_UNKNOWN_ACTION：判断是否对INIT做了特殊处理

-  返回值是一个reducer函数

```js
import isPlainObject from "./utils/isPlainObject"
import  actionTypes from './utils/action'


function validate(reducers) {
  if(typeof reducers !== 'object') {
    throw new TypeError('reducers must be a object')
  }
  if(!isPlainObject(reducers)) {
    throw new TypeError('reducers must be a plain-object')
  }
  for (const key in reducers) {
    if (Object.hasOwnProperty.call(reducers, key)) {
      const reducer = reducers[key];
      let state = reducer(undefined, actionTypes.INIT())
      if(state === undefined) {
        throw new TypeError('reducers can not be return undefind')
      }
      state = reducer(undefined, actionTypes.UNKNOWN())
      if(state === undefined) {
        throw new TypeError('reducers can not be return undefind')
      }
    }
  }
}

/**
 * combinerReducers，对于传入的reducers对象，要求必须是一个平面对象
 * 且执行的时候，会执行两次dispatch，第一次是初始化，要求reducer必须要有默认值
 * 第二次是验证，你是否写了特殊处理
 * @param {*} reducers 
 * @returns {function} 返回一个reducer函数
 */

export function myCombineReducers(reducers) {

  validate(reducers)

  return function (state = {}, action) {
    const newState = {}
    for (const key in reducers) {
      if (Object.hasOwnProperty.call(reducers, key)) {
        const reducer = reducers[key]
        newState[key] = reducer(state[key], action)
      }
    }
    return newState
  }

}
```



## applyMiddleWare中间件	

中间件的本质是在对原有的dispatch功能进行增强

中间件的应用的类似洋葱模型先组装最后一个中间件，然后一直往前到第一个，

但是执行的时候先执行第一个中间件的，然后第二个，直到最后一个执行完，又会返回来



**使用**

方式一： createStore(reducers ,  state(默认状态，可选), applyMiddleWare(中间件1，中间件2， ...))

方式二： applyMiddleWare（中间件1，中间件2，...）(createStore)(reducer)



**中间件函数**：是一个dispatch创建函数的函数，即调用后返回值是一个dispatch创建函数





