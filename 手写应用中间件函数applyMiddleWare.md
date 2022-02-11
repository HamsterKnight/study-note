# 手写应用中间件函数applyMiddleWare

1.applyMidleeWrare接受多个中间件函数

2.applyMiddle调用后返回一个innerOne函数，该函数接受一个store创建函数

3.innerOne函数调用后也返回一个innerTwo函数, 该函数接受两个参数，第一个是reducer，第二个是defaultState(可选)

4.innterTow调用后会返回一个对dispatch修改过的store对象

```js

function applyMiddleWare(...fns) {
  return function (creatStore) {
    return function (reducer, defaultState) {
      const store = creatStore(reducer, defaultState)
      let dispatch = () => {
        throw new Error('dispatch can not be use')
      }
      const { getState } = store
      const simpleStore = {
        dispatch: (...args) => dispatch(...args),//这里不能写store.dispatch,如果是store.dispatch那么中间件内触发的dispatch，无法触发所有中间件函数
        getState
      }

      // 获取组合dispatch创建函数的数组

      middleWareProducts = fns.map(fn => fn(simpleStore))
		
      // 获取到最终的dispatch
      // [a,b,c] 转成 (..args) => a(b(c(...args)))
      // (..args) => a(b(c(...args)))调用后返回最终的dispatch
      // 修改后的dispatch再执行，就会调用各个中间件里的最终函数
      dispatch = compose(...middleWareProducts)(store.dispatch)
        
      return {
        ...store,
        dispatch
      }

    }
  }
}


// 这里是将多个函数组合成洋葱模型
// [a,b,c] 转成 a(b(c()))
function compose(...fns) {
  if(fns.length === 0) {
    return (args) => args
  }
  if(fns.length === 1) {
    return fns[0]
  }

  return fns.reduce((a, b) => (...args) => a(b(...args)))
}

```

