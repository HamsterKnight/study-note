# redux-action第三方库

作用：用于简化action-types、action-creator以及reducer

[官网]: https://redux-actions.js.org/



**createAction**接受三个参数，返回一个action创建函数

- 第一个参数是actionType类型

- 第二个参数是一个函数，返回值是payload

- 第三个参数是一个函数，返回值是meta

  ```js
  createAction('INCREASE', payload => payload, meta => meta)
  ```

  

**createActions**接受一个参数对象,，返回多个action创建函数，

参数对象

- 属性名是type类型，会将大写下划线明明，转为小写驼峰
- 属性值可以是一个函数，用来返回payload
- 返回对应属性的创建函数，toString方法会被重写，调用会获取到type类型

**handleAction**接受三个参数，返回一个reducer对象

- 第一个参数可以是createActions返回action创建函数，或者是actionType

  - 如果传的action创建函数回调重写的toString方法，获取actionType

- 第二个参数是actionType处理逻辑函数

- 第三个参数是state默认值

  ```js
  const reducer = handleAction('increase', (state, action)=> state + 1, defaultState)
  ```

**handleActions**接收两个参数，第一个是参数对象(键名是actionType,键值是state处理函数)，第二个是state默认值

- 键名可以是**createActions**创建的action创建函数（会调用action创建函数重写的toString，获取到actionType）

- 键值是state处理函数，接受两个参数

  - 第一个参数是state
  - 第二个参数是action

  

**combineActions**用于处理多个actionType，对应同一个state处理函数, 一般和handleActions配合使用

```js
// 1. 创建action创建函数
const {increase, decrease} = createActions({
    INCREASE: () => 1
    DECREASE: () => -1
	ADD: num => num
})

// 2. 合并多个state处理函数
const fn = combineActions(increase, decrease)
// 配合handleActions使用
handleActions({
    [fn]: (state, {paylaod}) => state + payload
}, 1)
```









