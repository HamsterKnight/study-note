## redux中间件

- redux-logger:进行日志记录, 一般放在应用的中间件的最后一个

- 副作用处理
  - redux-thunk: 应用了redux-thunk中间件后，允许action是一个函数, thunk中间件一般放在第一个
    - 内部运作原理
      - 如果发现action是一个函数，那么action不会往后移交，而是直接调用action函数
  
  - redux-promise
    - 内部原理
      - 两种情况
        - 1.传入的action是一个标准的Promise
        - 2.传入的action的payload是一个标准的promise，type必须是一个字符串
    - 标注的action必须要有四个属性，type, payload, error, meta
  
  - redux-saga
  
    -  特点：纯净、强大、灵活
  
    - 基础
  
      - 最开始启动一个saga任务
  
      - saga任务是一个生成器函数
  
      - saga为任务提供了大量的功能以供使用
  
      - 功能是以指令的形式出现，指令出现的位置在yiled后面，这样外部就可以对内部进行控制
  
        ```js
        import createSagaMiddleWare from 'redux-saga'
        //创建saga中间件函数
        const saga = createSagaMiddleWare()
        
        //启动saga任务
        saga.run(myFn)
        
        function* myFn(){}
        ```
  
        
  
    - 指令都在redux-saga/effect中，以下介绍常用指令
  
      - **take**指令,监听某个action,仅监听一次，yield得到的是一个完成的action
  
        - 只接受一个参数
  
      - **all**指令，接受一个生成器数组，当数组里面所有的生成器完成后，才会继续往下执行
  
      - **takeEvery**指令，不会阻塞，接受两个参数，第一个是action类型，第二个参数是要执行的生成器；永远不会结束当前的生成器
  
      - **delay**指令, 延迟执行，接受一个参数
  
        - delay后的内容会延迟相应设置的时间执行
  
      - **put**指令，接受一个action参数，用于触发一个action
  
      - **call**指令，用于副作用函数调用，可以接收多个参数，[可能阻塞，当函数返回是promise的时候]
  
        - 第一种写法
  
          - 第一个参数是需要执行的副作用函数
  
          - 其他参数是传递给副作用函数的函数
  
            ```js
            call(fn, ...args)
            ```
  
            
  
        - 第二种写法
  
          - 第一个参数是一个数组，数组中有两项
  
            - 第一项是指定当前函数的上下文
  
            - 第二项是函数
  
              ```js
              call([context, fn], ...args)
              如上所以fn的的this指向context
              args是剩余参数
              ```
  
        - 第三种写法
  
          - 第一项是指定当前函数的上下文
  
          - 第二项是函数名
  
            ```js
            call([context, fnName], ...args)
            fnName是字符串，代表函数名
            ```
  
            
  
      - **apply**指令，接受多个参数
  
        - 第一个参数是指定当前函数执行上下文
  
        - 第二个是需要运行的函数
  
        - 剩余参数
  
          ```js
          apply(context, fn, [args])
          相当于call([context, fn], ...args)的另外一种写法
          ```
  
      - **select**指令，用户筛选获取仓库的数据
  
      - **cps指令**, 【可能阻塞】用于调用那些传统的回调方式的异步函数
  
        - cps指令会自动给传入的函数附加一个callback函数
          - callback回调函数接受两个参数，第一个参数用于报告错误，而第二个用于报告成功的结果
          - 成功或者错误的结果，均会返回给yiled赋值表达式
  
        ```js
        function mockData(condition, calblack) {
            setTimeout(()=> {
                callback(null, {a:123})
            }, 3000)
        }
        
        const reuslt = yield cps(mockData, condition)
        
        console.log(result)// {a:123}
        ```
  
        
  
    - 当saga发现yiled得到的结果是一个promise对象，他会自动等待promise完成，完成后将完成的结果作为值传递到下一次next; 如果是reject，会调用generator.throw抛出一个错误
  
    - 触发的生成器函数中yiled后面最好也是接指令，而不是函数，因为这样有利于单元测试
  
  - saga不常用指令
  
    - **fork**指令，用于开启一个新的任务，任务不会阻塞，该函数需要传递一个生成器函数
      - fork返回一个Task任务对象
    - **cancel**指令，用于取消一个任务
      - 原理，是调用generator.retrun方法，直接结束生成器函数
    - **takeLatest**指令，跟takeEvery差不多的功能，但是会取消上一次执行任务
    - **cancelled**
  
  - saga的指令的一些场景
  
    - 1.使用takeEvery监听一个actionType,触发一个自增函数，每次触发当前actionType的时候，就会触发一个自增函数，触发次数多了，自增函数会有很多个，现在需要触发actionType的时候，需要取消上一次的自增函数的执行，可以用**takeLatest**来处理该情况
  
      ```js
      // 通过fork实现以上的场景的解决方法
      function* autoIncreate() {
          let task;
          // 如果task有值，则取消该任务
          if(task) {
              yield cancle(task)
          }
          while(true) {
              yield take('autoIncreate')
              task = fork('autoIncreate'， autoIncreate)
          }
          
      }
      
      // 主函数
      
      function* mainFn() {
          yield fork('autoIncreate'， autoIncreate)
      }
      
      
      ```
  
    - 流程控制场景，比如先增加、后停止、先增加、后停止

​		

注意：

**redux-thunk和redux-promise都会改变action，让action不再纯净**

**redux-saga,会保存action的纯净,且功能强大**

