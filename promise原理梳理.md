# Promise原理及手敲源代码

Promise是一个构造函数，构造函数接受一个参数函数executor，

参数函数接受两个参数

- resolve
- reject

参数函数会在创建实例的时候运行

通过new 创建promise实例, 实例有两个属性

	- value 值
	- status 当前状态

status有三个状态值，`peding`,`fulfilled`,`rejected`,状态一旦发生更改后，不可再变



**promise实例有属性then方法**，执行后会返回一个新的promise实例，新实例状态跟当前的promise一样和值是当前promise的值,

该方法接受两个参数函数

参数函数不会立马执行，只有当promise的状态发生改变的时候，根据改变后的状态，执行对应的参数函数，

所以我们需要将参数函数推进队列中(推进队列中的不单单只有参数函数，还应该有状态值，返回promise的resolve, reject方法)

```js
promise.then((data)=> {}, (reason)=>{})
```



当用then方法将函数参数推进队列后，promise实例状态发生改变后，会执行队列中符合状态的函数(推进`微队列`中执行)，并清空队列

如果then方法主动返回内容是一个promise,那么返回的promise的状态和值要跟这个promise一致



**创建一个微队列函数**

```js
function microTask(fn) {
  // 判断是否是node环境
  if(process && process.nextTick) {
      process.nextTick(fn)
      return
  } else if (MutationObserver) {
      const p = document.createElement('p')
      const obs = new MutationObserver(()=>{
          fn()
      })
      obs.observe(p, {
          attributes: true,// 属性发生变化触发
          childList: true,// 子元素发生变化触发
          subtree: true // 子树发生辩护触发
      })
      p.innerHTML = '1'
  } else if(setImmediate) {
      // ie环境
      setImmediate(fn)
  } else {
      //其他环境
      setTimeout(fn, 0)
  }
  
}
```









# 整体源码

```js
// 创建一个微队列函数
function microTask(fn) {
  // 判断是否是node环境
  if(process && process.nextTick) {
      process.nextTick(fn)
      return
  } else if (MutationObserver) {
      const p = document.createElement('p')
      const obs = new MutationObserver(()=>{
          fn()
      })
      obs.observe(p, {
          attributes: true,// 属性发生变化触发
          childList: true,// 子元素发生变化触发
          subtree: true // 子树发生辩护触发
      })
      p.innerHTML = '1'
  } else if(setImmediate) {
      // ie环境
      setImmediate(fn)
  } else {
      //其他环境
      setTimeout(fn, 0)
  }
  
}


const PENDING = 'peding'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        this._value = undefined
        this._status = PENDING
        this._handlers = []
        // 这里的resolve之所以要绑定this，是因为外层直接调用的时候，但class 内的函数会处于严格模式下，this会是undefined
        executor(this._resolve.bind(this), this._reject.bind(this))
    }
    
    _resolve(data) {
        this._changeStatus(data, FULLILLED)
    }
    
    _reject(reason) {
        this._changeStatus(reason, REJECTED)
    }
    
    _changeStatus(data, status) {
        this._value = data
        this._status = status
       	this._runHandlers()
        
    }
    
    isPromise(obj) {
        return !!(obj && typeof obj === 'obj' && typeof obj.then === 'function')
    }
    _pushHandler(executor, status, resolve, reject) {
        this._handlers.push({
            executor,
            status,
            resolve,
            reject
        })
    }
    _runHandlers() {
        if(this._status === PENDING) return
        while(this._handlers[0]) {
            const handler = this.hanlders[0]
            this._runOneHandler(handler)
           	this._handlers.shift()
        }
    }
    _runOneHandler({executor, status, resolve, reject}) {
        microTask(()=> {
            // 排除不符合状态的函数
            if(this._status !== status) return
            // 如果传递的参数不是函数，那么会有promise穿透，及then返回的promise的状态和值都跟当前实例的一致
            if(typeof executor !== function) {
                this._status === FULFILLED ? resolve(this._value) reject(this._value)
                return
            }
            // 函数执行过程报错，直接调用reject方法
            try {
                const result = executor()
                // 判断返回值是不是promise
                if(isPromise(result)) {
                    // 如果是promise那么等该promise状态改变后，触发返回的promise的相应方法即可
                    result.then(resolve, reject)
                } else {
                    // 如果是正常值，则直接调用resolve
                    resolve(result)
                }
            } catch(error) {
                resolve(error)
            }
        })
    }
    then(onFulFill, onReject) {
        return new MyPromise((resolve, reject)=> {
            this._pushHandler(onFulFill, FULFILLED, resolve, reject)
            this._pushHandler(onReject, REJECTED, resolve, reject)
            this._runHandlers()
        }) 
    }
    
}
```















