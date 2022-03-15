## Vue3中的reactive api（响应式api）理解

**reactive api**：用来处理数据响应式的api

- ref

- computed

- watchEffect



将数据转成响应式数据的api

| API  | 传入参数 | 返回 | 备注 |
| ---- | -------- | ---- | ---- |
|  reactive    |   plain-object       |  返回一个proxy    | 深度代理对象中的所有成员     |
| readonly| plain-object、proxy| 返回一个proxy| 只能读取对象中的成员，不能更改；如果传入的是一个proxy,相当于给proxy外套了一层，再返回一个新的proxy，旧proxy可以更改原来的对象中的成员 |
| ref | any | {value:...} | 对value的访问是响应式的，如果传入的是一个对象，那么使用reactive进行代理；如果已经是代理的，则直接使用代理|
|computed|function|{value:...}| 当读取`value`时，会根据情况决定函数是否要运行函数|

reactive是用来代理对象，
ref用来代理普通值的，但是也可以代理对象

```js
value:... 代表着访问器属性（注释：防备忘）
相当于
get Value() {
    // 获取值
}
set Value() {
    设置值
}
```



### watchEffect 和 watch

**watchEffect**：刚开始的时候函数会**先执行一次**(同步运行)，收集相关数据的依赖，当依赖项发生变化的时候会重新执行函数，函数会放在微队列里面执行

```js
watchEffect(()=> {
    console.log(state.a)
})
```

**watch**: 接受两个参数，第一个参数是收集的依赖，第二个是依赖发生变化后执行的函数，执行的函数会推进微队列中，第三个参数是相关配置（也可以配置立即执行执行）

- 参数细节

  - 第一个参数可以是一个对象、函数、数组

    - proxy对象可以正常监听

      ```js
      const count = ref(0)
      watch(count,(newvalue, oldValue)=> {
          console.log(newValue, oldValue)
      })
      ```

      

    - 但需要监听基本值的变化的时候，需要传入一个函数，函数返回值是基本类型

      ```js
      const count = ref(0)
      watch( ()=> count.value, (newvalue, oldValue)=> {
          console.log(newValue, oldValue)
      })
      ```

    - 当需要**监听多个数据**的时候使用数组，监听函数的newValue和oldValue也会对应的变成一个数组，数据跟监听的顺序一样

      ```js
      const count = ref(0)
      const num = ref(1)
      watch([count, num], ([countNew, numNew], [countOld, numOld])=> {
          console.log(newValue, oldValue)
      })
      ```

      

**watchEffect和watch的应用场景**

- 使用watchEffect的场景
  - 进入组件的时候，会自动执行一次的函数
  - 当相关的依赖发生变化的时候，才执行
  - 不需要新旧数据
- 使用watch的应用场景
  - 需要监听无关依赖的变化
  - 需要新旧数据
  - 进来后，不需要立马自动执行一次（当然也可以配置）





## unRef 、toRef、toRefs

**unRef**: 如果参数是一个ref，那么返回内部值，否则返回参数本身，是` val = isRef(val) ? val.value : val`的语法糖

**toRef**: 将源响应式对象的某个属性，用ref包裹，并返回;

​			接受两个参数，第一个是源响应式对象，第二个是属性名

注意：返回的新的响应式数据，会对源数据保持响应式链接，及新的更改了，原来的对象属性也会更改

```js
const person = reactive({
    name: 'person'
})
const nameRef = toRef(person, 'name')

```

**toRefs**：将源响应式对象的所有属性，都用ref进行包裹，并返回一个新的普通对象，普通对象的属性值都是用ref进行包裹的
