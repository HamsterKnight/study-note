react-router的使用（4.x版本）

react-router-dom包含了需要导出使用的组件

- 使用路由，需要在应用最外层包裹的组件，组件有两种（BrowserRouter、HashRouter）	
  - `BrowserRouter` H5路由,路径跟我们平常访问的路径比较像，比较好记忆
    - /abc/d
  - `HashRouter` 哈希路由，路由地址是在#后之后, 兼容性比较好
    - /xxx#/abc/d

- 控制路由渲染的组件形式的有两种（Switch、Routes）

  - `Switch`组件，循环遍历路由组件，当匹配到一个路由之后，便不会继续往下匹配
  - `Routes`组件，只要满足条件的路由都会继续往下匹配

- 路由组件的渲染`Route`，控制渲染的组件内容，有以下属性

  - path (字符串) ,匹配的路径
  - component，渲染的组件
  - children在Routes的包裹下，无论路径是否匹配都会渲染； 在Switch的包裹下，则需要匹配才会渲染（因为渲染时由Switch控制的）
  - render函数，无论路径是否匹配都会渲染
  - sensitive 路径匹配是否大小写敏感
  - exact 路径是否精确匹配

  `Route`会向渲染的组件，注入三个属性

  	- history包含了路由操作的属性和方法
  	- location路径地址的数据和属性
  		- pathname 当前的路径
  		- search 搜索的参数
  		- state 记录的状态
  		- hash 哈希的值
  	- match匹配的地址信息
  	  - isExact 是否是精确匹配
  	  - path 传递的路径匹配规则
  	  - url 匹配的路径
  	  - params 路径动态匹配的参数

- 路由跳转组件有三种（Link、NavLink、Redirect）

  - Link组件，控制路由的跳转
    - to属性
      - 可以是一个对象{pathanme: xxxx, state: xxx}
      - 可以是一个路径
    - replace, 是否是替换栈记录的形式
    - innerRef，是一个函数，通过函数参数可以获取到里面的dom
  - NavLink,控制路由的跳转
    - 这部分属性跟Link一样
      - to属性
        - 可以是一个对象{pathanme: xxxx, state: xxx}
        - 可以是一个路径
      - replace, 是否是替换栈记录的形式
      - innerRef，是一个函数，通过函数参数可以获取到里面的dom
    - 比Link多的属性
      - activeClassName 自定义匹配路由的类名（默认是active）
      - activeStyle 匹配路由的样式
      - exact是否精确匹配
      - strict是否严格匹配最后一个斜杠
      - isActive 一个为了确定链接是否处于活动状态而添加额外逻辑的函数
      - location
  - Redirect，路由重定向
    - to属性，重定向的地址，可以是一个string，也可以是一个Object
    - from属性，是一个string，匹配到什么路径后，就重定向到对应地址
    - exact是否精确匹配
    - strict是否严格匹配最后一个斜杠





**源码思路**

- 假设没有这个库，我们要实现相关的功能我们自己要怎么写
- react-router往路由组件中注入了三个属性`history、location、match`
  - history包含了对地址栈道一系列操作
  - location包含了已经配置的地址的一些状态信息
  - match包含了地址匹配的信息


#react-router-dom中监听路由的变换，使用`history`第三方库

history库中提供了三个方法`createBrowserHistory, createHashHistory, createMemoryHistory	`来决定路由对象的创建方式。

## history

- length属性 表示地址栈的长度
- action属性 表示最后一次操作的类型
  - 通过createBrowserHistory的方式创建对象, action固定为POP
  - PUSH 调用push方法
  - POP 调用go, goForward, goBack方法
  - REPLACE 调用replace方法
- go方法 接受一个number参数
  - 为0时会刷新当前页面
  - 为负值时会退到地址栈当前的位置的后几个位置
  - 为正值是会前进到地址栈中的前几个位置
- goForward方法： 地址栈前进一格
- goBack方法： 地址栈回退一格
- push方法: 往地址栈中加入一条数据，接受一个参数
- replace方法：用新的数据，替换当前地址栈的数据，接受一个参数
- location：表达当前地址中的信息
- createHref： 返回basename + location.pathname
- listen方法：路由发生变化后，会触发该函数，
  - 函数接受两个参数
    - location 记录**跳转后**的路由地址信息
    - action 跳转方式
  - 函数执行后会返回一个取消监听函数
- block方法，用来在跳转前增加一个阻塞，
  - 该函数参数类型有两个
    - 接收一个字符串作为参数，表示消息内容，
    - 接收一个函数作为参数，参数有两个参数
      - location跳转的地址信息
      - action跳转类型
    - 函数的返回值是消息内容
  - 该函数返回一个取消函数，调用取消函数可以解除阻塞
  - 注意：block需要跟createBrowserHistory的配置项getUserConfirmation配合使用

**createBrowserHistory**配置项

- baseUrl:string, 路由的基路径
- `getUserConfirmation`:function(msg, callback)， 只有通过block设置阻塞后，该函数在路由跳转时才会被调用
  - 接受两个参数
    - msg通过block方法传过来的参数
    - callback
      - callback返回值如果是true,就会进行路由跳转，否则不跳转
- forceRefresh: boolean，路由跳转时是否刷新页面
- keyLength:num, 生成的location中key值的长度

> 需要注意的地方：
>
> 1. location的存储不另外自己起一个数组去维护，而是直接采用window.history.location来获得
>
>    这样的好处是：不用自己做额外的维护操作
>
> 2. location中的state的值,有几种赋值情况,   window.history.location 以下称为w.h
>
>    1. w.h.location中state为null时，location中state为undefined
>
>    2. w.h.location中state为object时，
>
>       1. 有key属性时,location中state为{key: w.h.locatin.key,  state: w.h.location.state.state}
>       2. 无key属性时 location.state = w.h.location.state
>
>       > 为什么会有这个判断呢？
>       >
>       > 因为其他库可能也会对state进行操作, 为了给其他库留活路，history库设置状态时会将状态存进state.state中
>
>    3. 其他情况location.state = w.h.location.state
>
> 3. location创建方式有两种
>
>    1. 通过window.location及window.history.state组合
>    2. 通过push或者repalce方法传进来的参数进行生成

```js
import ListenerManager from "./utils/ListenerManager";
import BlockManager from "./utils/BlockManager";
function createBrowserHistory(options = {}) {
  const {
    basename = "",
    forceRefresh = false,
    keyLength = 6,
    getUserConfirmation = (message, callback) =>
      callback(window.confirm(message)),
  } = options;

  const listenerManager = new ListenerManager();
  const blockManager = new BlockManager(getUserConfirmation);

  function go(num) {
    history.go(num);
  }

  function goForward() {
    history.forward();
  }
  function goBack() {
    history.back();
  }

  function push(path, state) {
    changePage(path, state, true);
  }

  function replace(path, state) {
    changePage(path, state, false);
  }
  /**
   * 对push方法和replace方法进行公共代码提取
   * @param {*} path 路径对象
   * @param {*} state 状态
   * @param {*} isPush 是否是push类型
   */

  function changePage(path, state, isPush) {
    let action = "PUSH";
    if (!isPush) {
      action = "replace";
    }
    // 获取统一处理的参数
    const pathInfo = handleState(path, state, basename);
    // 当地址发生变化的时候，location需要重新设置
    // 这里也是需要注意的，就是只有地址发生变化从history上设置location才是正常的，否则只能通过参数来生成新的location
    // const location = createLocation(basename)
    const location = createLocationFromPathInfo(pathInfo, basename);
    // 触发阻塞
    // 只有允许跳转的时候才能进行跳转
    blockManager.triggerBlock(location, action, () => {
      // 有个坑
      // 如果是强制刷新的情况，需要先保存状态到state的状态,不然状态会丢失
      if (isPush) {
        window.history.pushState(
          {
            key: createKey(keyLength),
            state: pathInfo.state,
          },
          null,
          pathInfo.path
        );
      } else {
        window.history.replaceState(
          {
            key: createKey(keyLength),
            state: pathInfo.state,
          },
          null,
          pathInfo.path
        );
      }
      // 触发listener监听函数
      listenerManager.triggerListener(location, action);
      // 小坑
      // listener监听函数出发后，才将location更新到history上，因为listener函数执行过程中，history的location还是旧值
      // 需要在地址改变后，再对location进行更新及更新action
      history.location = location;
      history.action = action;
      if (forceRefresh) {
        window.location.href = pathInfo.path;
      }
    });
  }

  addEvent();
  function addEvent() {
    // popstate方法只能监听用户在浏览器上的操作
    window.addEventListener("popstate", () => {
      const location = createLocation(basename);
      const action = "POP";
      blockManager.triggerBlock(location, action, () => {
        // 用户在浏览器上的操作，action都是POP
        listenerManager.triggerListener(location, action);
        // 在listener中history本身的location还是之前，所以需要在触发完监听函数后再赋值
        history.location = location;
      });
    });
  }

  function listen(listen) {
    listenerManager.addListener(listen);
  }

  function block(msg) {
    blockManager.block(msg)
  }


  const history = {
    go,
    goForward,
    goBack,
    push,
    replace,
    listen,
    block,
    length: window.history.length,
    action: "POP", // 默认创建的值都为POP
    location: createLocation(basename),
  };

  return history;
}
/**
 * 内部不用数组来维护栈中的location，因为挺麻烦的，需要增删改查之类的操作
 * 直接使用window.location来获取栈location的内容
 * location 有四个属性
 *          pathname
 *          search
 *          hash
 *          state
 */

function createLocation(basename) {
  // 对pathname进行处理
  let pathname = window.location.pathname;

  // 去除basename
  const reg = new RegExp(`^${basename}`);
  pathname = pathname.replace(reg, "");

  const location = {
    pathname,
    search: window.location.search,
    hash: window.location.hash,
  };

  let state,
    historyState = window.history.state;
  // 对state进行处理
  // 为什么不能直接用history中的state呢？
  // 因为还有第三方的插件会对state进行更改，我们不能直接更改别人的，给其他插件留活路
  // 直接取自己的就好, 自己的内容会存在state.state里面
  if (historyState === null) {
    state = undefined;
  } else if (typeof state === "object") {
    if ("key" in historyState) {
      state = historyState.state;
      location.key = historyState.key;
    } else {
      state = historyState;
    }
  } else {
    state = historyState;
  }

  location.state = state;

  return location;
}

// 对传进来的参数进行统一处理
function handleState(path, state, basename) {
  if (typeof path === "string") {
    return {
      path: basename + path,
      state,
    };
  } else if (typeof path === "object") {
    let { pathname, search = "", hash = "" } = path;
    pathname = basename + pathname;
    if (search && search.chartAt(0) !== "?") {
      search = "?" + search;
    }

    if (hash && hash.chartAt(0) !== "#") {
      hash = "#" + hash;
    }
    pathname += search;
    pathname += hash;

    // 如果传进来的path是一个对象，那么第二个参数失效
    return {
      path: pathname,
      state: path.state,
    };
  }
}

/**
 * 通过路径参数生成location
 * @param {*} pathInfo
 * @return location
 */
function createLocationFromPathInfo(pathInfo, basename) {
  const { path, state } = pathInfo;

  // 去除basename
  const reg = new RegExp(`^${basename}`);
  let pathname = path.replace(reg, "");
  pathname = pathname.replace(/[?#].*/, "");

  const quesIndex = path.indexOf("?");
  const sharpIndex = path.indexOf("#");

  let search = "",
    hash = "";
  // 情况1 问号存在，且在#前
  // 情况2 问号存在，不存在#
  if (
    quesIndex >= 0 &&
    ((sharpIndex >= 0 && quesIndex < sharpIndex) || sharpIndex < 0)
  ) {
    search = path.slice(quesIndex, sharpIndex);
    hash = path.slice(sharpIndex);
  } else if (quesIndex >= 0) {
    hash = path.slice(sharpIndex);
  }

  return {
    pathname,
    search,
    hash,
    state,
  };
}

function createKey(keyLength) {
  const key = Math.random().toString(36).slice(2);
  return key.slice(0, keyLength);
}

export default createBrowserHistory;

```



### match

> match属性的匹配，使用了一个第三方库，path-to-regex,该库会对匹配规则转化成正则表达式

- isExact地址是否精确匹配
- path 路由的匹配规则
- url 当前的地址路径
- params 可变内容的匹配对象

```js
/**
 * 主要返回route中注入的match信息
 * match包含的信息
 * isExact 收否精准匹配，也就是匹配的路径跟正则是否完全匹配
 * path 当前路由的匹配路由
 * url 当前正则匹配的路由地址
 * params 动态匹配参数
 *
 */
import { pathToRegexp } from "path-to-regexp";

export function mathPath(pathReg, pathname, options = {}) {
  // 获取路由匹配的默认的配置
  const opts = getOpts(options);
  // 获取匹配的key
  let keys = [];
  // 根据路由路径得到正则表达式
  const reg = pathToRegexp(pathReg, keys, opts);
  const result = reg.exec(pathname);
  // 如果没有匹配则返回空
  if (!result) {
    return null;
  }
  // 如果有匹配路由
  const paramsKeys = result.slice(1);
  //1.组合返回的params
  const params = {};
  for (let i = 0; i >= 0; i--) {
    params[keys[i].name] = paramsKeys[i];
  }
  return {
    isExact: keys.length === paramsKeys.length,
    params: params,
    path: pathReg,
    url: pathname,
  };
}

function getOpts(opts) {
  const baseOptions = {
    sensitive: false,
    strict: false,
    exact: true,
  };

  const options = {
    ...baseOptions,
    ...opts,
  };

  return {
    sensitive: options.sensitive,
    strict: options.sensitive,
    end: options.exact,
  };
}
```



### react-router组件的作用及原理

**BrowserRouter**：主要给Router组件提供history对象

**Router组件**：主要向下提供上下文，以及location发生变化的时候，触发整个组件树的更新

上下文的内容包括：{history,location,match}

**Route组件**：主要根据传入的路径来进行组件渲染

​						具体做的内容：

​							1.消耗Router组件提供的上下文，从而得到hitory和location

​							2.提供一个上下文，history，location及匹配的match

> 渲染规则
>
> 1.当有传入children时，无论路径是否匹配都会渲染
>
> 2.当路径匹配时
>
> ​	1.存在render函数，渲染render函数的内容
>
> ​    2.当不存在render函数，存在component属性时，渲染component内容 	



